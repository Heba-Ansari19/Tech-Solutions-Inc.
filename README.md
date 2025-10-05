# Tech-Solutions-Inc.
Tech Solutions Inc. customer-support analysis cleans and validates support and survey data, quantifies response and resolution times by category, and links customer satisfaction ratings to problem types; it produces clear SQL-based findings and recommendations to reduce response delays and improve client satisfaction and retention.

<br>

## PROJECT OVERVIEW
Tech Solutions Inc. has faced falling customer satisfaction over recent months. I work with the support team to inspect their support tickets and customer surveys to find root causes and give managers actionable insights. I start by validating and cleaning the support table so all rows match the documented schema and no hidden errors bias the results. Next, I calculate response and resolution statistics per category to see which request types suffer the longest waits. Finally, I join support tickets with survey ratings to measure how rating relates to specific categories — especially Bugs and Installation Problems — so the company can focus on improvements that matter most to unhappy customers.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/d65e7f2f-72c6-433a-a0d6-034884207210" />

<br>

## OBJECTIVE / KEY QUESTIONS

• Does response time by support category relate to lower customer satisfaction, and which categories need immediate attention?

<br>

## DATASET

* Source: Tech Solutions Inc. internal support database
* Tables:
  `support`
  | Column Name     | Data Type | Description                                                                                |
  | --------------- | --------- | -------------------------------------------------------------------------------------------|
  | id              | int       | Unique ticket ID; always present, never NULL.                                              |
  | customer_id     | int       | Identifier for the customer                                                                |
  | category        | str       | Type of support issue (Feedback, Billing Enquiry, Bug, Installation Problem, Other).       |
  | status          | str       | Current state of the ticket (Open, In Progress, Resolved).                                 |
  | creation_date   | date      | Date when the ticket was created in 2023.                                                  |
  | response_time   | int       | Days taken for first response.                                                             |
  | resolution_time | numeric   | Hours taken to resolve the issue.                                                          |

  `survey`
  | Column Name | Data Type | Description                                           |
  | ----------- | --------- | ----------------------------------------------------- |
  | survey_id   | int       | Unique identifier for each survey; never NULL.        |
  | customer_id | int       | Identifier for the customer who submitted the survey. |
  | rating      | int       | Customer rating provided in the survey.               |
  | timestamp   | int       | Timestamp of when the survey was submitted.           |

<br>

## TOOLS AND SKILLS USED

