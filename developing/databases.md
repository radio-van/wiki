# Contents

- [tools](#tools)
    - [pgcli](#tools#pgcli)
        - [connect](#tools#pgcli#connect)
    - [psql](#tools#psql)
        - [dump](#tools#psql#dump)
        - [restore](#tools#psql#restore)
        - [write to file](#tools#psql#write to file)
- [sql](#sql)
    - [constraints](#sql#constraints)
    - [roles](#sql#roles)
    - [queries](#sql#queries)
        - [filter values](#sql#queries#filter values)
        - [search for partial value](#sql#queries#search for partial value)
        - [limit query](#sql#queries#limit query)
        - [sort order](#sql#queries#sort order)
        - [query from several tables](#sql#queries#query from several tables)
        - [aggregate data with same values](#sql#queries#aggregate data with same values)
        - [JOINS](#sql#queries#JOINS)
            - [inner join](#sql#queries#JOINS#inner join)
            - [left join](#sql#queries#JOINS#left join)
    - [tables](#sql#tables)
        - [add column](#sql#tables#add column)
        - [rename column](#sql#tables#rename column)
        - [create temp table](#sql#tables#create temp table)
        - [read from CSV file](#sql#tables#read from CSV file)
        - [write to CSV file](#sql#tables#write to CSV file)
    - [values](#sql#values)
        - [duplicate](#sql#values#duplicate)
        - [insert](#sql#values#insert)
        - [partial update](#sql#values#partial update)
        - [update](#sql#values#update)
        - [delete](#sql#values#delete)

# tools 
## pgcli 
### connect 
`pgcli -h localhost -u postgres`

## psql
### dump 
`pg_dump -U username > filename` dump database  
`pg_dumpall -U username > filename` dump all databases  

### restore 
`psql dbname < filename` restore single database  
`psql -f <dump> -U <user>` full restore  

### write to file 
`\o filename`

# sql 

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

## queries 
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

### JOINS 
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
```sql
   INSERT INTO table (col1, col2) VALUES (val1, val2);
```
### partial update 
```sql
  UPDATE table SET value = REPLACE(value, 'which', 'whith') WHERE condition;
```
### update 
```sql
  UPDATE table SET value = '<value>' WHERE condition;
```
### delete 
```sql
  DELETE FROM table WHERE condition;
```
