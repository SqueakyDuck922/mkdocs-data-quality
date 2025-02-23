# Specific Data Quality test Examples using DMF’s 

## NOT NULL

```sql title="NOT NULL"  linenums="1"
--Test
SELECT SNOWFLAKE.CORE.NULL_COUNT(
  SELECT first_name
  FROM DQ_TUTORIAL_DB.SCH.CUSTOMERS
);

--Appy to Table
ALTER TABLE DQ_TUTORIAL_DB.SCH.CUSTOMERS
  ADD DATA METRIC FUNCTION SNOWFLAKE.CORE.NULL_COUNT
    ON (first_name);
```

## Unique
### Single Column

```sql title="Unique test signel column"  linenums="1"
--manually test function
SELECT snowflake.core.duplicate_count(
  SELECT account_number
  FROM DQ_TUTORIAL_DB.SCH.CUSTOMERS
);

--appy to table
ALTER TABLE DQ_TUTORIAL_DB.SCH.CUSTOMERS
  ADD DATA METRIC FUNCTION SNOWFLAKE.CORE.DUPLICATE_COUNT
    ON (account_number);
```

### Multiple Columns
Seems have to use custom DMF, and not possible to have a single generic custom DMF; Seems each DMF can only have input arguments that are tables with specific columns defined, so need a different DMF for combos that consist of a different number of columns and/or data types e.g:
unique_combination_of_columns_2_strings (ARG_T table(ARG_C1 STRING, ARG_C2 STRING))
unique_combination_of_columns_3_mixed (ARG_T table(ARG_C1 STRING, ARG_C2 NUMBER, ARG_C3 NUMBER ))

TODO wait outcome of support call to check if possible to do this with single generic DMF

```sql title="Unique combination of columns"  linenums="1"
CREATE DATA METRIC FUNCTION IF NOT EXISTS
  unique_combination_of_columns_2_strings (ARG_T table(ARG_C1 STRING, ARG_C2 STRING))
  RETURNS NUMBER AS
  'select count(1) from (
select ARG_C1, ARG_C2
from ARG_T
group by ARG_C1, ARG_C2
having count(1) > 1)'


ALTER TABLE customers ADD DATA METRIC FUNCTION
  unique_combination_of_columns_2_strings ON (last_name, city);
```

## Referential Integrity

```sql title="referential integrity"  linenums="1"
CREATE OR REPLACE DATA METRIC FUNCTION referential_check(
  arg_t1 TABLE (arg_c1 INT), arg_t2 TABLE (arg_c2 INT))
RETURNS NUMBER AS
 'SELECT COUNT(*) FROM arg_t1
  WHERE arg_c1 NOT IN (SELECT arg_c2 FROM arg_t2)';


ALTER TABLE customers
  ADD DATA METRIC FUNCTION referential_check
    ON (account_number, TABLE (accounts_daft(account_xx)));
```

## Equal Rowcount
TODO If possible to pass just table to DMF, as columns are irrelevant here

```sql title="referential integrity"  linenums="1"
--e.g. can do this without requiring arg_c1 & arg_c2 ?
CREATE OR REPLACE DATA METRIC FUNCTION referential_check(
  arg_t1 TABLE (arg_c1 INT), arg_t2 TABLE (arg_c2 INT))
RETURNS NUMBER AS
 'select (select count(1) from arg_t1) - (select count(1) from arg_t2)';


--For calling system DMF ROW_COUNT – possible to call without passing column?
ALTER TABLE DQ_TUTORIAL_DB.SCH.CUSTOMERS
  ADD DATA METRIC FUNCTION SNOWFLAKE.CORE.ROW_COUNT
    ON (<[a column here is irelevant]>);
```
