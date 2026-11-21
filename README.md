# customer_behavior_analysis-
An end-to-end customer analytics pipeline: Python (pandas) cleans the data and engineers features, PostgreSQL stores it, and SQL (CTEs, window functions) answers business questions. A Power BI dashboard presents the insights on customer segments, category revenue, subscriptions and discounts, with recommendations for retention and marketing.

Dashboard Preview

Customer Behaviour Dashboard

Overview

This project analyses a customer shopping dataset of roughly 3,900 records to answer practical business questions:

Who are the customers, and what do they buy?
Which categories and age groups drive the most revenue?
Do subscriptions, discounts and shipping options influence spending?
How loyal are customers, and where is the retention opportunity?

The work is organised as a reproducible pipeline: raw CSV → cleaned data in pandas → PostgreSQL table → SQL analysis → Power BI dashboard.

Architecture
customer_shopping_behavior.csv
            │
            ▼
   Python (pandas) ── clean, impute, engineer features
            │
            ▼
   PostgreSQL ── cleaned `customer` table (via SQLAlchemy)
            │
            ├──► SQL analysis (CTEs, window functions, subqueries)
            │
            ▼
   Power BI ── interactive dashboard
Tech Stack
Layer	Tools
Data cleaning	Python, pandas, numpy
Database connectivity	psycopg2, SQLAlchemy
Storage and analysis	PostgreSQL, SQL
Visualisation	Power BI
Repository Structure
.
├── data/
│   └── customer_shopping_behavior.csv
├── notebooks/
│   └── data_cleaning_and_loading.ipynb
├── sql/
│   └── analysis_queries.sql
├── dashboard/
│   └── customer_behaviour_dashboard.pbix
├── images/
│   └── dashboard.png
├── docs/
│   └── project_documentation.pdf
├── .env.example
├── requirements.txt
└── README.md

Adjust the paths above to match your actual folder layout.

Data Preparation (Python)
Step	What was done	Why
Missing values	Filled missing Review Rating values with the median of each category	Median is robust to outliers; grouping keeps values realistic
Column standardisation	Lowercased names, replaced spaces with underscores, renamed purchase_amount_(usd) to purchase_amount	Clean names avoid quoting issues in SQL
Age groups	Created age_group with pd.qcut into four quartile bins (Young Adult, Adult, Middle Aged, Senior)	Equal-sized groups make revenue comparisons fair
Purchase frequency	Mapped text frequencies to payment_frequency_days	Numeric values can be compared and averaged
Redundant column	Dropped promo_code_used after confirming it was identical to discount_applied	Removes duplicate information
Load to database	Wrote the cleaned DataFrame to PostgreSQL with df.to_sql	Enables SQL analysis and the Power BI connection
SQL Analysis

The queries in sql/analysis_queries.sql answer the following questions:

What is the total revenue by gender?
Which customers used a discount and still spent more than the average purchase?
What are the top 5 products by average review rating?
How does average spend compare between standard and express shipping?
How many customers are New, Returning or Loyal?
Do subscribed customers spend more than non-subscribers?
What are the top 3 best-selling items in each category?
Are subscribers more likely to be repeat buyers?
Which age group generates the most revenue?

SQL concepts used: aggregation, GROUP BY, subqueries, CASE expressions, CTEs, and window functions (ROW_NUMBER() OVER (PARTITION BY ...)).

Example: top 3 items per category

sql
WITH item_counts AS (
    SELECT category,
           item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (
               PARTITION BY category
               ORDER BY COUNT(customer_id) DESC
           ) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
Dashboard

The Power BI dashboard connects to the PostgreSQL customer table and includes:

KPI cards: number of customers, average purchase amount, average review rating
Subscription split: percentage of customers by subscription status
Category performance: revenue and sales by category
Age group analysis: revenue by age group
Slicers: filter every visual by subscription status, gender and category
Key Insights
Subscription is low: only 27% of customers are subscribed, so there is clear room to grow.
Clothing leads: it is the top category by both revenue (104,264) and number of sales, followed by Accessories, Footwear and Outerwear.
Spending is evenly spread by age: Young Adult and Middle Aged groups generate similar revenue.
Moderate satisfaction: the average review rating is 3.75.
Recommendations
Introduce subscriber-only perks to grow the 27% subscriber base.
Focus inventory and marketing on Clothing and Accessories.
Target discount-responsive, high-spending customers with personalised offers.
Reward Loyal customers and nudge Returning customers toward loyalty.
Feature top-rated products in campaigns and review low-rated ones.
Getting Started
Prerequisites
Python 3.10 or later
PostgreSQL installed and running
Power BI Desktop (to open the .pbix file)
Installation
bash
# Clone the repository
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>

# Install dependencies
pip install -r requirements.txt
Configure database credentials

Create a .env file (never commit it) based on .env.example:

PG_USER=your_username
PG_PASSWORD=your_password
PG_HOST=127.0.0.1
PG_PORT=5432
PG_DATABASE=customer
Run the pipeline
Create an empty PostgreSQL database named customer.
Run notebooks/data_cleaning_and_loading.ipynb to clean the data and load the customer table.
Execute the queries in sql/analysis_queries.sql in your SQL client.
Open dashboard/customer_behaviour_dashboard.pbix and point the data source to your PostgreSQL database.
requirements.txt
pandas
numpy
psycopg2-binary
sqlalchemy
python-dotenv
jupyter
Limitations and Future Work
The analysis is descriptive; churn prediction and customer lifetime value modelling would be natural extensions.
Age groups are quartile-based, so boundaries depend on this dataset.
The dataset has no purchase dates, so time trends and seasonality cannot be analysed.
Possible additions: RFM segmentation, an automated pipeline, and scheduled dashboard refreshes.
Author

Nathan D'Souza MSc Big Data Analytics, St. Xavier's College (Autonomous), Mumbai

LinkedIn · GitHub

License

This project is licensed under the MIT License. See the LICENSE file for details.
