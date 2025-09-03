# Walmart Sales: SQL + Python Project
![Walmart Logo](https://github.com/i-am-rahularora5504/walmart_pandas_sql_project/blob/main/Walmart_logo.jpg)

## Overview
This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions.

---

## Project Steps
![Project Layout](https://github.com/i-am-rahularora5504/Walmart_Sales_SQL_Python_Project/blob/main/walmart_project_layout.png)

### 1. Set Up the Environment (my_env1)
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (MySQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Set Up Kaggle API
   - **API Setup**: Obtain your Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to your profile settings and downloading the JSON file.
   - **Configure Kaggle**:
      - Create `.kaggle` folder in home directory.
      - Place the downloaded `kaggle.json` file in your local `.kaggle` folder.
      - Use the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into your project.
      - 
### 3. Download Walmart Sales Data
   - **Data Source**: Use the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://github.com/i-am-rahularora5504/Walmart_Sales_SQL_Python_Project/blob/main/Walmart.csv)

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas
     pip install pymysql
     pip install sqlalchemy
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

```python
#importing Dependencies
import pandas as pd #Process and clean data
import pymysql #act as an adapter
from sqlalchemy import create_engine #export dataframe from pandas to mysql
#create_engine feature is imported to make connection with mysql.
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
df['total_price']=df['unit_price'] * df['quantity']
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
#mysql+pymysql://user:password@localhost:3306/db_name
engine_mysql = create_engine("mysql+pymysql://root:rjovia!5@localhost:3306/walmart_db")

try:

    engine_mysql
    print("connection succeeded")
except:
    print("unable to connect")

df.to_sql(name='walmart', con=engine_mysql, if_exists='append', index =False)
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
use walmart_db;
select * from walmart;

select count(*) from walmart;

#Find the number of sales from different payment methods.
select 
     count(*),
     payment_method
     from walmart
	group by payment_method;
    
#Find the number of distinct Branchat which order is made.
select
     count(distinct Branch)
from walmart;

#Find the the minimum and maximum number of quantities ordered.
select min(quantity),max(quantity) from walmart;
```
### Business Problems

**1. Find the different payment method and their number of transaction, number of quantity sold.**
```sql
select 
payment_method ,
count(invoice_id) as Transactions,
Sum(quantity) as Quantity_Sold
from walmart
group by payment_method
```

**2. Find the highest rated category in each branch displaying branch, category and avg rating.**
```sql
select branch,
     category,
     Rating_Average
from
(select branch,
     category,
     avg(rating) as Rating_Average,
     rank() over (partition by branch order by avg(rating) desc) as Ranking
from walmart
group by branch, category) as t1 where Ranking = 1
```

**3. Find the busiest day for each branch based on number of transactions. 
    Also find at which day of the week , most transactions are completed .**
```sql
select branch ,
     Day,
     No_of_Transaction
from
(select branch ,
     dayname(date) as Day,
     count(invoice_id) as No_of_Transaction,
     rank() over (partition by branch order by count(invoice_id) desc) as ranking
from walmart
group by branch, Day) as t1
where ranking = 1;
```
```sql
select Day , 
     sum(No_of_Transaction)
     from(
select branch ,
     Day,
     No_of_Transaction
from
(select branch ,
     dayname(date) as Day,
     count(invoice_id) as No_of_Transaction,
     rank() over (partition by branch order by count(invoice_id) desc) as ranking
from walmart
group by branch, Day) as t1
where ranking = 1)
as t2 group by 1
order by 2  desc;
```

**4. Calculate the total quantity of items sold per payment method.**
```sql
select payment_method,
     sum(quantity) as Total_quantity
from walmart
group by payment_method;
```

**5. Determine the average, minimum and maximum rating of category for each city.
List the city, category, average_rating,minimum_rating and maximum_rating.**
```sql
select city,
     category,
     avg(rating) as average_rating,
     min(rating) as minimum_rating,
     max(rating) as maximum_rating
from walmart 
group by city, category;
```

**6. Calculate the total profit for each category. 
List total_profit and category ordered from hghest to lowest.**
```sql
select category , 
      sum(total_price * profit_margin) as total_profit
from walmart group by category
order by 2 desc;
```

**7. Display the most common payment method for each branch. 
list the branch and preffered_payment_method.**
```sql
select branch ,
payment_method as preffered_payment_method
from
(select branch ,
payment_method,
count(*) as no_of_tranaction,
rank() over(partition by branch order by count(*) desc) as Ranking
from walmart 
group by 1,2) as t1
where Ranking =1;
```
```sql
With cte 
as
(select branch ,
payment_method,
count(*) as no_of_tranaction,
rank() over(partition by branch order by count(*) desc) as Ranking
from walmart 
group by 1,2)

select * from cte where 
Ranking =1;
```

**8. Categorize sales into three groups MORING, AFTERNOON, EVENING
Find out each of the shift for different branch and their no. of invoices.**
```sql
select branch ,
CASE 
           WHEN (HOUR) < 12 Then 'Morning'
           WHEN (HOUR) Between 12 and 17 Then 'Afternoon'
           Else 'Evening'
	 End as shift_of_the_day,
     count(*)
     from
     (select branch,
     time, 
     hour(time) as HOUR from walmart)
     as t1

group by 1,2
order by 1,3 desc;
```

**9. Which one of th following is the highest performing branch.
Also how much above the avg. revenue produced by a branch.**

```sql
select branch ,
     sum(total_price) as revenue 
from walmart 
group by branch 
order by revenue desc;

select avg(revenue)
from(select branch ,
     sum(total_price) as revenue 
from walmart 
group by branch 
order by revenue desc)
as t1;
```
### 10. Project Publishing and Documentation
   - **Documentation**: Maintain well-structured documentation of the entire process in VScode(Jupyter Notebook-kernel) and mysql Workbench.
   - **Project Publishing**: Publish the completed project on GitHub or any other version control platform, including:
     - The `README.md` file (this document).
     - Jupyter Notebooks (if applicable).
     - SQL query scripts.
     - Data files (if possible) or steps to access them.

---
    
## Results and Insights

This section will include your analysis findings:
- **Sales Insights**: Key categories, branches with highest sales, and preferred payment methods.
- **Profitability**: Insights into the most profitable product categories and locations.
