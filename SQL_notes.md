# sql order of execution
    * FROM/JOIN 
    * WHERE
    * GROUP BY 
    * HAVING 
    * SELECT (if aggregates are metioned, them and then window functions)
    * WINDOW
    * ORDER BY 
    * OFFSET 
    * LIMIT
    * FETCH
  
  *Even though the statement like group by 1,2 works, the order of execution is still the same.*

# group by and window in one sql 
-> group by aggregates works first and then the select clause. Hence, we can use aggregations inside a window function.
-> imagine the result of group by query without window function. Window is applied on top of that result set 

ex1: ranking the sum of payments of each user 

      select 
        user_id, 
        sum(payment) as total, 
        rank() over(order by sum(payment)) -> window is applied after group by
      from payments 
      group by user_id

  This gives the total_payment for each user and then applies rank over it.


ex2: include one more col in the group by - country 

      select 
        country, 
        user_id, 
        sum(payment) as total, 
        rank() over(order by sum(payment)), 
        rank() over(paritition by country order by sum(payment))
      group by
      from payments 
      group by user_id, country; 

  Whatever window function is run, it's on top of the grouped result. so, imagine if there is no window function and just group by, the result will be showing total_payment for each country, user combination. On this result set, rank is applied. First window doesn't have any partitions, and the second one is partitioned by country.

# OFFSET 
skips the result of select "column" by n, where n is specified by offset and, the "column" is determined by the order by clause. Offset is used in combination with ORDER BY. OFFSET is run after select and order by 

      SELECT DISTINCT salary FROM Employee ORDER BY salary DESC OFFSET 1

      Employee =
      | id | salary |
      | -- | ------ |
      | 3  | 1      |
      | 2  | 1      |
      | 1  | 0      |
      | 4  | 1      |
      | 5  | 2      |

      Output
      | salary |
      | ------ |
      | 1      |
      | 0      |

    - 3 distinct values in the salary
    - order by desc 2, 1, 0 
    - OFFSET 1 skips "2" and returns 1,0
    - this result is skipping distinct values as we selected distinct values, otherwise, the output would show 1, 1, 1, 0


# FETCH 

      FTECH NEXT n ROWS ONLY 

fetch is used in combination with the offset, used to specify how many rows to fetch after offset

      SELECT DISTINCT salary FROM Employee ORDER BY salary DESC OFFSET 1 FETCH NEXT 1 ROWS ONLY  

      Output
      | salary |
      | ------ |
      | 1      |


# EXISTS 
checks if a sql returns something 

      SELECT 
        CASE WHEN EXISTS (SELECT salary FROM employee) THEN 'yes' 
        ELSE 'no' END 
      FROM another_table 
      LIMIT 1 (without this, yes is printed n times, where n is the num of rows in another table)

  - if employee was empty to begin with, the above statement doesn't return any value if its selected on "employee". Below wouldn't return any rows 

        SELECT 
          CASE WHEN EXISTS (SELECT salary FROM employee) THEN 'yes' 
          ELSE 'no' END 
        FROM employee --empty table 

- if we select anything from a table that is empty, even if its simple select 1 from empty_table. we wouldn't get any results. on the other hand, if we select 1 from table_w_n_recs, we print 1 n times in the output.
- we can use a trick, to get null from an empty table 

        SELECT 
          max(salary) -- returns null
        FROM empty_table


# Length 
- postgres provides length(str_col) and returns length of string

      select tweet_id      
      from Tweets
      where length(content) > 15


# POSTGRES SQL DATE oprators 

## EXTRACT 
    select current_date, current_timestamp, 
      extract(day from  current_date),
      extract(month from  current_date),
      extract(year from  current_date),
      extract(quarter from  current_date),
      extract(week from  current_date)
      extract(doy from  current_date), -- day of year
      extract(dow from  current_date)   -- day of week
      

