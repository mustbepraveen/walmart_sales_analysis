# Walmart Sales: SQL + Python Project
![Walmart Logo](https://github.com/i-am-rahularora5504/walmart_pandas_sql_project/blob/main/Walmart_logo.jpg)

## Overview
This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions.

---

## Project Steps
![Project Layout](https://github.com/i-am-rahularora5504/Walmart_Sales_SQL_Python_Project/blob/main/walmart_project_layout.png)

### 1. Set Up the Environment (my_env1)
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (PostgreSQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Set Up Kaggle API
   - **API Setup**: Obtain your Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to your profile settings and downloading the JSON file.
   - **Configure Kaggle**:
      - Create `.kaggle` folder in home directory.
      - Place the downloaded `kaggle.json` file in your local `.kaggle` folder.
      - Use the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into your project.
        
### 3. Download Walmart Sales Data
   - **Data Source**: Use the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://github.com/i-am-rahularora5504/Walmart_Sales_SQL_Python_Project/blob/main/Walmart.csv)

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas
     pip install psycopg2-binary
     pip install sqlalchemy
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

```python

import pandas as pd
from sqlalchemy import create_engine #export dataframe from pandas to mysql
#create_engine feature is imported to make connection with mysql.
import psychopg2
```

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Use functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

```python
df = pd.read_csv('Walmart.csv', encoding_errors='ignore')
df.shape
df.describe()
df.dtypes
df.info()
```

### 6. Data Cleaning
   - **Remove Duplicates**: Identify and remove duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Drop rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Use `.replace()` to handle and format currency values for analysis.
   - **Validation**: Check for any remaining inconsistencies and verify the cleaned data.

```python
#all duplicates
df.duplicated().sum()
df.drop_duplicates(inplace=True)
df.duplicated().sum()
df.shape
df.isnull().sum()
#droppping all rows with missing records
df.dropna(inplace=True)
df.dtypes
#Assigning back to unit_price column in walmart sales dataset.
df['unit_price']=df['unit_price'].str.replace('$','').astype(float)
df.head()
df.info()
```

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

```python
df.columns
df['total_amount']=df['unit_price'] * df['quantity']
df.head()
df.columns
df.to_csv('walmart_clean_data.csv', index=False)
```

### 8. Load Data into MySQL and PostgreSQL
   - **Set Up Connections**: Connect to MySQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in MySQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Run initial SQL queries to confirm that the data has been loaded accurately.

```python
#mysql connection
#engine_psql = create_engine("postgresql+psycopg2://postgres:root@localhost:5432/walmart_db")

try:
    engine_psql
    print("connection successful")
except:
    print("connection failed")

df.to_sql(name="walmart" , con=engine_psql , if_exists = 'append', index='False')
```

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as:
     - Revenue trends across branches and categories.
     - Identifying best-selling product categories.
     - Sales performance by time, city, and payment method.
     - Analyzing peak sales periods and customer buying patterns.
     - Profit margin analysis by branch and category.

```sql
-- Basic Exploration of data in mysql
SELECT * FROM walmart;

SELECT DISTINCT(payment_method),COUNT(*)
FROM walmart
GROUP BY payment_method;

SELECT DIsTINCT(branch),COUNT(*)
FROM walmart
GROUP BY branch;

SELECT MIN(quantity)
FROM walmart;
```
### Business Problems

**Find different payment methods,number of transactions and total quantity sold.**
```sql
SELECT DISTINCT(payment_method), COUNT(*) AS Total_Transactions, SUM(quantity) AS Total_Quantity
FROM walmart
GROUP BY payment_method;
```

**Find the highest-rated category in each branch,displaying the branch, category and average rating**
```sql
WITH CTE AS (
SELECT branch,
       category,
	   ROUND(CAST(AVG(rating) AS INTEGER),2) AS average_rating,
	   DENSE_RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) ) AS rank
FROM walmart
GROUP BY branch,category)

SELECT branch,category,average_rating
FROM CTE
WHERE rank = 1;
```

**Identify the busiest day for each branch based on number of transactions**
```sql
SELECT branch,day,total_transactions
FROM(
       SELECT 	branch,TO_CHAR(TO_DATE(date , 'DD/MM/YY') , 'day') AS day,COUNT(*) AS total_transactions,
                DENSE_RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) ) AS rank
                FROM walmart
                GROUP BY branch,day
)
WHERE rank = 1;
```
**Identify different payment method and quantity sold by each payment method**
```sql
SELECT DISTINCT(payment_method),SUM(quantity) AS total_quantity_sold
FROM walmart
GROUP BY payment_method;
```

**Determine the average,minimum and maximum rating for each city.
List the city,average rating,minimum rating and maximum rating.**
```sql
SELECT city,category,ROUND(CAST(AVG(rating) AS INTEGER),2),MIN(rating),MAX(rating)
FROM walmart
GROUP BY city,category;
```

**Calculate the total profit for each category by considering total_profit as (unit price * quantity * profit_margin).Order the result by highest to lowest profit.**
```sql
SELECT category, ROUND(CAST(SUM(unit_price*quantity*profit_margin) AS INTEGER),2) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit;
```

**Find the most common payment method for each branch and display the branch and preferred_payment_method.**
```sql
SELECT branch,payment_method AS preferred_payment_method,total_payments FROM(
SELECT branch,payment_method,COUNT(payment_method) AS total_payments,DENSE_RANK() OVER(PARTITION BY branch ORDER BY COUNT(payment_method)) AS rank
FROM walmart
GROUP BY branch,payment_method)
WHERE rank = 1;
```

**Categorize sales into 3 groups Morning,Afternoon and Evening and also calculate the number of invoices for each category**
```sql
SELECT branch,
       CASE WHEN EXTRACT(HOUR FROM time::time) < 12 THEN 'Morning'
	        WHEN EXTRACT(HOUR FROM time::time) BETWEEN 13 AND 17 THEN 'Afternoon'
			ELSE 'Evening'
	   END AS timeotheday,COUNT(*) AS total_transactions
FROM walmart
GROUP BY 1,2;
```

**Identify 5 branches with highest decrease ratio in revenue as compared to last year (this year 2023 and last year 2022)**

```sql
WITH revenue_2023 AS(
SELECT branch,
       SUM(total_amount) AS revenue
FROM walmart
WHERE EXTRACT(YEAR FROM TO_Date(Date , 'DD/MM/YY')) = 2023
GROUP BY branch
)
,revenue_2022 AS(
SELECT branch,
       SUM(total_amount) AS revenue
FROM walmart
WHERE EXTRACT(YEAR FROM TO_Date(Date , 'DD/MM/YY')) = 2022
GROUP BY branch
)

SELECT ly.branch,
       ly.revenue AS last_year_revenue,
	   cy.revenue AS current_year_revenue,
	   ROUND((ly.revenue - cy.revenue)::numeric/ly.revenue::numeric*100,2) AS revenue_decrease_ratio
FROM revenue_2022 AS ly
INNER JOIN revenue_2023 AS cy
ON ly.branch = cy.branch
WHERE ly.revenue > cy.revenue
ORDER BY 4 DESC
LIMIT 5;
```
### 10. Project Publishing and Documentation
   - **Documentation**: Maintain well-structured documentation of the entire process in VScode(Jupyter Notebook-kernel) and PGadmin
   - **Project Publishing**: Published the completed project on GitHub or any other version control platform, including:
     - The `README.md` file (this document).
     - Jupyter Notebook
     - Data file.

---
    
## Results and Insights

This section will include your analysis findings:
- **Sales Insights**: Key categories, branches with highest sales, and preferred payment methods.
- **Profitability**: Insights into the most profitable product categories and locations.
