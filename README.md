# FHIR API Data Ingestion and Analytics

## Overview

This project is created using Databricks to ingest healthcare data from the public HAPI FHIR API.

The following FHIR resources are used:

- Patient
- Encounter
- Observation
- Condition

The data is processed using Medallion Architecture:

FHIR API → Raw → Bronze → Silver → Gold


## Architecture

### Raw Layer
The API response is stored in JSON format in Databricks Volume.

The files are stored resource-wise and date-wise.

Example:

fhir_raw/
├── Patient/
├── Encounter/
├── Observation/
└── Condition/


### Bronze Layer

The JSON data is converted into Delta tables.

Additional metadata columns are added:

- extraction_timestamp
- api_url_or_params
- load_date
- source_resource


### Silver Layer

In the Silver layer:

- Duplicate records are removed.
- Record changes are identified using record hash.
- SCD Type 2 logic is used to maintain historical records.

SCD2 columns:

- effective_from
- effective_to
- is_current
- record_hash


### Gold Layer

Gold views are created using the current records from the Silver layer.

The following Gold views are created:

- gold_patient
- gold_encounter
- gold_observation
- gold_condition

A Patient-Encounter analytical view is also created for reporting.


## Incremental Loading

FHIR API data is loaded using pagination.

Load information is captured using metadata columns and an audit table.

The SCD Type 2 logic checks whether a record has changed.
If a record changes, the previous version is marked as historical and a new
current version is created.


## Databricks Workflow

A Databricks Workflow is created to run the notebooks in the required order:

Patient
↓
Encounter
↓
Observation
↓
Condition
↓
Gold Layer


## Reusability

Common configuration and functions are kept separately.

The same processing logic is reused for different FHIR resources instead of
writing separate ingestion logic for every resource.


## Project Structure

notebooks/
├── 01_Notebook
├── 02_FHIR_Data_Processing
├── 03_Silver_SCD2
├── 03_Gold_Layer
├── 05_Run_Patient
├── 06_Run_Encounter
├── 07_Run_Observation
└── 08_Run_Condition


## Results

The project successfully implements:

- FHIR API ingestion
- Pagination
- Raw JSON storage
- Bronze Delta tables
- Metadata tracking
- Data deduplication
- SCD Type 2 versioning
- Gold views
- Databricks Workflow orchestration
- Audit logging
