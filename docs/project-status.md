# Project Status and Work Tracker

**Overall status: In Progress**  
**Status baseline: 13 November 2026**

## Project preservation decision

The current implementation is being retained as the working baseline. Existing
ADF configuration, Databricks notebooks, documentation and previously recorded
results are not being replaced or re-executed as part of this status update.

This tracker organizes remaining work. Items previously identified as TODO are
tracked under **In Progress** so the project has an active completion plan.
“In Progress” does not mean that a task has been executed or completed; the
specific next action is stated for each item.

## Status summary

| Workstream | Status | Notes |
|---|---|---|
| Azure foundation and storage | Completed / recorded | Existing resource configuration is retained; no new provisioning planned |
| ADF ingestion configuration | In Progress | Five individual pipeline exports and six datasets are present; SQL linked service is excluded because its supplied JSON is invalid/redacted; Orders schema mismatch remains open |
| Databricks Bronze ingestion | In Progress | Notebook retained; review schema/source-contract alignment |
| WWI Silver transformations | In Progress | Notebook retained; review deterministic deduplication and transformation assumptions |
| Weather Silver | In Progress | Verify and document source units and validation behavior |
| Holiday Silver | In Progress | Review dataframe flow and county-scope handling |
| Silver-to-Gold model | In Progress | Retain current dimensions, fact and aggregate; document grain and limitations |
| Gold data quality | In Progress | Expand documented coverage and define failure/severity behavior |
| Documentation and evidence | In Progress | Align README, architecture, mappings, data dictionary and recorded results |
| Repository hygiene | In Progress | Identify incomplete exports and organize artifacts without changing implementation |
| CI/CD | In Progress | Define and implement source control, validation and deployment workflow |
| Power BI dashboard | Deferred | Deliberately skipped to avoid additional time and cost |

## In-progress work items

### 1. ADF configuration and source contracts

- [x] Review the supplied non-empty ADF JSON exports and validate dataset references.
- [ ] Reconcile repository exports with the actual deployed factory configuration.
- [ ] Re-export or securely recreate the SQL Server linked service from the
      actual ADF factory; the redacted source file is intentionally excluded.
- [ ] Correct the Orders positional schema mismatch between the ADF source
      dataset and the Auto Loader schema before any fresh ingestion.
- [ ] Keep credentials, storage keys and connection secrets out of source control.
- [x] Document that no master pipeline or trigger is included in the supplied export.
- [ ] Confirm the `.csv` dataset filenames versus `.txt` Copy sink extension settings.

### 2. Databricks notebooks

- [ ] Review the Orders Auto Loader schema against the source dataset and CSV
      positional order.
- [ ] Review WWI Silver transformations and customer deduplication assumptions.
- [ ] Verify Weather units against the API payload and `daily_units`.
- [ ] Review Holiday Silver transformation flow and `county_scope` behavior.
- [ ] Add or document unmatched OrderLine reconciliation before the FactSales
      inner join.
- [ ] Document the customer dimension as a latest/current-state representation,
      not a completed SCD Type 2 implementation.
- [ ] Extend the DQ design to cover Weather and Holiday datasets.
- [ ] Document full-refresh writes, DQ history retention and pipeline failure
      policy as current limitations or future improvements.

### 3. Gold model and validation evidence

- [ ] Preserve the existing FactSales grain: one row per OrderLine.
- [ ] Preserve the recorded counts and DQ outcomes as historical evidence;
      do not describe them as newly revalidated.
- [ ] Document the non-overlapping sales, weather and holiday date ranges.
- [ ] Align source-to-target mapping and data dictionary with the Gold tables
      actually implemented: DimDate, DimCustomer, FactSales and AggDailySales.
- [ ] Record known limitations and future enhancements separately from
      implemented functionality.

### 4. Documentation and repository

- [ ] Complete the architecture diagram and end-to-end data-flow description.
- [ ] Reconcile README and business requirements with implemented scope.
- [ ] Complete Gold data dictionary and source-to-target mapping.
- [ ] Add design decisions, validation evidence and interview explanations.
- [ ] Review empty folders and empty/incomplete ADF JSON exports; label gaps
      rather than inventing missing artifacts.

### 5. CI/CD — In Progress

- [ ] Confirm the source-control strategy and repository branch conventions.
- [ ] Add a CI validation workflow for changed files (JSON parsing, notebook
      structure checks, Python syntax/static checks where applicable, and
      documentation checks).
- [ ] Define environment-specific configuration and parameterization for
      development and any future higher environments.
- [ ] Define secret handling through Azure Key Vault or approved service
      connections; never commit credentials.
- [ ] Define an Azure Data Factory deployment approach using published
      templates or the team's chosen deployment mechanism.
- [ ] Define a controlled Databricks notebook/job deployment approach.
- [ ] Add approval gates and deployment documentation before enabling
      deployment to shared or production environments.
- [ ] Add a rollback/recovery procedure and deployment evidence.

**Current CI/CD state:** planning and repository preparation are in progress.
No completed automated CI/CD pipeline or automated deployment is claimed yet.

## Explicitly deferred

### Power BI dashboard

Power BI dashboard implementation is deferred by decision to avoid further
project cost and execution time. The existing Gold tables and prior connection
work may be described as groundwork, but a completed dashboard should not be
claimed.

## Execution and cost guardrails

- Do not rerun ADF pipelines, Databricks jobs or notebooks as part of this
  documentation/repository work.
- Do not provision new Azure resources or enable paid compute.
- Use static code review, existing artifacts and recorded evidence.
- Any future execution must be separately approved after reviewing expected
  cost and runtime.
