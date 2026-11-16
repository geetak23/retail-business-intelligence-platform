# Security and Credential Management

Security is an important consideration throughout the Retail Business Intelligence Data Platform. The project follows secure development practices for credential management, Azure resource access, data handling and repository maintenance.

## Credential and Secret Management

- Do not commit passwords, storage account keys, connection strings, SAS tokens, API tokens, client secrets, private keys, certificates or populated local environment files.
- Prefer managed identities for Azure-to-Azure authentication wherever supported.
- Store required secrets in Azure Key Vault or approved secure service connections.
- Reference secrets through secure service configurations rather than embedding credentials in source code or configuration files.
- Use environment variables or approved secret-management mechanisms for local development.

## Azure Access and Permissions

- Apply least-privilege access to Azure resources.
- Use managed identities where supported instead of embedding credentials in linked services or notebooks.
- Review access permissions for ADLS Gen2, Azure Data Factory, Databricks and Key Vault.
- Avoid granting broader permissions than required for a component's responsibilities.

## Data Protection

- Review source data, sample datasets and API payloads before publishing repository content.
- Do not publish production records, personally identifiable information (PII) or otherwise sensitive data.
- Use representative or non-sensitive sample data when examples are needed.
- Avoid exposing sensitive information in notebook output, logs, screenshots or documentation.

## Git and Repository Security

- Review changed files for credentials and sensitive information before every push and pull request.
- Use `.gitignore` to exclude local configuration files, credentials, temporary files and generated artifacts where appropriate.
- Consider automated secret scanning as part of the future CI validation workflow.
- Review repository visibility and published files before making the project public.

## Credential Exposure and Recovery

If a credential is accidentally committed or exposed:

1. Revoke or rotate the affected credential immediately.
2. Review available logs and access activity for potential unauthorized use.
3. Update the relevant Azure service configuration with the replacement credential.
4. Remove the exposed value from repository history where appropriate.
5. Review the incident and strengthen preventive checks.

Deleting a credential in a later commit does not make an exposed credential safe. The credential must be considered compromised until revoked or rotated.

## Project Security Implementation

The project incorporates or is designed around the following security practices:

| Area | Approach |
|---|---|
| ADLS Gen2 access | Managed identity-based access |
| Azure Key Vault | Centralized secret management |
| Azure Data Factory | Secure linked-service configuration |
| Databricks | Unity Catalog and controlled data access |
| Git repository | No credentials or secrets in source control |
| CI/CD | Secret scanning and secure deployment are planned enhancements |

**Note:** This document describes the project's security practices and intended controls. It does not imply that every security control has undergone an independent security audit or that automated secret scanning is already configured.
