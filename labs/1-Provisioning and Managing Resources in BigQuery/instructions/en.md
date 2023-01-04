# Provisioning and Managing Resources in BigQuery

## Overview

In this lab, you use the Google Cloud console and the CLI to create BigQuery datasets and tables. You also control access to and share data and monitor your BigQuery usage. 

## Objectives

In this lab, you learn how to:
* Create datasets
* Create native and external tables
* Control access to and share data
* Monitor resource utilization 


## Setup and Requirements

![[/fragments/start-qwiklab]]


![[/fragments/cloudshell]]


## Task 1. Create datasets

1. In the Google Cloud console, from the Navigation menu (![Navigation Menu Icon](img/nav-menu.png)), select __BigQuery__. 

2. In the BigQuery UI Explorer pane, click the __Action__ menu next to your Project ID, and select __Create dataset__.

![Create Dataset Menu](img/create-dataset.png)

3. In the resulting dialog, set the Dataset ID to `wikipedia_data`, and set the Data location to __US (multiple regions in United States)__. Then, click the __Create dataset__ button. 

## Task 2. Create native and external tables

1. In the Explorer pane, click the Action menu next to `wikipedia_data`, and select __Create table__.

![Create Table Menu](img/create-table.png)

2. In the Create table dialog, set the Source to __Google Cloud Storage__. Then, set the file pattern to `bigquery-demo-bucket/wikipedia-data/csv/Wiki1B-*.csv`

__Note:__ Notice the astericks (*) in the file pattern. There is approximately a gigabyte of data being imported from a number of CSV files. This would be a very common way of importing large datasets into BigQuery. 

3. Confirm the file format was changed to CSV. In the Destination section, confirm the dataset is set to `wikipedia_data`, and set the table name to `wiki_table_csv`

4. In the Schema section, select the __Auto detect__ checkbox. 

5. Expand the __Advanced options__ section, and set the __Header rows to skip__ property to 1. Finally, click the __Create Table__ button. 

6. When the load operation completes, in the Exporer pane, select the __Action__ menu next to the `wiki_table_csv` table and select __Query__. 

7. In the Query editor, paste the following query and run it. _This query counts the number of views for Wikipedia articles with "GOOGLE" in the title_. 

```
SELECT
  Title, COUNT(views) AS views
FROM
  wikipedia_data.wiki_table_csv
WHERE
  CONTAINS_SUBSTR(title, 'GOOGLE')
GROUP BY
  Title
ORDER BY
  views DESC
LIMIT
  100;
```

8. Examine the Query results.

![Query Results](img/query-results.png)

9. Click __Job information__ and examine it as well. 

![Job Information](img/job-information.png)

### Create an external table. 

10. As before, click the __Action__ menu next to the `wikipedia_data` dataset, and click __Create table__. 

11. Set the Source to __Google Cloud Storage__, and set the file pattern to `bigquery-demo-bucket/wikipedia-data/parquet/Wiki1B-*`. Also, set the file format to __Parquet__.

__Note:__ This is the same data that you imported earlier, but this data is in Parquet format. This is a columnar format similar to BigQuery's native format and also Redshift's native format. 

12. In the Destination section, confirm the dataset is set to `wikipedia_data`, set the table name to `wiki_external_table_parquet`, and set the table type property to __External table__. 

__Note:__ External tables are not imported into BigQuery Storage. The data stays in Cloud Storage, and only the schema is added to BigQuery. The table can be queried in the same way though. 

13. As before, select the __Auto detect__ checkbox in the schema section. Then, click the __Create table__ button. 

14. Run the same query as before, but using the external table. Compare the __Query results__ and __Job information__ from each one. You should find that the query using the native BigQuery table ran faster than the one that used the external table.   

```
SELECT
  Title, COUNT(views) AS views
FROM
  wikipedia_data.wiki_external_table_parquet
WHERE
  CONTAINS_SUBSTR(title, 'GOOGLE')
GROUP BY
  Title
ORDER BY
  views DESC
LIMIT
  100;
```

