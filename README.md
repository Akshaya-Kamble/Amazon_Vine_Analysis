# Amazon_Vine_Analysis
## Overview of the analysis:
This project is about analyzing Amazon reviews written by members of the paid Amazon Vine program.The Amazon Vine program is a service that allows manufacturers and publishers to receive reviews for their products. Companies pay a small fee to Amazon and provide products to Amazon Vine members, who are then required to publish a review.
The dataset selected for this project is about musical instruments.For the analysis we will use PySpark to perform the ETL process to extract the dataset, transform the data, connect to an AWS RDS instance, and load the transformed data into pgAdmin.Next using PySpark we can determine if there is any bias towards favorable reviews from Vine members in the dataset.

The following versions of softwares are used
PySpark spark-3.0.1
PgAdmin 4.24
Amazon Web Services - S3 and RDS

## Results:
### Deliverable 1: Perform ETL on Amazon Product Reviews
For this deliverable we have selected the musical instruments reviews dataset from the Amazon Review datasets.
```https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Musical_Instruments_v1_00.tsv.gz```
Using PySpark we will extract the dataset into a DataFrame,transform the DataFrame into four separate DataFrames that match the table schema in pgAdmin and finally upload the transformed data into the appropriate tables and run queries in pgAdmin to confirm that the data has been uploaded.

#### A. Using the provided schema we have created four tables in the pgAdmin named customers_table, products_table, review_id_table, and vine_table.Below are the details how the data is collected in these tables using PYSpark.
#### 1. customers_table
In this table we will have a count of customer_id and we will get the output using groupby() and agg() functions.
``` customers_df = df.groupby("customer_id").agg({"customer_id":"count"}).withColumnRenamed("count(customer_id)", "customer_count")
customers_df.show(20) ```
![]() 

#### 2. products_table
In this table we will have the unique product_ids and the product_title buy using the drop_duplicates()
```products_df = df.select(["product_id","product_title"]).drop_duplicates()
products_df.show(20)```
![]() 

#### 3. review_id_table
In this table we will have columns like the pgAdmin schema and we will also convert the review_date column to date using the below code.
``` review_id_df = df.select(["review_id","customer_id","product_id","product_parent", to_date("review_date", 'yyyy-MM-dd').alias("review_date")])
review_id_df.show(20)```
![]() 

#### 4.vine_table
In this table we will have all the columns like the pgAdmin schema.
```vine_df = df.select(['review_id',"star_rating","helpful_votes","total_votes","vine","verified_purchase"])
vine_df.show(20)```
![]() 

#### B. Using PySpark we will Connect to the AWS RDS instance and write each of the above created DataFrames to its corresponding tables in pgAdmin.

Configure settings for AWS RDS
```mode = "append"
jdbc_url="jdbc:postgresql://awschallenge.cw3ezyluysbs.us-east-2.rds.amazonaws.com:5432/Challenge"
config = {"user":"postgres", 
          "password": "Newme2020", 
          "driver":"org.postgresql.Driver"}```

Write all the four tables to pgAdmin using the below code.
```review_id_df.write.jdbc(url=jdbc_url, table='review_id_table', mode=mode, properties=config)
   products_df.write.jdbc(url=jdbc_url, table='products_table', mode=mode, properties=config)
customers_df.write.jdbc(url=jdbc_url, table='customers_table', mode=mode, properties=config)
vine_df.write.jdbc(url=jdbc_url, table='vine_table', mode=mode, properties=config)
```

After the data is sent to the tables, we can query and check the data in all the tables.Below are the screen shots from pgAdmin.

### Deliverable 2: Determine Bias of Vine Reviews
#### A. In this deliverable using PySpark we will sort and filter data from the AWS dataset to get the required dataframes by performing some calculations. 
1. vine_df : This dataframe contains all the schema as required in pgAdmin table.
2. vote_count_df : This dataframe has total_votes count equal to or greater than 20
3. voting_percent_df : This dataframe has the number of helpful_votes divided by total_votes equal to or greater than 50%.
4. paid_reviews_df : This dataframe has all the rows where a review was written as part of the Vine program (paid)
5. unpaid_reviews_df : This dataframe has all the rows where a review was written as part of the Vine program (unpaid)


#### B. Using the above created dataframes paid_reviews_df and unpaid_reviews_df we can answers the following questions.

#### 1. How many Vine reviews and non-Vine reviews were there?
There are 60 Vine reviews and 14477 non-Vine reviews.

#### 2. How many Vine reviews were 5 stars? How many non-Vine reviews were 5 stars?
There are 34 Vine reviews which are 5 stars and 8212 non-Vine reviews which are 5 stars.

#### 3. What percentage of Vine reviews were 5 stars? What percentage of non-Vine reviews were 5 stars?
56.66 % of Vine reviews are 5 stars and 56.72 % percentage of non-Vine reviews are 5 stars.

Below are snapshots for the code.
![]()
![]()

### Summary: 