# Fast-Food-Ordering-Final-DDDM

# Optimizing Balaji Fast Food Operations through Sales Data Analysis

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
The dataset consists of **500 rows and 8 columns**. [cite_start]It contains transactional data used to understand customer behavior and delivery efficiency[cite: 84, 91].

[cite_start]**Source File:** `fast_food_ordering_dataset.csv` [cite: 82]

**Data Dictionary:**
* **order_id:** Unique identifier for each order.
* **order_time:** Timestamp of the order.
* **city:** Customer location.
* **cuisine_type:** Type of cuisine (Categorical).
* **order_value:** Order value (Numerical).
* **delivery_time_minutes:** Delivery duration in minutes.
* **payment_method:** Method of payment (Example: Cash, UPI).
* [cite_start]**items_count:** Quantity of items ordered[cite: 85, 86, 87, 88, 89].

**Data Quality Check:**
* **Missing Values:** 0 (The dataset is complete).
* **Duplicates:** 0 (There are no duplicate orders).
* [cite_start]**Consistency:** All metrics are reasonable and there are no negative order values[cite: 97, 98, 99, 101].

## Executive Summary
We built a dashboard to track Revenue, Orders, and Average Order Value in real time. [cite_start]The analysis highlights peak operational hours and identifies the most profitable cuisine categories[cite: 149].

## Business Questions
We used data to answer three specific questions for the business:

1.  **Product Strategy:** Which cuisine type has the highest Average Order Value (AOV)?
    * [cite_start]**Purpose:** To focus on promoting high profit groups like North Indian and Mexican cuisines[cite: 107, 108].
2.  **Operations:** Which city has the longest average delivery time?
    * [cite_start]**Purpose:** To optimize the delivery fleet or reroute drivers in that area[cite: 110, 111].
3.  **Growth:** Which city contributes the most in total revenue?
    * [cite_start]**Purpose:** To identify key markets for further marketing investment[cite: 113, 114].

## Key Insights
Our analysis of the temporal trends and product dynamics revealed the following:

* [cite_start]**Peak Operations:** There is a significant surge in order volume between **12:00 PM and 1:00 PM** which is the Lunch Rush[cite: 154].
* [cite_start]**Downtime:** There is minimal activity observed after **10:00 PM**[cite: 155].
* [cite_start]**Top Performers:** South Indian and Italian cuisines generate the highest revenue share[cite: 158].

## Recommendations
Based on the insights we propose the following strategic actions:

1.  **Resource Allocation:**
    * Increase kitchen and delivery staffing levels during the **11:00 AM to 2:00 PM** window. [cite_start]This will help mitigate delays during the lunch rush[cite: 161].
2.  **Inventory Optimization:**
    * Prioritize stock for **South Indian** ingredients. [cite_start]This will prevent stockouts during high demand periods[cite: 164].
3.  **Marketing Strategy:**
    * Implement "Happy Hour" promotions during off peak hours from **3:00 PM to 5:00 PM**. [cite_start]This will flatten the demand curve[cite: 167].

## Technical Methodology
[cite_start]We followed a 5 week work breakdown structure[cite: 41].

* **Week 1 and 2:** We defined the business questions and explored the dataset. [cite_start]We cleaned the data and standardized formats for dates and currency[cite: 23, 34].
* **Week 3:** We focused on SQL modeling. We removed null values and created clean master tables. [cite_start]We established a Star Schema to optimize query performance[cite: 169, 193].
* [cite_start]**Week 4:** We built the dashboard with an Executive Scorecard and Time Series Analysis[cite: 140, 149].

### Data Model
[cite_start]We used a Star Schema structure[cite: 143].
* [cite_start]**Fact Table:** `Fact_Orders` contains metrics like Revenue and Quantity[cite: 146].
* [cite_start]**Dimensions:** `Dim_City` and `Dim_Cuisine` contain attributes for filtering[cite: 147].

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
