# Customer Shopping Behavior Analysis

Automated pipeline that ingests retail transaction data from email, cleans it, runs analysis in Snowflake, and feeds a live Power BI dashboard.

![](https://github.com/sahilansari79923-byte/Customer-Shopping-Behavior-Analysis/blob/main/Snaps/Screenshot%202026-08-07%20145524.png)

## The problem

A retail business had 3,900 transactions worth of raw data and no clear read on what was driving purchases or repeat business. This started as a manual analysis to answer that. It's since grown into an automated pipeline that picks up new transaction reports on its own and keeps the dashboard current without anyone re-running a script.

## Pipeline

```
Gmail → n8n → S3 → pandas (clean) → Snowflake (analyze) → Power BI
```

1. **Ingestion** - n8n watches Gmail for the transaction report and uploads the attachment to S3
2. **Storage** - S3 holds the raw file
3. **Cleaning** (pandas) - imputes missing Review Ratings using the category median, standardizes column names, builds `age_group` and `purchase_frequency_days`, drops the redundant `promo_code_used` column
4. **Loading** - cleaned data goes into Snowflake
5. **Analysis** (SQL) - revenue splits, customer segmentation, discount behavior, product performance
6. **Dashboard** - Power BI connects to Snowflake and refreshes automatically

## Automation

n8n watches Gmail for the transaction report and pushes new attachments to S3 automatically. No manual upload step.

![n8n workflow](images/n8n-workflow.png)

The file lands in the bucket within seconds of the trigger firing.

![S3 bucket](images/s3-bucket.png)

S3 sits between Gmail and Snowflake on purpose. It decouples ingestion from transformation and keeps a raw copy of every file that comes in, so a bad load into Snowflake never means re-pulling from Gmail.

## Dataset

- 3,900 transactions, 18 columns of transaction and customer data
- 37 missing values in Review Rating, imputed using the median per product category

## Cleaning (pandas)

Missing ratings filled per category instead of a flat median, so a low-rated category doesn't get pulled toward the dataset average:

```python
df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(
    lambda x: x.fillna(x.median())
)
```

Age split into quartile-based groups rather than fixed bins, so each group holds roughly the same number of customers:

```python
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)
```

## Analysis (Snowflake SQL)

Customer segments by purchase history:

```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
    CASE 
        WHEN previous_purchases = 1 THEN 'New'
        WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
        ELSE 'Loyal'
    END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```

Customers who used a discount but still spent above the dataset average, the "smart shopper" group:

```sql
SELECT customer_id, purchase_amount
FROM customer
WHERE discount_applied = 'Yes'
  AND purchase_amount >= (SELECT AVG(purchase_amount) FROM customer);
```

Top 3 products per category, ranked by order count:

```sql
WITH item_counts AS (
    SELECT category, item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

## Key findings

- Female customers generated slightly higher total revenue than male customers (revenue-by-gender query)
- Express shipping customers spent 12% more per transaction, $65 vs. $58 average (shipping comparison query)
- Subscribers spent 68% more, drove 45% of total revenue, and had a 78% loyalty rate (subscription query)
- Customer base splits into Loyal (15%), Returning (35%), New (50%). Biggest opportunity is moving New into Returning and Returning into Loyal (segmentation query)
- The smart-shopper segment above spends more than average even with a discount applied, worth targeting with premium offers instead of deeper discounts
- Blouses, dresses, and shirts had the strongest customer satisfaction scores (top-rated products query)

## Recommendations

- Grow the subscriber base with subscriber-only perks
- Build a loyalty program to retain repeat buyers
- Target high-revenue and express-shipping segments directly
- Feature top-rated products in marketing campaigns

## Project structure

```
├── data/                  # Raw and cleaned datasets
├── notebooks/             # pandas cleaning and feature engineering
├── sql/                   # Snowflake analysis queries
├── automation/            # n8n workflow export
├── dashboard/             # Power BI dashboard file
├── images/                # Screenshots referenced in this README
├── report/                # Project report and presentation
└── README.md
```

## Author

Md Sahil Ansari
