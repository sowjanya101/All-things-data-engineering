# reading df 
    df = spark.read.format('csv') \
                .option('header', 'true') \ 
                .option('inferschema', 'true') \ 
                .load('path')



# word count from a text file 

    from pyspark.sql.functions import explode, col, split
    path = 'C:\\Users\sowja\OneDrive\Desktop\work\DE_notes\word_count.txt'
    df = spark.read.format('text').load(path)
    df_words = df.withColumn('words', explode(split(col('value'), ' '))).drop('value')
    df_words.groupBy('words').count().show()

1. split(col('value'), ' '): converts the text column to a list 
2. explode: converts an array/map col to rows 
   1. for array creates one col and pivots the array data to rows 
   2. for map creates two cols (key, value) and adds a row for each key value pair.



# maptype and arraytype columns 
## map type
- defined using MapType(keytype, valuetype)
- it does allow multiple keys and values 


        data1 = [
        ({'fname': 'sowjanya', 'lname': 'j'}, 26),
        ({'fname': 'Anusha', 'mname': 'sow', 'lname': 'j', 'age': 12}, 24)
        ]
        schema1 = StructType([
                        StructField('name', MapType(StringType(), StringType())), 
                        StructField('age', IntegerType())
        ]) 
        df1 = spark.createDataFrame(data1, schema1)


        # accessing the data
        df1.select("name.fname", "name.mname", "name.age", "age").show(truncate=False)
        +--------+-----+----+---+
        |fname   |mname|age |age|
        +--------+-----+----+---+
        |sowjanya|NULL |NULL|26 |
        |Anusha  |sow  |12  |24 |
        +--------+-----+----+---+

        # convert col into rows (explode)
        df1.select(explode("name"), 'age').show(truncate=False)
        +-----+--------+---+
        |key  |value   |age|
        +-----+--------+---+
        |fname|sowjanya|26 |
        |lname|j       |26 |
        |fname|Anusha  |24 |
        |lname|j       |24 |
        |mname|sow     |24 |
        |age  |12      |24 |

        # map_keys and map_values provides an array of keys and values respectively.



# providing schema 
    1. text file: providing df.read.format('text').schema(schema_var) doesnt work. the default col name is "value"



