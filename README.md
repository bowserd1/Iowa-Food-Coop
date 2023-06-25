# Customer Analysis: Predicting Sales to Each Iowa Food Coop Customer

The Covid-19 pandemic disrupted most businesses and the Iowa Food Cooperative (IFC) was no exception. Early during the pandemic, sales skyrocketed, and farms sold out of product as customers sought out different options to purchase food. As the pandemic progressed, many customers stopped shopping at the IFC and sales have gradually declined to levels near what they were before the pandemic. Seeing the need to increase sales, the IFC plans to conduct a marketing campaign to existing members. The purpose of this project is to predict how much every customer member will purchase during an ordering period so the IFC can target specific customers with their marketing campaign.

The IFC is located in Des Moines, IA and connects customers to local farms around the state of Iowa. As a cooperative, the IFC requires membership and operates on ordering cycles of 2 weeks. Customers order direct from farms. Every two weeks, those farms deliver produts to the IFC and the customers pick up their orders at a variety of pickup locations around central Iowa or opt for home delivery.

## 1.Considerations and Methodology
There are several things I needed to consider with this project:
First, predicting how much each customer will buy isn't a perfect fit for machine learning. Once the data is cleaned, every row will be how much money a customer spent during a given ordering period (along with many other features). Since there have been about 50 ordering periods in the data, some customers will have 50 rows, while newer customers will have fewer rows. This isn't ideal because the rows are not independent of each other.

Two natural projects that were considered were: building a recommendation system to recommend additional items to customers based on what they purchased and predicting how much money a new customer would spend over the course of a year once they had ordered for a month. While both of these ideas would be good projects for machine learning, it was decided that the most actionable idea was predicting what a customer could spend so that the IFC could offer incentives to increase customer order size.

Knowing that having rows that aren't entirely independent of each other, I moved forward with predicting each customer's next order amount because this was the most meaningful project for the business need.

## 2. Data
The data for this project all came from the IFC's database, and I was able to access the database using SQL queries once I was given access to the database. Some of the sales data is available to be downloaded from the website in a csv file with the right access. I used both a csv download and data acquired from SQL queries for this project. The raw data is confidential.

The data contains sales data for every purchase since early 2021. Every sale is linked to a unique customer id, along with the item purchased, its category and subcateogry.

## 3. Data Wrangling
The data wrangling can be found in two files in this repository. A few challenges in data wrangling and cleaning are outlined below:
**1.** The sales data had a row for every sale made, so the sales needed to be grouped by customer and by ordering period. When grouping the data, however, we lose information about the individual items that each customer purchased. In the database, every purchase has a category, subcategory and the item. I made each category a feature in the dataframe, and used pivot tables to extract the sales by week and category for each customer. This was then joined back into the dataframe. Thus, the new dataframe shows us how much each customer had spent within a certain cateogry on date. There are about 20 categories and examples of these categories include: Produce, Meat-Beef, Meat-Pork, and Prepared Foods.

**2.** Once the sales were grouped by category and week, I needed to sum the columns cumulatively for each customer so that we could see how much the customer had spent within each category up to any given date. To avoid data leakage, all the sales information also needed to be shifted back by one ordering period. This is extremely important so that we avoid using future data to predict sales for the given week.

**3.** Another feature that I added was the distance in miles each customer lives from their pickup location. Customers can choose among several different pickup locations. To calculate the distance to pickup, I needed to convert every customer address to a geolocation, then use this to calculate the distance from the address to pickup location. Before converting the addresses to a geolocation, significant cleaning needed to happen since customers had typed their own addresses in. After correcting several errors, Google's geopy API worked remarkably well.

**4.** I added several other features which included: the month, year and quarter corresponding to the date, how long each customer had been a member, the number of orders a customer had made, and the customer's average order. I also added a categorical column showing if the date was before, after or not near a holliday.


## 4. Exploratory Analysis
I began the exploratory analysis by visualizing where customers live. This heatmap shows where customers live who purchased during the most recent ordering period. The red indicates more dollars spent.
![Screen Shot 2023-06-25 at 1 13 08 PM](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/d18475a2-fee4-4f1c-a254-43248fb6832d)

The following graph shows how much customers spend at each of the different pickup locations:

![Sales_by_location](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/316059aa-8825-4d91-9f10-3b8460131797)

This final graph shows the number of people who ordered during the most recent ordering period and places them into bins based on how many years they have been a member.
![Orders_by_years](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/387f1090-a5aa-4ef0-8e93-702c4349fbc3)

Each of these graphs starts to give us a better feel for who the customers are.

## 5. Preprocessing

