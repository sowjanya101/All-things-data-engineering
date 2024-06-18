# date dimensions
- If the fact tables stores facts at only date level granularity, having a date dim and using the datekey as FK in the fact table is recommended. 
- If we decide to keep 10 years of dates in the date dim, date dim would contain 365*10=3650 records. 
- datedim would ideally store all the info related to a date, like year, month, dow, doy, quarter etc
- the reason for using date dim instead of directly storing date in the fact table is, its easier computationally to look up the year from the datedim than actually using extract(year from datr) on each col.

## creating datedim in postgres

        select 
            '2021-12-31'::date + (seq.day || ' day')::interval
        from generate_series(0, 3649) as seq(day); 

        generate_series(0, 3649) - generates numbers from 0 to 3649
        we get the output of 10 years worth of dates from 2021-12-31 to 2031-12-31. Total 3650 records


# Time dimension 
- If the event/fact is noted at hour/min/second granulatiy, basically if a ts is needed in the fact table. It is advised to create a time dim along with the date dim 
- if only minute level details are needed, we would be having 24*60=1440 records, and if second level details are also needed, then 24*60*60 records needs to be created in the time dim
- The fact table would then store, a datekey and timekey FKs from both these tables. Sometimes, event_ts would also be stored in the fact table along with these for simpler queries.


## create time dim in postgres 

### second level detail 

        SELECT '00:00:00'::time + (sequence.second || ' second')::interval AS second
        FROM generate_series(0,86399) AS sequence(second)
        order BY sequence.second

### minute level detail 

        SELECT '00:00:00'::time + (sequence.minute || ' minute')::interval AS minute
        FROM generate_series(0,1439) AS sequence(minute)
        order BY sequence.minute

These works as the base inner queries that generates sequence of days, minutes, seconds. Other extractions are made on top of these and a date and dim dimensions are created.

### sample extraction 

        SELECT minute, 
            extract(hour from datum.minute) as hour_of_day, -- 0 to 23
            CASE WHEN to_char(datum.minute, 'hh24:mi') BETWEEN '00:00' AND '11:59' THEN 'AM'
            else 'PM' END  AS AM_PM
        FROM 
        (SELECT '00:00:00'::time + (sequence.minute || ' minute')::interval AS minute
            FROM generate_series(0,1439) AS sequence(minute)
        ) datum
        order by hour_of_day 


# degeneare dim 
- when the dim doesn't have any more attributes to create a sep table for it, we simply keep the dim in fact table

# CDC (Changge data capture)
- Capturing the changes from relational database system thats supporting your website into warehouse. can be done using below - 
  1. using simple incremental SELECT queries hitting the RDBMS and picking inc data, often identified by a timestamp col. 
     - select * from source_db where ts_col > last_successful_pull_ts; 
     - this is straightforward and works well for batch cases where we dont need to capture changes in real-time, but comes w few disadvantages
       * if its not possible having a ts col in the source
       * the updates made in between will be lost, ex: if a col is updated twice in a day, the first update will be overwritten and will not be passed onto the DW.
       * deletes will not be captured, unless its a soft delete (a col marked as 'delete' or something in source system)

  2. Using database triggers(inbuilt) that perform some action when an action like insert/delete/update is performed on the source table of interest 
     - all updates will be captured, but this strategy may not work on a db that has high freq inserts/updates, as it may incur latencies and the affects the performance of the db
     - this captures all the changes though
     
  3. another way is using WAL(write ahead logs), which is again a feature of most of the dbs, and all the changes are captured in a log file, and reading these files doesnt really affect the performance of DB.



# PROJECTS 

## Clickstream Analysis 

#### source
Adobe analytics:
Adobe Analytics is a popular web analytics platform used to track user interactions on websites and mobile apps. It manages websites and provides clickstream data of the websites and mobile applications. clickstream data like user clicks, page views, scrolls etc

Dimesions that can be grabbed from Adobe analytics: 
* Evars: (Event variables)
    * Purpose: Primarily used to capture data associated with user interactions or events on your website or app.
    * Persistence: By default, eVars persist across user sessions (visits) until a new value is set or a specific expiration time is reached. You can configure custom expiration times for eVars in Report Suite settings.
* Props: (Hit properties)
  * Purpose: Designed to capture data specific to a single hit (page view or event) on your website or app.
  * Persistence: Props do not persist beyond the hit they are set on. The value is only available for that specific user interaction.

