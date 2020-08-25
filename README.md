# Data Acquisition

## Overview

The first section of the project describes the loading of the Yelp dataset into SQL in order to explore the data and prepare it for futher analysis and predictive modeling of number of stars that a user assigns.


The analysis for this project was performed in MySQL.

## Description of the Yelp Dataset

The Yelp dataset `yelp_db.sql` has been downloaded from the [Yelp Open Dataset](https://www.yelp.com/dataset) website.

The association and structure of the tables is shown below:

![](images/yelp_dataset_schema.png)

## Loading Yelp Dataset into SQL

First the directory is changed to `/usr/local/mysql`, logging in to mysql from the command line: 

```ShellSession
cd /usr/local/mysql
bin/mysql -u root -p
```

Then creating a database `yelp_db`:

```sql
mysql> create database yelp_db
```

Exit from mysql by typing `exit`, then changing the directory in the command line to the location of the dataset `yelp_db.sql` that we want to load, and load the data:

```ShellSession
cd /Users/mansoor/Documents/"Data Science"/"SQL Data Analysis"
/usr/local/mysql/bin/mysql -u root -p yelp_db < yelp_db.sql
```
# Data Preparation



This section explores Yelp dataset and prepares the data for predictive modeling of the number of stars that users assign in their reviews.

Data exploration was performed by running a SQL script file `yelp_sql_code.sql` which is included the code discussed in detail below.  In order to run the script, we execute the following lines of code:

```sql
mysql> use yelp_db
mysql> source /Users/mansoor/Documents/Data Science/Portfolio/Project Code/yelp_sql_code.sql
```

The output was automatically recorded into an output file `yelp_output.txt`.  This was accomplished by adding 

```sql
tee /Users/mansoor/Documents/Data Science/Portfolio/Project Code/yelp_output.txt
```

in the beginning and `notee` in the end of `yelp_sql_code.sql`.

## Yelp Dataset Profiling and Understanding

The following code profiles the data by finding the total number of records for each of the tables in the database:

```sql
select count(*) from Attribute;
select count(*) from Business;
select count(*) from Category;
select count(*) from Checkin;
select count(*) from elite_years;
select count(*) from friend;
select count(*) from hours;
select count(*) from photo;
select count(*) from review;
select count(*) from tip;
select count(*) from user;
```

The number of records for each of the tables is summarized below:

| Table Name | No. Records |
|:--- | ---:|
| Attribute table | 1,310,575 | 
| Business table | 174,567 | 
| Category table | 667,527 | 
| Checkin table | 3,911,218 | 
| elite_years table | 187,125 | 
| friend table | 49,626,957 | 
| hours table | 824,595 | 
| photo table | 206,949 | 
| review table | 5,261,669 | 
| tip table | 1,098,325 | 
| user table | 1,326,101 | 


Next, we check if there are any NULL values in the Users table:

```sql
select id from user where id is NULL;
select id from user where name is NULL;
select id from user where review_count is NULL;
select id from user where yelping_since is NULL;
select id from user where useful is NULL;
select id from user where funny is NULL;
select id from user where cool is NULL;
select id from user where fans is NULL;
select id from user where average_stars is NULL;
select id from user where compliment_hot is NULL;
select id from user where compliment_more is NULL;
select id from user where compliment_profile is NULL;
select id from user where compliment_cute is NULL;
select id from user where compliment_list is NULL;
select id from user where compliment_note is NULL;
select id from user where compliment_plain is NULL;
select id from user where compliment_cool is NULL;
select id from user where compliment_funny is NULL;
select id from user where compliment_writer is NULL;
select id from user where compliment_photos is NULL;
```

The output below shows that there are no NULL values in the user table:

| Feature Name | No. NULL Values |
|:--- |:---:|
| id | - |
| name | - |
| review_count | - |
| yelping_since | - |
| useful | - |
| funny | - |
| cool | - |
| fans | - |
| average_stars | - |
| compliment_hot | - |
| compliment_more | - |
| compliment_profile | - |
| compliment_cute | - |
| compliment_list | - |
| compliment_note | - |
| compliment_plain | - |
| compliment_cool | - |
| compliment_funny | - |
| compliment_writer | - |
| compliment_photos | - |


To find the minimum, maximum, and average value for selected features, which include `stars`, `likes`, and `counts` from different tables, we run the following code: 

```sql
select min(stars), max(stars), avg(stars) from review;
select min(stars), max(stars), avg(stars) from business;
select min(likes), max(likes), avg(likes) from tip;
select min(count), max(count), avg(count) from checkin;
select min(review_count), max(review_count), avg(review_count) from user;
```

The above code generates the following output:

| Table | Column | Min | Max | Average |
|:--- |:--- |:---:| ---:| ---:|
| Review | Stars | 1 | 5 | 3.7277 |
| Business | Stars | 1 | 5 | 3.6322 |
| Tip | Likes | 0 | 15 | 0.0166 |
| Checkin | Count | 1 | 1,478 | 4.2566 |
| User | Review_count | 0 | 11,954 | 23.1172 |


In order to identify the cities with the most reviews, we list top 25 cities in terms of the number of reviews in descending order:

```sql
select city, sum(review_count) as all_review_count
from business
group by city
order by all_review_count desc
limit 25;
```

The output is shown below:

```
+-----------------+------------------+
| city            | all_review_count |
+-----------------+------------------+
| Las Vegas       |          1605343 |
| Phoenix         |           576709 |
| Toronto         |           430985 |
| Scottsdale      |           308529 |
| Charlotte       |           237118 |
| Pittsburgh      |           179471 |
| Henderson       |           166884 |
| Tempe           |           162772 |
| Mesa            |           134156 |
| Montréal        |           128285 |
| Chandler        |           122343 |
| Gilbert         |            97204 |
| Cleveland       |            92280 |
| Madison         |            86614 |
| Glendale        |            76293 |
| Edinburgh       |            48838 |
| Mississauga     |            43147 |
| Peoria          |            42584 |
| Markham         |            38840 |
| North Las Vegas |            37928 |
| Champaign       |            26260 |
| Surprise        |            25740 |
| Stuttgart       |            25537 |
| Goodyear        |            21508 |
| Richmond Hill   |            18329 |
+-----------------+------------------+
25 rows in set (0.12 sec)
```


Next, we will look into the distribution of star ratings to the business in two cities, Avon and Beachwood:

```sql
select stars, count(id) as number_of_businesses
from business
where city = 'Avon'
group by stars;

select stars, count(id) as number_of_businesses
from business
where city = 'Beachwood'
group by stars;
```

Output for Avon:

```
+-------+----------------------+
| stars | number_of_businesses |
+-------+----------------------+
|   2.5 |                   15 |
|     4 |                   34 |
|     5 |                   31 |
|     2 |                   10 |
|   3.5 |                   40 |
|   1.5 |                    4 |
|   4.5 |                   30 |
|     3 |                   33 |
|     1 |                    3 |
+-------+----------------------+
9 rows in set (0.04 sec)
```


Output for Beachwood:

```
+-------+----------------------+
| stars | number_of_businesses |
+-------+----------------------+
|     5 |                   44 |
|     3 |                   39 |
|   4.5 |                   24 |
|     4 |                   36 |
|     1 |                    7 |
|   3.5 |                   57 |
|     2 |                   24 |
|   2.5 |                   28 |
|   1.5 |                    6 |
+-------+----------------------+
9 rows in set (0.04 sec)
```

The code below outputs the top 3 users based on their total number of reviews: 

```sql
select id
       ,name
       ,review_count
from user
order by review_count desc
limit 3;
```

The code generated the following output:

```
+------------------------+--------+--------------+
| id                     | name   | review_count |
+------------------------+--------+--------------+
| 8k3aO-mPeyhbR5HUucA5aA | Victor |        11954 |
| RtGqdDBvvBCjcu5dUqwfzA | Shila  |        11323 |
| P5bUL3Engv-2z6kKohB6qQ | Kim    |         9788 |
+------------------------+--------+--------------+
3 rows in set (0.56 sec)
```

Are there more reviews with the word "love" or with the word "hate" in them?  The code below performs counts for each of the two words:

```sql
select count(id) as number_of_reviews
from review
where text like '%love%';

select count(id) as number_of_reviews
from review
where text like '%hate%';
```

The output below shows that the number of reviews with the word "love" is substantially larger than the number of reviews with the word "hate":

| Word | No. Reviews Containing the Word |
|:--- | ---:|
| "love" | 957,428 |
| "hate" | 131,187 |


The following code lists the top 10 users with the most fans:

```sql
select id, name, fans
from user
order by fans desc
limit 10;
```

The results are shown below:

```
+------------------------+-----------+------+
| id                     | name      | fans |
+------------------------+-----------+------+
| 37cpUoM8hlkSQfReIEBd-Q | Mike      | 7009 |
| hizGc5W1tBHPghM5YKCAtg | Katie     | 2499 |
| iLjMdZi0Tm7DQxX1C1_2dg | Ruggy     | 2135 |
| eKUGKQRE-Ywi5dY55_zChg | Cherylynn | 1962 |
| UsXqCXRZwSCSw0AT7y1uBg | Candice   | 1872 |
| nkN_do3fJ9xekchVC-v68A | Jeremy    | 1820 |
| ITa3vh5ERI90G_WP4SmGUQ | Peter     | 1803 |
| VHdY6oG2JPVNjihWhOooAQ | Jessica   | 1791 |
| peuxbSQwXed-81cSqL7Ykw | Brittany  | 1707 |
| j14WgRoU_-2ZE1aw1dXrJg | Daniel    | 1631 |
+------------------------+-----------+------+
10 rows in set (0.70 sec)
```

Next, we group restaurants in the city with the largest number of reviews (which is Las Vegas as the analysis above shows) by their overall star rating. Then we compare the restaurants with 2-3 stars to the restaurants with 4-5 stars in terms of the number of reviews:
	
```sql
select city
       ,category
       ,avg(review_count) as av_rev_count			-- number of reviews per business within each group
       ,count(business.id) as num_of_businesses,  

case								-- add categorical variable for the "2-3 stars" and "4-5 stars" categories
  when stars >= 2 and stars <= 3 then '2-3 stars'
  when stars >= 4 and stars <= 5 then '4-5 stars'
  else NULL
end star_group

from business 							-- city comes from the "business" table and category comes from the "category" table
     left join category
     on business.id = category.business_id

where city = 'Las Vegas' and category = 'Restaurants'
group by star_group;
```

The output below shows that restaurants with 4-5 stars have a substantially larger number of reviews compared to restaurants with 2-3 stars.  This makes sense as better / highly rated restaurants likely have a larger number of customers.

```
+-----------+-------------+--------------+-------------------+------------+
| city      | category    | av_rev_count | num_of_businesses | star_group |
+-----------+-------------+--------------+-------------------+------------+
| Las Vegas | Restaurants |     227.8907 |              2370 | 4-5 stars  |
| Las Vegas | Restaurants |      79.7724 |              1964 | 2-3 stars  |
| Las Vegas | Restaurants |     148.6626 |              1568 | NULL       |
+-----------+-------------+--------------+-------------------+------------+
3 rows in set (0.26 sec)
```

We also would like to see the number of stars and number of reviews for open and closed business: 

```sql
select is_open
       ,count(id) as num_of_businesses
       ,avg(stars) as stars_per_business
       ,avg(review_count) as num_reviews_per_business

from business 
group by is_open;
```

It is intuitive that open businesses have, on average, a larger number of stars as the output below shows, because it is easier for better / highly rated restaurants to survive.  The number of reviews is also larger for open businesses, likely for two reasons: first, highly rated businesses tend to have more customers (and hence more reviews) and second, time could have elapsed between the closure of closed businesses and extraction of the Yelp dataset, during which open businesses continued accumulating reviews, while closed businesses no longer accumulated any reviews.

```
+---------+-------------------+--------------------+--------------------------+
| is_open | num_of_businesses | stars_per_business | num_reviews_per_business |
+---------+-------------------+--------------------+--------------------------+
|       1 |            146702 | 3.6548547395400197 |                  31.6508 |
|       0 |             27865 |  3.512901489323524 |                  22.1675 |
+---------+-------------------+--------------------+--------------------------+
```

As the final step, we prepare data for predictive modeling of the effect of user characteristics on the number of stars that 
users assign in their reviews. The dependent variable (or the target) in this classification analysis is the number of stars that a business gets in a review. The features are:

1. The amount of time that the user has been yelping, defined as the difference between the review date and yelping_since date (the number of stars may depend on user yelping experience);
2. The average number of stars assigned by the user (to multiple businesses), i.e. average_stars (whether the user is a soft or hard grader or, alternatively, whether the user lives in an area in which businesses are of a particularly high or a particularly low quality);
3. The total number of reviews provided by the user (representing both user yelping experience and user engagement level);
4. The number of fans that the user has (do hard or soft graders have more fans?);
5. The number of people who found the user useful or funny or cool.

The code below extracts the data required for this analysis:

```sql
select review.id as review_id           -- review variables
       ,review.stars
       ,review.date as review_date
       ,user.id as user_id              -- user variables
       ,user.review_count
       ,user.yelping_since
       ,user.useful
       ,user.funny
       ,user.cool
       ,user.fans
       ,user.average_stars

from review 
inner join user on review.user_id = user.id
limit 25;
```

The first 25 rows of the resulting dataset are listed below:

```
+------------------------+-------+---------------------+------------------------+--------------+---------------------+--------+-------+------+------+---------------+
| review_id              | stars | review_date         | user_id                | review_count | yelping_since       | useful | funny | cool | fans | average_stars |
+------------------------+-------+---------------------+------------------------+--------------+---------------------+--------+-------+------+------+---------------+
| ----X0BIDP9tA49U3RvdSQ |     4 | 2014-02-17 00:00:00 | gVmUR8rqUFdbSeZbsg6z_w |          484 | 2009-01-02 00:00:00 |    300 |    75 |  105 |   29 |          3.78 |
| ---0hl58W-sjVTKi5LghGw |     4 | 2016-07-24 00:00:00 | Y6qylbHq8QJmaCRSlKdIog |            8 | 2014-07-02 00:00:00 |      0 |     0 |    0 |    0 |          3.13 |
| ---3OXpexMp0oAg77xWfYA |     5 | 2012-04-07 00:00:00 | SnXZkRN9Yf060pNTk1HMDg |          106 | 2009-03-26 00:00:00 |    187 |   114 |   63 |    5 |          4.08 |
| ---65iIIGzHj96QnOh89EQ |     5 | 2015-09-11 00:00:00 | VcmSgvslHAhqWoEn16wjjw |           13 | 2015-03-23 00:00:00 |      0 |     0 |    0 |    0 |          3.38 |
| ---7WhU-FtzSUOje87Y4uw |     5 | 2016-01-22 00:00:00 | NKF9v-r0jd1p0JVi9h2T1w |           16 | 2014-09-09 00:00:00 |      0 |     0 |    0 |    0 |          4.47 |
| ---94vtJ_5o_nikEs6hUjg |     5 | 2014-09-17 00:00:00 | c2MQ_LPuvtiiKFR_-OY9pg |          326 | 2012-07-23 00:00:00 |     62 |    47 |   29 |   44 |          3.87 |
| ---aCMjwX1f4XdJMkJmZUw |     5 | 2013-04-09 00:00:00 | 3PnZeTU8S5_JUWkd8Y3CnA |            1 | 2013-04-09 00:00:00 |      1 |     0 |    0 |    0 |             5 |
| ---D6-P4MpS86LYldBfX7w |     4 | 2016-05-23 00:00:00 | mCa0WvcSCwtdfeUyhhfg8w |           19 | 2013-07-28 00:00:00 |      0 |     0 |    0 |    0 |          2.81 |
| ---EJJ7Ou0mbUhB_KLHwvA |     1 | 2016-01-04 00:00:00 | M-LtQxAxEVZ-LdNLXbpQnA |           22 | 2012-03-21 00:00:00 |      6 |     2 |    0 |    0 |          4.26 |
| ---g6CDTpsZqTYdcDoI2Hw |     2 | 2012-01-04 00:00:00 | 47f21XiNtNr2lRjCTbDPvA |           24 | 2011-12-31 00:00:00 |      5 |     2 |    1 |    0 |          3.63 |
| ---ikxExF5hcu8H6gsUizQ |     5 | 2014-09-16 00:00:00 | dTYPv_TVl44gsdgxR2xnQA |          348 | 2011-08-22 00:00:00 |    699 |   342 |  675 |   20 |          3.93 |
| ---nya_pjxWmNFDFyAcfsA |     1 | 2012-06-27 00:00:00 | 5QOtcHU1SoqEqBCRR6FhsA |          145 | 2009-03-03 00:00:00 |     18 |     2 |    1 |    0 |          3.06 |
| ---OVMmOizYt30bx3-D0gA |     5 | 2015-08-12 00:00:00 | WH0D3U5UAtNy0X5U5ZeJnw |           27 | 2010-06-02 00:00:00 |     20 |     9 |    1 |    1 |          3.46 |
| ---p28WNWGZuG6gLAt-V1w |     2 | 2015-05-16 00:00:00 | -8nmj3B-tfY_vFiimtBOsw |           80 | 2014-11-23 00:00:00 |     18 |     6 |    7 |    5 |          3.83 |
| ---PvBlKcw1y_z72nHCcGw |     2 | 2014-05-25 00:00:00 | r0j4IpUbcdC1-HfoMYae4w |          307 | 2011-11-30 00:00:00 |    284 |   117 |  265 |   16 |          3.39 |
| ---S_MXC9CMrOMHmS_RDVw |     4 | 2016-05-19 00:00:00 | YItDcd3bEJAKTnb-pTwduA |           21 | 2016-05-19 00:00:00 |      0 |     0 |    0 |    1 |          3.86 |
| ---uo0XC7NK0hxnmw4ha2g |     1 | 2015-02-15 00:00:00 | n-Hit0Y3O6FxTiEhdEEsZw |           88 | 2014-03-21 00:00:00 |     16 |     1 |    1 |    6 |          3.73 |
| ---u_Of5Mg6iTfczVL79ug |     5 | 2014-05-07 00:00:00 | 4nk5Wn4ElB2oanaKscQ8pQ |           74 | 2009-08-08 00:00:00 |     14 |     0 |    0 |    0 |          3.87 |
| ---votkOF2sfhUkUTRyCYw |     5 | 2009-01-03 00:00:00 | zh9vXKAaUAErsqxY-0mHWw |           11 | 2007-06-03 00:00:00 |     14 |    14 |    4 |    0 |          3.27 |
| ---WDP9kwKyVQiw9GTgNmQ |     1 | 2014-03-17 00:00:00 | tRXe5HRTDsUNqL3yNSquMw |           16 | 2013-01-24 00:00:00 |     27 |     6 |    2 |    0 |           3.5 |
| ---z1dGqzrcq_MD5mE-dqA |     3 | 2015-08-11 00:00:00 | o2nV6JuhnaEyPE0k2M1y0Q |         1250 | 2007-11-06 00:00:00 |    347 |    68 |  460 |   62 |          3.67 |
| ---zHMCae68gIbSbtXxD5w |     4 | 2015-08-29 00:00:00 | rypcWiSNGM0suWsiSLh9xA |          294 | 2014-04-06 00:00:00 |    665 |   269 |  429 |   20 |          4.22 |
| ---Zx9mUxJlC6VP38INJNg |     5 | 2017-08-28 00:00:00 | 27T4_n9GyhIBHeI-51OgCA |            4 | 2015-07-17 00:00:00 |      0 |     0 |    0 |    0 |          3.75 |
| --007YIsRSNb33JY0dyqpg |     4 | 2011-08-29 00:00:00 | tJlsvyij1G8e8e1onQ4yBQ |          242 | 2010-04-21 00:00:00 |    187 |    91 |  159 |   20 |          4.09 |
| --01ogTXqLH2TzILZfrEYw |     5 | 2014-08-04 00:00:00 | _VO8DxfrQk8edRyrHQNsPg |            6 | 2011-12-05 00:00:00 |     22 |     1 |    3 |    0 |          4.33 |
+------------------------+-------+---------------------+------------------------+--------------+---------------------+--------+-------+------+------+---------------+
25 rows in set (0.01 sec)

```