# SQL - creating DB and table 
    using spark.sql, below commands can be executed 
    1. create database if not exists test 
    2. show databases 
    3. <!-- This creates a permanent table in Hive, and stores the data in a parquet file -->
        create table if not exists test.cust_tbl (
            id int, 
            name varchar
        ) using parquet  <!-- parquet is default -->
    4. 
        df.createOrReplaceTempView('cust') - saves as temp table
        df.write.saveAsTable('cust', mode='append/overwrite/ignore/error/errorifexists) - saves as permanent table 
            - overwrite: no need for the new schema to be same as existing when overwrite is used 
            - ignore: quietly exits if table is already present 
            - error or errorifexists: Throw an exception if data already exists. (error is default)

    5. insert into test.cust_tbl select * from cust 
    6. can also insert using regular syntax like insert into values()...
    7. truncate table test.cust <!-- cannot use delete -->
    8. Read a spark table: df = spark.read.table('cust'). use this instead of reading file, when the table is bucketed, to make sure metastore is accessed, and the bucketing information is considered.



# date operations in postgresql and spark and python
*todo*
- common date operations
  - string to date, datetime
  - date/dateformat formatting 
  - extracting diff parts of dates like month, year, day etc
  - adding/subtracting from dates

    to_date(dtstr_col, input_format)  <=> datetime.strptime(dt_str, input_format) <=> to_date
    date_format(dt_col, output_format) <=> datetime.strftime(dt, output_format)   <=> to_char


# Transformations and Actions
    1. Actions: 
       1. read
       2. write
       3. collect
       4. show
       5. count
    2. Transformations: craetes a new df
       1. df.count() - action, whereas df.groupBy('city').count() - transformation
       2. filter - narrow 
       3. join, groupby, distinct, orderby - wide transformations
       4. df.repartition(2) - wide transformation
          * syntax: df.repartition(n, *cols). we can either specify n or the cols or both, but one of them should be specified. This is a transformation, so the df needs to be reassigned to new df
       5. df.limit(10) -> returns df after limiting results
       6. regexp_extract(str, pattern, group_to_pick): this is similar to re.match in python/regexp_match in postgresql
    
## collect_list collect_set
-  opp of explode, collects the rows into a list including dups, where as collect_set collects the rows and dedup them
- works with Agg functions and window functions
- same in spark.sql as well 
  
        select collect_list(col) from table
        group by another_col

- example: 

        from pyspark.sql.functions import collect_list

        data = [('James','Java'),
        ('James','Python'),
        ('James','Python'),
        ('Anna','PHP'),
        ('Anna','Javascript'),
        ('Maria','Java'),
        ('Maria','C++'),
        ('James','Scala'),
        ('Anna','PHP'),
        ('Anna','HTML')
        ]


        df = spark.createDataFrame(data, ['name', 'lang'])

        (df.groupBy('name').agg(collect_list('lang').alias('languages'), 
                                collect_set('lang').alias('unique_languages')
                            ).show(truncate=False)
        )

        +-----+-----------------------------+-----------------------+
        |name |languages                    |unique_languages       |
        +-----+-----------------------------+-----------------------+
        |James|[Java, Python, Python, Scala]|[Scala, Java, Python]  |
        |Anna |[PHP, Javascript, PHP, HTML] |[PHP, Javascript, HTML]|
        |Maria|[Java, C++]                  |[Java, C++]            |
        +-----+-----------------------------+-----------------------+





# spark configurations: 
## set config inside spark_program.py 
    <!-- Either do this when creating spark -->
    from pypsark import SparkConf
    conf = SparkConf.conf()
    conf.set('spark.app.name', 'test')
    conf.set('spark.master', 'local[*]')
    spark = SparkSession.builder.conifg(conf=conf).getOrCreate()

    <!-- or do it during runtime -->
    spark.conf.set('spark.sql.shuffle.partitions', 10)


## parameterize setting spark conf 
### spark.conf file in the project dir: 
    [SPARK_APP_CONFIGS]
    spark.app.name=Hello Spark 
    spark.master=local[3]
### create a utils.py file where spark.conf file is read 
    from pyspark import SparkConf 
    import configparser
    def get_spark_config():
        conf = SparkConf()
        config = configparser.ConfigParser()
        config.read('spark.conf') # the conf file with sections created earlier 
        for k, v in config.items('SPARK_APP_CONFIGS'):
            conf.set(k, v)
        return conf
### import the utils file into the main hellospark.py 
    from utils import get_spark_config
    conf = get_spark_config()
    spark = SparkSession.builder.conifg(conf=conf).getOrCreate()

## spark-defaults config file 
    All spark configs are present at SPARK_HOME/conf/spark-defaults.conf

## precedence of configs 
    SparkConf < spark-submit command line < spark-defaults.conf < environment variables (SPARK_HOME/HADOOP_HOME etc)

## Read spark configs 
    get_conf = spark.sparkContext.getConf()
    get_conf.get('spark.app.name')
    get_conf.toDebugString() -> prints all the spark configs

    <!-- 0r -->
    spark.conf.get('spark.sql.shuffle.partitions')

## other config 
    spark.sql.autoBroadcastJoinThreshold: -1 (disables autobroadcast), defaut value is 10MB


# Logging in Spark 
    logging is enabled by log4j. because of the dist architecture of spark, we need this to collect logs and display it in either console/log file 
        1. set up a log4j.properties file in project dir -> contains configs needed, what log level to print(INFO, WARN, ERROR), which dir to write the logs to, across the master and executor containers, log file name etc
        2. Provide this configuration file name to the spark JVM using config in the spark-defaults.conf file 
        3. 


# Pyspark Architecture
    1. Each executor container has some memory and cpu cores alloacted 
    2. Each core gets a partition to work with and overall partitions are divided across executors
    3. action - job, wide transformation- stage. the number of tasks in a stage depends on the partitions.
    4. Initial partitions/tasks (if a file is read), depends on the num of partitions in that file 
    5. After a shuffle operation, the number of partitions depends on the "spark.sql.shuffle.partitions" config
    6. Each task is given to a core in an executor and the core works on that partition.
    7. read - although creates a new job, doesnt load data into driver until an action is hit or the df is cached.
    8. for df.read.format('csv') - if inferschema option is provided, it would create 2 jobs in the UI. one for read and one for inferschema 
    9. spark UI is at localhost:4040 and jobs only live until the app is active for local spark 
    10. serialization: data is often serialized in spark, meaning data is converted into a format that is effecient for data transmition. Data is converted into a byte-stream, thats easier for data transmition. this doesnt involve compression and its often a sep step we need to take. This happens during 
        1. shuffle operations, where data is moved between read and write exchange buffers
        2. UDFs/communication between driver and executor, inferring schemas etc.,
     11. Sometimes, serialized data's size is more.


# Spark APIs
    > SparkSQL > Dataframe > Dataset 
    > catalyst optimizer 
    > RDD API 

    1. RDD API is the low level API and the other 3 at the top are converted to the RDD internally after going through catalyst optimizer
    2. RDD doesn't have catalyst optimizer 
    3. The ease of use is from left to right, but things like debugging, using python for manipulating columns makes case for using DF APIs 
    4. Datasets are only available in Scala
   
   ## RDD API: 
   1. Distributed: data is distributed across the executor cores, but RDDs doesnt have a row col structure and neither a schema. Everything has to be done by the developer, providing the schema. 
   2. Mosty worked on the lambda functions, even the most common functionalities like filter, groupby etc. But, since its the most fundamental structure, its flexible, but optimizations through the catalyst optimizer are not present as spark is not aware of the structure, the code in the lambda functions etc.
   3. Resilient: as in, its fault tolerant. Even when one of the executor fails, spark remember how the RDD is created, and hence the lost partitions are easily recreated.
   4. No out of box support for various mostly used functions is provided, unlike dataframes.
   5. RDD is created using sparkcontext: sc = spark.SparkContext. sc was older entry point for spark programs.

    
   ## catalyst optimizer 
   6. part of spark SQL engine, which also works as a compiler effectively converting the API code to optimized javabyte code
   7. simply using SQL, DF, and dataset APIs makes use of catalyst optimizer. RDD doesn't have this facility.
   8. when the code is submitted to the spark engine using spark-submit, catalyst optimizer takes over in 4 phases 
      1. Analysis: col names, table names, function names etc are resolved.
      2. Logical optimization: applies rule based optimizations like partiton pruning, projection pruning, predicate pushdown on resolved plan.
      3. Physical optimization: generates cost-based optmizations
      4. code generation: chooses best physical plan and generates java byte code


   ## data sources and sinks 
   1. ingestion and data load are usually sep from the data processing which is main capability of the spark. We first get data from sources like JDBC, warehouses, log servers etc and load it into a lake, and not directly read from the source everytime, atleast for batch processing. Few reasons for doing so would be 
      1. load balance: source systems might not be designed to handle the load from this
      2. modularity and maintainability: ingestion itself is a complicated process
      3. security and flexibility
   2. Similarly, this is why we dont usually directly write to the consumers, and instead write the output to the datalake.

# Spark dataframe reader
    - usually the actions are provided as methods, and the config is given as options. method: mode, schema, format etc
    spark.read.format('csv/json/parquet') \
                .schema(schema_var) # optional schema, schema is a method
                .option('header', 'true') \ # only for csv 
                .option('inferSchema', 'true) \ # only for csv 
                .option('path', 'file path') \ 
                .option('mode', 'FAILFAST') # mode is also a method, not an option
                .option('schema', schema) \ # schema is a method and not a config of option, wrong approach for providing schema
                .option('dateFormat', 'M/d/y') \ # only used when schema is provided and we want to say how the date strings are formatted in the file 
                .load()

    1. mode: 
       1. FAILFAST: fails when a record is corrupted 
       2. PERMISSIVE - tries to read as many values as possible, and populated null if an attribute is corrupted. DEFAULT.
       3. DROPMALFORMED - drops malformed rows 
    

## Other options to deal with bad data: 
    .option('badRecordsPath', 'path') 
        -> all the bad records will be moved here 
        -> available out of the box in databricks, but in Apache spark, use custom approaches to move the bad data
    
    .option('columnNameOfCorruptRecord', 'bad_record') 
        -> works in default mode, PERMISSIVE
        -> the column bad_record needs to be specified in the schema as a StringType()
        -> a new col called bad_record is created in the df, where the  corrupt record is moved, for good records, 
        this col remains null
        -> the col with corrupt record becomes null and corrupted record is moved to the new col bad_record
        -> if "abc" is in id col. the df with this setting shows null in id and moves "abc" to bad_record col.
        -> this col can be used to filter the records and move them to locations accordingly.
        -> if the columnNameOfCorruptRecord is not specified, default column nameis _corrupt_Record, but this also needs to be given in the schema as string type

### common datatypes
    IntegerType(), StringType(), DateType(), TimestampType(), ArrayType(), MapType(), StructType(), StructField()
    - Equivalent to datetime.date,datetime.timestamp, list/tuple, dict in python


##  providing schema 
    inferring schema may not always result in correct schema inference. we either need to provide schema externally or use formats such as parquet with well defined schema. (binary format - parquet)
    schema = StructType

    ## programmatic schema
    schema = StructType([
        StructField('name', StringType(), True), 
        StructField('input_dt', DateType(), True)
    ])

    df = spark.read.format('csv')
                    .schema(schema)
                    .option('header', 'true')
                    .option('dateFormat', 'M/d/y')
                    .option('path', path)
                    .load()

    ## DDL string schema
    ddl_schema = "name String, input_dt date"
        df = spark.read.format('json')
                    .schema(schema)
                    .option('dateFormat', 'M/d/y')
                    .option('path', path)
                    .load()

## read csv 
- when a csv file is read without schema, a job is created to read metadata of the file (1 rec)
- when a csv file is read with inferschema option, spark creates 2 jobs, one for reading header, and another for reading few records for inferring schema


# dataframe writer 

    df.write.format('csv')
            .mode('overwrite/append/errorifexists/ignore')
            .option('path', '')
            .save()
    - mode is errrorifexists by default. overwrite, deletes everything and recreates. Hence the new schema doesn't need to be the same as old schema.
    
    - partitioning data 
      - the number of files in the output depends on the number of partitons and each executor core writes a partition in parallel.
      
    - controlling output partitions 
        - df.write.partitonBy(col1, col2).save('path') - allows for usage of cols to repartition with. more meaningful
        - df.repartition(n) and then write. n is a number
        - .option('maxRecordsPerFile', 10000) - controls the number of recs per file
        - df.write.format('json').mode('overwrite').option('path', '').partitionBy(country, city).option('maxRecordsPerFile', 1000).save() -- order doesnt matter after df.write
        - bucketBy(n, col1, col2):
          - only avaiable in spark managed tables as spark also needs to store info on how the table is bucketed, and its only possible w metastore -> df.write.saveAsTable('table')
          - sortBy(col1, col2)
            - sortBy works for bucketBy: df.write.bucketBy(5, country, state).sortBy(countr, state).saveAsTable('customer')
    
# spark tables: 
  * Both managed and unmanaged tables are permananent. Temp tables are created only if createOrReplaceTempView or create temp table syntaxes are used.
  * Need enableHiveSupport to get persistent storage.

## managed: 
  * where metadata and data are managed by spark itself.
  * spark uses in-memory catalog, so when creating managed tables, we need Hivemetastore for persistent metastore. 
        
        spark = SparkSession.builder.master('yarn').appName('test').enableHiveSupport().getOrCreate()

  * using managed tables provides seamless integration with other SQL engines, that works on JDBC/ODBC connections. Ex: HIve metastore can be integrated w Glue catalog and data can be directly viewed in both hive and glue.
  * simply, managed tables are where location is not specified by the user. The tables could be created by using SQL or using saveAsTable syntax
  
        spark.sql("""
            create table if not exists test.cust_tbl(
                id int, 
                name string 
            ) using parquet
        """)
        or 
        df.write.mode('append').saveAsTable('cust_tbl', format='parquet')

## Unmanaged Tables: 
* These are used when user already has a files, but would like to use SQL like queries on it. cataloging enables that feature.
* Unmanaged tables are when user specifies the location to write the files to in both the SQL and saveAsTable case.
  
        spark.sql("""
            create table if not exists test.cust_tbl(
                id int, 
                name string 
            ) using parquet
            LOCATION "c:/user/data/"
        """)
        or 
        df.write.mode('append').saveAsTable('cust_tbl', format='parquet', path="c:/user/data/")


# MISC      
    - get number of partitions in a df 
      - df.rdd.getNumPartitions() 
    
    - get number of records in each partition. 
      - from pyspark.sql.functions import spark_partition_id
      - df.groupBy(spark_partition_id()).count().show()


  -  Note: 
     -  usually the classes we import from the modules(small case) will be in proper CamelCase
        -  ex: from pyspark import SparkSession, SparkConf 
        -  pyspark module, and SparkSession, SparkConf classes
     - the methods/config we use on df writer or df reader or df are in "camelCase" where the first letter of the first word is small, but the rest is normal camelcase
       - ex: df.write.mode, df.read.schema, df.write.partitionBy(), df.write.bucketBy, df.read.options('inferSchema', 'header', 'columnNameOfCorruptRecord', 'badRecordsPath') etc.
       - Note that, when the parameter has only one word, its starts with small case

  - df.forEach(lambda x: x something): action where lambda is applied for each row *todo*
  
## Spark dataframe write method writing many small files

    df.write.partitionBy('country').save(path)
    problem: this is creating multiple small files inside each country partition

    Sol: 
    - df might have many number of partitions, and hence while writing, that many tasks must have written data parallelly to respective partition. 
    - repartition the data by country, so that only n partitions are created, n being the unique country values, and we will get only one file per partition in the output

    df.repartition('country').write.partitionBy('country').save(path)

    - this could potentially cause a large file if there is a skewed partition. Make use of .maxRecordsPerFile to control it

    df.repartition('country').write.option('maxRecordsPerFile', 10000).partitionBy('country').save(path)

## coalesce is pushed up and changes the parallelism of prev transformations
*todo*

## would spark consider the file partitioning in s3, when join is applied?
*todo*



    
# TRANSFORMATIONS

## ROW
    from pyspark.sql.types import Row 
    data = [Row(1, 'sowj'), Row(2, 'an')]
    df = spark.createDataFrame(data, schema=schema)

    df = spark.range(1, 10) # creates spark dataframe with col id

    df.collect() -> returns list of row objects
    for row in df.collect():
        row['id'] -> accessing attr 

## COLUMN 
    I think df functions are designed to operate on entire cols, where as when python comes into picture, each attribute has to be taken out for processing.
    df.withColumn(col1, spark_func(col(col1))) -> spark func on a col

    * can be accessed as col strings or col objects
    * col string ex: 
      * df.select("id", "name")
    * col object ex: 
      * df.select(col("id"), column("name"), df."city") -- all of these
    
    select: 
    - takes columns, but if there is any exp that needs to be resolved. For ex, to_date(travel_dt, 'MM/dd/yyyy') as travel_dt, we need to resolve it into column first. It can be done using either 
    - expr: 
      - can convert expressions and returns column, which is what select expects
      - df.select("id", expr("to_date(travel_dt, 'MM/dd/yyyy') as travel_dt"))
    - column objects: 
      - this is by using spark functions taken from pyspark.sql.functions
      - df.select("id", to_date("travel_dt", 'MM/dd/yyyy').alias("travel_dt"))
    - refer dataframe, columns, built-in functions in the docs


        from pyspark.sql import Row
        #Creating the pysql row
        row = Row(field1=12345, field2=0.0123, field3=u'Last Field')

        #Convert to python dict
        temp = row.asDict()

        #Do whatever you want to the dict. Like adding a new field or etc.
        temp["field4"] = "it worked!"

        # Save or output the row to a pyspark rdd
        output = Row(**temp)



## UDF 
* steps for UDF function: (The UDF will not be registered in the catalog)
  1. create a UDF function
  2. register the UDF. registering would help driver serialize the UDF and send it to executors  
  3. use it for col transformation 
        `def add_one(index_col): 
            return index_col + 1
        add_one_udf = udf(add_one, IntegerType())
        df.withColumn('new_index', add_one_udf("index"))`

* steps for UDF function using SQL: (The UDF will be registered in the catalog)
  1. create a UDF function
  2. register the UDF. 
  3. use it for col transformation 
        `def add_one(index_col): 
            return index_col + 1
        add_one_udf = udf(add_one, IntegerType())
        df.withColumn('new_index', add_one_udf("index"))`

        spark.udf.register("plus_one_udf", plus_one, LongType())
        spark.sql("select id, plus_one_udf(id) as id2 from tb").show()

    ** withColumn doesn't take column strings, hence needs to be resolved using expr

**spark.catalog.listFunctions()**: lists all the functions in the catalog. 


## MISC transformations
    expr is used to resolve expressions inside below methods to columns
        * select 
        * withColumn
        * sort
        * OrderBy
        * groupBy().agg(expr(), expr())..
    it resolves an expr to a column. for multiple cols, use multiple 
    * where takes an expr, and not col. So expr isn't needed in where clause
    * As long as we are applying a transformation, we get a df, so we can apply anything. Ex: after join drop can be chained, followed by select, followed by withColumn etc,.

            
    df.groupBy('country', 'city').count().orderBy('count', expr('country desc') ).show()
    df.groupBy('country', 'city').count().sort(expr('country desc')).show()


## translate 
    replaces the given substr in the col with replace string

    from pyspark.sql.functions import translate 
    df.withColumn('s', translate(lit('ada d'), 'ad', '1')).show()
        ==> '11 '
    df.withColumn('s', translate(lit('abcd'), 'ad', '1')).show()
        ==> '1bc'


----
*use all these in withColumn*
# monotonially increasing id 
    monotonically_increasing_id()

# case when then 
    expr('case when then else end').cast(IntegerType()) 
            .drop("col1", "col2")
            .dropDuplicates(["col1", "col2"]) # list 
            .sort("col1", expr("col2 desc"))
            .orderBy("col1", expr("col2 desc"))

# fillna, dropna 
- df.dropna(how='any/all', thresh=, susbset=[])
- df.fillna(value, subset=[])
- df.fillna({'name':'unknown', 'salary': 0})  


# WINDOW functions 
    # running sum of total by weeknumber for each country
    from pyspark.sql import Window

    running_total_window = (Window.partitionBy('country')
                            .orderBy('weeknumber')
                            .rowsBetween(Window.unboundedPreceding, Window.currentRow))
    # rowsbetween default is the above
    # below considers current row and above two rows
    running_total_window = (Window.partitionBy('country')
                            .orderBy('weeknumber')
                            .rowsBetween(-2, Window.currentRow))    

    # Window.unboundedFollowing - other possible value

    (dfagg.withColumn('running_total', 
                            sum('total').over(running_total_window)).show())

    # sum, row_number etc are pyspark.sql.functions

    # win_spec = Window.orderBy(desc('sal')) -- define the desc order using desc method


# JOIN 
    - left 
    - right 
    - full
    - left_anti
    - right_anti
    - inner

    customer - left df 
    product - right df 

    join_exp = customer.prod_id == product.prod_id
    customer.join(product, join_exp, 'inner'). 
        select('*') - shows all cols, including prod_id from both the cols.

    other join types are specified as below:
    * outer
    * left 
    * right 

    * spark stores the col names with internal ids, when we ask for *, all of them are returned, but when we ask for a single ambiguous col, like prod_id, in the analysis phase, spark tries to convert it into id, and thats when it throws error.
  

## selecting ambiguous columns
* rename the column beforehand in one of the tables with "withColumnRenamed"
    
        `customer = customer.withColumnRenamed('prod_id, 'product_id')`
* drop the column from one of the tables in the join iteself
        
        `customer.join(product, join_exp, 'inner')
                .drop(customer.prod_id)
                .select("prod_id", "qty")`
* select the col with alias
        
        `.select(customer.prod_id)`


## shuffle sort-merge join internals

* imagine two dfs are joined and each has 2 partitions.
* step1: df1 is read. one job, one stage, two tasks because of 2 partitons. same for df2
* step2: imagine two executors. can configure with master(local[2]) when creating a spark session. Each has 1 parition of df1 and 1 partition of df2. The join is on id
* step3: Each df's partition's id is mapped and written to a buffer. Shuffling is done. 
* step4: set spark.sql.shuffle.partitions to 2, so that the shuffle resulst in 2 partitions for each df 
* step5: each partition from df1 and df2 with same keys are read into one executor. we will have 2 executors in total.
* step6: sort-merge join will be applied.
* step7: join will have 3 stages. df1 shuffled, df2 shuffled, and 3rd stage where df1 and df2 are joined.
* the result of shuffle in a join operation also results in n tasks, each task contaning a partition of table1 and a partition of table2. n is determined by spark.sql.shuffle.partitions
* Both the tables have 3 partitions each and spark.sql.shuffle.partitions=3
  * stage1: 3 tasks for reading table1
  * stage2: 3 tasks for reading table2
  * stage3: 3 tasks that join table1 and table2

## steps to make joins efficient 
* reduce the size of the dfs to be shuffled
  - can be done by applying pre-filters and pre-aggregations
* increase the parallelism and make use of the big cluster
  - parallelism is controlled, by following three 
        1. number of executors 
        2. spark.sql.shuffle.partitions
        3. number of unique keys 
  - Even if we have 100 executors, but shuffle.partitiosn is only 20, we get max parallelism of 20, and even if shuff.partitions are 20, if number of unique keys used for join is 10, we get max parallelism of 10. we should look for ways to increase parallelism.
*  avoid data skew. while joining sales data on product, and if one of the product is very fast moving compared to other. the join has to wait, until all partitions are done executing.


## broadcast join
if one of the tables is small enough to fit into an executor.

        from pyspark.sql.functions import broadcast
        df1.join(broadcast(df2), join_expr, 'inner')
        df1.join(df2.hint('broadcast'), join_expr, 'inner')
when we broadcast, shuffle doesnt happen. The data from the broadcasted dataset is directly read into all the executors that needs it I guess.
- only works for equi joins (df.a == df1.b, not the > or lt ops in join cond)
- doesnt work for full outer join 

## bucket join 
- use hivemetastore: .enableHiveSupport()
- 
prebucket the dfs using the join key, when writing them to spark tables. Bucketing is only allowed for spark tables. Assuming 3 executors.
    df1.write.bucketBy("3", "id").saveAsTable('db.sales')
    df2.write.bucketBy("3", "id").saveAsTable('db.product')

    # now the data from each df is bucketed into 3 partitions and colocated on the executors
    df1.join(df2, df1.id==df2.id, "inner")

    # this is supposed to perform sort-merge join without shuffle, owing to data being collocated from the two dfs

    # bucket join has lot of gotchas, for it to perform correctly. while these are some of the considerations for it to avoid shuffle, these doesn't ensure that shuffle will be skipped. on the flip side, shuffle may also be skipped even without some of these: 
        # both tables needs to be bucketed on the same cols into same number of buckets
        # the spark.sql.shuffle.partitions needs to same as number of buckets. otherwise, spark might end up repartitioning the bucketed table again.
        # the data needs to be bucketed when written to a table. and it needs to be sorted using sortBy with same cols used in bucketBy. otherwise there is a chance to produce more files than intended.
        # read the table instead of reading the files, this is to ensure we are reading the metadata related to buckets.

    # One-side shuffle-free join: 
        - if only one of the tables are bucketed, and the shuffle partitions are same or less than the buckets, spark will only shuffle the unbucketed table. If shuffle partitions are more than the buckets, either change that config or repartition the un-bucketed table into same number of buckets.

                df1 - 20 buckets, df2 - non bucketed
                df1.join(df2.repartition(20), id)

    # bucketed data is read into the cluster in the same format its bucketed, hence avoids the shuffle. i think same thing happens when broadcast join is used.
    # bucket pruning is also available 
    # bucketing also helps w aggregations


# AQE 
By default disabled. To enable-

        set spark.sql.adaptive.enabled to True
        *There are other 4 configs that help fine tune the AQE*

AQE helps with: 
    * Dynamically coalesce shuffle partitions 
    * Dynamically switch join strategies (to broadcast, shuffle hash)
    * Dynamically optimize skew joins 

For all these individual tasks, there are sep configs to play with along w enabling the AQE

### Dynamically coalesce shuffle partitions:  
By default, spark.sql.shuffle.partitions is 200 and whenever a shuffle happens, the result is split into 200 partitions. If the dataset had only 5 unqiue keys, even then we would have 200 partitions, out of which 195 are empty. This means, the spark driver schedules and creates 200 tasks causing resource wastage. To avoid this, during shuffle stage, spark dynamically coalesces the partitions, and this includes 
    - removing empty partitions and 
    - coalescing smaller partitions, so that relatively same size partitions are created, resulting in all the tasks finishing up around the same timeframe.
*while reading from the write exchange buffer, spark gets access to info on most upto date partitions, like unique values, # of rows in each partition etc*
    
### dynamically switching join strategies 
#### switch to broadcast hash join 
If one of the tables is smaller
Ex query: 

            select * from big_table1 join big_table2 on col 
            where big_table2.id = "20"

- Ideally this results in shuffle sort-merge join. it  would have 3 stages: 
    - reading 2 tables (2 statges) and writing to write exchange
    - reading data from these 2 exachanges into read exchange and collocate data from both tables with same keys, then perform sort and then merge
- note: the table size needs to be less than spark.sql.autoBroadcastJoinThreshold to be considered for broadcastjoin by spark automatically. default value is 10
- this would happen automatically, if the table is read from somewhere and spark has latest statistics of the same
- Lets say after the filter big_table2 becomes very small table. But we might not know this beforehand if the big_table2 is created after a series of df transformations, and there is no way to tell it was going to be a small table
- usually, spark recomputes the statistics at the shuffle stages, so of AQE is enabled, it would know that big_table2 is actually small and then switches the join to broadcasthash join.
- By this point, the two tables would have been shuffled, but because of the switch, the now small_table2 would be broadcasted again, but the sort is skipped as its not sort-merge anymore
- AQE respects autoBroadcastJoinThreshold.
#### switch to shuffle hash join 
If both the tables turn out to be small after spark calculates the statistics in shuffle stage, SPARK switches to hash join instead of sort-merge. hash join meaning, one side of the tables key is hashed and it probes the other side for the same hash.
- for ShuffleHashJoin and BroadcaseHashJoin, the hash table should fit into the memory, otherwise it causes OOM exceptions.

### dynamically handle join skew
- after partitioning two tables involved in the join, if one of the partition on one side of table is skewed, spark breaks it up and the corresponding partition from the other table is duplicated across the tasks that has the breaked up partitions from table1.


# Dyanmic partition pruning 
- usually if we specify a filter(predicate) condition on a column, on which the file is already partitioned on, then we pushdown the predicate, to only read those partitions that matches the predicate and prune (ignore) the other partitions. This is for direct filtering though.
- Ex: file1 is partitioned on date, and if we specify a query: 

        # predicate pushdown
        from pyspark.sql.functions import sum, count
        df = spark.read.format('parquet').option('path', "C:/Users/sowja/Downloads/customer_country/").load().where('country="canada"')
        df.agg(count('index').alias('some')).collect()

- But in the cases, where the filter is not directly applied on the partition column, DPP comes to the rescue. 
- one example could be as follows

            orders_df.join(date_dim, 'date_id', 'inner). 
                    .where(date_dim.year=='2021' && date_dim.month=='12')
                    .show()

- here, we know to pick only the partitions from order_df(partitioned by date) with 2021-12 data, but the query ends up having to pick everything, as its not a direct filter on the partitioned table order_df
- the DPP can help, if date_dim is broadcasted. looking at this broadcasted date_dim in the broadcast exchange, a filter cond is injected into the read query of the order_df, and the unnecessary partitions are pruned. Now, for all the executors that has the parts of order_df, the date_dim is broadcasted and the join happens
- the DPP is enabled by default, but for it to work, below conditions has to be met 
    - fact and dim like tables, meaning, one table small enough to be broadcasted
    - the partitions of fact table and the filter on dim table should make sense, like the example above


# cache and persist
* Cache and persist are same, except persist is more customizable about how data can be cached.
* to uncache - df.unpersist()
* df.cache() - default is memory and disk in deserialized format 
* df.persist(StorageLevel(
                useDisk,  
                useMemory, 
                useOffHeap, 
                deserialized, 
                replication=1
            ))
* data is stored in serialized format on disk, which takes less space, and when its loaded into memory, it has to be deserialized into java objects. So when caching happens, there is an option to choose whether data needs to be stored in serialized/deserialized formats in memory. we dont have these options for offheap and disk. If serialized is chosen, additional CPU capacity is needed to deserialize it when working on that data.
* replication is the replication factor.
* cache is a lazy operation. It also loads only those partitions that are needed. If we use take(10), it loads only one partition and gives 10 records back. 

        df = spark.range(1, 100000).repartition(10).cache()




# Databricks specific commands: 
1. df.createGlobalTempview('df_temp')
2. use magic command: %sql in the cell and start running the queries directly. 
    %sql 
    select * from global_temp.df_temp; 

    %fs head dbfs_path 
    previews data in dbfs path 


# Repartition 
- when repeated filters are happening on a col
- when there is data skew, and to get even partitions for parallel processing 

        df.repartition(numPartitions, *cols) -- uses hashing for repartitioning 
        df.repartitionByRange(numPartition, *cols) -- uses sampling, and divide the data by ranges (0-10 in one part etc)

# coalesce 
- use it to reduce the num of partitions. use this for reducing and not repartitioning.
- combines local partitions first, and hence results may have uneven partitions
        
        df.coalesce(10)


# SPARK UI 
## storage: 
  - shows details of partitons and their size, if a df is cached


# JOIN HINTS 
- broadcast 
- shuffle_merge *todo*
- shuffle_hash
- shuffle_replicate_NL



# broadcasting a variable
- a variable can be used without broadcasting as well, and its called closure. A broadcasted variable is serialized to every "node" that needs has a worker that needs it and caches it there. Its lazy. 
- A closure, is serialized and sent to every executor/worker that needs it.
- If we had 20 nodes and 50 executors running on it, the closure takes up more netwotk usage as the driver has to serialize it 50 times.

    closure_var = spark.read.csv('path').rdd.collectAsMap() --gives dict 
    broad_var = spark.sparkContext.broadcast(closure_var)

    def func(col): 
        return broad_var.get(col)

    func_udf = udf(func)
    df.withColumn('value', func_udf('col'))


# Accumulator
- is a mutable global variable, that can be used as a counter, or to keep track of the sum from across the tasks, its on driver.
- the ex shows that accumulator is applied from a transformation, but it can also be applied from an action.
- usually, when a task fails, it is retried, in this case there is a chance for the accumulator to be incremented twice. So using an accumulator from inside an action like "foreach" has gaurenteed accurate results.

        null_vals = spark.sparkContext.accumulator(0)
        -- adding to the null_valls accumulator 
        null_vals.add(1)
        -- accesisng the value 
        null_vals.value

ex: a col contains non-int values that needs to be replaced with nulls. Along with it, we need to count the number of nulls as well. If we trnasform and then take count of nulls, that results in exchange, instead we can use accumuator

        def replace_nonints(val): 
            if isinstance(val, int):
                return val 
            else: 
                null_vals.add(1)
                return None

        replace_nonints_udf = udf(replace_nonints)
        df.withColumn('final', replace_nonints_udf(col)
        )
        print(null_vals.value)

# speculative execution 
- if there is a slow running task that takes more time than other tasks in that stage, the stage has to wait until all the tasks are completed.  
- sometimes a task may run slow because of hardware issues, or memory crunch, or dataskew
- if its due to hardware issue, the task can be benifitted from starting another task on a new executor/core, and it may finish faster. This is what happens when spark speculative execution is enabled
- spark.speculation = True (Default: False)
- spark just identifies slow running tasks and starts new tasks, which basically means more resource usage, hence its off by default, and spark doesnt check if the reason for slowness is hardware, or other issues
- spark also provides, other config to fine tune the speculation.

# Resource allocation 
    - between applications
    - within an application
# DynamicAllocation (between apps)
- spark cluster can run multiple applications at once
- 2 allocation strategies are available: 
  - static allocation (Default)
  - Dynamic allocation
- the default allocation strategy is as follows: app is submitted to the cluster, cluster manager starts the App Master, AM immediately asks for the executors specified in the spark-submit command, and these are given to the AM. The AM master will release these executors, only once the app is finished. If this is a big app, and requests all the resouces, and a smaller app comes at later point, the small one has to wait until the big app is completed.
- Ex: cluster with 100 executors (4 cores, 32GB).
    - App had 4 stages, and first stage needed 400 cores
    - second, third and fourth only needed 100, 200, 200 respectively. 
    - The app master requests for 100 executors from RM and doesnt release executors after the first stage. To release them we have to enable dynamic allocation

        spark.dynamicAllocation.enabled=True
        spark.dynamicAllocation.shuffleTracking.enabled=True  (keeps executors with shuffle exchanges from active tasks from being released)
        spark.dynamicAllocation.executorIdleTimeout=60s  (after 60s of inactivity, an executor is released)
        spark.dynamicAllocation.schedulerBacklogTimeout=1s (if the scheduler doesnt find an empty core within 1s, it requests for executors)

# spark scheduler (resource allocation  between jobs within an application)
- usually, the jobs(actions) are executed in sequence. Ex: two tables are read sep and count action is being taken on them sep, even though they dont have any relation, the reading and count of job2 happens after job1
  - df1 = spark.read.csv() >> df1.count()
  - df2 = spark.read.csv() >> df2.count()
- we can use multi-threading from to make these jobs parallel, by creating parallel threads
- Now, these jobs are further divided into stages and tasks and they need to scheduled. This is where spark.scheduler.mode comes into picture.

                spark.scheduler.mode = FIFO (default, all tasks from job1 are given resources first and then the tasks from job2)
                spark.scheduler.mode = FAIR (does round-robin resource allocation for tasks into CPU slots)


# multi-threading (python lib)
*todo*

        import threading 

        def func(file):
            spark.read.csv(file).count() 

        file_locs = [filepath1, filepath2]
        for file_loc in file_locs: 
            threads = threading.Thread(target=func, args=(file_loc))

        for job in threads: 
            job.start()

        -- both the jobs are run in parallel now


# memory allocation and management 
*todo*


# projection pruning while reading from a parquet file 


    df = spark.read.format('csv').option('header', 'true').option('inferSchema', 'true').option('path', "C:\\Users\sowja\OneDrive\Desktop\work\DE_notes\project_youtube_analytics\IN_youtube_trending_data.csv").load()



# interview questions 

## probem1: 

        movieId,title,genres
        1,Toy Story (1995), Adventure|Animation|Children|Comedy
        1,Toy Story (1995),Adventure|Drama
        2,Toy Story 2 (2020),Adventure|Fantasy
        3,Jumanji 2 (2001),Adventure|Children|Fantasy
        4,Grumpier Old Men (1995),Comedy|Romance
        5,Waiting to Exhale (1995),Comedy|Drama|Romance
        6,Father of the Bride (1995) Part II,Comedy
        7,Heat (1995),Fantasy
        
        OUTPUT: ( Make sure that duplicates are removed)
        Movie_key, movieId,title,year, genres
        1,1, Toy Story , 1995, Adventure
        2, 1, Toy Story , 1995, Animation
        3, 1, Toy Story , 1995, Children
        4, 1, Toy Story , 1995, Comedy
        5, 1,Toy Story , 1995,Drama
        6, 2, Toy Story 2, 2020, Adventure
        7, 2, Toy Story 2, 2020, Fantasy
        8, 3, Jumanji 2, 2001, Adventure
        9, 3, Jumanji 2, 2001, Children
        10, 3, Jumanji 2, 2001, Fantasy

---

## solution1: 
        from pyspark.sql  import Window
        from pyspark.sql.functions import row_number, regexp_extract, explode

        # create DF 
        movie_df = spark.read.format('csv').option('header', 'True').load('C:\\Users\sowja\OneDrive\Desktop\work\DE_notes\data_files\movie_genre_csv.csv')
        movie_df.show()

        # transformation
        window_spec = Window.orderBy('movieId')
        (
                movie_df
                .withColumn('title_new', regexp_extract('title', r'([a-zA-Z 0-9]+) \((\d+)\)', 1))
                .withColumn('year', regexp_extract('title', r'([a-zA-Z 0-9]+) \((\d+)\)', 2))
                .drop('title')
                .withColumn('genres', explode(split('genres', '\|')) )  # escapinging the pipe as the split reads it as "or" opeartor of regex
                .dropDuplicates()
                .withColumn('Movie_key', row_number().over(window_spec))
            .show(truncate=False)
        )




# deltatables 
- deltatable is a framework that allows ACID transactions on files
- with this, we can perform insert, update and delete ops on a file, which means this allows for type2 on a file without ever loading to a db
- lakehouse architecture allows for reading data from sources, creating facts and dims including type2 ops and then expose the data for analytics.
- simply, a warehouse with files in the background.
- azure synapse analytics is one such service that provides these capabilities to build a lakehouse on object storage.
- it has storage integrations to connect to the sources, pyspark/sql and delta lake tables to process the data and then provides endpoints for analytics
- databricks follows madallion arch (bronze, silver, gold layers.)

# merge in spark 
- spark has merge syntax, w the help of delta tables, this is possible
- General logic using merge: 
  * source: has an new rec(insert), update and a ignore record
  * insert: will be processed when PK doesnt exist in the target
  * ignore: will be processed when PK matches, but there is no change in dims(hash of tables is also same as the latest record - active_flag=Y)
  * update: 
    1. the current active rec needs to updated with a stop date and a new rec needs to be inserted with active_flag=Y
    2. source=new data, target=warehouse
    3. wherever there is an update, we need a rec to update the target ts and a rec to insert into source 
    
        # create a source_temp 
        source.join(target.where(active_flag=Y, src.pk=tgt.pk and src.hk!=tgt.hk, 'left'), src.pk=tgt.pk and src.hk!=tgt.hk).select(tgt.pk as pk, src.*)
        UNION 
        source.join(target.where(active_flag=Y), src.pk=tgt.pk and src.hk!=tgt.hk, 'inner').select(null as pk, src.*)

        - now the source_temp has 3 rows(for the update row where same PK but diff HK), and only one record for the new records, and one row for ignore rec
        - merge on matching pks (between tgt and the source_temp)
        - when matched(1 update row with pk not null) and hks are not equal then update the target end ts to current-timestamp. pk with same hash would be ignored
        - when not matched (1 update rec w null pk, and a new rec), insert into tgt 
   
        merge into TargetProducts t
        using 
            (
                select src.productid as pk, src.* from SourceProducts src
                union 
                select null as pk, src.* 
                from SourceProducts src 
                join TargetProducts tgt 
                on src.productid = tgt.productid 
            )s 
        on s.pk = t.productid and s.price != t.price and t.end_date = '9999-12-31'  (we can also just put pk in the on cond and include the otehr criteria in when clauses)
        when matched 
        then update set end_date = current_date
        when not matched 
        then insert values(s.productid, s.productname, s.price, current_date, '9999-12-31')


        merge into TargetProducts t
        using 
            (
                select src.productid as pk, src.* from SourceProducts src
                union 
                select null as pk, src.* 
                from SourceProducts src 
                join TargetProducts tgt 
                on src.productid = tgt.productid 
            )s 
        on s.pk = t.productid 
        when matched and s.price != t.price and t.end_date = '9999-12-31'  
        then update set end_date = current_date
        when not matched 
        then insert values(s.productid, s.productname, s.price, current_date, '9999-12-31')

        # some dbs provide delete option as well 
        merge into myTable as target
        using @tmpTable as source
            on  ( source.GroundID   = target.GroundID )
            and ( source.GroupID    = target.GroupID )
        when
            not matched by target  # PK not present in tgt
            then
                insert ( GroundID, GroupID, AcceptingReservations )
                values
                (
                    source.GroundID,
                    source.GroupID,
                    source.AcceptingReservations
                )
        -- If there is a row that matches, update values;
        when matched
            then
                update set
                    target.AcceptingReservations = source.AcceptingReservations
        -- If they do not match, delete for that GroundID only;
        when
            not matched by source  # PK not present on the source side
            and target.GroundID = @GroundID
                then
                    delete;



# salting technique examples

# hash agg over sort aggergates

# how to handle slow running spark job 
- partition skew (enable AQE, spark3 has it enabled by default)
- considerably more number number of partitions than executor cores - increase the number of executors, provide more resources
- considerably less number of partitions than available executor cores - adjust the partitions accordingly if possible 
- optimize join stratgies (broadcast > shuffle hash > shuffle sort merge join) 
- hash agg over sort aggergates **what is this**
- consider caching in right places 
- consider using right file formats like parquet, to prune over cols 
- lookout for disk spills - provide right memory 

# mapreduce
- heavy disk IOs as map phase writes to disk and reduce phase reads from disk, and the API was complex, and was only available in Java

# spark session
- sparksession provides all the contexts needed like hive context, spark context etc. No need for creating each context individually

# lazy evaluation 
- because spark is lazy, it can push predicates to the top and more intensive shuffles down. This is possible because spark waits until an action and come up with an optimized plan
- in terms of reading a partitioned parquet file, spark can pushdown the predicate and select only cols needed, instead of reading the entire file right when its read, because it waits for an action
- gives scope for skipping operations that are not needed
- lineage is how an RDD is created from another RDD when a transformation is called, DAG is created by spark with series of transformations to be performed


# pivot and stack 
*todo*

# coming up with cluster config 
- start with the size of data that needs to be processed and then the size of each partition
- By default, we can go with 128MB block size
- Ex: to process 25GB of data, where each partiton is of 128MB 
- Num partitions= 
        25*1024 = 25600MB/128MB = 200 partitions 
- Each partiton is processed by a core, Hence number of cores = 200
- Num of executors
        - ideally between 2 to 5 cores for each exec, we can go with 4 
        - 200/4 = 50 executors
- Executor memory 
        - Memory for each core is usually 4x*partiton size = 512MB
        - Each executor has 4cores*mem for each core = 4*512MB = 2GB
- Total Memory 
        - 50 executors*2GB = 100GB 
- so, to process 25GB of data, we need a cluster size of 100GB, with 50 executors, 4 cores each 
- driver memory: 
  - If there are no collect operations on driver (ideally we wont do it in prod), we can have driver memory=executor mem = 2GB or twice executor mem 
  - In prod, we mostly write results back to s3 or something, which doesnt involve data collected at driver 



# deoitte Notes
Locality level:
-  PROCESS LOCAL - the task  is being executed on the same exececutor
-  NODE LOCAL - the task is being exececuted on same node but on diff exececutor
-  RACK LOCAL - the task is being exececuted on an executor in diff node 
Lower locality level lesser the latency 

- Ex:
For an app. a driver started on node x and executor on node y (only one executor)
>> df.count(): 
- count is taken across all partitions (since one executor all partitions will be on it) - PROCESS LOCAL 
	- the counts are written to an exchange from all partitions 
	- all the counts are read from exchange and aggregated - NODE LOCAL 
	
	
>> df.collect()
	- all records are collected to driver and in client mode, driver is mostly on a node diff than executor 
		making this a RACK_LOCAL or atleast NODE_LOCAL process if the executor is on same node as driver.
		

TASKS Division:
-> Num. of partitions = no. of tasks = each task to a core 
-> if there are more no. of tasks than cores, first few tasks gets executed, results get collected and same goes on

Coalesce Vs repartition:
-> Coalesce forces change in spark.sql.shuffle.partitions making forcing transformations before it to run in that many tasks mentioned by coalesce.
-> Due to less num of partitions, (if there are transformations before coalesce), the full parallel processing isn't reached and hence could slow down process. If we are using coalesce just to repartition after all Tx are performed, it is faster.
-> The same also could cause OOM, as it forces creating less no. of tasks there by creating less no. of partitions and hence each partition would be big.
	Ex: Processing a file of 30M recs which is cached in memory. On top of it there are very few trasnformations before writing it to s3, and used coalesce(8) here. 
	- Instead of 19200 tasks which it has created without using coalesce, it forced to create only 8 tasks ,making each task process huge data.
	- since each task/partition is sent to a core, a core wouldn't have that much mem, and then spark engine makes use of DISK space.
	- but due to big size, the disk space ran out as well.
	- without coalesce, it created 19200 tasks, having 50 cores, all these tasks acheieve a parallelism factor of 50 and data fit in-memory with liittle spill to the disk as mem associated to all 50 cores were used, as opposed to only using mem associated with 8 cores adn then spilling to disk.
-> This does reduce data shuffle, by only shuffling data from additional partitions, which also causes uneven partition sizes

-> Repartition does a full shuffle to get required num  of parts, and preserves spark.sql.shuffle.partitions and does a shuffle after that.


Memory assignment behaviour & task processing behaviour:
>> s3 file is being read and taken count 
- spark reads the file and creates x num of tasks if there are x partitions and each task takes count for each file and write to an exchange buffer, next stage reads from exchange and agg. the final count.
>> If a file is cached, it it fits in memory, it stores else spills to disk cahce storage level, for df's default is mem+disk 
>> If its only in-memory level, we get an OOM on executor if mem of asscoiated executors assigned for an application runs out.
>> If disk is also full, then spark terminates the exececutor.


Start of an application:
>> launches an AM  and in the same node 
>> launches a driver container 
>> driver (spark session) talks to cluster manager to see where it can launch executor container 
>> launches executor containers and driver sends all the artifacts (jars, py files) to executors 
>> driver creates a lineage graph for each action and divides that in stages by shuffle operations, which is further divided into tasks based on no. of data partitions 
>> each task is assigned to an executor and core takes up and completes it 


Check partitions:
df.rdd.getNumPartitions()


DISTINCT():
- lets say we have 5 partitions
- DISTINT first does hasaggregate(dedup) within each parition (reduces the data to be shuffled)
- shuffles the data so that duplicate values go into same hash exchange 
- then does the (dedup) hash aggregation on the shuffled data as well


df_prev.distinct('NK') join df_curr.distinct('NK') on NK:
- df_prev is shuffled to get DISTINCT values (above logic)
- df_curr is shuffled to get DISTINCt values 
- these two are again shuffled so that join data is collocated 

Remove 2 shuffles and make it 1:
- since join and distinct is on same CDE, use repartition('NK') shuffle #1
- now distinct happens without any shuffle, and (if we remove broadcast hint), join would also happen without any shuffle
(but for this to happen, we can't directly join, but need to join by mentioning just the columns like in case2)
Expl - case1:
df_prev.join(df_curr, (df_prev.NK=df_curr.NK), left)
case2:
df_prev.join(df_curr, NK, left)
This changes from 2 shuffles to 1 shuffle for each df in the join

Using Hive tables instead of Spectrum speeds up queries as there would be multiple files and hence that many partitions in HIve table, but for reading from DataFramereader.jdbc we need a column that has define min and max values based on which we cause option("numPartitions", x), "lowerBound", "upperbound", predicate etc.




Imagine 2 datasources - a local parquet file vs a remote table on a database. Spark will read the file itself as compared to Spark will depend on the database to read data. The plan for reading a local parquet file doesn't have asterisk vs the one reading from remote database does.

AQE:
Coalescing post-shuffle partitions and removing extra tasks on empty paritions:

spark.sql.adaptive.enabled true
spark.sql.adaptive.coalescePartitions.enabled true 
spark.sql.adaptive.coalescePartitions.parallelismFirst false
spark.sql.adaptive.advisoryPartitionSizeInBytes 64MB

spark_submit="spark-submit --master yarn --deploy-mode client --num-executors 12 --executor-cores 5 --executor-memory 18g --driver-memory 18g --conf spark.sql.broadcastTimeout=2000  --conf spark.sql.autoBroadcastJoinThreshold=50M --conf spark.yarn.maxAppAttempt=1 --conf spark.executor.memoryOverhead=2g --conf spark.dynamicAllocation.enabled=true --conf spark.dynamicAllocation.shuffleTracking.enabled=true --conf spark.dynamicAllocation.minExecutors=4 "


/usr/lib/spark/bin/spark-submit --deploy-mode cluster --master yarn --name LTCG_staff_effective_history_curated_append --jars s3://ta-individual-datalake-tst-codedeployment/onedatalake/pyspark/jars/postgresql-42.2.18.jar --driver-memory 18g --driver-cores 5 --conf spark.dynamicAllocation.enabled=true --conf spark.dynamicAllocation.shuffleTracking.enabled=true --conf spark.dynamicAllocation.minExecutors=50 --conf spark.yarn.submit.waitAppCompletion=true --conf spark.yarn.maxAppAttempt=1 --conf spark.executor.memoryOverhead=2g



---------------------------------------------------------------------
Nodes - 2 or 4
64Vcores 
256GB memory 

cores - 5 per executor
total cores - (64-4) leaving one core per executor to run Hadoop Deamon process, Secondary name node, Task tracker, Job tracker, node manager etc
num-executors - (60/5) - 12 - leaving 1 executor out for ApplicationManager to run 
executor-memory - (256/12) >> (21.3) >> leaving max(384mb, 7% of exec mem) to executor-memoryOverhead (1.5g) >> (21.3 - 1.5) >>  19.8g 

12 5 19g


#spark_submit="spark-submit --master yarn --deploy-mode client --num-executors 12 --executor-cores 5 --executor-memory 19g --driver-memory 8g --conf "spark.sql.broadcastTimeout=2000" --conf "spark.yarn.maxAppAttempt=1" "

Prod config:
4 nodes - each with 32VCPU 128 GB
vcores - 128
memory - 512GB 

total cores - 124 
num-executors - 23
memory - 18g




NEW calculation: 
master - 1
core - either 1 or 2
task - either 2 or 3

core capacity - 
128 GiB of Memory, 32 vCPUs

task capacity - 
256 GiB of Memory, 64 vCPUs

--num-executors 25 --executor-cores 10 --executor-memory 20g --driver-memory 18g


total cores - 96
total memory - 384

Lowest node count with same capacity: (3 task nodes)
for 3 nodes: each node can be assumed as -
32 cores and 
128GiB memory 


lets say 5 cores per executor  and leaving 1 core per node, we get 93 cores
--num-executors -> (93 cores / 5 cores per executor) - 18 executors and leaving one executor for app. manager -> 17 executors
--executor-cores -> 5
--executor-memory -> (384gb/18 executors) -> 21Gb per executor (-2 gb overhead) => 18gb per executor 
ideal method: (based on no. of nodes) ->  
--memory-overhead -> (7% of 21gb) -> 2GB

17 executors , with 18gb each and 5 cores each 

3 nodes 
48 cores 
192GB

5 cores -> 9 exececutors -> 26gb per executor (-3gb overhead) -> 23gb

2 units = 16Vcore and 64GB
8 units 
64vcores 
256GB

5 cores -> 12 executors -> 18g 





--METHOD IN BLOGS:
leave 1 core per node, we get 31 cores per node 
total cores = 31* 3 -> 93

no. of executors = 93/5(ideal cores for good HDFS throughput) = 18.6 -> 18
Leaving 1 executor for ApplicationManager => --num-executors = 17


no. of executors per node = 31/5 = 6.2 -> 6 executors
memory per executor = 128/6 = 21GB 
Counting off heap overhead = 7% of 21GB = 2GB. So, actual --executor-memory = 21 - 2 = 18GB

This also results in 17 executors with 18Gb each and 5 cores each with 2Gb of overhead memory for each exec.

--CONSIDERING 5 nodes:
for 5 nodes: each node can be assumed as -
19 cores and 
76 GiB memory 

leaving 1 core per node - we get 18 cores per node 
total cores = 18*5 = 90

no. of executors = 90/5 = 18 executors 
Leaving 1 executor for AM = 17 executors 

no. of executors per node = 18/5 = 3.6 
mem. per executor = 76/3.6 = 21 GB
Counting off heap overhead = 7% of 21GB = 2GB. So, actual --executor-memory = 21 - 2 = 18GB

This also results in 17 executors with 18Gb each and 5 cores each with 2Gb of overhead memory




FOR 16GiB and 4VCPUS for a task and core node cluster:
nodes: 2
memory: 32Gb
cores: 8

cores - leaving a core per node, 3 cores per node - 6 cores in total 
executors - 


sh /application/financedw/financedwgdq/scripts/run_gdqscript.sh -s ltcghybrid -t claimspayment -b 11606 -c 2022-03-11 -j TA-INDIVIDUAL-FINDW-FINANCEDW-LTCGHYBRID-GDQ  -e tst  -p financedw

spark_submit="spark-submit --master yarn --deploy-mode client --conf "spark.dynamicAllocation.enabled=true" "


Dynamic allocation enabled in spark app submit and emr property spark.emr.maximizeResourceAllocation was set to true:

properties in spark  (/etc/spark/conf/spark-defaults.conf)
spark.executor.instances         2
spark.executor.cores             4
spark.driver.memory              10647M
spark.executor.memory            9833M
spark.default.parallelism        16
spark.emr.maximizeResourceAllocation true
spark.emr.default.executor.memory 9833M
spark.emr.default.executor.cores 4
spark.emr.default.executor.instances 2
spark.dynamicAllocation.enabled true 

even the spark history server had the same properties
spark.dynamicAllocation.enabled	true
spark.emr.default.executor.cores	4
spark.emr.default.executor.instances	2
spark.emr.default.executor.memory	9833M
spark.emr.maximizeResourceAllocation	true

- creating EMR without MaximizeresourceAllocation, and will run with dynamicAllocation = true in spark-submit

spark.dynamicAllocation.enabled	true
spark.emr.default.executor.cores	4
spark.emr.default.executor.memory	9486M

----------------------------------------------------------------------------------------------------------
OLD calculations:

Points:
only task and core nodes are considered in the resource manager.
master node specs is entirely out of equation for planning the capacity of your apps.


http://10-128-83-203:8088/
(try changing these specs, works fine, check redshift connections and also retry logic)

in EMR: Memory - 584GB, Vcores - 160

num of nodes: (1-64G-16V, 4(2-128G-32V, 2-64G-16V), 6(4-128G-32V, 2-64G-16V)) --> 10
num of cores: 16 + (64 + 32) + (32 + 128) --> 272

(96 Vcores, 360G, 3nodes) --this was avaialble on resource manager (12 nodes were decommissioned)
each node had - 32Vcore, 128G (these were core and task nodes, excluding master node), and the given specs worked just fine.

10  nodes
27 cores per node (270 cores in total)
64G per node (640G)

tiny executore (one exececutor per core):
--num-executors 272
--num-cores 1
--executor-memory 4G

fat exececutor (one executor per node):
--num-executors 10
--num-cores 27
--executor-memory 64G

balanced:
--num-executors 25
--num-cores 10
--executor-memory 20g


8 nodes
48 cores per node 
185G per node (1480G)

calcs:
-> leaving one core per node for deamon process (48-1) -> 47 cores per node
in total -> 47*8(nodes) -> 376 cores available in overall cluster

-> lets consider 10 cores per executor -> no. of executors = 376(total cores)/10(cores per exececutor) => 37
-> Leaving 1 executor for ApplicationManager => --num-executors = 30

-> memory for each exececutor => 1400G/30 -> 40g


--num-executors 25
--num-cores 10
--executor-memory 25g

8 nodes
48 cores per node 
185G per node ()


--driver-memory 18g 
--driver-cores 5 
--executor-cores 5 
--executor-memory 18g 
--conf spark.default.parallelism=110




16 nodes - 16 vCore, 64 GiB memory
20 nodes -> 32 vcores, 256 GiB memory

36 nodes  
24 cores per node 
170 G per node 

--num-executors 50
--num-cores 15
--executo-memory 80g


96 Vcores, 360G, 3nodes


spark_submit="spark-submit --master yarn --deploy-mode client --num-executors 25 --executor-cores 10 --executor-memory 20g --driver-memory 18g --conf "spark-sql-broadcastTimeout=2000" --conf "spark-yarn-maxAppAttempt=1" --conf "spark-executor-memoryOverhead=2g" --driver-class-path /usr/share/aws/redshift/jdbc/redshift-jdbc41-1-2-37-1061-jar --jars /usr/share/aws/redshift/jdbc/redshift-jdbc41-1-2-37-1061-jar,/apps/oracle/ojdbc6-jar,/apps/postgres/postgresql-42-2-23-jar"

spark-submit --master yarn --deploy-mode client --num-executors 50 --executor-cores 15 --executor-memory 80g --driver-memory 30g --conf "spark-sql-broadcastTimeout=2000" --conf "spark-yarn-maxAppAttempt=1" --conf "spark-executor-memoryOverhead=2g" --driver-class-path /usr/share/aws/redshift/jdbc/redshift-jdbc41-1-2-37-1061-jar --jars /usr/share/aws/redshift/jdbc/redshift-jdbc41-1-2-37-1061-jar,/apps/oracle/ojdbc6-jar,/apps/postgres/postgresql-42-2-23-jar gdq_dynamic-py -s ltcg -t party -b 101 -c 2021-08-31




 --driver-memory 18g --driver-cores 5 --executor-cores 5 --executor-memory 18g --conf spark-default-parallelism=110 --conf spark-yarn-submit-waitAppCompletion=true --conf spark-yarn-maxAppAttempt=1 --conf spark-executor-memoryOverhead=2g
 
 
 
import boto3
client = boto3-client('emr') 
response = client-cancel_steps(
    ClusterId='j-21DJVIXDQ2X39',
    StepIds=a
)
 
response = client-list_steps(
    ClusterId='j-21DJVIXDQ2X39',
    StepStates=[
        'PENDING', 'RUNNING',
    ]
)

http://10-128-80-162:9870/
application_1632916944306_0037 --> pool?


HDFS Name Node	http://10-128-80-162:9870/
Tez UI	http://10-128-80-162:8080/tez-ui
Spark History Server	http://10-128-80-162:18080/
Ganglia	http://10-128-80-162/ganglia/
Hue	http://10-128-80-162:8888/
Resource Manager	http://10-128-80-162:8088/


------------------------------------------
cost estimations
------------------------------------------
	198858837
	5889746


198M recs - 5M recs
2.1G - 147.9 MB

storage costs for one source system (7 domains)- 
per month - 
60G - 4500MB (4.5G)
1.38 USD - 0.11 USD
per year - 
16.56 USD vs 1.32USD
	
storage costs for 10 source system (7 domains)- 
per year - 
160 USD vs 10 USD


EMR cost (an hour daily)
master - 1- m5.4xlarge
core - 4- m5.8xlarge
task - 6- m5.8xlarge
36.32 USD per month

Ec2:
master 1 - 30 USD (m5.4xlarge)
core - 1 - 60  USD (m5.8xlarge)
task - 3 - 90 USD (m5.4xlarge)
180 USD per month

Total EMR monthly cost - 200 USD

max config 
master 1 -  30 USD (m5.4xlarge)
core - 4 - 240 USD (m5.8xlarge)
task - 6 - 360 USD (m5.8xlarge)
630 USD per month 

Total EMR monthly cost - 660 USD (on full capacity)

avg would be - 430USD per month/per ss

general:
4x - 1 - 30.21 
8x - 1- 60

master - 1- m5.4xlarge
core - 4- m5.4xlarge
task - 6- m5.4xlarge
64.24 USD per month 


on avg EMR cost could be - 
per month - 76.105 USD
per year - 913.26 USD 
per 10 sources per year - 9130 USD



cost of 60GB per month of KVP - 
s3 cost -> 1.38 USD 
Tiered price for: 60 GB
60 GB x 0.0230000000 USD = 1.38 USD
Total tier cost = 1.3800 USD (S3 Standard storage cost)
210 GET requests in a month x 0.0000004 USD per request = 0.0001 USD (S3 Standard GET requests cost)
1.38 USD + 0.0001 USD = 1.38 USD (Total S3 Standard Storage, data requests, S3 select cost)
S3 Standard cost (monthly): 1.38 USD

spectrum -> 0.29 USD per month (60 GB)

cost of 5GB per month of KVP - 



s3 storage costs
emr cost
spectrum
ra3 std cost
ra3 storage cost

cost for opt1 v opt2

opt3
directly copy data into findw from IDL (49 tables), along RDM with RDM tables