## INTERVAL FUNCTION 
Add/Subtract year, month, week, day, hour, minute, second to the existing date using this function 

    SELECT 
      *,
      admitted_date + INTERVAL '1 day', 
      admitted_date + INTERVAL '1 week', 
      admitted_date + INTERVAL '1 month', 
      admitted_date + INTERVAL '1 year',
      admitted_date + INTERVAL '1 hour',
      admitted_date + INTERVAL '1 minute', 
      admitted_date + INTERVAL '1 second'
    FROM student;

    SELECT 
      current_timestamp, 
      current_timestamp - INTERVAL '1 day', 
      current_timestamp - INTERVAL '1 week', 
      current_timestamp - INTERVAL '1 month', 
      current_timestamp - INTERVAL '1 year',
      current_timestamp - INTERVAL '1 hour',
      current_timestamp - INTERVAL '1 minute', 
      current_timestamp - INTERVAL '1 second'
    FROM student;

admitted_date - could be current_date or current_timestamp as well 


## subtract two dates 
Use - operator. Returns the result in days when two dates are subtaracted. But the result specifies the label (days when result can be returned in days), or gives HH:MM:SS format if the difference is in hours/minutes/seconds

### diff between date columns
    SELECT 
    current_timestamp::date - (current_timestamp - INTERVAL '1 day')::date; ==> 1

    SELECT 
    current_timestamp::date - (current_timestamp - INTERVAL '1 week')::date; ==> 7

    SELECT 
    current_timestamp::date - (current_timestamp - INTERVAL '1 year')::date; ==> 366 


### diff between timestamp columns
    SELECT 
      current_timestamp - (current_timestamp - INTERVAL '1 month'), 
      current_timestamp - (current_timestamp - INTERVAL '1 year'), 
      current_timestamp - (current_timestamp - INTERVAL '1 day'), 
      current_timestamp - (current_timestamp - INTERVAL '1 week')

      "30 days"	
      "366 days"	
      "1 day"	
      "7 days"
--- 

      SELECT 
        current_timestamp - (current_timestamp - INTERVAL '1 hour'), 
        current_timestamp - (current_timestamp - INTERVAL '1 minute'), 
        current_timestamp - (current_timestamp - INTERVAL '1 second')

      "01:00:00"	
      "00:01:00"
      "00:00:01"

### difference between a date and a timestamp 
Works, but returns the result as combination of "x days HH:mm:ss"

      SELECT 
      current_timestamp - (current_timestamp - INTERVAL '1 month')::date  ==> "30 days 15:42:59.637529"

### negative differences 
    SELECT 
      (current_timestamp - INTERVAL '1 day') - current_timestamp, 
      (current_timestamp - INTERVAL '1 day')::date - current_timestamp::date


## date formatting 
### strings to date/datetime 
    to_date(date_str, input_format)
    to_timestamp(datetime, input_format)

### date/datetime to string formats
    to_char(date/datetime, output_format)

    DD - day of month (01–31)
    Day -	full capitalized day name (blank-padded to 9 chars)
    day - full lower case day name (blank-padded to 9 chars)
    DAY - full upper case day name (blank-padded to 9 chars)

    YYYY	year (4 or more digits)
    YYY	last 3 digits of year
    YY	last 2 digits of year
    Y	last digit of year

    MONTH	full upper case month name (blank-padded to 9 chars)
    Month	full capitalized month name (blank-padded to 9 chars)
    month	full lower case month name (blank-padded to 9 chars)
    MON	abbreviated upper case month name (3 chars in English, localized lengths vary)
    Mon	abbreviated capitalized month name (3 chars in English, localized lengths vary)
    mon	abbreviated lower case month name (3 chars in English, localized lengths vary)
    MM	month number (01–12)

    HH	hour of day (01–12)
    HH24	hour of day (00–23)

    MI	minute (00–59)

    SS	second (00–59)

# Frame specification in window functions 
  *todo*


# COUNT(*)/COUNT(1) V count(col)
- count(*)/count(1) counts every row, even if the entire row has nulls 
- count(col) returns 0 if any entire col is null, and only counts when there is a value in the col.
- 1280. Students and Examinations (Leetcode) [Link here] (https://leetcode.com/problems/students-and-examinations/description/?envType=study-plan-v2&envId=top-sql-50)


# MISC 
- multiply one of the ints with 1.0 to get float value in the output if both the inputs are ints
- any operation with null results in null. Like null+0, null/0 etc. this doesn't apply for agg operations. avg, sum, count doesnt include null. 

        salary 
        100 
        null 

        select avg(salary); 

        returns 100/1, not 100/2


# REGEX
*todo*