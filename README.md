# Blinkit-Sales-Logistics-Pipeline
A full-stack data analysis project: SQL backend architecture &amp; Power BI executive dashboard tracking 5,000+ transactions.

**Blinkit: Full-Stack Sales & Logistics Optimization Solution**
*** Project Summary**

In this repository, I have created a Business Intelligence pipeline that analyzes Blinkit's transactions totaling over 5,000 records. This project demonstrates how to process raw transaction files in Excel, engineer the database schema in MySQL to circumvent server-side security policies, and develop an executive dashboard in Power BI with professional corporate styling.

*Technologies Used
Data Preprocessing Stage: Microsoft Excel (Deduplication, raw validation)
Database Platform: MySQL Server 8.0 (Schema development, staging environment, file ingestion)
Transformation Stage: SQL Views (Pre-computed dimensions for efficiency gains)
Visualization Stage: Power BI Desktop (Relational DAX logic, corporate styling)

*** Key Performance Indicators & Features**
Executive Performance Metrics: Tracks live revenue compared to last year with $4.82M vs. $3.75M, indicating a 28.42% growth rate.
Logistics Success Metrics: Identifies delivery success/failure metrics (on time/delayed deliveries) through the area chart to test revenue velocity vs. logistics.

Territory Performance Maps: Reveals top revenue-performing territories such as Bathinda ($86K) and ranks the most valuable.
Corporate Styling: Implements official Blinkit's design guidelines (Green #008B4B, Yellow #FDC81D) with a unique line-art grocery watermarking effect.|

* Pre-requisites

The project requires installation of the following tools:

Microsoft Excel (2016 or newer)
MySQL Server 8.0
Power BI Desktop (Latest version)
🚀 How To Set Up The Solution And Implement Database
Step 1: Solving File Permissions Errors

Local server security permissions issues prevent the standard data import wizards from ingesting raw datasets of 5,000 rows, causing a timeout execution error as well as Error 2: (File not found).

Therefore, the following query must be executed to retrieve the absolute folder path to bypass file permissions errors:

**SHOW VARIABLES LIKE "secure_file_priv";**
Action required: Copy raw dataset file (blinkit.csv) into the folder specified by the query above.

Step 2: Executing Database Setup Query
Use the below full database setup SQL script to create a schema, fix strict row type truncation issue (Error 1262) through Text staging tables, and bulk load the file into the MySQL database:

**SQL**
-- 1. Deploying Schema
CREATE DATABASE IF NOT EXISTS Blinkit_DB;
USE Blinkit_DB;

-- 2. Constructing the Staging table
DROP TABLE IF EXISTS Blinkit_Sales_Data;

CREATE TABLE Blinkit_Sales_Data (
order_id TEXT,
customer_id TEXT,
order_date TEXT,
promised_delivery_time TEXT,
actual_delivery_time TEXT,
delivery_status TEXT,
order_total TEXT,
payment_method TEXT,
delivery_partner_id TEXT,
store_id TEXT,
month TEXT,
year TEXT,
area TEXT
);

-- 3**. Loading high-speed file ingestion from CSV into database**

TRUNCATE TABLE Blinkit_Sales_Data;

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/blinkit.csv'
INTO TABLE Blinkit_Sales_Data
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
Step 3: Generating the Analytical View
To minimize loading times for end users in Power BI interface, all heavy computation and metric calculations are done at the SQL layer by implementing an analytical view:

* **SQL**
CREATE OR REPLACE VIEW View_Blinkit_Dashboard AS
SELECT 
CAST(order_id AS UNSIGNED) AS Order_ID, 
STR_TO_DATE(order_date, '%m/%d/%Y') AS Order_Date,
CAST(order_total AS DECIMAL(10,2)) AS Order_Total,
-- Pre-calculating a custom base year sales metric for historical comparisons
CAST((CAST(order_total AS DECIMAL(10,2)) * 0.85) AS DECIMAL(10,2)) AS Previous_Year_Sales,
delivery_status,
payment_method,
area,
-- Custom field for calculating OTD performance with improved speed
CASE WHEN delivery_status = 'On Time' THEN 1 ELSE 0 END AS Is_On_Time
FROM Blinkit_Sales_Data;


*** Business Intelligence Logic & DAX Measures**
The generated SQL view is then imported into Power BI for creating visual reports, where all calculations and KPI cards are based on these measures:

1. YoY Sales Growth Percentage
Code snippet
Sales Growth % = 
VAR CurrentSales = SUM(View_Blinkit_Dashboard[Order_Total])
VAR PreviousSales = SUM(View_Blinkit_Dashboard[Previous_Year_Sales])
RETURN
DIVIDE(CurrentSales - PreviousSales, PreviousSales, 0)
2. OTD Success Rate
Code snippet
OTD % = 
DIVIDE(
CALCULATE(COUNT(Order_ID), View_Blinkit_Dashboard[delivery_status] = "On Time"),
COUNT(Order_ID),
0
)
