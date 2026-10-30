# sqlproject
SQL Postgresql Jupyter Data Analysis
E-commerce Sales and Delivery Analysis

A portfolio and learning project using PostgreSQL, SQL, Python, and Jupyter to explore sales and delivery performance in an online marketplace.

Status: Planning. This README is the starting specification. The database, scripts, queries, and notebook described below are planned deliverables; no results or performance improvements are claimed yet.

Project purpose

Build a relational database from real marketplace data, answer practical questions with SQL, and use Python to interpret and present the results.

The central business question is:

What is selling, where are delivery delays concentrated, and how are delays associated with customer satisfaction?

The project will connect database theory with implementation: identifying keys and functional dependencies, explaining relationships, choosing constraints, writing queries, and checking the meaning of the results.

Dataset

Source: Brazilian E-Commerce Public Dataset by Olist.

The provider describes approximately 100,000 anonymised Brazilian marketplace orders from 2016–2018, distributed across related CSV files.

The first version will use:

olist_customers_dataset.csv

olist_orders_dataset.csv

olist_order_items_dataset.csv

olist_products_dataset.csv

olist_sellers_dataset.csv

Later extensions will add payments and reviews. Category-name translations may be added as a lookup. Geographic coordinates are outside the initial scope.

The dataset is historical, so findings will describe this observation period. Download instructions and source attribution will be included rather than committing the raw dataset to the repository.

Dataset license listed by the provider: CC BY-NC-SA 4.0. Dataset terms are separate from any license later selected for the project code.

How PostgreSQL, SQL, and Python fit together

Component

Responsibility

PostgreSQL

Stores the actual database, tables, constraints, and committed records

SQL

Defines tables and expresses questions or changes to the data

Python

Reads CSV files, connects to PostgreSQL, sends SQL, and works with returned results

Psycopg

Provides the connection between Python and PostgreSQL

Jupyter notebook

Combines explanations, query results, charts, and statistical interpretation

A .sql file contains instructions; it is not the database. A Python program can execute SQL written in a string or loaded from a .sql file. PostgreSQL executes those instructions.

The planned workflow is to create the database structure, import records through Python, query PostgreSQL, and bring the query results into a notebook. The tables and committed records remain in PostgreSQL after the Python process closes.

Initial database model

The first version will stay close to the source structure while documenting and validating its design.

Table

Meaning of one row

Proposed primary key

Main relationships

customers

One source customer record with purchase-location information

customer_id

Referenced by orders

orders

One order and its lifecycle dates

order_id

References customers

order_items

One numbered item within an order

(order_id, order_item_id)

References orders, products, and sellers

products

One product and its characteristics

product_id

Referenced by order_items

sellers

One seller and its location

seller_id

Referenced by order_items

Customer identifiers require special care

In this dataset, customer_id is assigned per order. The same person can have several different customer_id values. The stable identifier supplied for recognising repeat purchasers is customer_unique_id.

Use customer_id to join the source orders and customer records. Use customer_unique_id when grouping purchases by the same customer.

This means that a row in the source-aligned customers table should not be interpreted as a distinct person. An optional modelling exercise will separate stable customer identity from order-specific location details.

Design decisions to explain

Why orders and order items are separate tables.

Why order_item_id is identified together with its order_id.

How order_items connects orders, products, and sellers.

Why the historical sale price belongs to an order item.

Which fields may be missing and what those missing values mean.

Which functional dependencies hold and whether the proposed tables satisfy the normal forms studied in class.

For example, the intended item-level dependency is:

(order_id, order_item_id) → product_id, seller_id, price, freight_value

The source files are a starting point for investigation. Their existence as separate CSV files does not, by itself, prove that the target model is normalised.

Integrity and data types

Profile the source before finalising constraints. Candidate rules include primary keys, foreign keys, appropriate NOT NULL constraints, and justified checks on monetary values.

Use exact decimal types such as NUMERIC for monetary amounts. Preserve source identifiers as opaque strings and parse timestamps consistently without inventing timezone information.

Planned SQL analysis

Keep the core joins, filtering, grouping, and metric calculations in SQL. Each analysis query should state its question, filters, and what one output row represents.

Question

Main SQL practice

How many orders have each status?

GROUP BY, COUNT

Which orders have missing lifecycle dates?

Filtering, IS NULL

Which categories have the highest product sales value?

Joins, SUM, sorting

What is the average product subtotal per delivered order?

Aggregation at the order level

What share of delivered orders arrived late?

Date comparisons, conditional aggregation

Which customers purchased more than once?

Joins, HAVING, stable customer identity

How did monthly product sales change?

CTEs, LAG

Which sellers rank highest within each category?

Window functions and ranking

Do item totals and payment totals reconcile?

Aggregation before joining; later extension

How do reviews differ between late and on-time orders?

Order-level joins; later extension

An illustrative query for the planned schema:

