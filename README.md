# ELT workflow with dbt & Airflow
## 📋 Overview

#### This project implements an ELT (Extract, Load, Transform) approach for workflows using dbt for data transformation, Airflow for orchestration and Snowflake as the data warehouse. The pipeline transforms TPC-H dataset tables into models which can be used for analytics following dbt best practices with a staging, intermediate, and final consumption marts layer architecture.

## 🏗️ Architecture

```
elt_snowflake/
├── macros/
│   └── pricing.sql               # Custom pricing macros
├── models/
│   ├── staging/                  # Raw data models
│   │   ├── stg_tpch_line_items.sql
│   │   └── stg_tpch_orders.sql
│   └── marts/                    # Business-ready models
│       ├── fct_orders.sql        # Fact table for orders
│       ├── generic_tests.sql     # Data quality tests
│       ├── int_order_items.sql   # Intermediate transformations
│       └── int_order_items_summary.sql
├── tests/
│   ├── fct_orders_date_valid.sql # Date validation tests
│   └── fct_orders_discount.sql   # Discount validation tests
├── dbt_project.yml               # Project configuration
└── requirements.txt              # Python and other dependencies
```
## 🌟 Features

- Modular Structure: Clear separation between staging and mart layers for maintainability and reusability
- Custom Macro: Reusable SQL logic ```discounted_amount``` to enforce consistent business logic across models
- Airflow Integration: Orchestrated using Astronomer's Cosmos package for seamless dbt workflow automation
- Data Quality Testing: Built-in tests for data integrity to ensure reliable analytics
- Containerization: Docker-ready deployment for consistent environments and easy scaling

## 🔄 Data Transformation Flow

### Staging Layer: Raw data from TPC-H sources is standardized and prepared for further transformation

- ```stg_tpch_orders```: Standardizes order header data with consistent naming
- ```stg_tpch_line_items```: Standardizes order line item details and creates a surrogate key

### Intermediate Layer: Business logic is applied to staged data to create intermediate calculations and relationships

- ```int_order_items```: Joins orders with line items and calculates discount amounts using the custom pricing macro
- ```int_order_items_summary```: Aggregates order items by order key to calculate gross sales and total discounts

### Marts Layer: Final analytics-ready (consumption) table that provides business value

- ```fct_orders```: Complete order data enriched with sales and discount amounts, ready for reporting and analysis

## ⚙️ Configuration
### dbt Configuration
The project uses the following materialization strategies:

- Staging models: Materialized as views for efficient storage and always-current data
- Mart models: Materialized as tables for optimized query performance

All models use the ```dbt_wh``` Snowflake warehouse for execution, as specified in the ```dbt_project.yml``` file.

### Airflow Configuration
The dbt DAG is configured with:
- Daily schedule: Runs once per day to refresh all models
- Cosmos integration: Uses the Cosmos package to manage dbt execution within Airflow
- Virtual environment: Executes dbt commands within a dedicated Python environment for dependency isolation
- Snowflake integration: Connects to Snowflake using the configured connection ID

## 🔬 Testing
The project includes basic data quality tests to ensure data reliability:

Generic Tests

- Uniqueness checks: Ensures order keys are unique to prevent duplicate orders
- Not null validations: Confirms that critical fields contain values
- Referential integrity: Verifies that relationships between tables are maintained
- Value acceptance: Validates that status codes are limited to allowed values ('P', 'O', 'F')

Custom Data Tests

- Date Validation: Ensures order dates are within a valid range to catch data ingestion errors
- Discount Validation: Identifies orders with valid discount amount for review

Run tests:
```
dbt test
```
This will execute all tests and report any failures in the command line output.

## 🛠️How to run this project
### Prerequisites
- Docker: Required to containerize and run the application consistently across environments
- Snowflake account: An active account
- dbt: dbt-core installed along with its snowflake adapter for connecting to snowflake warehouse and other artifacts

### Snowflake Setup
Before running this workflow, you'll need to set up your Snowflake environment by following these steps:
- Switch to administrator role: Log into Snowflake and use the ```ACCOUNTADMIN``` role, which has the necessary privileges to create resources.
- Create a warehouse: Create a warehouse named ```dbt_wh``` with X-Small size for now for cost efficiency. This warehouse will be used for all dbt transformations.
- Create a database: Set up a database named ```dbt_db``` that will store all the transformed data models and tables.
- Create a role: Establish a dedicated ```dbt_role``` with appropriate least-privilege access for dbt operations.
- Grant warehouse usage: Provide the ```dbt_role``` with usage permissions on the ```dbt_wh``` warehouse to execute queries.
- Assign the role: Grant the dbt_role to your Snowflake user account to inherit all necessary permissions.
- Grant database privileges: Provide the ```dbt_role``` with all privileges on the ```dbt_db``` database to create, modify, and read objects.
- Create schema: While using the ```dbt_role```, create a ```dbt_schema``` within the database to organize the data objects.

> 💡 Tip: These resources can be cleaned up when no longer needed by dropping the warehouse, database, and role as the ```ACCOUNTADMIN``` role to avoid incurring costs on Snowflake account.

### To configure Snowflake Connection in Airflow:
- Create an Airflow connection named ```snowflake_conn``` in the Airflow UI
- Set the connection type to "Snowflake"
- Provide your Snowflake username, password, account identifier, warehouse, database, and schema
- This connection will be used by the Cosmos package to execute dbt commands against Snowflake

### Run workflow through Airflow:
```
astro dev start
```
- The Dockerfile installs Airflow with the Astronomer runtime
- Creates a dedicated Python virtual environment for dbt
- Installs the dbt-snowflake adapter for Snowflake compatibility
- Maps port 8080 to access the Airflow UI

> 💡 Tip: To run a specific model in terminal use command: ```dbt run -s model_name```.

### Access Airflow UI and Activate DAG:
- Navigate to ```http://localhost:8080``` in your browser
- Log in with the default credentials (username: admin, password: admin)
- Locate the ```dbt_dag``` in the DAGs list
- Enable the DAG by toggling the switch in the UI
- The DAG will automatically run daily based on the schedule configuration
