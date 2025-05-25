# Intelligent Inventory Management for Hotel Bar Operations

**Developed by:** [Pratheek Nistala]("https://pratheek.tech")

## Table of Contents

1. Project Overview
2. The Business Problem
3. Challenges Faced and Approach
4. Assumptions
5. Solution Steps
6. System Performance and Improvements
7. Real-World Implementation
8. Scalability and Production Monitoring
9. Contact Information

## Project Overview

This project aims to solve a critical inventory management problem for a growing hotel chain operating multiple bars. The core issue is the frequent occurrence of stockouts for high-demand items and overstocking of slow-moving inventory. These problems lead to increased operational costs, lost sales opportunities, and decreased guest satisfaction.

This solution provides a data-driven forecasting and inventory recommendation system to help hotel managers make smarter decisions, ensuring optimal stock levels at each bar location.

## The Business Problem

The hotel chain's current inventory management practices are leading to two major issues:

**Increased Operational Costs:** Overstocking ties up capital in unsold inventory, incurs storage costs, and increases the risk of spoilage. Conversely, stockouts of popular items result in lost sales and revenue.

**Decreased Guest Satisfaction:** Guests unable to order their preferred beverages experience a negative impact on their overall perception of the hotel and their loyalty.

The goal is to develop a system that can:

- **Minimize Stockouts:** Ensure popular items are always available, maximizing revenue and guest satisfaction.
- **Reduce Overstocking:** Free up capital and reduce waste by holding less slow-moving inventory.
- **Automate and Standardize:** Provide a consistent and automated method for bar managers to make inventory decisions, reducing reliance on manual guesswork.

## Challenges Faced and Approach

### Challenges:

- **Irregular Data:** The historical consumption data might not be perfectly regular (e.g., not every item is consumed every day at every bar), requiring careful handling during aggregation.
- **Variability in Demand:** Demand for alcoholic beverages can be highly variable due to factors like day of the week, special events, and seasonality, which are not explicitly captured in the initial dataset.
- **Defining "Optimal" Inventory:** Determining the right balance between avoiding stockouts and minimizing overstocking requires a clear definition of service level and consideration of lead times.
- **Simulating Real-World Behavior:** Accurately simulating how the system would perform in a dynamic environment with daily consumption and replenishment cycles.

### Approach to Solve the Problem:

The approach involves a structured methodology, starting from data understanding and moving through forecasting, inventory logic, and practical simulation.

- **Data-Centric Understanding:** Begin by thoroughly exploring the provided historical inventory movement data to understand its structure, identify potential issues (like missing values or inconsistent formats), and gain insights into consumption patterns.
- **Robust Preprocessing:** Transform the raw data into a format suitable for time-series analysis, including handling dates, creating unique item identifiers, and aggregating consumption.
- **Strategic Forecasting Model Selection:** Start with a simple baseline (SMA) for comparison, then move to a more sophisticated yet interpretable model (Exponential Smoothing) that can capture trends and seasonality, even without explicit seasonal indicators in the initial data.
- **Standard Inventory Management Principles:** Apply established inventory management formulas (Safety Stock, Reorder Point, Par Level) to translate demand forecasts into actionable inventory recommendations.
- **Practical Simulation:** Build a simulation framework to test the system's effectiveness under realistic conditions, tracking key performance indicators like stockouts and service levels.

## Assumptions

To simplify the initial model and provide a workable solution, the following assumptions were made:

- **Lead Time:** A constant lead time of 3 days is assumed for all items. This is the time taken from placing an order to receiving the delivery. In a real-world scenario, this could be more dynamic and item-specific.
- **Service Level:** The system aims for a 95% service level, meaning the goal is to be in-stock 95% of the time. This corresponds to a Z-score of approximately 1.65 for normally distributed demand.
- **Review Period:** Inventory is assumed to be reviewed and orders are placed daily.

## Solution Steps

The solution is implemented in a Python notebook, following these structured steps:

### Step 1: Data Exploration and Preprocessing

The initial phase involves loading the dataset and preparing it for analysis.

- **Load Data:** The Consumption Dataset - Dataset.csv file is loaded into a pandas DataFrame.
- **Initial Inspection:** `df.head()`, `df.info()`, and `df.isnull().sum()` are used to understand the dataset's structure, data types, and identify any missing values.
- **Date Conversion:** The 'Date Time Served' column is converted to a datetime object and set as the DataFrame index.
- **Unique Item Identifier:** A new column 'Item' is created by concatenating 'Bar Name', 'Alcohol Type', and 'Brand Name' to uniquely identify each product at each location.
- **Daily Aggregation:** Consumption data ('Consumed (ml)') is aggregated by day for each unique 'Item' to create a time series suitable for forecasting. Missing dates within the time series for each item are filled with zeros to ensure continuity.

### Step 2: Demand Forecasting

This section focuses on predicting future demand for each item.

#### Baseline Model: Simple Moving Average (SMA)

A 7-day Simple Moving Average is used as a baseline to forecast the next day's demand. This provides a simple yet effective benchmark for more complex models.

