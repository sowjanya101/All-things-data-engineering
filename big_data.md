# file formats

Customer Table
Id      Name        Age    Graduate
1       'sowji'     27      True
2       'Anu'       25      True
3       'Manu'      24      False
4       'Vanu'      26      True

## Row based - OLTP 
- In any database data is stored in files/blocks in the background
- In row based, each block may have 1 or more rows stored back to back 

1|'sowji'|27|True|2|'Anu'|25|True -> block1
3|'Manu'|24|False|4|'Vanu'|26|True -> block2

- These are great for operations involving entire row, like insert, delete, update. DML ops

## col based - OLAP 
- The above format is not great for the OLAP use cases where we might only select one or two columns. When this happens the database has to scan all the blocks to retrieve the column we need. We can't skip any blocks.

1|2|3|4|'sowji'|'Anu'|'Manu'|'Vanu' -> block1
27|25|24|26|True|True|False|True -> block2 

- now, if we need id, we can only look at the block1 and skip block2
- But, row reconstruction again with this format takes cpu 

## Hybrid - Parquet
- do both horizontal as well as vertical partitioning 
- parquet uses hybrid strcuture 
- Data is divided into row groups. r1 and r2 in the above table might be part of rowgroup1. This is horizontal partitioning 
- each row group would have column chunks, cc0, cc1... The typical size of row group is 128MB
- row group would also store metadata 
  - row count(RC) of the group  
  - column chunks metadata, like the compression encoding, min, max, distinct values etc
  - and the actual column chunks
- column chunk again contains pages which stores the actual data related to that column. Page is the smallest storage unit and has 1MB size.
- since each page contains data from the same column, compression is efficient.
- dict encoding, where a dict is stored along with the pages within a column chunk. For instance, if there a country column with 50k values, then we can take the distinct values and create a dict. Now, the actual data can be ints that reference dict key. This reduces the storage a lot 
        dict = {
            0: 'India'
            1: 'Australia'
            2: 'US ...
        }
        data = [0|2|1|1|...]
- Further data can be compressed using bitpacking AND RUN LENGYH ENCODING , where if dups are found, instead of storing, say 1 5 times, we store it as [size, value] = [5, 1] compressing it further 
- if there is a query 
    select * from table where id < 3

    RowGroup0 
        id chunk 
            page0 -> 1|2
        name chunk 
            page0 -> 'sowji'|anu
        Age chunk 
            page0 -> 27|25
        graduate chunk 
            page0 ->  True|True
    RowGroup1
        id chunk 
            page0 -> 3|4
        name chunk 
            page0 -> 'manu'|vanu
        Age chunk 
            page0 -> 24|26
        graduate chunk 
            page0 ->  False|True

- now the query skips RowGroup1 entirely and just reads and proccess the 5 1MB  pages in RG0
- if only name is asked - select name from table where id = 1, only 2 pages are read from RG0 
- this is possible because of the RG metadata, and data stored by columns in sep pages
- Also, to make this more efficient, presort the data based on the query columns, so that the range within a row group is ordered. we dont want all the row groups having min value 1 and max value as 10. then we end up scanning all the RGs. similar to zone maps in RS
- entire RG or a col, depending on the select is read based on the metadata. It doesnt further go in to the id column page and check if id 1 exists. (assuming the min and max being 0 and 10, 1 may or maynot be present in this col chunk, but we end up reading the RG anyway)
- dict filtering could help in these cases, if the col has dict encoding, then all the possible unique values are stored in the dict in each col chunk. with the predicate id = 5, it simply looks in the dict and for sure can choose to skip theRG if 5 is not there. we can enable a config for this

### lot of small files, one big file 
- lot of small files, creates a overhead of reading the metadata
- where as enough number of files, parallelize the workload
- with one big file, we lose parallel processing, and we also has one huge footer(metadata) to process, which is not optimized for reads
- 1 Gb partitions are nice
 

* Columnar formats
  - parquet 
  - ORC 

* ROW based 
  - Avro 

- Avro is used for heavy DML operations and it facilitates full schema evolution 
- Mostly used for streaming applicatiosn like Kafka and Druid 

- parquet and ORC being columnar formats are majorly used in analytical uses cases. Both are splittable & supports schema evolution but not as good as Avro in terms of schema evolution
- ORC is used w Hive mostly as it supports good SerDe and heavy compression 
- parquet is used with datalakes mainly, as it supports nested data structures easily

### RLE - run length encoding optimization
- if we know there is a column that has very low cardinality, try sorting the data by the column before writing the df back to parquet, this redcues the physical size of the file
- in the unsorted case, parquet stores data as size, value - 1 2, 1 3, 1 4.. this actuall increases the space
- in sorted case, we will have all the values in one place and actaully store the count 5 1s, 6 2s, greatly reducing the space 
[Link here] http://webcache.googleusercontent.com/search?q=cache:https://subhamkharwal.medium.com/pyspark-optimize-parquet-files-d612df1a601e&strip=0&vwsrc=1&referer=medium-parser


## parquet vs ORC and which is preferred where 
[Link here] http://webcache.googleusercontent.com/search?q=cache:https://medium.com/@diehardankush/why-parquet-vs-orc-an-in-depth-comparison-of-file-formats-5fc3b5fdac2e&strip=0&vwsrc=1&referer=medium-parser

- Both are columnar formats, but used for diff cases
- parquet is for read heavy operations and has nice flexibility for schema evolution
- allows compression at column level
- whereas, ORC is used for write heavy use cases, that involves lot of inserts, updates and deletes. 
- this is possible because ORC provides ACID support 
- hence we mainly see it as default format for Hive, Hive is a warehouse that performs lots of inserts, some updates and deletes
- ORC has good overall compression with lightweight indexes to improve read performance.
- Basically, parquet is used for write once read many usecases, where as ORC when there are regular write, operations


# delta lake 
- isnt it just bringing ACID to parquet
- ORC had ACID and overall indexing, parquet had col level compression, and more flexibility for schema evolution and optimized for reads with metadata for each col chunk in a row group.