![Job Information External Table](img/job-information-external.png)

## Task 3. Control access to and share data

1. In the Explorer pane, click the __Action__ menu next to `wikipedia_data`, and select __Share__.

![Share Dataset Menu](img/share-dataset.png)

2. Make our colleague, Mike, a BigQuery Data Editor. Click the __Add principals__ button. Enter the email `michaelabel@google.com` and assign to him the role __BigQuery Data Editor__, and then click the __Save__ button. 

![Dataset Permissions](img/dataset-permissions.png)

__Note:__ You have given Mike permissions to edit data in the dataset. You now give everyone permissions to query the Wikipedia data. 

3. As you just did, click the __Action__ menu next to `wikipedia_data`, and select __Share__.  Click the __Add principals__ button. This time, set the new principal to the key word `allAuthenticatedUsers`and assign to that principal the __BigQuery Data Viewer__ role. When prompted, click __Allow Public Access__.

![Make Public Dataset](img/make-public-dataset.png)

## Task 4. Monitor resource utilization 

1. In the Google Cloud console, from the Navigation menu (![Navigation Menu Icon](img/nav-menu.png)), scroll down to the __Operations__ section and select __Monitoring__. If you are warned about unsaved work, select __LEAVE__.

2. Dismiss any popup windows and then from the left-hand pane of the Monitoring UI, select __Dashboards__. 

![Monitoring Dashboards](img/monitoring-icon.png)

3. In the __All Dashboards__ table, click the __BigQuery__ dashboard. 

4. There is not much data to see because you just started using this project, but there should be some. Note under Project metrics, you have charts for Queries in Flight, Query Times, and Slot Utilization. 

## Task 5. Create resources with the CLI

Everything you can do in the console, you can also do programatically using the Command Line Interface (CLI). This allows you to write scripts to create and destroy resources. It can also make migration and testing easier and more automated. 

1. On the Google Cloud console title bar, click __Activate Cloud Shell__ (![cloud shell icon](img/cloud_shell_icon.png)). If prompted, click __Continue__.

2. Enter the following command; you should see the dataset you created earlier. 

```
bq ls
```

3. Enter the following command; you should see the tables in the dataset you created earlier. 

```
bq ls wikipedia_data
```

4. Create a new dataset with the following command.

```
bq mk --location=US bikes_database
```

5. Now, add a table to the dataset you just created. The table is named `regions`, the data is in a CSV file in Cloud Storage. You use the --autodetect parameter to automatically detect the table schema. 

```
bq load --autodetect bikes_database.regions gs://bigquery-demo-bucket/bikes-database/regions.csv
```

6. There are two more files in the same bucket: `stations.csv` and `trips.csv`. Use the same command as above, but create two more tables, `stations` and `trips`, using those two files. 


```
bq load --autodetect bikes_database.stations gs://bigquery-demo-bucket/bikes-database/stations.csv

bq load --autodetect bikes_database.trips gs://bigquery-demo-bucket/bikes-database/trips.csv
```

7. Confirm that the tables were created using the following command.

```
bq ls bikes_database
```

8. Use the following command to see some data from the stations table. 

```
bq head bikes_database.stations
```

9. Use the following command to see some data from the trips table. 

```
bq head bikes_database.trips
```

10. Now run a query.

```
bq query """
SELECT
  s.name,
  COUNT(t.trip_id) AS trips
FROM
  bikes_database.stations s
JOIN
  bikes_database.trips t
ON
  s.name = t.start_station_name
GROUP BY
  s.name
ORDER BY
  trips DESC;
"""
```

11. In Cloud Shell, type `exit` and then refresh the BigQuery UI. In the Explorer pane, expand your project to find your newly added dataset and tables. 

12. Explore the UI, and try experimenting with some queries to get familiar with it. 


### **Congratulations!** You have used the Google Cloud console and the CLI to create BigQuery datasets and tables. You also controlled access to and shared data and momitored your BigQuery usage. 


![[/fragments/endqwiklab]]

![[/fragments/copyright]]
