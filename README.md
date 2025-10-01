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

### 2. Data Exploration: Understanding the `survey` Table

```sql
-- Overview of the survey table: preview first 10 rows

SELECT *
FROM survey 
LIMIT 10;
```

### 3. Check `id` for NULLs

```sql
-- Checking id column for missing, NULL, or inconsistent values

SELECT DISTINCT id
FROM support
WHERE id IS NULL;
```

### 4. Explore product_type


### 5. Explore product_type


### 6. Explore product_type


### 7. Explore product_type


### 8. Explore product_type


### 9. Explore product_type


### 10. Explore product_type


### 11. Explore product_type









<br>
<br>






<br>
<br>










<br>
<br>


























