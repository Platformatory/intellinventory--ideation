**Proposal**

<h1>FDH Dbx Ingestion Automated Setup</h1>

---

**Contents**:

- [Scenario](#scenario)
  - [Basic](#basic)
  - [Advanced](#advanced)
- [Context](#context)
  - [What is FDH Databricks Ingestion?](#what-is-fdh-databricks-ingestion)
  - [What Does FDH Databricks Ingestion Involve?](#what-does-fdh-databricks-ingestion-involve)
    - [ETL Framework](#etl-framework)
    - [Development Stages (`dev`)](#development-stages-dev)
    - [Execution Stages (`exe`)](#execution-stages-exe)
  - [What is FDH Databricks Ingestion?](#what-is-fdh-databricks-ingestion-1)
  - [What Does FDH Databricks Ingestion Involve?](#what-does-fdh-databricks-ingestion-involve-1)
    - [ETL Framework](#etl-framework-1)
    - [Development Stages (`dev`)](#development-stages-dev-1)
    - [Execution Stages (`exe`)](#execution-stages-exe-1)

---

# Scenario
## Basic
4 requirements docs:

1. Ingestion mapping workbook
   1. 1 sheet for source and target specifications
   2. 1 sheet for the actual mapping
2. Ingestion process flow
3. Job specifications
4. Other requirements

We need to use these to automatically create:

- Scripts
- Jobs/workflows
- Report on requirements-compliance

We can also include human feedback checkpoints (e.g. after the report is done).

On top of this, we could automate:

- Running the scripts in the right order once created
- Scheduling the job as per job specifications

## Advanced
- Identify discrepancies in requirements <br> E.g.: *Difference in target datatype and column description specification*
- Suggest changes to rectify discrepancies

# Context
## What is FDH Databricks Ingestion?
FDH (Fusion Data Hub) is ecosystem of all the data pipelines and peripheral data-handling applications needed to collect, process and distribute client-side data for JFC (Jollibee Food Corporation). FDH Databricks ingestion refers to the parts of FDH directly involved with (1) ingesting data from various data sources (e.g. Kafka, S3, etc.) into Databricks and (2) processing this data for various downstream use-cases. This ingestion into Databricks is done in order to unify the development, deployment and governance of data analytics and other downstream data-dependent applications (e.g. ML model training).

## What Does FDH Databricks Ingestion Involve?
### ETL Framework
The requirements of the ingestion setup are shaped by this ETL framework:

[`github.com/Jollibee-Foods-Corporation/jfc-fdh-etl-framework-dbx-platform`](https://github.com/Jollibee-Foods-Corporation/jfc-fdh-etl-framework-dbx-platform)

(See the `development` branch for the latest code)

For Databricks ingestion, we use classes and methods defined in this framework. But to use these, the code expects a number of things, such as: (1) configuration files (either in an expected location and with an expected naming format, or given as a method argument), (2) Databricks resources (e.g. control tables for onboarding configurations and lookup tables for knowing which transformations to apply to the source data), and (3) AWS resources (e.g. a secret in Secrets Manager with an expected naming format).

### Development Stages (`dev`)
FDH Databricks ingestion can be divided into 3 stages and their sub-stages:

**Stage 1: Architecture** (`dev.arc`):

1. Source data schema definition
2. Target table schema definition
3. Source-to-target ingestion mapping

**Stage 2: Ingestion Scripts + Configurations** (`dev.src`):

1. DDL statement(s) to create target external Delta table(s) <br> **NOTE**: *These tables store data in an S3 location which is not Databricks-managed*
2. DDL statement(s) to create external Snowflake table(s) <br> **NOTE**: *They use the same as the Delta tables*
3. Lookup table entries to perform source-to-target column transformations <br> E.g.: *Conversion of date string in source to datetime object in target*
4. Onboarding data-ingestion configurations <br> E.g.: *Data source (Kafka, S3, etc.), source data format (JSON, Avro, etc.)*
   1. Configurations file (this is a JSON file)
   2. Script to apply above configurations to control table <br> **NOTE**: *This table is used by the ETL framework to load ingestion configurations*
5. Source-to-target schema mapping and registry

**Stage 3: Job Scripts + Job Creation** (`dev.job`):

1. Initial manual run script (mostly for testing purposes)
2. Hourly/daily run job scripts
3. Job creation script (which creates the ingestion job in Databricks)

---

Development stages that have not been included but are needed for ingestion:

- Creation of a secret with the necessary Databricks and source configurations
- Outline of the permissions needed by the user doing the ingestion

*We must consider incorporating these stages into this automation framework.*

### Execution Stages (`exe`)
This is useful to understand how the created scripts will be used in practice.

**Stage 1: Table Creation & Setup** (`exe.tbl`):

- Create Delta tables using `dev.src.1` <br> *This script also contains DROP TABLE queries if necessary and table deletion instructions*
- Create external tables in Snowflake using `dev.src.2`
- Insert lookup table entries using `dev.src.3`
- Load onboarding configurations in `dev.src.4.1` using `dev.src.4.2`

**Stage 2: Manual Ingestion** (`exe.man`):

To ingest data manually, use `dev.job.1`.

**Stage 3: Job Creation** (`exe.job`):

Each data-ingestion job script ingests from a particular source, and all scripts together cover all the relevant sources for the particular pipeline. A job for automating this data ingestion must contain tasks, each task referring to an ingestion script. To facilitate the creation of this job along with the necessary configurations, we can use a script to do the following:

- Store a sufficiently authorised service principal's credentials
- Use these to obtain an AUTH token to authorise job creation API requests

Then, look up the created job ID in the "Jobs & Pipelines" tab, where we can:

- Add triggers (e.g. for hourly/daily schedule)
- Upgrade the compute version if necessary

## What is FDH Databricks Ingestion?
FDH (Fusion Data Hub) is ecosystem of all the data pipelines and peripheral data-handling applications needed to collect, process and distribute client-side data for JFC (Jollibee Food Corporation). FDH Databricks ingestion refers to the parts of FDH directly involved with (1) ingesting data from various data sources (e.g. Kafka, S3, etc.) into Databricks and (2) processing this data for various downstream use-cases. This ingestion into Databricks is done in order to unify the development, deployment and governance of data analytics and other downstream data-dependent applications (e.g. ML model training).

## What Does FDH Databricks Ingestion Involve?
### ETL Framework
The requirements of the ingestion setup are shaped by this ETL framework:

[`github.com/Jollibee-Foods-Corporation/jfc-fdh-etl-framework-dbx-platform`](https://github.com/Jollibee-Foods-Corporation/jfc-fdh-etl-framework-dbx-platform)

(See the `development` branch for the latest code)

For Databricks ingestion, we use classes and methods defined in this framework. But to use these, the code expects a number of things, such as: (1) configuration files (either in an expected location and with an expected naming format, or given as a method argument), (2) Databricks resources (e.g. control tables for onboarding configurations and lookup tables for knowing which transformations to apply to the source data), and (3) AWS resources (e.g. a secret in Secrets Manager with an expected naming format).

### Development Stages (`dev`)
FDH Databricks ingestion can be divided into 3 stages and their sub-stages:

**Stage 1: Architecture** (`dev.arc`):

1. Source data schema definition
2. Target table schema definition
3. Source-to-target ingestion mapping

**Stage 2: Ingestion Scripts + Configurations** (`dev.src`):

1. DDL statement(s) to create target external Delta table(s) <br> **NOTE**: *These tables store data in an S3 location which is not Databricks-managed*
2. DDL statement(s) to create external Snowflake table(s) <br> **NOTE**: *They use the same as the Delta tables*
3. Lookup table entries to perform source-to-target column transformations <br> E.g.: *Conversion of date string in source to datetime object in target*
4. Onboarding data-ingestion configurations <br> E.g.: *Data source (Kafka, S3, etc.), source data format (JSON, Avro, etc.)*
   1. Configurations file (this is a JSON file)
   2. Script to apply above configurations to control table <br> **NOTE**: *This table is used by the ETL framework to load ingestion configurations*
5. Source-to-target schema mapping and registry

**Stage 3: Job Scripts + Job Creation** (`dev.job`):

1. Initial manual run script (mostly for testing purposes)
2. Hourly/daily run job scripts
3. Job creation script (which creates the ingestion job in Databricks)

---

Development stages that have not been included but are needed for ingestion:

- Creation of a secret with the necessary Databricks and source configurations
- Outline of the permissions needed by the user doing the ingestion

*We must consider incorporating these stages into this automation framework.*

### Execution Stages (`exe`)
This is useful to understand how the created scripts will be used in practice.

**Stage 1: Table Creation & Setup** (`exe.tbl`):

- Create Delta tables using `dev.src.1` <br> *This script also contains DROP TABLE queries if necessary and table deletion instructions*
- Create external tables in Snowflake using `dev.src.2`
- Insert lookup table entries using `dev.src.3`
- Load onboarding configurations in `dev.src.4.1` using `dev.src.4.2`

**Stage 2: Manual Ingestion** (`exe.man`):

To ingest data manually, use `dev.job.1`.

**Stage 3: Job Creation** (`exe.job`):

Each data-ingestion job script ingests from a particular source, and all scripts together cover all the relevant sources for the particular pipeline. A job for automating this data ingestion must contain tasks, each task referring to an ingestion script. To facilitate the creation of this job along with the necessary configurations, we can use a script to do the following:

- Store a sufficiently authorised service principal's credentials
- Use these to obtain an AUTH token to authorise job creation API requests

Then, look up the created job ID in the "Jobs & Pipelines" tab, where we can:

- Add triggers (e.g. for hourly/daily schedule)
- Upgrade the compute version if necessary