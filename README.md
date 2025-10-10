# Tech-Solutions-Inc.
Analyzed **Tech Solutions Inc.** customer support data to clean, validate, and standardize support tickets and survey records for accurate insights. Addressed **missing values, inconsistent categories and statuses, non-numeric responses, resolution times, and timestamp issues**. Created a **fully reliable dataset to measure response and resolution performance** by category and linked customer satisfaction ratings to problem types. Enabled **actionable recommendations to reduce response delays, improve client satisfaction, and support retention strategies.**

<br>

## PROJECT OVERVIEW
**Tech Solutions Inc. is a customer support-focused company** facing declining customer satisfaction over recent months. Customers have reported **delays and issues, particularly with Bugs and Installation Problems**. This project focused on analyzing support tickets and customer surveys to identify patterns and areas needing improvement. The `support` dataset included fields such as `category`, `status`, `response_time`, `resolution_time`, and `survey_rating`. Cleaning and analyzing this data ensures reliable insights, helps prioritize improvements, and enables the company to enhance overall customer experience.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/d65e7f2f-72c6-433a-a0d6-034884207210" />

<br>

## OBJECTIVE / KEY QUESTIONS

• Does response time by support category relate to lower customer satisfaction, and which categories need immediate attention?

<br>

## DATASET

* Source: Tech Solutions Inc. internal support database
* ERD:

  <img width="300" alt="image" src="https://github.com/user-attachments/assets/2ddc3924-3464-4912-90b2-9d6bd169a634" />
  
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

I started by previewing a sample of rows from the `support` table to understand the **data structure and stored values**. This helped me identify column names, data formats, and obvious issues such as invalid entries or mixed date formats, guiding the **cleaning plan for parsing, default handling, and type corrections before further analysis**.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/4a84317c-bb3e-488a-b2d0-1401e19c5c57" />


### 2. Data Exploration: Understanding the `survey` Table

```sql
-- Overview of the survey table: preview first 10 rows

SELECT *
FROM survey 
LIMIT 10;
```

I started by previewing a sample of rows from the `survey` table to understand the **data structure and stored values**. This helped me identify column names, data formats, and obvious issues such as invalid entries or mixed date formats, guiding the **cleaning plan for parsing, default handling, and type corrections before further analysis**.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/44ed7313-d51e-46d1-bf1f-2fcd71019efb" />


### 1. Check the table structure

```sql
--- Understanding the table structure and data types of all the columns

SELECT 
    column_name, 
    data_type
FROM information_schema.columns
WHERE table_name IN('support', 'survey');
```
Checked the **data types of all columns** from both the tables to confirm they **match expectations and to identify any necessary type conversions during cleaning**.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/ea89b3f6-f351-4984-b6d8-8ab01687e0ee" />


### 4. Check `id` for NULLs

```sql
-- Checking id column for missing, NULL, or inconsistent values

SELECT DISTINCT id
FROM support
WHERE id IS NULL;
```

I verified whether `id` had any missing values. Since this is the **primary key**, it must **not contain NULLs**. This check confirmed **data integrity for unique identification**.


### 5. Explore customer_id column

```sql
-- Checking customer_id column for missing, NULL, or inconsistent values

SELECT DISTINCT customer_id
FROM support
ORDER BY  customer_id;
```

Checked all unique `customer_id` values to identify **NULLs, placeholders, or invalid entries**. This ensured that all customer identifiers were valid and ready for accurate joins with the `survey` table, allowing proper mapping of **support interactions to customer feedback**.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/cf70c11d-62f7-4791-854b-2de924569cf4" />


### 6. Explore the category column

```sql
-- Checking category column for missing, NULL, or inconsistent values

SELECT DISTINCT category
FROM support;
```

Checked all unique `category` values to identify **missing, inconsistent, or invalid entries**. This ensured that all categories matched the expected set — **Feedback, Billing Enquiry, Bug, Installation Problem, and Other** — for accurate grouping and reliable analysis.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/efe96d5d-09a9-4552-a83f-5a2a16e4b9ac" />


### 7. Explore the status column

```sql
-- Checking status column for missing, NULL, or inconsistent values

SELECT DISTINCT status
FROM support;
```

Checked all unique `status` values to identify **missing values (NULL), placeholder characters (-), capitalization, or spacing issues.** This ensured that all statuses matched the expected set — **Open, In Progress, and Resolved** — for accurate tracking of ticket progress and SLA compliance.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/68345519-3e4a-4980-9785-7acdbee72430" />


### 8. Explore creation_date

```sql
-- Checking creation_date column for missing, NULL, or inconsistent values

SELECT DISTINCT creation_date
FROM support
ORDER BY creation_date;
```

Listed distinct `creation_date` values to verify the **full date range and identify NULLs or malformed dates**. I ensure that **dates fall within 2023**, as required, and replace any NULLs with **'2023-01-01'** according to the data rules. Confirming date integrity is essential for **time-series analysis, trend detection, and aligning support activity with survey timestamps**.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/2971fdd0-edfb-4919-8d50-e7d08d8f3136" />


