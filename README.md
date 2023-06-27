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

**3.** Another feature that I added was the distance in miles each customer lives from their pickup location. Customers can choose among several different pickup locations. To calculate the distance to pickup, I needed to convert every customer address to a geolocation, then use this to calculate the distance from the address to pickup location. Before converting the addresses to a geolocation, significant cleaning needed to happen since customers had typed their own addresses in. After correcting several errors, Google's geopy API worked remarkably well. At this point, I also dropped all members from out of state since none of them ordered regularly and are likely to order again since the IFC does not ship product. A sidenote about distance calculations and geopy: getting each customer's geolocation and making the distance calculations take a long time, so I filtered the dataframe to only include each customer once, then joined that back onto the original dataframe.

**4.** I added several other features which included: the month, year and quarter corresponding to the date, how long each customer had been a member, the number of orders a customer had made, and the customer's average order. I also added a categorical column showing if the date was before, after or not near a holliday.


## 4. Exploratory Analysis
I began the exploratory analysis by visualizing where customers live. This heatmap shows where customers live who purchased during the most recent ordering period. The red indicates more dollars spent.
![Screen Shot 2023-06-25 at 1 13 08 PM](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/d18475a2-fee4-4f1c-a254-43248fb6832d)

The following graph shows how much customers spend at each of the different pickup locations:

![Sales_by_location](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/316059aa-8825-4d91-9f10-3b8460131797)

This graph shows the number of people who ordered during the most recent ordering period and places them into bins based on how many years they have been a member.
![Orders_by_years](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/387f1090-a5aa-4ef0-8e93-702c4349fbc3)

Lastly, this histogram shows how much customers spent in the most recent ordering period. We can see here that member orders have a right tail where most people who place an order spend less than $75, but a few customers spend over $200.

![Orders_Last_Period](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/30ad9f84-7ba2-403a-8f09-3fb3a08ef4df)


## 5. Preprocessing
At this stage several decisions were made about how to proceed:

**1.** I dropped all members who had never ordered. This effects members who have never ordered (over 5,000 people), and when using this in production in the future, it will effect new customers because there will be no predictions for them. Essentially, I decided to not set a coldstart value and wait to get more data before making a prediction. Only 16% of members have made an order in the last 2+ years, so it made the most sense to start without a prediction.

**2.** Next, set up some dummy predictions as a baseline for the model. I added columns of a 4-week rolling average, 6-week rolling average, and and 8-week rolling average. Since all of these contain information, I left them in as features for the model as well.

**3** Since all of the data is time-based, I needed to be very careful about setting up the training, validation and test data sets. To set this up correctly, I took the earliest 70% of orders for each customers as the training set, the next 15% for each customer as the validation set, and the last 15% of data as the test set. After grouping the dataframe by member ID and sorting by date, I used the following code to split the dataframe into train, validation and test sets:

![Screen Shot 2023-06-25 at 8 52 36 PM](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/42c3531f-d1b3-4776-afb7-eec300cc69ab)



**4.** Finally, there were several categorical variables that I one-hot-encoded to prepare for modelling. These varibles included: customer pick up locations, how the customer heard about the IFC, and the customer's membership status.

## 6. Modelling
Moving into the modelling stage, I decided to try a random forest regression and xgboost regression. As I mentioned previously, the rows in this problem are not independent, making it not a great candidate for linear regression. 

For accuracy metrics, I decided to evaluate the models using root mean squared error (rmse) because of how the IFC plans to use this project. This will penalize the model for having predictions that are farther from reality. (Once I predict the amount each customer will buy, the IFC plans to offer incentives to people to increase their purchase amount. If the model is off by a small amount it isn't as much of a problem. Consider the following situation: if the model predicts a customer will spend $20 next week, but they will actually already spend $150, giving them an incentive like $20 off your next order of $100 will cost the IFC money because they were already going to spend $100. However, if the model predict someone will spend $20, and they are acutally going to spend $50, the model is off by $20, but the incentive will still provide benefit to the IFC).

For all models, I did not use the standard k-fold validation library from sci-kit learn because the data is time based and shuffling the rows would cause the output to be invalid. Instead, I used randomized search to optimize the hyperparameters and used my validation data set as the data on which to optimize these hyperparameters. The following code does that for the XGBoost model:

![Screen Shot 2023-06-26 at 2 29 56 PM](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/57d4a820-5709-497f-b282-e9685998c973)

After training the models and tuning hyperparameters using the validation set, I combined the training and validation sets and fit the models on that data before finally running the optimized test set.

Also, after training the models and comparing them against the baseline predictions (4,6 and 8 week rolling averages), I noticed that neither of my machine learning models learned as quickly when a customer stopped ordering, or churned. In order to experiment with this, I added two more models which mixed the 8-week rolling average with both random forest and XGBoost. Anywhere that the 8-week rolling average was 0, I made these models predict 0, while anywhere else I left the random forest and XGBoost the same. This comes with a risk of overfitting on future data, but it also recognizes that once a customer stops ordering, they are far less likely to order again.

Evaluating the models based on the rmse, random forest had the best predictions on the training data, although its performance fell off significantly on the test data, suggesting some overfitting. The best model on the test data was the XGBoost model mixed with the rolling 8-week average.

![Model_Metrics](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/812a2e32-e64f-497e-872f-5137366020c6)

One interesting thing to note is that all models (including the baseline values) increased in accuracy from the training set to the test set. Although this isn't typical with machine learning, it makes sense for this specific business problem. The data starts during the Covid-19 pandemic when many customers were ordering (around 300 per ordering period) while more recently, this has dropped to about 200 orders per ordering period. As fewer people order, it is easier for all of the models to predict zeroes for the people who haven't ordered in a long time.

Finally, the top 15 most important features are shown below.

![best_features](https://github.com/bowserd1/Iowa-Food-Coop/assets/120436824/0d4f3198-975e-4044-9dc0-d194ff269419)

## 7. Conclusion and Further Work

The Iowa Food Coop now has a model that will help them to build a marketing campaign where they can incentivize customers to spend more money there. Moving forward, there are several projects that can build on this work in the future:

**1.** Once the IFC determines a marketing plan, we can randomly assign a control group and a test group for which customers get the incentives. Hypothesis testing can be done to determine whether or not the marketing campaign was efective.

**2.** A second project that uses the identical data would be to predict whether or not a customer will be active 12 months later. This project would use the same data and wouldn't need a lot of data manipulation to get the data in place to make these predictions. The goal of this project would be more focused around interpretability and understanding the role each feature plays in a customer being active after 1 year, so that the IFC can focus their marketing efforts during the early weeks of the customers' journeys.

**3.** Lastly, a recommendation system could be built to encourage customers to buy more products.

In conclusion, this project serves as a nice foundation to show the IFC what can be done with their data in the realm of machine learning. Racther than being a one-time project exploring the realm of meachine learning, the IFC can find benefit in several more machine learning applications.
