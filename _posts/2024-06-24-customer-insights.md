---
layout: post
title: Customer Insights and Sales
image: "/posts/grocery.jpg"
tags: [SQL, Python, Tableau]
---

# Business Case

The owners of a small grocery chain have asked for an analysis of their Sales, Customers, and a recent marketing campaign based on data stored in a database. They would like a basic statistical analysis on this data and a dashboard summarizing the key insights and trends in these 3 domains.

# Solution

In this solution I will show how I connected directly to the companies postgreSQL database using Python and performed exploratory data analysis, handled outliers, generated precurosor visualisations, and the final dashboard in Tableau presenting key statistics and trends back to the business.

---

## Running Our SQL Statement in Python

In order to begin this analysis, we first load our data from the PostgreSQL database into Pandas DataFrames using Python. This sets us up to be able to quickly perform EDA and run our statisical analysis on the tables provided to us by the business.

### Step 1 - Create the Access Parameters

### Step 2 - Establish Connection & Pass Acces Parameters

### Step 3 - Pass our Queries to Create our DataFrames 

```Python
from sqlalchemy import create_engine
import pandas as pd

db_params = {
    'dbname': 'data_science_infinity',
    'user': 'student',
    'password': '*******',
    'host': 'data-science-infinity.cpwa6tfdvnx2.eu-west-2.rds.amazonaws.com',
    'port': '5432'
}

connection_string = f"postgresql://{db_params['user']}:{db_params['password']}@{db_params['host']}:{db_params['port']}/{db_params['dbname']}"

# Create the SQLAlchemy engine
engine = create_engine(connection_string)

# Example query
try:
    
    customer_details = pd.read_sql('SELECT * FROM customer_details', engine)
    transactions = pd.read_sql('SELECT * FROM grocery_db.transactions', engine)
    campaign_data = pd.read_sql('SELECT * FROM grocery_db.campaign_data', engine)
    product_areas = pd.read_sql('SELECT * FROM grocery_db.product_areas', engine)
    loyalty_scores = pd.read_sql('SELECT * FROM grocery_db.loyalty_scores', engine)
    
except Exception as error:
    print(f"Error executing the query: {error}")
```
---
## EDA and Basic Statistics

Our next step...

### Step 1 - Create a Dictionary of our DataFrames

### Step 2 - Pass our DataFrame Names into an EDA Loop

```Python
# List of DataFrame names
dataframes = {
    'transactions': transactions,
    'customer_details': customer_details,
    'campaign_data': campaign_data,
    'product_areas': product_areas,
    'loyalty_scores': loyalty_scores
}

# Perform analysis for each DataFrame
for name, df in dataframes.items():
    print(f"\nAnalysis for {name} DataFrame:\n")
    
    # Check if DataFrame is not empty
    if df.empty:
        print(f"{name} DataFrame is empty. Skipping analysis.\n")
        continue

    # Shape of DataFrame
    print(f"{name}.shape:\n{df.shape}\n")
    
    # First 20 rows
    print(f"{name}.head(20):\n{df.head(20)}\n")
    
    # Last 10 rows
    print(f"{name}.tail(10):\n{df.tail(10)}\n")
    
    # Random 10 rows
    print(f"{name}.sample(10):\n{df.sample(10)}\n")
    
    # Grab 10% of data
    sample = df.sample(frac=0.1)
    print(f"10% sample of {name}:\n{sample}\n")
    
    # Describe DataFrame
    print(f"{name}.describe():\n{df.describe()}\n")
    
    # Outliers in sales_cost (if the column exists)
    if 'sales_cost' in df.columns:
        print(f"Top 25 largest values in sales_cost for {name}:\n{df.nlargest(25, 'sales_cost')}\n")
        print(f"Top 25 smallest values in sales_cost for {name}:\n{df.nsmallest(25, 'sales_cost')}\n")
    else:
        print(f"Column 'sales_cost' does not exist in {name} DataFrame.\n")
    
    # Number of unique values
    print(f"{name}.nunique():\n{df.nunique()}\n")
```
---
## Perform Basic Plotting and Handle Outliers

```Python
# Function to remove outliers using the IQR method
def remove_outliers(df, column):
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    return df[(df[column] >= lower_bound) & (df[column] <= upper_bound)]

# Remove outliers from the distance_from_store column
filtered_df = remove_outliers(customer_details, 'distance_from_store')

# Plot the distribution of the filtered data
plt.figure(figsize=(10, 6))
plt.hist(filtered_df['distance_from_store'], bins=10, edgecolor='k')
plt.title('Distribution of Customers by Distance from Store (Outliers Removed)')
plt.xlabel('Distance from Store')
plt.ylabel('Number of Customers')
plt.grid(True)
plt.show()
```
![alt text](/img/posts/ci_cust_dist.png "Coffee & Python - I love them!")

![alt text](/img/posts/coffee_python.jpg "Coffee & Python - I love them!")

```Python
sns.countplot(x='gender', data=customer_details)
plt.title('Distribution of Customers by Gender')
plt.show()

#analyze sales performance
sales_by_date = transactions.groupby('transaction_date')['sales_cost'].sum().reset_index()
plt.plot(sales_by_date['transaction_date'], sales_by_date['sales_cost'])
plt.title('Sales Trends Over Time')
plt.show()

sales_by_product_area = transactions.merge(product_areas, on='product_area_id') \
                                    .groupby('product_area_name')['sales_cost'].sum().reset_index()
sns.barplot(x='sales_cost', y='product_area_name', data=sales_by_product_area)
plt.title('Sales by Product Area')
plt.show()

# evaluate campaign effectiveness
campaign_performance = campaign_data.groupby(['campaign_name', 'mailer_type'])['signup_flag'].mean().reset_index()
sns.barplot(x='signup_flag', y='campaign_name', hue='mailer_type', data=campaign_performance)
plt.title('Campaign Signup Rates by Mailer Type')
plt.show()
```
---
## Send the Data to CSV for Tableau

```Python
#send data tables to csv for dashboard in Tableau
transactions.to_csv('transactions.csv', index=False)
customer_details.to_csv('customer_details.csv', index=False)
campaign_data.to_csv('campaign_data.csv', index=False)
product_areas.to_csv('product_areas.csv', index=False)
loyalty_scores.to_csv('loyalty_scores.csv', index=False)
```
---
## Visualize in Tableau

<iframe seamless frameborder="0" src="https://public.tableau.com/views/CustomerInsightandSales/CustomerProfiles?:embed=yes&:display_count=yes&:showVizHome=no" width = '1090' height = '900'></iframe>