### 9. Explore reponse_time column

```sql
-- Checking response_time column for missing, NULL, or inconsistent values

SELECT DISTINCT response_time
FROM support
ORDER BY response_time;
```

Listed unique `response_time`  values to inspect their **range, granularity, and any non-numeric entries**. This allows me to identify missing values **(NULL), anomalies, such as negative values, extreme outliers, or text, and determine whether to cast or impute them.**

<img width="800" alt="image" src="https://github.com/user-attachments/assets/63cc08b5-1c1b-4e05-a473-008e05da8903" />


### 10. Explore resolution_time column

```sql
-- Checking resolution_time column for missing, NULL, or inconsistent values

SELECT DISTINCT resolution_time
FROM support
ORDER BY resolution_time;
```

Listed unique `resolution_time` values to identify mixed formats, such as **text entries or empty strings**. This ensured that all values were **cleaned, converted to numeric form, and ready for accurate hour-based SLA reporting and category comparisons.**

<img width="800" alt="image" src="https://github.com/user-attachments/assets/2a29a9b2-b285-4eb9-ad2a-c8f7453974c7" />


After thoroughly exploring each column of the `support` table, we observed the following:

* `id`: This column was perfect with **1987 unique IDs**, no NULLs or inconsistencies, providing a reliable primary key. **No cleaning was needed.**
* `customer_id`: Contained **1237 unique customer IDs** with no missing or invalid values, ensuring accurate mapping to surveys.
* `category`: Had **5 distinct categories**. Any NULLs, missing, or inconsistent values were replaced with **'Other' to maintain consistent grouping**.
* `status`: Contained **4 distinct values**, including one inconsistent entry '-'. This was corrected to **'Resolved' to ensure accurate ticket tracking**.
* `creation_date`: Included 334 distinct dates. Any NULL or invalid entries were replaced with '2023-01-01' to maintain timeline consistency.
* `response_time`: Had **15 distinct values**. NULLs or missing entries were replaced with **0 to enable accurate response analysis.**
* `resolution_time`: **Contained 246 distinct hours**. Any NULL or non-numeric entries were replaced with **0 and cast carefully to numeric for reliable SLA reporting**.

By addressing these issues—**replacing missing values, correcting inconsistencies, and standardizing formats**—I created a **clean and reliable dataset**. This enables precise analysis of response times, ticket categories, SLA performance, and customer satisfaction trends, providing a solid foundation for operational improvements.


### 11. Explore customer_id column

```sql
-- Checking customer_id column for missing, NULL, or inconsistent values

SELECT DISTINCT customer_id
FROM survey
ORDER BY customer_id;
```

Listed unique `survey` `customer_id` values to see how many customers submitted feedback and to identify matches with `support.customer_id`. This ensured **accurate join coverage and guided handling of unmatched rows** when analyzing the relationship between response_time and survey ratings.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/fda2bb5e-b583-452e-b13a-a4a9e88e4922" />


### 12. Explore the rating column

```sql
-- Checking rating column for missing, NULL, or inconsistent values

SELECT DISTINCT rating
FROM survey
ORDER BY rating;
```

Listed unique `rating` values to check for unexpected numbers or NULLs. This ensured **all ratings aligned** with the expected scale and were ready for accurate category averages and correlation analysis with `response_time`.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/6140e3fc-18a2-416c-a6ca-f1e13f5226b8" />


### 13. Explore the timestamp column

```sql
-- Checking timestamp column for missing, NULL, or inconsistent values

SELECT DISTINCT timestamp
FROM survey
ORDER BY timestamp;
```
Listed distinct `timestamp` values to inspect their **format, granularity, and range**. This ensured alignment with `support.creation_date` and prepared the data for **accurate temporal joins and trend analyses.**

<img width="800" alt="image" src="https://github.com/user-attachments/assets/dd0d276a-9030-44eb-9cf1-2183a1fb12fa" />


After thoroughly exploring each column of the `survey` table, we observed the following:

* `survey_id`: This column was perfect with **1987 unique IDs**, no NULLs or inconsistencies, providing a reliable primary key. **No cleaning was needed.**
* `customer_id`: Contained **192 distinct customer IDs**, ensuring each survey could be accurately linked to a support ticket.
* `rating`: **Had 6 distinct values**, all valid and within the expected scale, enabling precise analysis of customer satisfaction.
* `timestamp`: Included **10 distinct timestamps with no missing or invalid entries**, supporting accurate temporal analysis of survey submissions.

By reviewing these columns and confirming their integrity, we ensured a **clean and reliable survey dataset**. This allows accurate correlation analysis with support tickets and meaningful insights into customer satisfaction trends.


### 14. Cleaning the Dataset

