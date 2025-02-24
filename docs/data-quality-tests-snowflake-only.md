# Data Quality Tests using Snowflake Data Metric Functions (DMFs)

Requires Enterprise Edition Snowflake license.

## Specific Data Quality tests using DMFs

### NOT NULL

```sql title="NOT NULL"  linenums="0"
--Manually test system DMF 
SELECT SNOWFLAKE.CORE.NULL_COUNT(
  SELECT first_name
  FROM YOUR_DATABASE.YOUR_SCHEMA.YOUR_TABLE
);

--Apply to table
ALTER TABLE YOUR_DATABASE.YOUR_SCHEMA.YOUR_TABLE
  ADD DATA METRIC FUNCTION SNOWFLAKE.CORE.NULL_COUNT
    ON (first_name);
```

### Unique
#### Unique Single Column

```sql title="Unique test single column"  linenums="0"
--manually test system DMF
SELECT snowflake.core.duplicate_count(
  SELECT account_number
  FROM YOUR_DATABASE.YOUR_SCHEMA.YOUR_TABLE
);

--Apply to table
ALTER TABLE YOUR_DATABASE.YOUR_SCHEMA.YOUR_TABLE
  ADD DATA METRIC FUNCTION SNOWFLAKE.CORE.DUPLICATE_COUNT
    ON (account_number);
```

#### Unique Combination of Columns
Requires custom DMF. Currently DMFs require an explicit definition of the input columns and their data types. There isn't a way to create a single generic DMF that can handle varying numbers of columns or different data types. So must create a different DMF for each varying number and datatype of columns that make up composite keys in your tables. For example:

unique_combo_2_strings (ARG_T table(ARG_C1 STRING, ARG_C2 STRING))
unique_combo_3_mixed (ARG_T table(ARG_C1 STRING, ARG_C2 NUMBER, ARG_C3 NUMBER ))



```sql title="Unique combination of columns"  linenums="0"
--Create custom DMF
CREATE DATA METRIC FUNCTION IF NOT EXISTS
  unique_combination_of_columns_2_strings (ARG_T table(ARG_C1 STRING, ARG_C2 STRING))
  RETURNS NUMBER AS
  'select count(1) from (
select ARG_C1, ARG_C2
from ARG_T
group by ARG_C1, ARG_C2
having count(1) > 1)'

--Apply to table
ALTER TABLE customers ADD DATA METRIC FUNCTION
  unique_combination_of_columns_2_strings ON (last_name, city);
```

### Referential Integrity

```sql title="Referential integrity test"  linenums="0"
--Create custom DMF
CREATE OR REPLACE DATA METRIC FUNCTION referential_check(
  arg_t1 TABLE (arg_c1 INT), arg_t2 TABLE (arg_c2 INT))
RETURNS NUMBER AS
 'SELECT COUNT(*) FROM arg_t1
  WHERE arg_c1 NOT IN (SELECT arg_c2 FROM arg_t2)';

--Apply to table
ALTER TABLE customers
  ADD DATA METRIC FUNCTION referential_check
    ON (account_number, TABLE (accounts(account_no)));
```

### Equal Rowcount
Note that for custom DMFs you must specify the column in the table argument, even if you don't use it in the function logic. This is a current limitation of the DMF framework. So this custom DMF takes in columns for each table although they are irrelevent for the test.

```sql title="referential integrity"  linenums="0"
--Create custom DMF. Note expects column names despite their irrelevance for this test
--Note the result will be negative if table arg_t2 has more rows. 
CREATE OR REPLACE DATA METRIC FUNCTION equal_rowcout(
  arg_t1 TABLE (arg_c1 INT), arg_t2 TABLE (arg_c2 INT))
RETURNS NUMBER AS
 'select (select count(1) from arg_t1) - (select count(1) from arg_t2)';

--Apply to table: Note must pass column names despite their irrelevance for this test
ALTER TABLE customers
  ADD DATA METRIC FUNCTION equal_rowcout
    ON (account_number, TABLE (accounts(account_no)));
```


## DMF Configuration

See [Reference documentation](#reference-documentation) for more details

```sql  linenums="0"
--See DMF's added to table
SELECT * FROM TABLE(INFORMATION_SCHEMA.DATA_METRIC_FUNCTION_REFERENCES(
  REF_ENTITY_NAME => 'YOUR_DATABASE.YOUR_SCHEMA.YOUR_TABLE',
  REF_ENTITY_DOMAIN => 'TABLE'));

--See all DMFs both system and custom
SHOW DATA METRIC FUNCTIONS IN ACCOUNT;
  
--Drop a DMF: Must get signature right for DMF or will get a “does not exist” error:
DROP FUNCTION YOUR_DATABASE.YOUR_SCHEMA.YOUR_CUSTOM_DMF (table(STRING))
DROP FUNCTION YOUR_DATABASE.YOUR_SCHEMA.YOUR_CUSTOM_DMF (table(STRING, STRING)) -–e.g diff signature

--View DMF results
SELECT scheduled_time, measurement_time, table_name, metric_name, value
FROM SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS
WHERE TRUE
-- AND METRIC_NAME = 'INVALID_EMAIL_COUNT'
-- AND METRIC_DATABASE = 'DQ_TUTORIAL_DB'
order by scheduled_time desc
LIMIT 100;

```

## Reference Documentation

[Data Quality Monitoring](https://docs.snowflake.com/en/user-guide/data-quality-intro)

[Schedule your DMFs to run](https://docs.snowflake.com/en/user-guide/data-quality-working#label-data-quality-schedule)

[System DMFs](https://docs.snowflake.com/en/user-guide/data-quality-system-dmfs#system-dmfs)

[DMF required parameters](https://docs.snowflake.com/en/sql-reference/sql/create-data-metric-function#required-parameters)