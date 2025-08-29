# Datapipe Process 

This is a process to show how I would develop a robust and automated datapipe process using Azure Data Factory to ingest and orchestrate data, Snowflake as my datawarehouse, DBT to transform the data, and Looker to visualize que results.

##

The main ideia of this project is to present a well documented datapipeline, following best pratices, ensuring governance over the process and good replicability as well. 

##
## Azure

Using the .csv example files I found in a Udemy Course, I created a Storage Account and a Container (showed in the image bellow) to store all the raw datasources. After this setup, it was also necessary to set the permissions in snowflake to configure the connection between both tools, allowing the ingesting of raw data into my datawarehouse of choice, snowflake in this case.

<p align="center">
<img width="12" />
<img src="Azure.png" height="600" alt="powerbi logo" width ="900"  />

##
## DBT

All the data transformations that made sense, and that in this example, would`ve been requested by the users, were done setting dim, fact and analytical tables and folders in DBT.

<p align="center">
<img width="12" />
<img src="dbt.png" height="1600" alt="powerbi logo" width ="400"  />

##
## Snowflake

Besides the connection to Azure and the table storage (with all tables being created in DBT), there`s no major process being done in Snowflake.

<p align="center">
<img width="12" />
<img src="snowflake.png" height="1600" alt="powerbi logo" width ="400"  />
  
##
## Looker

Here`s the dashboard developed using one of the analytical tables, in Looker.

<p align="center">
<img width="12" />
<img src="Captura de tela 2025-08-28 213337.png" height="600" alt="powerbi logo" width ="900"  />