The `sma_forecast` function calculates the SMA for a given time series and window.

#### Exponential Smoothing Model

- **Model Choice:** Exponential Smoothing (specifically, Holt-Winters) is chosen due to its robustness, ease of implementation, and ability to capture trends and seasonality in time series data. It's well-suited for demand forecasting where patterns might exist.
- **Implementation:** The `ExponentialSmoothing` function from `statsmodels.tsa.api` is used. The model is configured with `trend='add'` and `seasonal='add'` to account for additive trends and seasonality, and `seasonal_periods=7` to capture weekly patterns.
- **Forecasting:** The `fit().forecast(1)` method is used to predict the demand for the next period.

### Step 3: Inventory Recommendation System

Based on the demand forecasts, the system calculates optimal inventory levels.

#### Key Metrics Calculated:

- **Safety Stock:** Calculated as Z-score × Standard Deviation of Lead Time Demand. This buffer accounts for forecast inaccuracies and demand variability.
- **Reorder Point (ROP):** The inventory level at which a new order should be placed. Calculated as (Average Daily Usage × Lead Time) + Safety Stock.
- **Par Level (Order-up-to-Level):** The maximum desired inventory level. Calculated as Reorder Point + Average consumption during the review period.

`calculate_inventory_levels` Function: This function takes an item's consumption series, lead time, and service level Z-score to return these key inventory metrics.

### Step 4: Simulation

A simulation is run to evaluate the system's performance in a practical scenario.

- **Simulation Logic:** The `simulate_inventory` function tracks inventory levels over time for a given item's demand series.
- **Daily Cycle:** Each day, the system checks if the current inventory is less than the demand. If so, a stockout is recorded, and inventory is depleted. At the end of each day, inventory is replenished to the calculated Par Level.
- **Key Performance Indicators (KPIs):** The simulation tracks and reports:
  - Total Days in the simulation period.
  - Number of Stockout Days.
  - Overall Service Level (%).

## System Performance and Improvements

The simulation results indicate a high service level, demonstrating the system's effectiveness in meeting customer demand for the example item.

### Potential Improvements:

- **Dynamic Lead Times:** Integrate a mechanism to forecast or dynamically adjust lead times, as supplier lead times can vary.
- **Promotions and Events:** Enhance the forecasting model to incorporate external factors like promotions, holidays, and local events, which significantly impact demand.
- **More Sophisticated Models:** For items exhibiting highly complex or irregular demand patterns, explore advanced time series models (e.g., SARIMA, Prophet) or machine learning models (e.g., Gradient Boosting, LSTMs).
- **Cost Optimization:** Introduce cost parameters (holding costs, stockout costs, ordering costs) into the inventory optimization to find the most cost-effective par levels.

## Real-World Implementation

To deploy this solution in a live hotel environment, the following steps would be crucial:

- **Data Integration:** Establish an automated pipeline to collect and process real-time consumption data directly from the hotel's Point of Sale (POS) system.
- **Dashboarding:** Develop a user-friendly dashboard for bar managers. This dashboard should clearly display demand forecasts, recommended inventory levels, and proactive alerts for low-stock items.
- **Training:** Conduct comprehensive training sessions for bar managers on how to effectively use the system, interpret the recommendations, and integrate it into their daily operations.
- **Monitoring and Retraining:** Continuously monitor the model's accuracy and performance in a production environment. Periodically retrain the models with new data to ensure they remain accurate and relevant as demand patterns evolve.

## Scalability and Production Monitoring (Optional)

### Scalability

- **Robust Data Infrastructure:** To handle data from hundreds of bars and thousands of items, a scalable data infrastructure is essential. This would likely involve cloud-based services for data storage (e.g., a data warehouse like BigQuery or Snowflake) and distributed computing frameworks (e.g., Apache Spark on Dataproc or EMR) for processing large volumes of data.
- **Microservices Architecture:** Breaking down the system into smaller, independent services (e.g., a forecasting service, an inventory recommendation service) would improve scalability, resilience, and maintainability.

### Production Monitoring

In a production environment, continuous monitoring is vital to ensure the system's effectiveness and identify issues early. Key metrics to track include:

#### Model Accuracy:

- **Mean Absolute Error (MAE):** Measures the average magnitude of the errors in a set of forecasts, without considering their direction.
- **Root Mean Squared Error (RMSE):** A quadratic scoring rule that measures the average magnitude of the error. It's more sensitive to large errors.

#### Business KPIs:

- **Stockout Rates:** Percentage of times an item was out of stock when demanded.
- **Inventory Turnover:** How many times inventory is sold and replaced over a period, indicating efficiency.
- **Guest Satisfaction Scores:** Surveys or feedback related to beverage availability.
- **Operational Costs:** Track changes in storage costs, spoilage, and expedited shipping fees.

## Contact Information

For any questions or further discussions, please reach out:

- **GitHub:** https://github.com/prtk2403
- **X (formerly Twitter):** https://x.com/xyzprtk