This project had one big fact table that included dims as well. 
Reasons?
  - because click data has large volumes, and joins suffer, and anyway DWs with their columnar storage comes cheap?
  - what happens to type2 on dims, does it even exist in this concept?

### recordlinkage lib 
- used to identify whether two or more record belong to the same entity (like person)
- this is used when there is no identifier to identify that the records belong to the same person/org 
- Eg: when cookies are used, we cant use cookieid to identify the same person, in this case, we compare other attributes like dob, gender, address etc to identify if the records belong to the same person 
- recordlinkage: accepts two pandas dfs
- we have to create blocks, as its computationally intensive to compare all the records. Block is a set of records that has same attribute. Ex: we can create blocks of records that have same surname and then compare records within each block on multiple attributes like dob, gender, address and specify whether an attribute has to exactly match or a certain percent of match is okay. 
- the lib would compare and give us the matching records

## youtube analytics - sample 

source: 
  - trending_vids.csv
  - trending_vids_ref.json

  > trending_vids_ref.json had a nested json structure. coverted that nested json to df cols using pandas json_denormalize. 

          import pandas as pd
          df = pd.read_json('C:\\Users\sowja\OneDrive\Desktop\work\DE_notes\project_youtube_analytics\CA_category_id.json')
          pd.json_normalize(df['items'])

          > write this function in Lambda, and set up s3 trigger on the lambda 
          > Lambda "configuration" section provides an option for creating env variables, which can be accessed in the func using os.envrion['var_name']
          > Lamba allows to create a "s3 put event" test config while testing the lambda
          > lambda has a lot of predefined libs that can be attached as layers with most frequently used functionalities. ex: awswrangler

  > trending_vids.csv has incorrect datatypes, ex: category_id is labelled as str, where as the same col in ref file is int. Every time we do analytics we'd end up casting the col, which creates an overhead. the goal of the ETL is to do these on every day increments so that its not throttle the analytics queries.

          > create a glue visual ETL with source s3 and target s3
          > provide the target catalog to be created and opt the choice on how the catalog needs to be updated when there are schema changes *todo*
          > also provide the partition keys for the target s3
          > enable job bookmark, so that only the new unprocessed files coming into source s3 are processed every time the job is triggered
          > can add addtional steps for dedup further joining with the ref data (transformation)

  > Quicksight: 
      import data into qs using "datasets" and create "analysis" on top of that. analysis are the dashboards.






jdbc
redshift arch
clickstreAM


# columnar vs row based storage in Databases
- OLTP query pattern: usually selects all the rows, inserts an entire rows
- OLAP query pattern: select specific cols, agg by specific cols 
- OLTP stores "rows" in blocks that typically range from 2kB to 32KB 
- OLAP dbs stores "cols" in blocks (1MB blocks in Redshift)
EX: 

      dept	empid	salary
      HR	    1	    100
      OTHER	  2	    200

This table would be stored as - 
* in OLTP row based blocks:

        HR|1|100 => block1
        OTHER|2|200 => block2

* in OLAP col based blocks:

        HR|OTHER => block1
        1|2 => block2
        100|200 => block3

- based on the olap query pattern, if few cols are selected only few of the overall blocks can be scanned, hence only a smaller percentage of data will be moved to memory, leaving the rest out, along w the increased compression, owing to same data type within a block


# Dist keys - Redshift
Goals:
  - parallelize the workloads (make use of all the nodes available)
  - co-located joins (reduce the need to scan all the nodes in the cluster. If the data is colocated w the help of dist key, data related to same join keys from both the joining tabes sit in the same node, thus reducing the need to scan multiple nodes when join happens)
- use a col thats frequently joined on, so they are colocated and key based distribution is a great use case for joining two fact tables
- if a dim tables is smaller (<3M records), using dist style all helps. Dist style all keeps a copy of table in each node.
- having high cardinality for dist keys is ideal as the data would get evenly distributed without skews, hence evenly distributing the workload.
DIST STYLE: 
  - auto (RS decides based on the query patterns)
  - key 
  - all (table is distributed across each node, not each slice)
  - even 

# sort keys - Redshift 
Goals: 
  - prune blocks that doesn't contain data we are querying for, and reduce the data to be scanned 
