# Customer Shopping Behavior Analysis

Analysis of 3,900 retail transactions to identify what drives customer spend and repeat purchases, with a Power BI dashboard and a set of business recommendations.

## Overview

A retail business had raw transaction data and no clear read on what was driving purchases or repeat business. This project cleans the dataset, models it in a relational database, and analyzes it to answer: what customer segments matter most, how shipping and subscription behavior relate to spend, and which products are performing best.

## Dataset

- 3,900 total purchases
- 18 columns covering transaction and customer demographic data
- 37 missing values, all in the Review Rating column, imputed using the median

## Tech stack

- **Python (pandas)** — data cleaning, feature engineering
- **PostgreSQL** — structured storage and querying
- **Power BI** — interactive dashboard
- **SQL** — segment and revenue-driver queries

## Process

1. **Data loading** — imported the raw dataset with pandas
2. **Initial exploration** — structure check and summary statistics
3. **Missing data handling** — imputed Review Rating with median values
4. **Feature engineering** — created age groups and purchase frequency bands
5. **Database integration** — loaded cleaned data into PostgreSQL for querying

## Key findings

- **Revenue by gender**: Female customers generated slightly higher total revenue than male customers.
- **Shipping and spend**: Express shipping customers spent 12% more per transaction ($65 vs. $58 average).
- **Subscription impact**: Subscribers showed 68% higher spend, made up 45% of total revenue, and had a 78% loyalty rate vs. non-subscribers.
- **Customer segmentation**: Loyal (15%), Returning (35%), New (50%) — with the biggest opportunity in converting New to Returning and Returning to Loyal.
- **High-value discount users**: A "smart shopper" segment spends above average even while using discounts, a group worth targeting with premium offers.
- **Top-rated products**: Blouses, dresses, and shirts had the strongest customer satisfaction scores.

## Recommendations

- Grow the subscriber base through exclusive, subscriber-only benefits
- Build loyalty rewards to increase retention among repeat buyers
- Target high-revenue and express-shipping segments directly
- Feature top-rated products in marketing campaigns

## Project structure

```
├── data/                  # Raw and cleaned datasets
├── notebooks/             # Python scripts / notebooks for cleaning & feature engineering
├── sql/                   # SQL queries used for analysis
├── dashboard/             # Power BI dashboard file
├── report/                # Project report and presentation
└── README.md
```

## Author

Md Sahil Ansari