```sql
--- Cleaning and replacing missing values, NULLS, and inconsistent data from our dataset for accurate analysis.

SELECT
	-- Required, never NULL per specs
    id,  

	
	--- Replacing NULL in column customer_id
    COALESCE(customer_id, 0) AS customer_id,  -- NULL → 0


	---  Replacing NULL with 'Other' in the category column
    CASE 
        WHEN category IS NULL 
			THEN 'Other'
        WHEN category NOT IN ('Feedback',' Billing Enquiry', 'Bug', 'Installation Problem', 'Other') 
			THEN 'Other'
        ELSE category
    END AS category,  -- NULL/invalid → 'Other'


	--- Replacing NULL with 'Resolved' in the status column
    CASE
        WHEN status IS NULL 
			THEN 'Resolved'
        WHEN status NOT IN ('Open', 'In Progress',' Resolved') 
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

I cleaned the support dataset thoroughly to prepare it for reliable analysis. First, I replaced NULLs with safe defaults, such as 0 for `customer_id` and `response_time`, and ‘2023-01-01’ for `creation_date`, ensuring no record was incomplete or unusable. Next, I standardized key columns like `category` and `status`, mapping invalid or missing entries to canonical values so that inconsistencies would not break aggregations or groupings later on. I also cleaned `resolution_time` by removing stray text, converting blanks to NULLs, casting to numeric, and rounding for uniform precision. I tested each transformation individually using a CTE before combining everything into a single cleaned dataset. This cleaning process was critical because even minor inconsistencies could produce misleading results—for example, inaccurate SLA calculations or distorted response-time trends. By carefully preparing the dataset, I created a solid foundation for analysis, enabling confident exploration of trends, performance by category, and correlations between support efficiency and customer satisfaction.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/bf792794-4f3f-45f7-beca-61574a07b7bf" />


### 15. Calculate the minimum and maximum response time by category

```sql
-- Getting the min and max response time and rounding the values to 2 decimal places. Grouping by category to get the min and max for each group.

SELECT 
		category,
		ROUND(MIN(response_time), 2) AS min_response, 
		ROUND(MAX(response_time), 2) AS max_response
FROM support
GROUP BY category;
```

I analyzed the minimum and maximum `response_time` for each ticket `category` to understand the full range of support performance. By identifying which categories, such as Bugs or Installation Problems, experienced the slowest or most inconsistent responses, I could clearly see where delays were most common and which areas needed attention. This detailed understanding of response-time distribution allowed the team to pinpoint inefficiencies, prioritize improvements, and enhance overall support efficiency.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/7d650108-98b9-4638-8acb-db6c728b37b1" />


### 16. Link survey ratings with support tickets for Bug and Installation Problem categories

```sql
--- Joining the tables

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

I analyzed support `tickets` with survey `ratings`, focusing on categories like Bugs and Installation Problems. By examining how response times in these technical issue categories affected customer satisfaction, I could identify where faster resolutions would most improve `ratings`. This insight allowed the team to prioritize efforts on the areas that matter most to unhappy customers, optimize support workflows, and enhance overall customer experience.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/3b709568-bfaf-4da7-aa9f-29e59e697902" />


<br>

## INSIGHTS AND FINDINGS

* NULLs, invalid categories/statuses, missing dates, and non-numeric resolution_time were **replaced or corrected**, creating a **reliable dataset for analysis**.
* Bugs and Installation Problems show the **highest max_response (13–17)**, while Feedback and Other categories have lower, more consistent response times, highlighting **areas with potential delays**.
* Joining support with a survey for Bug and Installation Problem tickets returned 123 rows, showing that **longer response times in these categories correspond with lower customer satisfaction ratings**.

<br>

## RECOMMENDATIONS
* Build a persistent cleaned view to ensure **consistent defaults and category mapping across all analyses**.
* Prioritize **reducing max_response** for categories with the **slowest tickets and audit workflows** for large min–max gaps to fix bottlenecks.
* Compute average rating by response_time bins for Bugs and Installation Problems to link **SLA targets with improved customer satisfaction**.
* Replace imputed zeros in response_time with **meaningful markers or fix ingestion errors to avoid skewed aggregates**.
* Ensure resolution_time and other numeric fields are **complete and correctly typed** at ingestion to eliminate post-hoc cleaning.
* Perform **regular data quality checks** on counts, distinct values, and NULL percentages, reporting anomalies to the engineering team.

<br>

## FINAL NOTES
This analysis demonstrated how SQL can **transform raw, inconsistent support data into actionable insights for Tech Solutions**. By systematically validating and cleaning categories, statuses, dates, and numeric fields, the company can **measure response performance accurately and link it to customer satisfaction**. The focus on Bug and Installation Problem tickets provides a clear sample to understand **how delays impact ratings, guiding operational improvements where they matter most**. Future steps could include building a standard cleaned view, enforcing **stricter ingestion rules**, and using **SLA-driven targets** to improve slow-response categories and **boost overall customer retention**.

<br>

## REFERENCE

* [DataCamp](https://app.datacamp.com/)
* Internal Tech Solutions Inc. 
  
<br>
