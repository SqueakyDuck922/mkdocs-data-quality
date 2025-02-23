# Data Governance for Shared Databases

## DBT
- Data Quality Tests: Implement [DQ tests for shared databases](data-quality-tests-shared-databases.md) using generic DBT tests
- Data Contracts: Enforce contracts all DM models (enforce in project file for DM folder)

## Snowflake Only
- Data Quality Tests: Implement [DQ tests for shared databases](data-quality-tests-shared-databases.md):
    - Not nulls implement as table constraints
    - Other tests implement as [DMF DQ Tests](data-quality-tests-snowflake-only.md) (or possibly use streams & tasks if don’t have enterprise edition)
- Data Contracts: Potentially look into Snowflake Object Tagging to document and enforce data contracts.