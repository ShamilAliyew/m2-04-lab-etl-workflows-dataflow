![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Lab | ETL Workflows and Dataflow

## Overview

This lab puts the concepts of ETL pipelines, dataflow modes, and batch vs stream processing into practice. You will build a complete ETL pipeline from raw messy data to clean analytical tables, simulate how data passes between services through different dataflow modes, and implement both batch and stream processing patterns on the same dataset. The goal is to develop hands-on familiarity with the patterns that underlie every production data system.

## Learning Goals

By the end of this lab, you should be able to:

- Build a working ETL pipeline with explicit extract, transform, and load phases
- Implement data validation and rejection with logging during extraction
- Perform common transformations: cleaning, joining, deduplicating, and feature derivation
- Simulate the three modes of dataflow: through databases, through services, and through a message broker
- Implement both batch processing and stream processing on the same data
- Combine batch and stream features into a single feature vector

## Setup and Context

You will work in a Jupyter Notebook. All data is created in memory. No external databases, APIs, or message brokers are required — you will simulate these patterns using Python data structures. The focus is on the patterns and logic, not infrastructure setup.

## Requirements

- Fork this repository to your own GitHub account.
- Clone your fork to your machine.
- Make sure you have `pandas`, `numpy`, and `pyarrow` installed.

```bash
pip install pandas numpy pyarrow
```

## Getting Started

Create a new notebook and name it `m2-04-etl-workflows-dataflow-lab.ipynb`. Complete all tasks in this notebook with code and markdown explanations. Before you submit, restart your kernel and run the notebook top to bottom.

## Tasks

### Task 1: Build a Complete ETL Pipeline

Build an ETL pipeline that processes raw e-commerce order data into a clean analytical dataset.

**Step 1 — Generate raw data:** Create a list of 200 raw order records as dictionaries with these fields: `order_id`, `customer_id`, `product_name`, `quantity`, `unit_price`, `order_date`, `shipping_country`. Intentionally include at least 15 problematic records across these categories:
- 3 records with missing `product_name` (None or empty string)
- 3 records with negative `quantity` or `unit_price`
- 3 records with malformed `order_date` (e.g., "not-a-date", "2025-13-45")
- 3 records with duplicate `order_id` values
- 3 records where `quantity` or `unit_price` is a string instead of a number

**Step 2 — Extract with validation:** Write an `extract(raw_records)` function that:
- Parses each record and validates required fields exist
- Validates that `quantity` and `unit_price` are positive numbers
- Validates that `order_date` is a parseable date
- Returns two lists: `valid_records` and `rejected_records`
- Each rejected record should include the reason for rejection
- Print a summary: count of valid, count of rejected by reason

**Step 3 — Transform:** Write a `transform(valid_records)` function that:
- Converts to a pandas DataFrame
- Computes `total_amount = quantity * unit_price`
- Extracts `order_month` and `order_day_of_week` from the date
- Standardizes `shipping_country` to title case
- Removes duplicate `order_id` values (keep the first)
- Adds an `amount_category` column: "small" (<$25), "medium" ($25-$100), "large" (>$100)
- Returns the cleaned DataFrame

**Step 4 — Load:** Write a `load(df, path)` function that saves the DataFrame to a Parquet file. Then read it back and verify the row count and dtypes match.

**Step 5:** Run the full pipeline: `extract → transform → load`. Print a summary showing records at each stage (raw → valid → transformed → loaded).

### Task 2: ETL vs ELT Comparison

**Step 1:** Implement an ELT variant of the pipeline from Task 1:
- Extract: do minimal validation (only reject unparseable JSON/records)
- Load: save the raw (mostly unvalidated) data to a Parquet "data lake" file
- Transform: read from the data lake file, apply the same validation and transformation rules as Task 1

**Step 2:** Compare ETL and ELT approaches. Write a markdown cell addressing:
- How many records made it to the destination in each approach?
- At what stage were problems caught in each?
- What are the advantages of each approach?
- When would you choose one over the other?

### Task 3: Simulate Modes of Dataflow

Model a simplified analytics system with three components: an order ingestion service, a feature computation service, and a prediction service.

**Step 1 — Data passing through a database:** Simulate a shared database using a dictionary. The order service writes new orders to the database. The feature service reads orders and computes features (total orders, average amount, last order date per customer). The prediction service reads the features.

```python
# Shared database (simulated)
database = {"orders": [], "features": {}}

def order_service_write(order):
    database["orders"].append(order)

def feature_service_compute():
    # Read from database, compute features, write back
    ...

def prediction_service_read(customer_id):
    # Read features from database
    ...
```

**Step 2 — Data passing through services:** Refactor so the prediction service requests data from the feature service directly (function call simulating an API request). The feature service requests raw orders from the order service.

**Step 3 — Data passing through a message broker:** Implement a simple pubsub broker (a class with `publish`, `subscribe`, and `get_latest` methods). The order service publishes new orders to an "orders" topic. The feature service subscribes and computes features whenever new orders arrive, publishing them to a "features" topic. The prediction service subscribes to "features."

**Step 4:** Run 20 new orders through all three dataflow modes. Verify that the prediction service gets the same features in each case. Write a markdown cell comparing the three modes: coupling, latency characteristics, and what happens when one component goes down.

### Task 4: Batch Processing vs Stream Processing

**Step 1 — Batch processing:** Using the cleaned data from Task 1, write a batch processing function that computes daily aggregate features:
- Total orders per day
- Total revenue per day
- Average order size per day
- Number of unique customers per day
- Top product per day (by revenue)

Execute it and display the results.

**Step 2 — Stream processing:** Implement a `StreamProcessor` class that processes orders one at a time and maintains running statistics:
- Running total of orders and revenue
- Windowed average (last 50 orders) of order size
- Running count of unique customers
- Current most popular product (by count in the last 50 orders)

Process the same data from Task 1 through the stream processor, one record at a time. After every 50 records, print the current streaming statistics.

**Step 3 — Compare final results:** After processing all records through both approaches, compare the final totals. They should match for cumulative statistics (total orders, total revenue). Write a markdown cell explaining: Why might windowed streaming statistics differ from daily batch statistics even on the same data? What does each approach tell you that the other does not?

### Task 5: Combine Batch and Stream Features

**Step 1:** Define a set of batch features and streaming features for a customer in an e-commerce system:

Batch features (computed from all historical data):
- `total_lifetime_orders`
- `avg_order_amount`
- `days_since_first_order`
- `most_purchased_category`

Streaming features (computed from last 10 orders):
- `recent_order_count`
- `recent_avg_amount`
- `seconds_since_last_order`
- `recent_top_category`

**Step 2:** For 5 sample customers, compute both batch and streaming features from the Task 1 data.

**Step 3:** Combine them into a single feature dictionary per customer. Print the combined feature vectors for all 5 customers.

**Step 4:** Write a markdown cell explaining why an ML model would benefit from having both batch and stream features. Give an example of a prediction task where the batch features alone would be insufficient, and another where the stream features alone would be insufficient.

## Submission

### What to submit

Submit the following file:

- A notebook file `m2-04-etl-workflows-dataflow-lab.ipynb` containing all five tasks with code and markdown explanations

### Definition of done (checklist)

Before you submit, make sure:

- [ ] The notebook runs **top to bottom** without errors after a kernel restart.
- [ ] Task 1 includes a complete ETL pipeline with extraction validation, transformation, and Parquet loading.
- [ ] Task 2 compares ETL and ELT with a written analysis.
- [ ] Task 3 simulates all three dataflow modes and includes a comparison.
- [ ] Task 4 implements both batch and stream processing with result comparison.
- [ ] Task 5 combines batch and stream features with an explanation of their complementary roles.
- [ ] Markdown cells contain thoughtful analysis connecting code to concepts.

### How to submit (Git workflow)

When you are done, make sure all changes are saved, then run:

```bash
git add .
git commit -m "Solved m2-04 lab"
git push -u origin HEAD
```

- Make a pull request.
- Paste the link to your pull request in the Student Portal.

## Evaluation Criteria

Your work will be evaluated on pipeline correctness, design quality, and analytical reasoning. The ETL pipeline should handle messy data gracefully with clear validation logic. Dataflow simulations should correctly demonstrate the differences between modes. Batch and stream implementations should produce consistent results on the same data. Your markdown explanations should demonstrate understanding of when and why each pattern is used in production, not just describe what the code does.
