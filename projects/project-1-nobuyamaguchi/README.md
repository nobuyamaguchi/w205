# Project 1: Query Project

- In the Query Project, you will get practice with SQL while learning about
  Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven
  questions using public datasets housed in GCP. To give you experience with
  different ways to use those datasets, you will use the web UI (BiqQuery) and
  the command-line tools, and work with them in jupyter notebooks.

- We will be using the Bay Area Bike Share Trips Data
  (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement

- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the
  company running Bay Area Bikeshare. You are trying to increase ridership, and
  you want to offer deals through the mobile app to do so. What deals do you
  offer though? Currently, your company has three options: a flat price for a
  single one-way trip, a day pass that allows unlimited 30-minute rides for 24
  hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


---

## Part 1 - Querying Data with BigQuery

### What is Google Cloud?
- Read: https://cloud.google.com/docs/overview/

### Get Going

- Go to https://cloud.google.com/bigquery/
- Click on "Try it Free"
- It asks for credit card, but you get $300 free and it does not autorenew after the $300 credit is used, so go ahead (OR CHANGE THIS IF SOME SORT OF OTHER ACCESS INFO)
- Now you will see the console screen. This is where you can manage everything for GCP
- Go to the menus on the left and scroll down to BigQuery
- Now go to https://cloud.google.com/bigquery/public-data/bay-bike-share 
- Scroll down to "Go to Bay Area Bike Share Trips Dataset" (This will open a BQ working page.)


### Some initial queries
Paste your SQL query and answer the question in a sentence.

- What's the size of this dataset? (i.e., how many trips)
Query:
```sql
#standardSQL
SELECT count(*) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Answer:
983648

- What is the earliest start time and latest end time for a trip?
Query:
```sql
#standardSQL
SELECT min(start_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

```sql
#standardSQL
SELECT max(end_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Answer:
Earliest start time: 	
2013-08-29 09:08:00 UTC

Latest end time:
2016-08-31 23:48:00 UTC

- How many bikes are there?
Query:
```sql
#standardSQL
SELECT count(distinct bike_number)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

Answer:
700

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.
- Use the SQL tutorial (https://www.w3schools.com/sql/default.asp) to help you with mechanics.

- Question 1: When do people use bikes during the day?
  * Answer:	
hour	count		
0	2929	
1	1611	
2	877	
3	605	
4	1398	
5	5098
6	20519
7	67531	
8	132464	
9	96118	
10	42782	
11	40407
12	46950
13	43714	
14	37852
15	47626	
16	88755	
17	126302	
18	84569	
19	41071	
20	22747	
21	15258	
22	10270
23	6195

Most popular times are around 8am and 5pm. 


  * SQL query:
```sql
#standardSQL
SELECT extract(hour FROM start_date) AS hour, count(*) AS count 
FROM`bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY hour
ORDER BY hour
```

- Question 2: How many subscribers are there in each type?
  * Answer:
```
subscriber_type	count	
Customer	136809	
Subscriber	846839
```

  * SQL query:
```sql
#standardSQL
SELECT subscriber_type, COUNT(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY subscriber_type
```

- Question 3: How long on average do people use the bike share?
  * Answer: 17 min

  * SQL query: 
```sql
#standardSQL
SELECT ROUND(AVG(duration_sec)/60, 0)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```


---

## Part 2 - Querying data from the BigQuery CLI - set up 

### What is Google Cloud SDK?
- Read: https://cloud.google.com/sdk/docs/overview

- If you want to go further, https://cloud.google.com/sdk/docs/concepts has
  lots of good stuff.

### Get Going

- Install Google Cloud SDK: https://cloud.google.com/sdk/docs/

- Try BQ from the command line:

  * General query structure

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

### Queries

1. Rerun last week's queries using bq command line tool (Paste your bq
   queries):

- What's the size of this dataset? (i.e., how many trips)
```bq query
bq query --use_legacy_sql=false 'SELECT count(*) FROM `b
igquery-public-data.san_francisco.bikeshare_trips`'
```
```   
+--------+
|  f0_   |
+--------+
| 983648 |
+--------+
```

- What is the earliest start time and latest end time for a trip?
```bq query
bq query --use_legacy_sql=false 'SELECT min(start_date),
 max(end_date) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```
```   
+---------------------+---------------------+
|         f0_         |         f1_         |
+---------------------+---------------------+
| 2013-08-29 09:08:00 | 2016-08-31 23:48:00 |
+---------------------+---------------------+
```

- How many bikes are there?
```bq query
bq query --use_legacy_sql=false 'SELECT count(distinct b
ike_number) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```
```   
+-----+
| f0_ |
+-----+
| 700 |
+-----+
```

2. New Query (Paste your SQL query and answer the question in a sentence):

- How many trips are in the morning vs in the afternoon?


### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: How long do people use bike share from Monday to Friday? 

- Question 2: How long do people use bike share between 7 and 9am or 4 and 6pm on weekday? 

- Question 3: How many users are there from 7 to 9am and 4 to 6pm on weekday?

- Question 4: How many customers and subscribers are there from 7 to 9am and 4 to 6pm on weekday?

- Question 5: Where do people start using bike share from 7 to 9am and 4 to 6pm on weekday?

- Question 6: Where do people end using bike share from 7 to 9am and 4 to 6pm on weekday?

- Question 7: Where do people start using bike share from 7 to 9am and on weekday?

- Question 8: Where do people start using bike share from 4 to 6pm on weekday?

- Question 9: Where do people end using bike share from 7 to 9am and on weekday?

- Question 10: Where do people end using bike share from 4 to 6pm and on weekday? 

### Answers

Answer at least 4 of the questions you identified above You can use either
BigQuery or the bq command line tool.  Paste your questions, queries and
answers below.

- Question 1: How long do people use bike share from Monday to Friday?
  * Answer:
```
+-----+------+
| day | f0_  |
+-----+------+
|   2 | 14.0 |
|   3 | 13.0 |
|   4 | 13.0 |
|   5 | 14.0 |
|   6 | 16.0 |
+-----+------+
```

  * SQL query:
```bq query
bq query --use_legacy_sql=false '
 SELECT extract(dayofweek from start_date) as day, ROUND(AVG(duration_sec)/60, 0) 
 FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
 WHERE extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7 
 GROUP BY day ORDER BY day'
```
```sql
#standardSQL
SELECT extract(dayofweek from start_date) as day, ROUND(AVG(duration_sec)/60, 0)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7
GROUP BY day
ORDER BY day
```

- Question 2: How long do people use bike share between 7 and 9am or 4 and 6pm on weekday?  
  * Answer:
```
+-----+----------+
| day | duration |
+-----+----------+
|   2 |     11.0 |
|   3 |     11.0 |
|   4 |     11.0 |
|   5 |     11.0 |
|   6 |     13.0 |
+-----+----------+
```

  * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT extract(dayofweek from start_date) as day, ROUND(AVG(duration_sec)/60, 0) as duration
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND( (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
OR (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19))
GROUP BY day
ORDER BY day
'
```



- Question 3: How many users are there from 7 to 9am and 4 to 6pm on weekday?
  * Answer:
```
+------+--------+
| hour | count  |
+------+--------+
|    7 |  65900 |
|    8 | 128999 |
|    9 |  90264 |
|   16 |  79000 |
|   17 | 118332 |
|   18 |  78188 |
+------+--------+
```

  * SQL query:
```bq query
bq query --use_legacy_sql=false '
 SELECT extract(hour FROM start_date) AS hour, count(*) AS count 
 FROM `bigquery-public-data.san_francisco.bikeshare_trips`
 WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
 AND( (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
 OR (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19))
 GROUP BY hour
 ORDER BY hour
 '
```
- Question 4: How many customers and subscribers are there from 7 to 9am and 4 to 6pm on weekday?
  * Answer:
```
+-----------------+--------+
| subscriber_type | count  |
+-----------------+--------+
| Customer        |  30812 |
| Subscriber      | 529871 |
+-----------------+--------+
```

  * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT subscriber_type, COUNT(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND( (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
OR (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19))
GROUP BY subscriber_type
'
```

- Question 5: Where do people start using bike share from 7 to 9am and 4 to 6pm on weekday?
  * Answer:
```
+-----------------------------------------------+-------+
|              start_station_name               | count |
+-----------------------------------------------+-------+
| San Francisco Caltrain (Townsend at 4th)      | 52441 |
| San Francisco Caltrain 2 (330 Townsend)       | 40264 |
| Temporary Transbay Terminal (Howard at Beale) | 28880 |
| Harry Bridges Plaza (Ferry Building)          | 27298 |
| Steuart at Market                             | 25313 |
| 2nd at Townsend                               | 24103 |
| Townsend at 7th                               | 20323 |
| Market at Sansome                             | 18880 |
| Embarcadero at Sansome                        | 18587 |
| Market at 10th                                | 17273 |
+-----------------------------------------------+-------+
```

  * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT start_station_name, count(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND( (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
OR (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19))
GROUP BY start_station_name
ORDER BY count DESC LIMIT 10
'
```

- Question 6: Where do people end using bike share from 7 to 9am and 4 to 6pm on weekday?
  * Answer:
```
+-----------------------------------------------+-------+
|               end_station_name                | count |
+-----------------------------------------------+-------+
| San Francisco Caltrain (Townsend at 4th)      | 65411 |
| San Francisco Caltrain 2 (330 Townsend)       | 42095 |
| Harry Bridges Plaza (Ferry Building)          | 26851 |
| 2nd at Townsend                               | 26824 |
| Steuart at Market                             | 25521 |
| Temporary Transbay Terminal (Howard at Beale) | 24424 |
| Townsend at 7th                               | 23984 |
| Market at Sansome                             | 23805 |
| Embarcadero at Sansome                        | 20092 |
| Embarcadero at Folsom                         | 14154 |
+-----------------------------------------------+-------+
```
```bq query
bq query --use_legacy_sql=false '
SELECT end_station_name, count(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND( (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
OR (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19))
GROUP BY end_station_name
ORDER BY count DESC LIMIT 10
'
```

- Question 7: Where do people start using bike share from 7 to 9am and on weekday?
  * Answer:
```
+-----------------------------------------------+-------+
|              start_station_name               | count |
+-----------------------------------------------+-------+
| San Francisco Caltrain (Townsend at 4th)      | 38900 |
| San Francisco Caltrain 2 (330 Townsend)       | 30458 |
| Harry Bridges Plaza (Ferry Building)          | 20002 |
| Temporary Transbay Terminal (Howard at Beale) | 19085 |
| Steuart at Market                             | 14937 |
| 2nd at Townsend                               | 10724 |
| Grant Avenue at Columbus Avenue               | 10151 |
| Market at Sansome                             |  7825 |
| Market at 10th                                |  7696 |
| Civic Center BART (7th at Market)             |  7473 |
+-----------------------------------------------+-------+
```
  * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT start_station_name, count(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
GROUP BY start_station_name
ORDER BY count DESC LIMIT 10
'
```

- Question 8: Where do people start using bike share from 4 to 6pm on weekday?
  * Answer:
```
+-----------------------------------------------+-------+
|              start_station_name               | count |
+-----------------------------------------------+-------+
| Townsend at 7th                               | 14038 |
| San Francisco Caltrain (Townsend at 4th)      | 13541 |
| 2nd at Townsend                               | 13379 |
| Embarcadero at Sansome                        | 12279 |
| Market at Sansome                             | 11055 |
| 2nd at South Park                             | 10538 |
| Steuart at Market                             | 10376 |
| San Francisco Caltrain 2 (330 Townsend)       |  9806 |
| Temporary Transbay Terminal (Howard at Beale) |  9795 |
| Market at 10th                                |  9577 |
+-----------------------------------------------+-------+
```

 * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT start_station_name, count(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19)
GROUP BY start_station_name
ORDER BY count DESC LIMIT 10
'
```

- Question 9: Where do people end using bike share from 7 to 9am and on weekday?
  * Answer:
```
+-----------------------------------------------+-------+
|               end_station_name                | count |
+-----------------------------------------------+-------+
| San Francisco Caltrain (Townsend at 4th)      | 20490 |
| 2nd at Townsend                               | 17261 |
| Townsend at 7th                               | 16832 |
| Market at Sansome                             | 13109 |
| Embarcadero at Sansome                        | 11604 |
| San Francisco Caltrain 2 (330 Townsend)       | 10918 |
| Embarcadero at Folsom                         | 10778 |
| Steuart at Market                             | 10438 |
| Howard at 2nd                                 | 10425 |
| Temporary Transbay Terminal (Howard at Beale) |  9692 |
+-----------------------------------------------+-------+
```
  * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT end_station_name, count(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND (extract(hour from start_date) > 6 AND extract(hour from start_date) < 10)
GROUP BY end_station_name
ORDER BY count DESC LIMIT 10
'
```

- Question 10: Where do people end using bike share from 4 to 6pm and on weekday?
  * Answer:
```
+-----------------------------------------------+-------+
|               end_station_name                | count |
+-----------------------------------------------+-------+
| San Francisco Caltrain (Townsend at 4th)      | 44921 |
| San Francisco Caltrain 2 (330 Townsend)       | 31177 |
| Harry Bridges Plaza (Ferry Building)          | 18185 |
| Steuart at Market                             | 15083 |
| Temporary Transbay Terminal (Howard at Beale) | 14732 |
| Market at Sansome                             | 10696 |
| 2nd at Townsend                               |  9563 |
| Embarcadero at Sansome                        |  8488 |
| Powell Street BART                            |  7396 |
| Townsend at 7th                               |  7152 |
+-----------------------------------------------+-------+
```

  * SQL query:
```bq query
bq query --use_legacy_sql=false '
SELECT end_station_name, count(*) as count
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE (extract(dayofweek from start_date) > 1 AND extract(dayofweek from start_date) < 7)
AND (extract(hour from start_date) > 15 AND extract(hour from start_date) < 19)
GROUP BY end_station_name
ORDER BY count DESC LIMIT 10
'
```

---

## Part 3 - Employ notebooks to synthesize query project results

### Get Going

Use JupyterHub on your midsw205 cloud instance to create a new python3 notebook. 


#### Run queries in the notebook 

```
! bq query --use_legacy_sql=FALSE '<your-query-here>'
```

- NOTE: 
- Queries that return over 16K rows will not run this way, 
- Run groupbys etc in the bq web interface and save that as a table in BQ. 
- Query those tables the same way as in `example.ipynb`


#### Report
- Short description of findings and recommendations 
- Add data visualizations to support recommendations 

### Resource: see example .ipynb file 

[Example Notebook](example.ipynb)