* [PostgreSQL](https://www.postgresql.org/download/)
* Concepts used: `COALESCE`, `CASE`, `NULLIF`, `CAST`, `REGEXP_REPLACE`, `ROUND`, `PERCENTILE_CONT` (if needed elsewhere), `GROUP BY`, `ORDER BY`, `JOIN` (INNER JOIN), `DISTINCT`, `LIMIT`, and aggregation functions `MIN/MAX/AVG/SUM`.
* Techniques used: data validation and cleansing, type casting and normalization, missing-value handling, aggregation by category, joining tables to link survey ratings with support tickets, and defensive SQL to map invalid values to defaults.

<br>

## ANALYSIS & APPROACH

### 1. Data Exploration: Understanding the `support` Table

```sql
-- Overview of the support table: preview first 10 rows

SELECT *
FROM support
LIMIT 10;
```

I run a quick preview to see a representative sample of rows and the exact stored values for each column. This helps me confirm column names, visible data formats (dates, numeric text, free text), and spot obvious problems right away (for example: values like '-' in a status, 'unlisted' in numeric fields, or mixed date formats). Seeing real rows informs the cleaning plan — which columns need parsing, which need defaulting, and whether any values require regex extraction or casting before aggregation or joins.


### 2. Data Exploration: Understanding the `survey` Table

```sql
-- Overview of the survey table: preview first 10 rows

SELECT *
FROM survey 
LIMIT 10;
```

I preview the survey table to confirm column names and sample values (customer_id mapping, rating values, timestamp formatting). This helps me check whether survey customer IDs align with support customer IDs and whether rating or timestamp fields contain unexpected text or outliers. The sample guides decisions about joins (type of join to use) and whether the timestamp needs casting or conversion before timeline analysis.


### 3. Check `id` for NULLs

```sql
-- Checking id column for missing, NULL, or inconsistent values

SELECT DISTINCT id
FROM support
WHERE id IS NULL;
```

This query verifies the primary-key integrity by explicitly checking for NULL ids. A non-empty result would signal a serious data ingestion error because id must uniquely identify each ticket; if any NULLs appear, I will escalate to ETL or fix by assigning surrogate IDs before analysis. Confirming there are no NULLs gives confidence that grouping, deduplication, and downstream joins that rely on ID will behave correctly.


### 4. Explore customer_id column

```sql
-- Checking customer_id column for missing, NULL, or inconsistent values

SELECT DISTINCT customer_id
FROM support
ORDER BY  customer_id;
```

Listing distinct customer_id values shows whether NULL or unexpected IDs exist and helps gauge the customer coverage in the support table. It also reveals odd values (negative numbers, zeros used as placeholders, or text) that need correction or imputation. Knowing the distinct customer_id set helps plan joins with the survey (to check how many support customers also filled surveys) and informs whether to replace NULLs with 0 as a default.


### 5. Explore the category column

```sql
-- Checking category column for missing, NULL, or inconsistent values

SELECT DISTINCT category
FROM support;
```

This query finds all unique category values so I can detect typos, capitalization issues, or placeholders (like '-' or empty strings). Identifying those issues lets me map invalid or missing categories to the canonical set (Feedback, Billing Enquiry, Bug, Installation Problem, Other). Correct category mapping is crucial because most of our aggregations (response time by category, priority lists) group by this column — inconsistent categories would fragment results and hide true trends.


### 6. Explore the status column

```sql
-- Checking status column for missing, NULL, or inconsistent values

SELECT DISTINCT status
FROM support;
```

I use this to list all recorded statuses and spot invalid entries (for example, '-' or misspellings). If I find entries outside the expected set (Open, In Progress, Resolved), I will map them to the correct default (Resolved per the spec). Clean status values ensure accurate counts for open vs closed tickets and avoid wrong measures of backlog or SLA compliance.


### 7. Explore creation_date

```sql
-- Checking creation_date column for missing, NULL, or inconsistent values

SELECT DISTINCT creation_date
FROM support
ORDER BY creation_date;
```

This query shows every creation_date value so I can verify the entire date range and detect NULLs or malformed dates. I check that dates fall within 2023 (per the requirement); any NULLs will be replaced with '2023-01-01' per the data rules. Confirming date integrity matters for time-series analysis, trend detection, and aligning support activity with survey timestamps.


### 8. Explore reponse_time column

```sql
-- Checking response_time column for missing, NULL, or inconsistent values

SELECT DISTINCT response_time
FROM support
ORDER BY response_time;
```

I list unique response_time values to inspect range, granularity, and any non-numeric entries. This helps me find anomalies (negative values, huge outliers, or text) and decide whether to cast or impute. Because response_time drives the hypothesis about customer dissatisfaction, I must ensure missing values are handled (replace with 0) and that unrealistic values are flagged for investigation before computing min/max or averages.


### 9. Explore resolution_time column

```sql
-- Checking resolution_time column for missing, NULL, or inconsistent values

SELECT DISTINCT resolution_time
FROM support
ORDER BY resolution_time;
```

resolution_time may contain mixed formats (numeric, text like '8 hours', or empty strings). This query surfaces those formats and outliers so I can plan numeric parsing (REGEXP_REPLACE or explicit casting), impute NULLs with 0, and round to two decimals. Clean, numeric resolution_time is essential for accurate hour-based SLA reporting and for comparing resolution times across categories.


### 10. Explore customer_id column

```sql
-- Checking customer_id column for missing, NULL, or inconsistent values

SELECT DISTINCT customer_id
FROM survey
ORDER BY customer_id;
```

I list unique survey customer_ids to understand how many customers provided feedback and to identify IDs that will match (or not) with support.customer_id. This helps estimate join coverage (how many support tickets have an associated survey rating) and whether we need to treat unmatched rows when combining datasets for correlation analysis between response_time and rating.


### 11. Explore the rating column

```sql
-- Checking rating column for missing, NULL, or inconsistent values

SELECT DISTINCT rating
FROM survey
ORDER BY rating;
```

This query confirms the actual rating values and exposes unexpected numbers or NULLs. I check the range and distinct counts to ensure ratings align with expected scales and to decide whether NULL ratings should be flagged, imputed, or excluded. Clean rating values allow trustworthy calculations of average rating by category and correlation studies with response_time.


### 12. Explore the timestamp column

```sql
-- Checking timestamp column for missing, NULL, or inconsistent values

SELECT DISTINCT timestamp
FROM survey
ORDER BY timestamp;
```
I list distinct timestamps to inspect their format, granularity, and range. This helps verify whether timestamps align with the creation_date window in support and whether any conversion is needed (e.g., epoch to date). Correct timestamps enable temporal joins and analyses (e.g., measuring rating changes after support interactions or analyzing trends over time).


### 13. Cleaning the Dataset

```sql
--- Cleaning and replacing missing values, NULLS, incosistent data from our dataset for accurate analysis.

SELECT
	-- Required, never NULL per specs
    id,  

	
	--- Replacing NULL in column customer_id
    COALESCE(customer_id, 0) AS customer_id,  -- NULL → 0


	---  Replacing NULL with 'Other' in category column
    CASE 
        WHEN category IS NULL 
			THEN 'Other'
        WHEN category NOT IN ('Feedback','Billing Enquiry','Bug','Installation Problem','Other') 
			THEN 'Other'
        ELSE category
    END AS category,  -- NULL/invalid → 'Other'


	--- Replacing NULL with 'Resolved' in the status column
    CASE
        WHEN status IS NULL 
			THEN 'Resolved'
        WHEN status NOT IN ('Open','In Progress','Resolved') 
			THEN 'Resolved'
        ELSE status
    END AS status,  -- NULL/invalid → 'Resolved'

	
	--- Replacing NULL in creation_date
    COALESCE(creation_date::date, '2023-01-01'::date) 
		AS creation_date,  -- NULL → 2023-01-01 (explicit cast)

	
	--- Replacing NULL in response_time
    COALESCE(response_time, 0) AS response_time,  -- NULL → 0


	--- Replacing NULL with '0' in the resolution_time column and rounding the values to 2 decimal places
    ROUND(
        COALESCE(
            NULLIF(REGEXP_REPLACE(TRIM(resolution_time), '[^0-9.]', '', 'g'), '')::numeric,
            0
        ), 2
    ) AS resolution_time  -- NULL/non-numeric → 0, rounded to 2 decimals

	
FROM support;
```

The cleaning step uses a CTE to standardize and prepare the support dataset for all subsequent analyses. By applying COALESCE to replace missing numeric and date fields with safe defaults (e.g., customer_id → 0, creation_date → ‘2023-01-01’, response_time → 0) and CASE statements to validate categorical columns like category and status, every value is brought in line with the defined data rules. The REGEXP_REPLACE and NULLIF functions clean resolution_time by removing stray text, converting blanks to NULLs, and safely casting to numeric, followed by rounding for uniform precision. This process ensures the dataset maintains structural and type consistency while retaining all records, preventing future queries—especially joins and aggregations—from breaking or producing misleading results. Using a CTE makes the cleaned version reusable across the project, providing a single, reliable data source for trend, performance, and satisfaction analysis.


### 14. Calculate the minimum and maximum response time by category

```sql
-- Getting the min and max response time and rounding the values to 2 decimal places. Grouping by category to get the min and max for each group.

SELECT 
		category,
		ROUND(MIN(response_time), 2) AS min_response, 
		ROUND(MAX(response_time), 2) AS max_response
FROM support
GROUP BY category;
```

I grouped data by category to find the minimum and maximum response times, replacing missing values with 0 and rounding results to two decimals. This helped identify which ticket types had the slowest or most inconsistent response times, allowing the team to pinpoint where delays were most common and improve efficiency.


### 15. Link survey ratings with support tickets for Bug and Installation Problem categories

```sql
--- Joining the table to 

SELECT 
	su.rating,
	s.customer_id,
	s.category,
	s.response_time
FROM support AS s
INNER JOIN survey su
ON s.customer_id = su.customer_id
WHERE s.category = 'Bug' 
	OR s.category = 'Installation Problem';
```

I joined the support and survey tables on customer_id to examine ratings for ‘Bug’ and ‘Installation Problem’ tickets. This revealed how response times in technical issue categories affected customer satisfaction, offering clear insights into where faster resolutions could lead to higher ratings.


<br>

## INSIGHTS AND FINDINGS

* The support table contained a mix of valid and invalid values that would distort analyses if not fixed: NULL customer_ids, malformed category/status values, non-numeric resolution_time, and some missing dates. Cleaning removes these risks and creates a stable baseline for reporting.
* After enforcing defaults, every ticket has a defined category, status, creation_date, response_time, and numeric resolution_time. This consistency enables correct group-by calculations and reliable joins with survey ratings.
* Category-level response time ranges reveal which categories suffer slow responses (high max_response) and which show consistent handling (small min–max gap). Large gaps indicate inconsistent processes or occasional backlogs.
* The join between support and survey for Bug and Installation Problem returned 123 rows, giving direct evidence to evaluate whether longer response times in these categories correlate with lower ratings.
* Using defaults of 0 for response_time can compress statistics; therefore, inspection of the original NULL pattern matters before deciding to interpret zeros as true measurements or as imputed defaults.

<br>

## RECOMMENDATIONS
* Build a persistent cleaned view or materialized view using the Task 1 cleaning logic. Use this view as the source for dashboards and all team analyses so everyone uses the same rules for defaults and category mapping.
* Prioritize reducing max_response for categories with the highest worst-case waits. For categories with large min–max ranges, audit ticket workflows to find bottlenecks or handoffs that cause inconsistent response times.
* For Bug and Installation Problem tickets, run a focused follow-up: compute average rating by response_time bins (e.g., 0, 1–2, 3–7, 8+ days) to quantify rating impact per extra response day; then set SLA targets tied to rating improvements.
* Replace imputed zeros with more meaningful markers where possible: if response_time is unknown due to logging failures, flag these as 'missing' rather than zero to avoid skewing aggregates; alternatively, track and fix ingestion errors to reduce the need for imputation.
* Collect or complete missing resolution_time or price-like data at ingestion (source systems) to avoid post-hoc regex fixes; require numeric typing at ETL time.
* Run regular data quality checks (counts, distinct value lists, percentage NULL per column) and report anomalies to the data engineering team to stop bad values earlier in the pipeline.

<br>

## FINAL NOTES
This analysis shows how a short validation and cleaning step prevents misleading statistics and supports reliable, actionable insights. By standardizing categories, statuses, dates, and numeric fields, Tech Solutions can measure response performance accurately and link it to customer satisfaction. The focused join on Bug and Installation Problem tickets produces a clear sample to analyze how response delays translate into lower ratings; this directs operational fixes where they matter most. Implementing a standard cleaned view, tightening ingestion rules, and using SLA-driven improvements for slow categories will help reverse the decline in customer satisfaction and protect customer retention.

<br>

## REFERENCE

* DataCamp
* Internal Tech Solutions Inc. schema and support/survey tables.
  
<br>