- RS has data stored in cols in 1MB blocks, and these blocks have zone maps(metadata like min and max value of the block). These blocks are well compressed owing to the columnar storage, i.e same col data type, hence more compression
- the cols that are more frequently queried on are great candidates for the sort keys, while the col w low cardinality being the first one in the list of sort keys 
- sort keys are like indexes in oltp systems
- compound sort key: sorting level is in the order of keys provided, eg: deptid, empid. sorts by dept first and then empid within each dept.
  - using a select with direct filter on the empid doesnt take sort keys into consideration. it has to provide filters by dept first and then emp, or just the dept for the sort keys to work
- interleaved sort keys: gives equal weightage to all the keys provided in the sort key. Each col provided in the sort key is sorted individually and the rows are logically linked.
  - now, sort keys takes effect even when we use empid individually in the where clause

## sort and dist key recommendations
### DIST 
- In most of the cases, distributing the fact table on date columns is effective as the fact tables are mostly aggregated/filtered on date ranges, and co-locating the data helps parallelize workloads. And in typical scenarios, dims are usually small and using "ALL" DIST STYLE makes for effecient queries, as in when joined with fact table.
- In few cases, where one of the dim table is larger, and fact and this dim table are frequently joined, then distributing both the fact and the larger dim on their join key and the other dims using "ALL" style would be effecient.
### SORT
- if fact table is already distributed on date, and it's also mostly queried on the range of dates, sorting it on the date itself makes the queries prune over unnecessary blocks of data. since facts are typically larger compared to dims, we can make this consideration as opposed to below 
- On the other hand, in the instance of a fact table and a large dim table that is frequently joined on - sorting the fact and dim on this join key makes the join effecient by converting it to a merge join.
- For dims, we may consider sorting on the cols that are typically used as predicates.
- These are general cases, but the actual considerations depends on the query patterns. 
- For the query engine to apply the optimizations provided, the statistics needs to be up-to-date and regularly updating the stats helps a great deal.


- choose fact and one dim table(bigger one amongst all the dims) and dist them  on the join key. define the sort key as the same.
- ideally, the distkey should have high cardinality
- if possible, dist the othe dims with ALL (if the size is relatively small)


# Ingestion into REdshift - COPY 
Copy is the most optimal way to ingest data, RS nodes are further divided into node slices, and if there is only one file, one slice will load it. So, divide the file(partition) the file into multiples of # slices, this will increase the parallelism.

# VACUUM
Redshift follows logical deletes and logical updates approach. 
- soft deletes 
- for update, it soft deletes the existing rec and inserts a new record
- vacuum clears up the space, and resorts the table globally
- this creates empty spaces, and the sort order is messed up, so in LTCG, after ETL, we used to run the vacuum and analyze to cleanup/sort data and to update statistics

# ANALYZE
- updates the table statistics for optimal query plans
- done automatically, but running manually after a specific ingestion helps
- usually all the auto activities like analyze and vacuum happens during no workloads, so performing them manually right after ingestion/some activity can help, and we dont have to wait for the cluster maintanenance time slot

# WLM, SQA 
- use WLM for setting up query monitoring rules, setting concurrency, setting up queues and their priority, concurrency scaling(if set up auto scales the cluster during spike)
- short query acceleration for  prioritizing short queries
- can turn on automatic worklod management, where ML would set up queues and other stuff


# resize the EMR cluster?
*todo*


# vertical partitioning vs horizontal partitioning
- vertical partitioning is a technique where some columns are moved to new tables. Each table contains the same number of rows but fewer columns. This feels closer to what redshift does through columnar storage
- horizontal partitioning or sharding is where number of rows is split across multiple nodes
 

# facts 
# lucidchart for data modeling
- new rows doesnt necessarily need to get appended to the target table. Depending on the use case, we can have new rows that would negate/subtract the original fact measure, if it was incorrect or in an actual case where a product is returned and amount has to be refunded etc. 
- For the negation case, tha amount(which is the fact in point) would be correct once facts are summed across diff rows
- There could also be cases where facts are replaced if incorrect or some other reasons 

# date_lists
- is a concept used in fact tables in cases where there are multiple rows of facts for a certain user
- an ex would be, if FB stored user clicks data per day in a fact table.
- Now, querying this huge table would get so inefficient considering the number of users and the event volume
- here, a concept called date_list can be used, where instead of rows per user with click count per day, we use one row per user and a col which would store a map of date as key and click count as value for that day 

  user_id   first_date        last_date         date_list 
    1       1st June 2021     5th June 2021     {'01-06-2021': 5, '02-06-2021': 6...}

