# CI/CD Plan

**Status: In Progress — design and repository preparation**

## Objective

Introduce controlled source control, validation and deployment for the Azure
Data Factory and Databricks assets in this repository. CI/CD is a planned
workstream; the current project does not yet claim a completed automated
deployment pipeline.

## Proposed workflow

1. Developer changes ADF JSON, Databricks notebooks, configuration or docs.
2. Changes are committed to a feature branch.
3. Pull request validation runs static checks.
4. Review and approval are required before merging to the main branch.
5. Deployment artifacts are generated or published from the approved branch.
6. A controlled release deploys ADF and Databricks assets to the selected
   environment using environment-specific configuration.
7. Deployment outcome and release version are recorded.

## CI validation scope

Initial validation should be non-deploying and should not start Azure compute.

- Parse non-empty ADF JSON files and report empty/incomplete files.
- Check repository structure and required documentation.
- Validate notebook JSON structure and code-cell extraction.
- Run Python syntax checks on standalone `.py` files, when present.
- Add notebook static checks where practical; do not execute notebooks as part
  of ordinary CI.
- Detect likely secrets and prevent credentials from being committed.
- Publish validation results with the pull request.

## CD deployment scope

### Azure Data Factory

- Establish the authoritative ADF source-control and publish workflow.
- Use generated deployment templates or the selected supported deployment
  mechanism.
- Parameterize factory names, linked services, storage endpoints and other
  environment-specific values.
- Keep secrets outside JSON source files and resolve them through Key Vault or
  approved secure service connections.
- Require approval before deployment to shared or production environments.

### Azure Databricks

- Select and document a consistent deployment method for notebooks and jobs.
- Keep workspace-specific paths, catalog names, storage locations and other
  environment settings configurable.
- Deploy notebooks and job definitions only through an approved release.
- Avoid starting compute or executing jobs automatically unless explicitly
  intended and approved.

## Security and governance

- Never commit account keys, passwords, tokens or client secrets.
- Use managed identities or approved service connections where supported.
- Apply least-privilege permissions to build and deployment identities.
- Separate development and production configuration.
- Record approvals, deployment versions and outcomes.

## Rollback and recovery

- Retain the last known-good deployment artifacts.
- Document how to redeploy the prior approved version.
- Define how to restore ADF and Databricks configuration after a failed release.
- Treat data rollback separately from code rollback; Delta data changes require
  their own recovery strategy.

## Current state versus target state

| Capability | Current status |
|---|---|
| Project assets stored in repository | Partially present |
| ADF export completeness | In Progress — five pipeline exports available; SQL linked service intentionally excluded pending secure recreation |
| CI validation workflow | Planned / In Progress |
| Secret scanning and secure configuration | Planned / In Progress |
| ADF automated deployment | Planned / In Progress |
| Databricks automated deployment | Planned / In Progress |
| Approval and rollback process | Planned / In Progress |

## Acceptance criteria

CI/CD can be considered implemented when:

- A pull request triggers repeatable validation.
- Invalid or incomplete deployment artifacts are reported.
- Secrets are not stored in the repository.
- ADF and Databricks deployment steps are documented and repeatable.
- Environment configuration is parameterized.
- Deployment requires the appropriate approvals.
- A successful release and a rollback/recovery path are evidenced.

No Azure pipeline execution or deployment is part of this planning document.
