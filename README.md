# Fast Food Operations Analysis - Data Driven Decision Making Project

**Tools Used:** MySQL, Power BI, Excel

## Table of Contents
* [Project Background](#project-background)
* [Data Structure](#data-structure)
* [Executive Summary](#executive-summary)
* [Business Questions](#business-questions)
* [Key Insights](#key-insights)
* [Recommendations](#recommendations)
* [Technical Methodology](#technical-methodology)
* [SQL Logic](#sql-logic)

## Project Background
This project focuses on building a sales performance dashboard for Balaji Fast Food. The business required a way to analyze their rich dataset to gain insights.

The main goal is to optimize operations by analyzing transaction data. We examine the time of sale, item popularity, payment types, and staff performance.

**Team 3:**
* We used a diverse skill set including coding, design, and business thinking.
* We followed a fixed weekly meeting schedule to ensure progress.

## Data Structure
The dataset consists of **500 rows and 8 columns**. It contains transactional data used to understand customer behavior and delivery efficiency.

**Source File:** `fast_food_ordering_dataset.csv`

**Data Dictionary:**
* **order_id:** Unique identifier for each order.
* **order_time:** Timestamp of the order.
* **city:** Customer location.
* **cuisine_type:** Type of cuisine (Categorical).
* **order_value:** Order value (Numerical).
* **delivery_time_minutes:** Delivery duration in minutes.
* **payment_method:** Method of payment (Example: Cash, UPI).
* **items_count:** Quantity of items ordered.

**Data Quality Check:**
* **Missing Values:** 0 (The dataset is complete).
* **Duplicates:** 0 (There are no duplicate orders).
* **Consistency:** All metrics are reasonable and there are no negative order values.

## Executive Summary
We built a dashboard to track Revenue, Orders, and Average Order Value in real time. The analysis highlights peak operational hours and identifies the most profitable cuisine categories.

## Business Questions
We used data to answer three specific questions for the business:

1.  **Product Strategy:** Which cuisine type has the highest Average Order Value (AOV)?
    * **Purpose:** To focus on promoting high profit groups like North Indian and Mexican cuisines.
2.  **Operations:** Which city has the longest average delivery time?
    * **Purpose:** To optimize the delivery fleet or reroute drivers in that area.
3.  **Growth:** Which city contributes the most in total revenue?
    * **Purpose:** To identify key markets for further marketing investment.

## Key Insights
Our analysis of the temporal trends and product dynamics revealed the following:

* **Peak Operations:** There is a significant surge in order volume between **12:00 PM and 1:00 PM** which is the Lunch Rush.
* **Downtime:** There is minimal activity observed after **10:00 PM**.
* **Top Performers:** South Indian and Italian cuisines generate the highest revenue share.

## Recommendations
Based on the insights we propose the following strategic actions:

1.  **Resource Allocation:**
    * Increase kitchen and delivery staffing levels during the **11:00 AM to 2:00 PM** window. This will help mitigate delays during the lunch rush.
2.  **Inventory Optimization:**
    * Prioritize stock for **South Indian** ingredients. This will prevent stockouts during high demand periods.
3.  **Marketing Strategy:**
    * Implement "Happy Hour" promotions during off peak hours from **3:00 PM to 5:00 PM**. This will flatten the demand curve.

## Technical Methodology
We followed a 5 week work breakdown structure.

* **Week 1 and 2:** We defined the business questions and explored the dataset. We cleaned the data and standardized formats for dates and currency.
* **Week 3:** We focused on SQL modeling. We removed null values and created clean master tables. We established a Star Schema to optimize query performance.
* **Week 4:** We built the dashboard with an Executive Scorecard and Time Series Analysis.

### Data Model
We used a Star Schema structure.
* **Fact Table:** `Fact_Orders` contains metrics like Revenue and Quantity.
* **Dimensions:** `Dim_City` and `Dim_Cuisine` contain attributes for filtering.

## SQL Logic
We used the following SQL scripts to clean, normalize, and analyze the data in MySQL.

```sql
-- =============================================
-- 1. DATABASE SETUP & RAW DATA IMPORT
-- =============================================

-- Create the database
CREATE DATABASE IF NOT EXISTS FastFoodDB;
USE FastFoodDB;

-- Create the Raw Data table (Staging Layer)
-- Note: We use VARCHAR for time and values to prevent import errors.
CREATE TABLE Raw_Data (
    order_id VARCHAR(50),
    order_time VARCHAR(50),
    city VARCHAR(100),
    cuisine_type VARCHAR(100),
    order_value VARCHAR(50),       -- Text type to handle potential special characters
    delivery_time_minutes INT,
    payment_method VARCHAR(50),
    items_count INT
);

-- =============================================
-- 2. DIMENSION TABLES CREATION (STAR SCHEMA)
-- =============================================

-- Dimension Table: City
CREATE TABLE Dim_City (
    city_id INT AUTO_INCREMENT PRIMARY KEY,
    city_name VARCHAR(100)
);

-- Dimension Table: Cuisine
CREATE TABLE Dim_Cuisine (
    cuisine_id INT AUTO_INCREMENT PRIMARY KEY,
    cuisine_type VARCHAR(100)
);

-- =============================================
-- 3. FACT TABLE CREATION
-- =============================================

-- Fact Table: Orders
CREATE TABLE Fact_Orders (
    fact_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id VARCHAR(50),
    order_date DATE,            -- Separated date for daily analysis
    order_time TIME,            -- Separated time for "Peak Hour" analysis
    city_id INT,                -- Foreign Key to Dim_City
    cuisine_id INT,             -- Foreign Key to Dim_Cuisine
    order_value DECIMAL(10, 2), -- Standard numeric currency format
    delivery_time_minutes INT,
    payment_method VARCHAR(50),
    items_count INT,
    FOREIGN KEY (city_id) REFERENCES Dim_City(city_id),
    FOREIGN KEY (cuisine_id) REFERENCES Dim_Cuisine(cuisine_id)
);

-- =============================================
-- 4. DATA TRANSFORMATION & LOADING (ETL)
-- =============================================

-- Populate City Dimension (Unique Cities)
INSERT INTO Dim_City (city_name)
SELECT DISTINCT city 
FROM Raw_Data 
WHERE city IS NOT NULL AND city <> '';

-- Populate Cuisine Dimension (Unique Cuisines)
INSERT INTO Dim_Cuisine (cuisine_type)
SELECT DISTINCT cuisine_type 
FROM Raw_Data 
WHERE cuisine_type IS NOT NULL AND cuisine_type <> '';

-- Transform and Load Data into Fact Table
INSERT INTO Fact_Orders (
    order_id, 
    order_date, 
    order_time, 
    city_id, 
    cuisine_id, 
    order_value, 
    delivery_time_minutes, 
    payment_method, 
    items_count
)
SELECT 
    r.order_id,
    
    -- Handle Date: Format in file is 'YYYY-MM-DD HH:MM:SS'
    DATE(STR_TO_DATE(r.order_time, '%Y-%m-%d %H:%i:%s')), 
    
    -- Handle Time
    TIME(STR_TO_DATE(r.order_time, '%Y-%m-%d %H:%i:%s')),
    
    -- Get City ID (Lookup)
    dc.city_id,
    
    -- Get Cuisine ID (Lookup)
    dcu.cuisine_id,
    
    -- Handle Currency: Convert text to decimal
    CAST(r.order_value AS DECIMAL(10, 2)),
    
    r.delivery_time_minutes,
    r.payment_method,
    r.items_count

FROM Raw_Data r
-- Join to retrieve IDs instead of text names
LEFT JOIN Dim_City dc ON r.city = dc.city_name
LEFT JOIN Dim_Cuisine dcu ON r.cuisine_type = dcu.cuisine_type;

-- =============================================
-- 5. VERIFICATION & ANALYSIS
-- =============================================

-- View the first 10 rows of the clean table
SELECT * FROM Fact_Orders LIMIT 10;

-- Test Query: Average Revenue and Delivery Time by City
SELECT 
    c.city_name,
    COUNT(f.fact_id) AS Total_Orders,
    ROUND(AVG(f.order_value), 2) AS Avg_Order_Value,
    ROUND(AVG(f.delivery_time_minutes), 0) AS Avg_Delivery_Time
FROM Fact_Orders f
JOIN Dim_City c ON f.city_id = c.city_id
GROUP BY c.city_name
ORDER BY Avg_Order_Value DESC;
