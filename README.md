# Data-Engineer-Capstone-Project-Udacity

## Introduction

In this project our goal is to create an ETL pipeline with the 'I94 immigration data' and 'city temperature data' to create a database that is much optimized for immigration event queries. With the use of this database we can answer questions relating to immigration behavior to destination temperature For example, do people prefer immigrating to warmer places than other places?

## Steps followed in the project

* Step 1: Scope the Project and Gather Data
* Step 2: Explore and Assess the Data
* Step 3: Define the Data Model
* Step 4: Run ETL to Model the Data
* Step 5: Complete Project Write Up

### Step 1: Scope the Project and Gather Data

#### Scope
We are going to aggregate I94 immigration data by destination city to form our 1st Dimension table. Next we are going to aggregate city temperature data by city to form the 2nd Dimension table. The two datasets will be joined on the destination city to form the Fact table. The final database is optimized accordingly to query on immigration events to determine if temperature affects the selection of destination cities. 'Apache Spark' will be used to process the data present.

#### Describe and Gather Data
This I94 immigration data comes from the US National Tourism and Trade Office and it is provided in SAS7BDAT format. We will be using the attributes such as,

i94cit = 3 digit code of origin city
i94port = 3 character code of destination USA city
arrdate = arrival date in the USA
i94yr = 4 digit year
i94mon = numeric month
i94mode = 1 digit travel code
depdate = departure date from the USA
i94visa = reason for immigration

The temperature data comes from Kaggle. It is provided in csv format. We will be using attributes such as,

Latitude= latitude
Longitude = longitude
AverageTemperature = average temperature
City = city name
Country = country name

### Step 2
#### Explore the Data and Cleaning Steps¶
As we can see from the dataframe outputs above, many values in i94port column is not normal. so for the I94 immigration data we should drop all the entries where the destination city code i94port is not a valid value such as XXX, 99, etc... which is as described in I94_SAS_Labels_Description.SAS file. And similarly for the temperature data, we should drop all the entries where AverageTemperature is NaN. Also drop all entries with duplicate locations, and then add the i94port of the location in each entry.

### Step 3: Define the Data Model
#### 3.1 Conceptual Data Model
The 1st dimension table will contain events from the I94 immigration data.

The columns below are to be extracted from the immigration dataframe:

i94yr = 4 digit year
i94port = 3 character code of destination city
arrdate = arrival date
i94mode = 1 digit travel code
i94mon = numeric month
i94cit = 3 digit code of origin city
depdate = departure date
i94visa = reason for immigration
The 2nd Dimension table will contain city temperature data.

The columns below are to be extracted from the temperature dataframe:

i94port = 3 character code of destination city (mapped from immigration data during cleanup step)
Latitude= latitude
Longitude = longitude
AverageTemperature = average temperature
City = city name
Country = country name
The Fact table will contain information from the I94 immigration data which is joined with the city temperature data on i94port:

i94yr = 4 digit year
i94port = 3 character code of destination city
arrdate = arrival date
i94mode = 1 digit travel code
i94mon = numeric month
i94cit = 3 digit code of origin city
depdate = departure date
i94visa = reason for immigration
AverageTemperature = average temperature of destination city
The tables that are to be created will be saved to Parquet files format partitioned by the city(i94port).

#### 3.2 Mapping Out Data Pipelines
Described below are the pipeline steps to be executed:

Clean the I94 data as mentioned in step 2 to create a Spark dataframe immigration_df for each month present.
Clean the temperature data as mentioned in step 2 to create a Spark dataframe temp_df.
Create immigration dimension table by selecting relevant columns from immigration_df and write to parquet file format partitioned by i94port
Create temperature dimension table by selecting relevant columns from temp_df and write to parquet file format partitioned by i94port
Create fact table by joining immigration and temperature dimension tables on i94port and write to parquet file format partitioned by i94port

### Step 4: Run Pipelines to Model the Data
#### 4.1 Create the data model¶
We're building the data pipelines to create the data model.

#### 4.2 Data Quality Checks
This data quality check will ensure that there are adequate number of entries in each table.

### Step 5: Complete Project Write Up
* Clearly state the rationale for the choice of tools and technologies for the project.

Ans: I chose Apache Spark since it has the ability to easily handle multiple file formats including SAS, containing large amounts of data. Spark SQL was used to process the large input files into dataframes and was manipulated via standard SQL join operations to form the additional tables.

* Propose how often the data should be updated and why.

Ans: For the best result output the data should be updated monthly in conjunction with the current raw file format.

* Write a description of how you would approach the problem differently under the following scenarios:

    * The data was increased by 100x.

      Ans: For scenario where the data is increased by 100x, we could no longer process the data available as a single batch job. One of the idea would be to do incremental updates using a tool such as Uber's Hudi. We could also consider moving to a cloud service where we can take advantage of services such as AWS EMR or Azure databrick where Spark can be run in cluster mode using a cluster manager such as Yarn.

    * The data populates a dashboard that must be updated on a daily basis by 7am every day.

      Ans: If the data needs to populate a dashboard daily to meet an SLA then we could use a AWS EMR in conjunction with Control-M to setup scheduled jobs or we can use scheduling tool such as Airflow to run the ETL pipeline.

    * The database needed to be accessed by 100+ people.

        Ans: If the database is needed to be accessed by 100+ people, we could consider publishing the parquet files to AWS S3 (object store)/HDFS and giving read access to users that need it. If the users want to run SQL queries on the raw data, we could then consider publishing to HDFS using a tool such as Impala.
