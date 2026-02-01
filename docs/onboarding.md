# Onboarding Guide: Using SUNDT_DW_CICD_PIPELINE

## Purpose
This guide explains how a data warehouse repository adopts the standardized CI/CD platform provided by `SUNDT_DW_CICD_PIPELINE`.

By following this guide, teams can:
- Enable CI and CD with minimal setup
- Inherit standardized security, testing, and promotion rules
- Deploy Databricks Asset Bundles consistently across environments

---

## Prerequisites

Before onboarding, confirm the following:

- Repository hosted in the Sundt GitHub organization
- Databricks workspace(s) available for Dev/Test/Prod
- Asset inventory registered in the metadata table
- Ownership assigned for the consuming repository
- Required secrets created in Azure Key Vault and/or GitHub

> Environment naming note: We currently use DataBricks-* (legacy ADO naming) to match existing environment keys and approvals. A future cleanup may rename these to Databricks-* after migration stabilization.
---

## Step 1: Add required branches

Ensure the repository has the following branches:

- `dev`
- `test`
- `prod`

Recommended:
- Protect `prod` from direct commits
- Protect `test` to require CI checks

---

## Step 2: Add GitHub Environments

Create GitHub Environments in the consuming repo:

- `DataBricks-Dev`
- `DataBricks-Test`
- `DataBricks-Prod`

For each environment:
- Add required secrets (see Step 3)
- Configure approval rules (Prod requires approval)
- Scope deployments to appropriate branches

---

## Step 3: Configure secrets

### GitHub Environment secrets (example)

**Required**
- `DATABRICKS_HOST`
- `DATABRICKS_TOKEN` (or OIDC config)
- `DATABRICKS_CLUSTER_ID` (if applicable)

**Optional**
- Feature flags
- Deployment toggles

Secrets must never be committed to source control.

---

## Step 4: Reference reusable workflows

Add a workflow in the consuming repository:

`.github/workflows/ci.yml`

Example:

```yaml
name: CI / CD

on:
  push:
    branches: [dev, test, prod]
  pull_request:

jobs:
  ci:
    uses: sundt/SUNDT_DW_CICD_PIPELINE/.github/workflows/reusable-databricks.yml@main
    with:
      bundle_path: databricks/
      environment: ${{ github.ref_name }}
```

This delegates CI/CD logic to the platform repo.

---

## Step 5: Register assets in metadata

Ensure all data assets are registered in `assets.metadata` with:

- Correct source connection reference
- Expected schema information
- Freshness expectations
- Enabled and critical flags

Only registered assets are validated by CI.

---

## Step 6: Validate locally (optional)

Before pushing to `test`:

- Run bundle validation locally:
  ```bash
  databricks bundle validate
  ```
- Run unit tests or notebook smoke tests as applicable

---

## Step 7: Promotion flow

- Merge feature branches into `dev`
- Promote tested changes to `test`
- Ensure `ci-test` passes
- Open PR from `test` to `prod`
- After approval, merge to trigger production deploy

---

## Common onboarding issues

### CI fails immediately
- Missing secrets in GitHub Environment
- Incorrect Databricks workspace URL
- Metadata entries incomplete or disabled

### Asset validation fails
- Schema mismatch with source
- Freshness threshold too strict
- Incorrect sample query

---

## Support and escalation

For onboarding support:
- Data Platform team
- DevOps / CI maintainers

Include:
- Repo name
- Branch
- GitHub Actions run ID
- Databricks job run ID (if applicable)

---

## Governance expectations

By onboarding, repositories agree to:
- Use standardized CI/CD workflows
- Respect branch protection rules
- Keep metadata accurate
- Participate in incident and improvement processes

---

## Related documents
- architecture.md
- branching-and-promotion.md
- security-and-secrets.md
- runbooks/failed-ci-test.md
- runbooks/prod-rollback.md
