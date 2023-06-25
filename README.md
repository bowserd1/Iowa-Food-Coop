# Customer Analysis: Predicting Sales to Each Iowa Food Coop Customer

The Covid-19 pandemic disrupted most businesses and the Iowa Food Cooperative (IFC) was no exception. Early during the pandemic, sales skyrocketed, and farms sold out of product as customers sought out different options to purchase food. As the pandemic progressed, many customers stopped shopping at the IFC and sales have gradually declined to levels near what they were before the pandemic. Seeing the need to increase sales, the IFC plans to conduct a marketing campaign to existing members. The purpose of this project is to predict how much every customer member will purchase during an ordering period so the IFC can target specific customers with their marketing campaign.

The IFC is located in Des Moines, IA and connects customers to local farms around the state of Iowa. As a cooperative, the IFC requires membership and operates on ordering cycles of 2 weeks. Customers order direct from farms. Every two weeks, those farms deliver produts to the IFC and the customers pick up their orders at a variety of pickup locations around central Iowa or opt for home delivery.

## 1.Considerations and Methodology
There are several things I needed to consider with this project:
First, predicting how much each customer will buy isn't a perfect fit for machine learning. Once the data is cleaned, every row will be how much money a customer spent during a given ordering period (along with many other features). Since there have been about 50 ordering periods in the data, some customers will have 50 rows, while newer customers will have fewer rows. This isn't ideal because the rows are not independent of each other.

Two natural projects that were discussed were: building a recommendation system to recommend additional items to customers based on what they purchased and predicting how much money a new customer would spend over the course of a year once they had ordered for a month. While both of these ideas would be good projects for machine learning, it was decided that the most actionable idea was predicting what a customer could spend so that the IFC could offer incentives to increase customer order size.

Knowing that having rows that aren't entirely independent of each other, I moved forward with predicting each customer's next order amount.

## 2. Data
The data for this project all came from the IFC's database, and I was able to access the database using SQL queries once I was given access to the database. Some of the sales data is available to be downloaded from the website in a csv file with the right access. I used both a csv download and data acquired from SQL queries for this project. The raw data is confidential.

The data contains sales data for every purchase since early 2021. Every sale is linked to a unique customer id, along with the item purchased, its category and subcateogry.

## 3. Data Wrangling
The data wrangling can be found in two files in this repository. The primary problems in data wrangling and cleaning are outlined below:
**1.** The sales data had a row for every sale made, so the sales needed to be grouped by customer and by ordering period. When grouping the data, however, we lose information about the individual items that each customer purchased. In the database, every purchase has a category, subcategory and the item.
I made each category a feature in the dataframe, and 


## 4. Exploratory Analysis
