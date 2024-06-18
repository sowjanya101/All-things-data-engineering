# Glue Crawler 
- if the s3 has partitions w just folder name, then if a new partition is added, we cant use "MSCK repair table table_name" on it. The partition needs to be in key=value format for msck to work
- instead we can use "alter table add partiton(partiton_0='Tokyo') PATH 'path to new partition" in Athena
- Glue crawler has options to choose, when the schema changes in the s3
  - update the catalog (If a manual change was made in the catalog like col rename, it would be overwritten w the new data)
  - add new cols only 
  - ignore 

# Glue 
- catalog
- crawler
- Glue studio
  - script, job details, runs, data quality, schedule *todo*

# Glue studio ETL 
- contains notebooks, script editor(python, Spark) and visual ETL 
- Visual ETL has regular mostly used transformations, and we can drag and drop them as needed to build a pipeline, and the script is generated automatically or script can be authored from scratch.
- Job Bookmarks: if the above ETL had a source s3 location, and it is procssed, whenever new files come, only they should be processed. Glue provides job bookmarks to store the state information, so that only inc(new) files are processed
- This also works for the records in the file. Bookmarks can just read the data that has not been processed yet.


# Data ingestion flow could be as following
- a csv file is received in s3, validate the schema (match all columns), if yes, then perform transformation(update col dtypes, agg data and write to parquet)

- create a lambda, that reads a s3 put event and takes header and compares it to the predefined header and based on it send the success/fail json
- create a glue etl job, that converts str to int(dtype changes) where needed, does agg on a col and writes data to s3 in parquet format
- create a step fn, the UI has drag and drop functionality as well, first step as lambda and then glue job
- we need the s3 event to trigger the step func and pass along the event to lambda, for this 
  - turn eventbridge triggers switch in s3 bucket to 'on'
  - create a event rule in eventbridge that goes off on a put s3 event on a required bucket and then targets the prepared step function.


# connecting Athena to powerBI 
- PowerBI is a data visualization tool available as a desktop app. 
- AWS provides downloadable ODBC drivers, once downloaded and installed on the system
- Add a new source in the driver as Athena and provide AWS access key and secret keys (create a user w access to s3 and athena)
- Now this driver is connected to Athena. Open power BI and select the source as ODBC driver and all the tables in Athena shows up in powerBI

# AWS DMS (Database Migration Service)
- migrates data (historical + CDC) between 2 data databases 
- SCT - schema conversion tool helps in migrating the database schemas between two heterogeneous databases (oracle to mysql)
- create a replication instance (ec2), endpoints for source and tgt db and then a replication task (can choose between migrating hist, hist+cdc, just cdc)
- has serverless option as well without creating an instance, specified in the above step.


# AWS data sync
- Syncs data (two way) between onprem servers to (s3, EFS) or other way around, preserving the security and the metadata
- can alos sync between aws (s3, sfs)
- doesnt support for continuous sycn, only have options starting from hourly (have to schedule that)
- supports upto 10GBPs (one data sync agent). can adjust the bandwidths