-- Delivered orders with items, grouped by purchase month.
-- Product sales value excludes freight charges.
SELECT
    DATE_TRUNC('month', o.order_purchase_timestamp)::date AS purchase_month,
    COUNT(DISTINCT o.order_id) AS delivered_orders_with_items,
    SUM(i.price) AS product_sales_value
FROM orders AS o
JOIN order_items AS i
    ON i.order_id = o.order_id
WHERE o.order_status = 'delivered'
GROUP BY 1
ORDER BY 1;

COUNT(DISTINCT o.order_id) counts orders once even when an order contains several items. The inner join means this query includes only orders with matching item records.

Metric definitions

Metric

Initial definition

Product sales value

Sum of item prices for the selected delivered orders, excluding freight

Average order product subtotal

Average of per-order item-price sums among selected delivered orders with items

Delivery duration

Elapsed time from purchase to delivery, for records with valid timestamps

Late delivery

Actual delivery calendar date is after the estimated delivery calendar date

Late-delivery rate

Late delivered orders divided by delivered orders with both dates available

Repeat buyer

A customer_unique_id with at least two qualifying orders within the dataset

Report the eligible record count for every metric. Treat unavailable dates separately from on-time deliveries. Define the selected period and check whether the first and last months are complete before comparing monthly trends.

Product sales value is not a measure of platform profit.

Python and statistical analysis

Python will support a small, repeatable workflow:

Read and profile the CSV files.

Apply documented conversions and cleaning rules.

Load parent records before dependent records.

Send parameterised SQL to PostgreSQL.

Retrieve query results for charts and statistics.

The notebook will explain the questions, show selected SQL results, and interpret approximately three useful charts.

Planned statistical work:

Describe delivery duration using distributions, medians, and selected percentiles.

Compare review-score distributions for late and on-time deliveries.

Optionally estimate the difference in low-rating proportions with a confidence interval.

Before the review analysis, inspect repeated order identifiers and define a documented rule for producing the required order-level review dataset.

Report group sizes, exclusions, and limitations. Observed differences will be described as associations. Order-level delivery dates and reviews also require care when an order contains items from multiple sellers.

Data quality and query correctness

The project will record and explain:

Missing identifiers and dates.

Duplicate candidate keys and unmatched foreign-key references.

Unexpected prices, freight values, or timestamp sequences.

Cleaning decisions and counts of retained or excluded records.

Reconciliation between source row counts and imported records.

Keep raw inputs unchanged. Investigate problematic records before deciding how to handle them.

A specific query-correctness exercise will address joins between tables with multiple rows per order. For example, three item rows joined directly to two payment rows produce six rows. Aggregate each source to the intended level before combining totals.

Planned repository layout

The following files are proposed; they have not yet been implemented.

Path

Purpose

README.md

Project scope, methods, progress, and eventual findings

sql/schema.sql

Table definitions, keys, and constraints

sql/quality_checks.sql

Validation and reconciliation queries

sql/analysis/

Commented analytical queries

scripts/load_data.py

Repeatable CSV import

notebooks/analysis.ipynb

Charts, statistics, and interpretation

docs/database_design.md

Relationships, functional dependencies, and design decisions

docs/data_dictionary.md

Column meanings, units, and missing-value rules

docs/performance.md

One measured query-plan investigation

data/raw/

Local source files, excluded from version control

Development roadmap

Read the source documentation and inspect the five initial CSV files.

Explain five example orders and their related records.

Draft the relationship diagram, keys, and functional dependencies.

Create the initial PostgreSQL tables and validate a small sample.

Implement the repeatable Python import and data-quality checks.

Write approximately ten SQL analyses with explicit metric definitions.

Add reviews and, optionally, payments.

Build the notebook with charts and written interpretation.

Investigate one query using EXPLAIN ANALYZE and measure any justified change.

Add verified setup instructions, findings, and limitations to this README.

First milestone: a small database whose rows, relationships, and query results can all be explained independently.

Setup status and reproducibility

Planned prerequisites are PostgreSQL, Python, Psycopg, Jupyter, and suitable plotting and statistical packages.

Executable installation and run instructions will be added after the corresponding files exist and the workflow has been tested. Record the actual dependency versions and connection configuration at that stage.

The completed workflow should allow another person to obtain the source data, create the schema, load records, run the SQL, and reproduce the notebook. Connection credentials should remain in local configuration outside version control.

The initial scope is the database and analysis workflow. Dashboards, machine learning, and a web application are optional future work.

Results

Pending implementation and analysis.

This section will eventually contain a small set of verified findings, the SQL or notebook supporting each finding, and the practical limitations of the data. No example finding should be presented as an observed result.

References

Olist dataset and source documentation

PostgreSQL constraints

PostgreSQL window functions

PostgreSQL query plans and EXPLAIN

Psycopg basic usage

