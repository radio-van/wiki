# Contents

- [basics](#basics)
    - [layers](#basics#layers)
    - [types](#basics#types)
- [clickhouse](#clickhouse)
    - [disadvantages](#clickhouse#disadvantages)
    - [suitable for](#clickhouse#suitable for)
- [postgres](#postgres)
    - [constraints](#postgres#constraints)
    - [roles](#postgres#roles)
    - [index](#postgres#index)
    - [queries](#postgres#queries)
        - [logic](#postgres#queries#logic)
        - [dates](#postgres#queries#dates)
        - [filter values](#postgres#queries#filter values)
        - [search for partial value](#postgres#queries#search for partial value)
        - [limit query](#postgres#queries#limit query)
        - [sort order](#postgres#queries#sort order)
        - [query from several tables](#postgres#queries#query from several tables)
        - [aggregate data with same values](#postgres#queries#aggregate data with same values)
        - [EXCLUDE](#postgres#queries#EXCLUDE)
        - [JOINS](#postgres#queries#JOINS)
            - [inner join](#postgres#queries#JOINS#inner join)
            - [left join](#postgres#queries#JOINS#left join)
        - [WHERE and HAVING](#postgres#queries#WHERE and HAVING)
    - [tables](#postgres#tables)
        - [add column](#postgres#tables#add column)
        - [rename column](#postgres#tables#rename column)
        - [create temp table](#postgres#tables#create temp table)
        - [read from CSV file](#postgres#tables#read from CSV file)
        - [write to CSV file](#postgres#tables#write to CSV file)
    - [values](#postgres#values)
        - [duplicate](#postgres#values#duplicate)
        - [insert](#postgres#values#insert)
        - [partial update](#postgres#values#partial update)
        - [update](#postgres#values#update)
        - [update on conflict](#postgres#values#update on conflict)
        - [delete](#postgres#values#delete)
    - [tools](#postgres#tools)
        - [connect](#postgres#tools#connect)
        - [dump](#postgres#tools#dump)
        - [restore](#postgres#tools#restore)
        - [write to file](#postgres#tools#write to file)
        - [type of field](#postgres#tools#type of field)

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

## tables 
### add column 
```sql
  ALTER TABLE table ADD column_name datatype; 
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

## values 
### duplicate 
```sql
  INSERT INTO table (col1, col2, ...) SELECT col1, col2 FROM table WHERE <condition>
```
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
