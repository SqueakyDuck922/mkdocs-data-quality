# Data Quality Tests in DBT

Configured in yaml files for each model.

## NOT NULL

At column level

```yaml  linenums="0" hl_lines="6"
columns:
    - name: your_column
    data_type: TEXT
    description: Business key for something
    tests:
        - not_null  
```

## Unique
### Unique Single Column

At column level

```yaml linenums="0" hl_lines="6"
columns:
    - name: your_column
    data_type: TEXT
    description: Business key for something
    tests:
        - unique
```

### Unique Combination of Columns

At model level - using package dbt-labs/dbt_utils

```yaml linenums="0" hl_lines="6"
models:
  - name: your_model
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
           - column1 
           - column2
           - column3
```

## Referential Integrity

At column level

```yaml   linenums="0" hl_lines="6"
columns:
    - name: your_column
      data_tests:
       - relationships:
            to: ref('ref_model')
            field: column_in_ref_model
```

## Equal Rowcount

At model level - using package dbt-labs/dbt_utils

```yaml   linenums="0" hl_lines="6"
models:
  - name: your_model
    tests:
      - dbt_utils.equal_rowcount:
          compare_model: ref('ref_model')
```

## Test for tests

At project folder level (in dbt_project.yml) - using calogica/dbt_expectations

```yaml  linenums="0" hl_lines="6"
models:
  your_project:
    your_sub_folder:
      +required_tests: {"unique_combination_of_columns" : 1}
```