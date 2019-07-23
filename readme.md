Aviral Bhardwaj And Shubhangi Gautam Major Project On Airline Data Analysis At CDAC
=============

This repo contains data set and queries I use in my presentations on SQL-on-Hive (i.e. Impala and hive) at various conferences. This started off as a repo that was use in my presentation at CloudCon in San Francisco, so the name of the repo reflects that but now this repo has morphed into a single repository that contains my dataset for demos and such at various different presentations on Hive and Impala.

Files in the repository
=======================
* [2008.tar.gz](http://stat-computing.org/dataexpo/2009/2008.csv.bz2): Flight delay dataset from 2008.
* [airports.csv](https://github.com/aviralb/cdacproject/blob/master/airport.csv): Dataset linking airport codes to their full names. More details in [Introduction](http://stat-computing.org/dataexpo/2009/) section.


Datasets
========
There are 2 datasets in the repo.

a) The first dataset contains on-time flight performance data from 2008, originally released by [Research and Innovative Technology Administration (RITA)](http://www.transtats.bts.gov/Fields.asp?Table_ID=236). The source of this dataset is http://stat-computing.org/dataexpo/2009/the-data.html. The dataset 

b) The second dataset contains listing of various airport codes in continental US, Puerto Rico and US Virgin Islands. The source of this dataset is http://www.world-airport-codes.com/ The data was scraped from this website and then cleansed to be in its present CSV form.

Setup
=====

You will need a box with Hadoop and Hive installed. Easiest way to get it to install one of the Demo VMs or use packages available from Apache Bigtop. Cloudera Demo VMs are available from [Cloudera's website](https://ccp.cloudera.com/display/SUPPORT/Cloudera+QuickStart+VM). You can learn more about Apache Bigtop and install integration test Apache Hadoop and Hive by going to the [project's main page](bigtop.apache.org) and the [project's wiki](https://cwiki.apache.org/confluence/display/BIGTOP/Index).
* We download that data and upload in the hive table


Hive commands
=============
You can create tables for the datasets in Hive. These tables can directly be used in Impala, since Hive and Impala share metadata. Of course, you can create these tables directly in Impala itself too, in case you don't want to use Hive. We will only show you commands for creating tables in Hive only though.

* On hive shell: Create hive table, *flight*:

<pre>
<code>
CREATE TABLE flight(
   year INT,
   month INT,
   day INT,
   day_of_week INT,
   dep_time INT,
   crs_dep_time INT,
   arr_time INT,
   crs_arr_time INT,
   unique_carrier STRING,
   flight_num INT,
   tail_num STRING,
   actual_elapsed_time INT,
   crs_elapsed_time INT,
   air_time INT,
   arr_delay INT,
   dep_delay INT,
   origin STRING,
   dest STRING,
   distance INT,
   taxi_in INT,
   taxi_out INT,
   cancelled INT,
   cancellation_code STRING,
   diverted INT,
   carrier_delay STRING,
   weather_delay STRING,
   nas_delay STRING,
   security_delay STRING,
   late_aircraft_delay STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
</code>
</pre>
<pre>

Some time this can be create problem you can use 

<code>
create table flight(Year int,Month int,DayofMonth int,DayOfWeek int,DepTime int,CRSDepTime int,ArrTime int,CRSArrTime int,UniqueCarrier string,FlightNum int,TailNum string,ActualElapsedTime int,CRSElapsedTime int,AirTime int,ArrDelay int,DepDelay int,Origin string,Dest string,Distance int,TaxiIn int,TaxiOut int,Cancelled int,CancellationCode string,Diverted int,CarrierDelay string,WeatherDelay string,NASDelay string,SecurityDelay string,LateAircraftDelay string) row format delimited fields terminated by ',' stored as textfile; 
</code>

</pre>

* Load the data into the table:

<pre>
<code>
LOAD DATA INPATH '/user/cloudera/flight.csv' overwrite into table flight;
</code>
</pre>

* Ensure the table got created and loaded fine:

<pre>
<code>
SHOW TABLES;

SELECT * FROM flight LIMIT 10; 

</code>
</pre>

* Query the table. Find average arrival delay for all flights departing SFO in January:

<pre>
<code>
SELECT avg(arr_delay) FROM flight WHERE month=1 AND origin='SFO';
</code>
</pre>

* On hive shell: create the airports table

<pre>
<code>
CREATE TABLE airports(
   name STRING,
   country STRING,
   area_code INT,
   code STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';


or 



CREATE TABLE airports(name STRING,country STRING,area_code INT,code STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' stored as textfile;


</code>
</pre>

* Load data into airports table:

<pre>
<code>
LOAD DATA INPATH '/user/cloudera/airport.csv' overwrite into table airports;
</code>
</pre>

* On hive shell, list some rows from the airports table:

<pre>
<code>
SELECT * from airports limit 10;
</code>
</pre>

* On hive shell: run a join query to find the average delay in January 2008 for each airport and to print out the airport's name:

<pre>
<code>
SELECT name,AVG(arr_delay) FROM flight_data f INNER JOIN airports a ON (f.origin=a.code) WHERE month=1 GROUP BY name;
</code>
</pre>

Impala commands
===============

* Now, we will run the same queries on Impala to compare its performance with Hive. No need to re-create these tables in Impala since Hive and Impala share the same metastore. You can directly access them in Impala.

<pre>
<code>
bash> impala-shell
impala> connect localhost;
impala> show tables;
</code>
</pre>

* On Impala shell, query the table. Find average arrival delay for all flights departing SFO in January:

<pre>
<code>
SELECT avg(arr_delay) FROM flight WHERE month=1 AND origin='SFO';
</code>
</pre>

* On Impala shell: run a join query to find the average delay in January 2008 for each airport and to print out the airport's name:

<pre>
<code>

   SELECT name,AVG(arr_delay) FROM flight_data f INNER JOIN airports a ON (f.origin=a.code) WHERE month=1 GROUP BY name;
</code>
</pre>