- this makes retrieving data very efficient


# how to check the file size in python

    # for general files 
    import os
    os.path.getsize(file_path)

    # for s3 files
    boto3_client.head_object(bucket, key) - provides the metadata of the file, including the filesize (content length)


# trigger a step func from lambda
    boto3 client start_execution

# read and write to s3 in csv and parquet formats
# convert a string to csv in lambda


           
#  Optimizing S3 Data Storage
## optimizing for data scans 
  1. partitioning 
  2. columnar vs row format 

### partitioning 
* partitoning data on the most frequently queried columns reduces the data to be scanned by a query 
* In most cases, an ideal candidate for partiton keys would be date 

### columnar vs row 
* using columnar format helps only pick the columns needed, which means scan only the blocks that contain the columns we need as opposed to scanning all the blocks(in row format) and then picking the columns we need
* parquet is an ideal candidate for storing columnar data.

## optimizing for IO operations
  1. compression 
  2. choosing optimal file size 
   
### compression 
* with parquet, the default recommended compression is snappy, this reduces the data size transferred over network. Another advantage is reduced footprint on the storage
* but this would take additional compute to uncompress the data.

### choosing optimal file size 
* between large number of small files, and small number of large files, choosing the right size improves the throughput and optimizes the number of network calls made
  



# partitioning and bucketing 
- we can partition data by most frequently queried cols, in this case we usually choose the partitions w low cardinality
- lets say we partition on country and we get 3 partitions with 400MB each. This is still lot of data and needs further filtering down to get to what we want to query
- if there is no further partition key w low cardinality, and we only have most frequently queried cols w high cardinlity, then going w bucketing helps. 
- bucketBy(age, 3)
- now, all the ages 1 to 100 will be bucketed into 3 files inside each country partition. This gives healthy file size and also parallelizes workloads, reduces the data to be scanned
- s3 supports bucketing as well

# difference between hive and spark bucketing 
- hive:
  * has one file for one bucket 
  * sorts the data inside a bucket(==file)
  * this is because, each bucket is written by a reducer and requires shuffle
- spark: 
  * writes one or more files for one bucket 
  * doesnt sort the data inside a bucket 
  * spark task write to a file, but metadata identifies which bucket it belongs to. hence shuffling on write doesnt happen

# issues when migrating from hive to spark because of these differences


# difference between ORC and Parquet  *todo*
https://www.youtube.com/watch?v=NZLrJmjoXw8



# data mesh vs data fabric 
- data mesh is decentralized where data fabric is controlled centrally
- the underlying concept is same, democratizing data across multiple teams to benefit custom applications


# micro partitioning



# transformations 
- phone number editing 


# early arriving fact/late arriving dim 
- if facts arrive early, before dim had that NK, the fact would end up with no FK
- an order transaction is recieved for a product that has not been loaded into the product dim yet
- two solutions: 
    - store the facts with no rec in dims in another temp table and process then when the dim arrives 
    - keep a rec in dim with 0 SK and use it as FK-SK for facts that doesnt have any dims yet, so its not missed in the joins and update it later


# data model 
Ref Link [Link here] https://www.youtube.com/watch?v=fRs0H47fhaM&list=PLTjcBkvRBqGFSVUtFhYu6hu5yrrAoMVg4&index=4
## conceptual
- identify the type of queries that needs to be answered w this data model
- based on this, arrive at the facts, dims and their realtion
- when building dims, consider normalization when necessary based on usecase
- one dim can have multiple hierarchies, and everything doesn't need to be a linear hierarchy 
- For example, in case of a library system
  - we can have books, magazines to be lent 
  - each book/magazine has multiple parts/issues
  - each part/issue could have mutiple copies available 
  - each book has authors and publishers. For each part/issue there could be author/publisher 
  - usually, when a person borrows a book, its going to be a copy 
  ### Design of these 
  - product_line(one) - (one to many)product(one) - (one to many)product_item  
  - Author(one) - (one to many)product
  - Publisher(one) - (one to many)product


## warehouse design considerations
partitioning of fact: 
  - what is the data retention of warehouse
  - Ex:3 years = 365*3 =~ 1000 (say there is another partition country (5 countries)) 
  - apprx partitions would be 1000*5 = 5k partitions
  - usually partitions can go upto a 100,000 and it'd be okay 

agg tables
SKs