# Security notes

- Do not commit passwords, storage account keys, connection strings, SAS tokens,
  API tokens, client secrets, private keys, certificates or populated local
  environment files.
- Prefer managed identities for Azure-to-Azure access where supported.
- Store required secrets in Azure Key Vault or approved secure service
  connections and reference them from the relevant service configuration.
- Review customer data and sample payloads before publishing a public
  repository. Do not publish production or otherwise sensitive records.
- Review changes for secrets before every push and pull request.
- If a credential is committed accidentally, revoke or rotate it immediately.
  Deleting it in a later commit does not make the exposed credential safe.
