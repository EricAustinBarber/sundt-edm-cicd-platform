# Security and Secrets Management

## Purpose
This document defines the security model, secret-handling standards, and identity strategy for Sundt Data Warehouse CI/CD pipelines.

Goals:
- Minimize long-lived credentials
- Enforce least privilege by environment
- Provide auditable, compliant access patterns
- Reduce operational and security risk during deployments

---

## Security design principles

1. **No secrets in source control**
   - No credentials, tokens, or connection strings committed to GitHub.
   - Metadata tables store secret *references*, not values.

2. **Least privilege**
   - Access scoped by environment (Dev/Test/Prod).
   - Separate identities for CI/CD vs human access.

3. **Short-lived credentials preferred**
   - Favor federated identity (OIDC) over static secrets.

4. **Defense in depth**
   - GitHub Environments + Key Vault + Databricks permissions.

> Environment naming note: We currently use DataBricks-* (legacy ADO naming) to match existing environment keys and approvals. A future cleanup may rename these to Databricks-* after migration stabilization.

---

## Identity and authentication options

### Option 1: OIDC federation (preferred)

**What it is**
- GitHub Actions authenticates to Azure using OpenID Connect.
- Azure issues short-lived tokens dynamically.
- No client secrets stored in GitHub.

**How it works**
```
GitHub Action
   |
   | OIDC token
   v
Azure AD (Federated Credential)
   |
   | Access token
   v
Azure Resources / Databricks
```

**Benefits**
- No long-lived secrets to rotate
- Strong audit trail in Azure AD
- Reduced blast radius
- Aligns with Zero Trust best practices

**When to use**
- Production deployments
- Shared CI/CD pipelines
- Environments with compliance requirements

---

### Option 2: Service principals with secrets (fallback)

**What it is**
- Traditional Azure AD service principal
- Client ID + client secret stored as GitHub Environment secrets

**Benefits**
- Simple to set up
- Works without tenant-level OIDC configuration

**Risks**
- Long-lived secrets
- Manual rotation required
- Higher audit overhead

**Mitigations**
- Environment-scoped secrets only
- Rotation schedule enforced
- Limited permissions per environment

---

## GitHub Environments vs Azure Key Vault

### GitHub Environments
Used for:
- Environment-specific deployment credentials
- Databricks workspace URLs
- Cluster IDs
- Deployment toggles

Characteristics:
- Scoped to workflows and branches
- Support required approvals
- Visible audit trail of deployments

Example environments:
- DataBricks-Dev
- DataBricks-Test
- DataBricks-Prod

---

### Azure Key Vault
Used for:
- Source system credentials (e.g., JDBC connection strings)
- Third-party API keys
- Shared secrets used by Databricks jobs

Characteristics:
- Centralized secret lifecycle management
- Integrated with Azure RBAC
- Rotation and expiration support

**Rule of thumb**
- GitHub Environments → deployment context
- Key Vault → runtime data access

---

## Secret resolution flow

```
GitHub Actions
   |
   | (OIDC or SPN)
   v
Azure Key Vault
   |
   | Secret values
   v
Databricks Job / Cluster
```

- Metadata tables reference secrets by name (e.g., `sec-e1-connection-string`).
- Databricks jobs resolve secrets at runtime using managed identity or service principal access.
- GitHub Actions never logs secret values.

---

## Secret naming conventions

All secrets must follow a consistent naming pattern.

### Prefixes
- `sec-` : generic secret
- `kv-`  : Key Vault name reference
- `sp-`  : service principal identifier (non-secret)

### Examples
- `sec-e1-connection-string`
- `sec-databricks-access-token`
- `kv-dw2023-prod-sdt`
- `sp-dw-cicd-prod`

Environment-specific secrets should be scoped using GitHub Environments, not name suffixes.

---

## Rotation and lifecycle expectations

### Service principal secrets
- Rotation interval: every 90 days (maximum)
- Rotation responsibility: platform / DevOps team
- Rotation must be non-breaking (overlapping validity)

### Key Vault secrets
- Expiration dates set on creation
- Rotation documented and automated where possible
- Dependent jobs tested after rotation

### OIDC credentials
- No rotation required
- Federated credentials reviewed annually

---

## Logging and audit

### Required audit signals
- GitHub Actions run ID
- Commit SHA and artifact version
- Databricks job run ID
- Identity used (OIDC/SPN)
- Environment deployed

### Logging rules
- Secrets masked in logs
- No connection strings or tokens printed
- Structured logs preferred

---

## Risk reduction summary

This design reduces risk by:
- Eliminating hardcoded credentials
- Limiting credential scope per environment
- Ensuring all access is logged and attributable
- Preventing unauthorized production deployments
- Making security posture reviewable and repeatable

---

## Common anti-patterns (explicitly disallowed)

- Storing secrets in pipeline YAML
- Reusing the same credential across environments
- Copying secrets between GitHub and Databricks manually
- Bypassing CI for production deployments

---

## Related documents
- architecture.md
- branching-and-promotion.md
- runbooks/failed-ci-test.md
