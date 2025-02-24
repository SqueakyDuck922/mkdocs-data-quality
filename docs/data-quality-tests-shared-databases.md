# Data Quality Tests for Shared Database 
## Data Mart

### Test on One Database Object
- Grain of table: key/unique combo of columns –test_coverage e.g."unique_combination_of_columns|unique" : 1
- Not nulls – generally all columns in a data mart should not be null – use default values instead (“N/A”, “unknown”..)
- Accepted values

### Test how One Database Object refers to another Database Object
- Referential integrity : facts to dims
- Row count consistency between source and transformed tables

### Test Something Unique about Your Data
- Test for specific business rules


### Test the Freshness of your Raw Source Data
- Freshness tests on raw sources

### Test Coverage: Test for Tests
- For each model test for existence of unique test 

