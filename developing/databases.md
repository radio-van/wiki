# Contents

- [basics](#basics)
    - [layers](#layers)
    - [types](#types)
- [clickhouse](#clickhouse)
    - [disadvantages](#disadvantages)
    - [suitable for](#suitable-for)
- [postgres](#postgres)
    - [constraints](#constraints)
        - [add to existing table](#add-to-existing-table)
        - [drop constraint](#drop-constraint)
        - [list constraints](#list-constraints)
    - [roles](#roles)
    - [index](#index)
    - [queries](#queries)
        - [logic](#logic)
        - [dates](#dates)
        - [filter values](#filter-values)
        - [search for partial value](#search-for-partial-value)
        - [limit query](#limit-query)
        - [sort order](#sort-order)
        - [query from several tables](#query-from-several-tables)
        - [aggregate data with same values](#aggregate-data-with-same-values)
        - [aggregate data with same values in JSON field](#aggregate-data-with-same-values-in-json-field)
        - [EXCLUDE](#exclude)
        - [JOINS](#joins)
            - [inner join](#inner-join)
            - [left join](#left-join)
        - [WHERE and HAVING](#where-and-having)
        - [count more then one param](#count-more-then-one-param)
        - [group by month](#group-by-month)
    - [tables](#tables)
        - [add column](#add-column)
        - [change column type](#change-column-type)
        - [rename column](#rename-column)
        - [create temp table](#create-temp-table)
        - [read from CSV file](#read-from-csv-file)
        - [write to CSV file](#write-to-csv-file)
        - [copy column](#copy-column)
        - [add primary key](#add-primary-key)
        - [check if column exists](#check-if-column-exists)
    - [values](#values)
        - [duplicate](#duplicate)
        - [duplicate wwith modification](#duplicate-wwith-modification)
        - [insert](#insert)
        - [partial update](#partial-update)
        - [update](#update)
        - [update on conflict](#update-on-conflict)
        - [delete](#delete)
    - [window functions](#window-functions)
    - [tools](#tools)
        - [connect](#connect)
        - [dump](#dump)
        - [restore](#restore)
        - [write to file](#write-to-file)
        - [type of field](#type-of-field)
        - [find duplicates](#find-duplicates)
        - [delete duplicates](#delete-duplicates)

# basics
Data is stored in *pages* in RAM. If data is changed, then *page* is marked as *durty*.  
*Write-ahead-log* allows to recover state in case of power loss before data from RAM is stored on disk.  
It's done with *checkpoints* - syncing of *durty* pages to disk.

Data on disk is **not** organized into tables. *Tables* are abstractions of *query plan*.

## layers
* *data access*
* *data keep* cause keeping may differ from accessing
* *hardware layer*

## types
* *relative*: quieries + optimization

*Common DB table* is pseudo two-dimensional and *row*-oriented, e.g. [postgres](#postgres)
| row id | entity id | entity name | some entity param |
| 001    | 1         | smth1       | param1            |
| 002    | 2         | smth2       | param2            |

*Row*-oriented DB is efficient in returning the whole *row*, e.g. `001:1,smth1,param1`.  
However, such DB is not efficient in operations on the whole DB, because of full-scan of the table, which is  
stored in several blocks and that causes multiple disk operations.  
To solve the problem, indexes can be used, i.e. some values are stored separately with the row index of original table.  
It is useful for values which assume to be filtered (e.g. salary value).

Index example:
| row id | param  |
| 001    | param1 |
| 002    | param2 |

*Column*-oriented DB (e.g. [clickhouse](#clickhouse) serializes all values of a column together, e.g.:  
```
1:001,2:002
smth1:001,smth2:002
param1:001,param2:002
```
This looks like *row*-oriented table with index for each column, however, in *column*-oriented table key is column value,  
not the row number. This means that columns with the same value are compressed: `paramN:001,002`.  

# clickhouse
    Data is stored *by column* instead of *by row* (like in [sql](#sql)). Everything is pretty the same with SQL,  
    but access to data is more precise and scanning rows is eliminated, i.e. HDD seeks are lower comparing to other DB.
    
## disadvantages
* no transactions
* aggregations must fir in RAM
* no full-fledged UPDATE/DELETE

## suitable for
* small number of tables with large number of columns
* queries use a large number of rows with small subset of columns
* queries are rare
* query results is mostly filtered or aggregated
* batch update without transactions

# postgres

## constraints 
*Constraints* are the rules enforced on a data columns on table.
These are used to limit the type of data that can go into a table.
Could be _column_ level and _table_ level.

Commonly used:
* `NOT NULL` ensures that a column cannot have *NULL* value
* `DEFAULT` provides a default value for a column when none is specified
* `UNIQUE` ensures that all values in a column are different
* `PRIMARY Key` uniquely identifies each row/record in a table
* `CHECK` ensures that all values in a column satisfies certain conditions

Usage:
```sql
CREATE TABLE COMPANY(
  ID INT PRIMARY    KEY     NOT NULL,
  NAME              TEXT    NOT NULL UNIQUE,
  AGE               INT     NOT NULL CHECK(AGE > 18),
  SALARY            REAL    DEFAULT 50000.00
```

### add to existing table
```sql
ALTER TABLE <table> ADD CONSTRAINT <const name> UNIQUE (columns);
```
### drop constraint
```sql
ALTER TABLE <table> DROP CONSTRAINT <constraint_name>;
```
### list constraints
```sql
\d+ <table_name>
```

## roles 
`CREATE ROLE` or `createuser`, useful `options` are:
- `-s` superuser
- `-r` allow role creation
- `-l` allow login
- `d` allow create db

see also `ALTER ROLE`, `DROP ROLE`

## index
check indexes
```sql
SELECT
    tablename,
    indexname,
    indexdef
FROM
    pg_indexes
```

## queries 
### logic
```sql
  SELECT * FROM table WHERE condition AND (condition OR condition);
```

### dates
```sql
  SELECT * FROM table WHERE some_date_field > now(0 - interval 'N hours');
```
```sql
  SELECT * FROM table WHERE now() - some_date_field > interval '1 day');
```
```sql
  SELECT * FROM table WHERE date_trunc('day', some_date_field)=date_trunc('day', now());
```
### filter values 
```sql
  SELECT * FROM table WHERE string IN (value1, value2,...);
```
### search for partial value 
```sql
  SELECT * FROM table WHERE column LIKE 'pattern';
```
> HINT: use `%` as wildcard, e.g. `pattern%` or `%pattern%`
### limit query 
```sql
  SELECT * FROM table LIMIT num;
```
### sort order 
```sql
  SELECT * FROM table ORDER BY id asc/desc;
```
### query from several tables 
```sql
  SELECT * FROM table1 INNER JOIN table2 ON condition_of_joining WHERE ...;
```
### aggregate data with same values 
```sql
  SELECT DISTINCT column FROM ...
  SELECT column2, MAX(column1) FROM ... GROUP BY column2;
``` 
### aggregate data with same values in JSON field
```sql
  SELECT <field>->>'<json key>' AS key, COUNT(*) AS count FROM <table> GROUP BY <json key>
``` 

### EXCLUDE
```sql
  SELECT * FROM table WHERE ...
  EXCLUDE
  SELECT * FROM table WHERE ...
```

### JOINS 
**JOIN** adds columns and values from table *B* to selection from table *A*.

* **INNER** fetches data if present in both tables
* **CROSS** fetches data from both tables
* **OUTER**
    * **LEFT** fetches data if present in the left table
    * **RIGHT** fetches data if present in right table
    * **FULL** fetches data if present in either of the two tables

#### inner join 
select smth from one table where particular values are the same in both tables
```sql
  SELECT
    A.*
  FROM
    table_A A
  INNER JOIN table_B B ON A.id = B.id
```
#### left join 
select everything from one table plus values from another table that are the same with the first one
```sql
  SELECT
    A.*
  FROM
    table_A A
  LEFT JOIN table_B B ON A.id = B.id
```
select values from one table that dont exist in the second table
```sql
  SELECT
    A.*
  FROM
    table_A A
  LEFT JOIN table_B B ON A.id = B.id
  WHERE table_B.id IS NULL; 
```

### WHERE and HAVING
```sql
SELECT department_id, count(*) AS employees_no
    FROM employee
    WHERE gender = 'F'
    GROUP BY department_id
    HAVING employees_no < 10;
```

The query counts the number of female employees in each department,  
and only returns the departments where this number is less than 10.

- `WHERE` excludes non-female employees. Those rows are not read at all by the query.
- `GROUP BY` groups (or aggregates) the found rows, producing only one row for each distinct department_id.
- `HAVING` eliminates the aggregated rows where employees_no is less than 10.

Note that:

- `WHERE` employees_no < 10 would fail with an error, because that value doesn’t exist before aggregation.
- `HAVING` gender = 'F' would fail with an error, because the gender column doesn’t exist in the aggregated  
         rows (or, if you prefer, in the SELECT clause).

### count more then one param
```sql
SELECT COUNT(CASE <value><condition> THEN 1 END) AS VAL_COND1, COUNT(...) AS VAL_COND2 FROM ...;
```

### group by month
```sql
SELECT DATE_TRUNC('month', created_at) AS month, COUNT(...) GROUP BY month ORDER BY month;
```


## tables 
### add column 
```sql
  ALTER TABLE table ADD column_name datatype; 
```
### change column type
```sql
  ALTER TABLE table ALTER COLUMN column_name TYPE column_new_type; 
```
### rename column 
```sql
  ALTER TABLE table RENAME COLUMN column_name TO column_new_name; 
```
### create temp table 
```sql
  CREATE TEMPORARY TABLE table (column1 TYPE, column2 TYPE);
```
> HINT: date field type is `TIMESTAMP WITH TIME ZONE`
### read from CSV file 
```sql
  COPY table (column1, column2, ...) FROM 'filename' WITH CSV DELIMITER ',' QUOTE '"' ESCAPE '"';
```
### write to CSV file 
```sql
  \copy (SELECT ...) to 'filename' with csv;
```
Alternative method for read-only FS:
`psql ... -c '<command> to STDOUT with csv;' > <filename>`

### copy column
```sql
ALTER TABLE <table> ADD COLUMN <new_column> <data type>;
UPDATE <table> SET <new_column> = <old_column>;
```
### add primary key
```sql
ALTER TABLE <table> ADD COLUMN id SERIAL PRIMARY KEY;
```
### check if column exists
```sql
SELECT column_name 
FROM information_schema.columns 
WHERE table_name='your_table' and column_name='your_column';
```

## values 
### duplicate 
```sql
  INSERT INTO table (col1, col2, ...) SELECT col1, col2 FROM table WHERE <condition>
```
### duplicate wwith modification
```sql
  INSERT INTO table (col1, col2, ...) SELECT <new_value> as col1, col2 FROM table WHERE <condition>
```
in this case value of `col1` from table is ignored, `<new_value>` is used instead.
### insert 
* single row:
```sql
   INSERT INTO table (col1, col2) VALUES (val1, val2);
```
* multiple rows:
```sql
   INSERT INTO table (col1, col2) VALUES
       (val1, val2),
       (val3, val4);
```
### partial update 
```sql
  UPDATE table SET value = REPLACE(value, 'which', 'whith') WHERE condition;
```
### update 
```sql
  UPDATE table SET value = '<value>' WHERE condition;
```
### update on conflict
```sql
  INSERT INTO table (col1, col2, ...) VALUES (val1, val2, ...) ON CONFLICT (id) DO NOTHING
  INSERT INTO table (col1, col2, ...) VALUES (val1, val2, ...) ON CONFLICT (id) DO UPDATE other_field=EXCLUDED.other_field, ...
```
### delete 
```sql
  DELETE FROM table WHERE condition;
```


## window functions
`func() OVER <window> as <col_name>`

`()` - empty __window__, i.e. all results

functions:
* `row_number()`
* `rank()`
* `lead()`
also:
* `sum()`
* `count()`

Examples:
* add row numbers:
    ```sql
    SELECT id, col1, col2, row_number() OVER () as num FROM table;
    
    id | col1 | col2 | num
    1  | foo  | bar  | 1
    2  | bar  | foo  | 2
    ```
* calculate rating:
    ```sql
    SELECT id, score, row_number() OVER (ORDER BY score DESC) as rating FROM table ORDER BY id;
    
    id | score | rating
    1  |    50 |      2
    2  |   100 |      1
    ```
    Note that rating acts as id for __window__ because __window__ uses its own ordering

* using `PARTITION`
    ```sql
    SELECT id, section, score, row_number() OVER (PARTITION BY section ORDER BY score DESC) as rating FROM table ORDER BY id;
    
    id | section | score | rating
    1  |       1 |    50 |      2
    2  |       1 |   100 |      1
    3  |       2 |    70 |      1
    ```
    Note that rating is assigned according to section

* using `sum()`:
    ```sql
    SELECT transaction_id, change, sum(change) OVER (ORDER BY transaction_id) as balance FROM transactions ORDER BY transaction_id;
    
    transaction_id | change | balance
                 1 |    1.0 |     1.0
                 2 |   -2.0 |    -1.0
                 3 |   10.0 |     9.0
    ```
    OR
    ```sql
    SELECT transaction_id, change, sum(change) OVER () as balance FROM transactions ORDER BY transaction_id;
    
    transaction_id | change | balance
                 1 |    1.0 |     9.0
                 2 |   -2.0 |     9.0
                 3 |   10.0 |     9.0
    ```
    Note that without window only final balance is shown
    
* multiple functions:
    ```sql
    SELECT sum(salary) OVER w avg(salary) OVER w FROM salaries WINDOW w AS (PARTITION BY worker ORDER BY salary DESC);
    ```
    
    
## tools 
* `pgcli`
* `psql`

### connect 
`pgcli|psql -h localhost -U postgres`  
- `-W` ask password

### dump 
`pg_dump -U username > filename` dump database  
`pg_dumpall -U username > filename` dump all databases  

### restore 
`psql dbname < filename` restore single database  
`psql -f <dump> -U <user>` full restore  

### write to file 
`\o filename`

### type of field
`SELECT pg_typeof(<field>) from ...;`

### find duplicates
```sql
select Column1, Column2, count(*)
from <TABLE>
group by Column1, Column2
HAVING count(*) > 1
```

### delete duplicates
```sql
delete from <TABLE>
where <PK> not in (
  select min(<PK>) from <TABLE>
  group by <Col must be uniq>
);
```

on heavy data:
```sql
create table tmp as select min(id) from <TABLE> group by <UNIQ Column>;  /* a table with the first row from duplicates */
create table tmp2 as select * from <TABLE> left join tmp on <TABLE>.id=tmp.min where tmp.min is null;  /* a table with the rest of duplicates */
delete from <TABLE> where id in (select id from tmp2);  /* delete duplicates */
```
