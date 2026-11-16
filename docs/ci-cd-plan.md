# CI/CD Strategy and Deployment Plan

**Status: Planned Enhancement — CI/CD design and repository preparation**

## Objective

Establish a structured CI/CD approach for managing, validating and deploying the Azure Data Factory and Azure Databricks assets developed as part of the Retail Business Intelligence Data Platform.

The project is designed to demonstrate practical Azure Data Engineering implementation skills, including data ingestion, medallion architecture, PySpark transformations, Delta Lake, Unity Catalog and analytical data modeling. CI/CD is identified as a complementary enhancement to make the solution more maintainable, repeatable and deployment-ready across environments.

The proposed approach focuses on source control, automated validation, secure configuration management and controlled releases without introducing unnecessary Azure compute costs.

## Proposed CI/CD Workflow

The following workflow describes the intended development and release process:

1. **Development:** Modify ADF JSON definitions, Databricks notebooks, configuration files or documentation.
2. **Source Control:** Commit changes to a feature branch in the Git repository.
3. **Pull Request:** Submit changes for review before merging into the main branch.
4. **Automated Validation:** Run static checks against ADF definitions, notebook files, Python code and repository structure.
5. **Review and Approval:** Review validation results and approve changes before merging.
6. **Release Preparation:** Generate or publish deployment artifacts from the approved source.
7. **Controlled Deployment:** Deploy ADF and Databricks assets to the selected environment using environment-specific configuration.
8. **Release Tracking:** Record deployment versions, approvals and outcomes for traceability.

## CI — Continuous Integration

The initial CI implementation will focus on non-deploying validation. These checks are intended to run without starting Azure Databricks compute or executing data pipelines.

### Planned validation checks

- Validate non-empty ADF JSON files and report incomplete or invalid definitions.
- Verify the expected repository structure and required documentation.
- Validate Databricks notebook JSON structure and code-cell extraction.
- Run Python syntax checks against standalone `.py` files, where available.
- Introduce notebook static checks where practical, without executing notebooks during routine CI.
- Scan repository changes for accidentally committed credentials or secrets.
- Publish validation results as part of the pull request.

These checks will help identify common repository and configuration issues early in the development lifecycle.

## CD — Continuous Deployment

The deployment strategy will support controlled promotion of Azure Data Factory and Databricks assets using environment-specific configuration.

### Azure Data Factory

- Establish the authoritative ADF source-control and publishing workflow.
- Use generated deployment templates or another supported deployment mechanism.
- Parameterize factory names, linked services, storage endpoints and other environment-specific settings.
- Keep credentials outside source-controlled JSON files and retrieve secrets through Azure Key Vault or approved secure service connections.
- Require appropriate approval before deploying to shared or production environments.

### Azure Databricks

- Select and document a consistent deployment approach for notebooks and job definitions.
- Externalize workspace-specific paths, catalog names, storage locations and other environment settings.
- Deploy notebooks and job configurations through an approved release process.
- Keep deployment separate from job execution so that compute is not started automatically unless explicitly required and approved.

## Security and Governance

Security will be incorporated into the CI/CD design from the beginning.

- Never commit storage account keys, passwords, access tokens or client secrets.
- Use managed identities or approved service connections wherever supported.
- Apply least-privilege permissions to build and deployment identities.
- Separate development and production configuration.
- Store and retrieve sensitive configuration through approved secret-management mechanisms.
- Maintain deployment approvals, release versions and execution records.

## Rollback and Recovery Strategy

A controlled deployment process should include a recovery approach.

- Retain previously approved deployment artifacts.
- Document how to redeploy the last known-good version.
- Define recovery steps for failed ADF and Databricks configuration deployments.
- Manage code rollback separately from data rollback.
- Use Delta Lake recovery capabilities and an appropriate data recovery strategy when data changes need to be reversed.

## Current State and Target State

| Capability | Current status |
|---|---|
| Git repository and project source control | Established |
| ADF asset exports | Five pipeline exports available; SQL linked service intentionally excluded for secure recreation |
| Databricks notebook source control | Project notebook assets maintained in the repository |
| CI validation workflow | Planned enhancement |
| Secret scanning and secure configuration | Planned enhancement |
| ADF automated deployment | Planned enhancement |
| Databricks automated deployment | Planned enhancement |
| Environment-specific parameterization | Target design |
| Approval and rollback process | Target design |

## Acceptance Criteria

The CI/CD workstream will be considered implemented when:

- Pull requests trigger repeatable automated validation.
- Invalid or incomplete deployment artifacts are identified and reported.
- Secrets are excluded from source control and handled securely.
- ADF and Databricks deployment procedures are documented and repeatable.
- Environment-specific configuration is parameterized.
- Appropriate approvals are required before deployment.
- A successful release and a rollback or recovery procedure are demonstrated.

## Project Implementation Perspective

The Retail Business Intelligence Data Platform establishes the core Azure Data Engineering implementation, including data ingestion, ADLS Gen2 storage, Bronze/Silver/Gold processing, Databricks transformations and analytical Gold datasets.

This CI/CD plan extends that foundation with a defined approach to source control, automated quality checks, secure deployment and release governance.

**The focus is on building a maintainable and deployment-ready Azure Data Engineering solution, with CI/CD automation identified as a logical next enhancement.**
