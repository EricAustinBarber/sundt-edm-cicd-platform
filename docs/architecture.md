# Architecture

## Purpose
This document describes the reference architecture for Sundt Data Warehouse CI/CD using GitHub Actions and Databricks, with metadata-driven testing as a first-class control.

Primary goals:
- Build once, promote the same artifact through Dev → Test → Prod
- Enforce automated validation before production promotion
- Provide repeatable, auditable deployments with strong security boundaries

---

## High-level system diagram

```
+-------------------------------+
| GitHub Repositories           |
|                               |
|  feature / dev / test / prod  |
|                               |
|  GitHub Actions Workflows     |
|   - ci-dev                    |
|   - ci-test                   |
|   - cd-prod                   |
+---------------+---------------+
                |
                | Build / Package
                v
+-------------------------------+
| Artifact Store                |
| (GitHub Artifacts or External)|
+---------------+---------------+
                |
                | Deploy / Validate
                v
+-----------------------------------------------------------+
| Databricks                                                |
|                                                           |
|  Dev Workspace                                            |
|   - Bundle deploy                                         |
|   - Smoke tests                                           |
|                                                           |
|  Test Workspace                                           |
|   - Bundle deploy                                         |
|   - Metadata-driven asset validation                      |
|   - assets.metadata                                       |
|   - assets.test_results                                   |
|                                                           |
|  Prod Workspace                                           |
|   - Bundle deploy                                         |
|   - Post-deploy smoke / monitors                          |
+-----------------------------------------------------------+
```

---

## Core components

### GitHub Actions (Control Plane)
GitHub Actions orchestrate build, test, and deployment.

- **ci-dev**
  - Trigger: PRs to `dev`
  - Purpose: fast feedback (lint, unit tests, bundle validation)

- **ci-test**
  - Trigger: commits to `test`
  - Purpose: integration testing and data validation
  - Output: required status check for `prod` promotion

- **cd-prod**
  - Trigger: merge to `prod`
  - Purpose: deploy previously validated artifact to production

---

## Artifact promotion model

Artifacts are:
- Built once
- Identified by commit SHA or version
- Promoted unchanged through environments

This eliminates environment-specific rebuild drift.

---

## Metadata-driven validation

### assets.metadata
Authoritative inventory of data assets.

Typical fields:
- Source system and connection secret reference
- Database / schema / object
- Expected columns
- Sample query or filter
- Freshness / watermark column
- Enabled and critical flags

### assets.test_results
Execution log of automated validations.

Captured per run:
- Asset ID
- Environment
- Commit SHA
- Check results (pass / fail / warn)
- Metrics (row count, max timestamp, null rates)
- Diagnostic messages

These results are used for CI gating and audit.

---

## Branch execution flow

### Feature → Dev
- Developer opens PR
- ci-dev runs and must pass before merge

### Dev/Test
- Commit to `test` triggers ci-test
- Bundle deploys to Test workspace
- Asset validation runs for all enabled assets
- Results persisted and evaluated

### Test → Prod
- PR to `prod` requires ci-test to be green
- After merge, cd-prod deploys promoted artifact
- Post-deploy smoke checks run

---

## Security and environments

- GitHub Environments:
  - DataBricks-Dev
  - DataBricks-Test
  - DataBricks-Prod

- Secrets are:
  - Scoped per environment
  - Retrieved at runtime (Key Vault preferred)
  - Never stored in source control

- Authentication:
  - Preferred: Azure OIDC federation
  - Fallback: environment-scoped secrets

---

## Gating policy (baseline)

A run is considered successful when:
- All enabled + critical assets pass:
  - Connectivity
  - Schema
  - Freshness (if configured)
- No schema or connectivity failures exist

Policies are configurable per domain.

---

## Observability and audit

- GitHub run ID ↔ Databricks job run ID mapping
- Persisted test history in assets.test_results
- Dashboards for:
  - Pass/fail trends
  - Stale data detection
  - Asset-level quality signals

---

## Failure handling

- Fail fast on critical connectivity or schema issues
- Retry transient compute failures (bounded)
- Require approval for production overrides

---

## Assumptions

- Metadata table is authoritative
- Validation executes inside Databricks
- Workflows and actions are reusable across repos

---

## Related documents
- branching-and-promotion.md
- security-and-secrets.md
- runbooks/failed-ci-test.md
