# Branching and Promotion Strategy

## Purpose
This document defines the standardized branching model, promotion rules, and CI/CD gating requirements for Sundt Data Warehouse repositories.

The goal is to:
- Enable continuous integration with fast feedback
- Enforce automated validation before promotion
- Ensure production deployments are auditable, repeatable, and low-risk

---

## Branch model overview

```
feature/*  →  dev  →  test  →  prod
   |          |        |        |
   |          |        |        +-- Production deployment
   |          |        +----------- Integration + data validation
   |          +-------------------- Developer integration
   +------------------------------- Feature development
```

---

## Branch definitions

### feature/*
**Purpose:** Individual feature or fix development

- Created from: `dev`
- Merged into: `dev`
- CI requirements:
  - Linting
  - Unit tests
  - Basic bundle validation
- Deployment: none

---

### dev
**Purpose:** Developer integration branch

- Accepts PRs from `feature/*`
- CI workflow: `ci-dev`
- Required checks:
  - `ci-dev / lint`
  - `ci-dev / unit-tests`
  - `ci-dev / bundle-validate`
- Deployment:
  - Optional deploy to Dev workspace
  - Smoke tests only

Failures here block feature integration but do not impact promotion.

---

### test
**Purpose:** Release candidate integration and validation

- Accepts merges from `dev` (or controlled direct commits)
- CI workflow: `ci-test`
- Required checks:
  - `ci-test / build`
  - `ci-test / bundle-deploy-test`
  - `ci-test / asset-validation`
- Deployment:
  - Mandatory deploy to Test Databricks workspace
  - Full metadata-driven asset validation

This branch represents “ready for production”.

---

### prod
**Purpose:** Production source of truth

- Accepts PRs from `test` only
- CI workflow: `cd-prod`
- Deployment:
  - Automatic or approval-gated production deployment
  - Uses promoted artifact only (no rebuild)

No direct commits are allowed.

---

## Required status checks

### dev branch
Required:
- `ci-dev` workflow must succeed

Recommended:
- Code owner review (optional but encouraged)

---

### test branch
Required:
- `ci-test / asset-validation` must succeed
- No failed critical asset checks

Recommended:
- At least one reviewer approval

---

### prod branch
Required:
- `ci-test` status check from latest `test` commit
- `cd-prod / pre-deploy` (if implemented)
- Code owner approval

Optional:
- Change window enforcement
- Manual approval via GitHub Environment

---

## Approval rules

### Code reviews
- `dev`: minimum 1 reviewer recommended
- `test`: minimum 1 reviewer required
- `prod`: minimum 1–2 reviewers required (Code Owners)

### Environment approvals
Use GitHub Environments for deployment gating:

- **DataBricks-Dev**
  - No approval required

- **DataBricks-Test**
  - Optional approval (team-dependent)

- **DataBricks-Prod**
  - Required approval before deployment
  - Approval recorded and auditable

---

## CI gating and promotion logic

### Promotion to test
Promotion occurs when:
- Feature is merged into `dev`
- Changes are merged or promoted to `test`
- `ci-test` executes successfully

### Promotion to prod
Promotion occurs when:
- PR from `test` → `prod` is opened
- Required checks confirm:
  - `ci-test` passed for the commit
  - All critical assets validated
- Required approvals are granted
- Merge to `prod` triggers `cd-prod`

---

## Artifact handling rules

- Artifacts are built once per commit
- Artifacts are immutable
- Same artifact is deployed to Test and Prod
- Artifact identity is recorded (commit SHA / version)

Rebuilding artifacts during promotion is prohibited.

---

## Failure scenarios

### CI failure on dev
- Feature PR cannot merge
- Developer fixes issues before retry

### CI failure on test
- Promotion halts
- Asset test failures must be resolved or explicitly waived
- Results are visible in `assets.test_results`

### CI failure on prod
- Deployment halts
- No automatic rollback unless configured
- Incident process may be triggered

---

## Exceptions and overrides

- Overrides require:
  - Explicit approval
  - Documentation of reason
  - Recorded audit trail

- Temporary waivers must:
  - Be time-bound
  - Reference affected assets
  - Be removed after remediation

---

## Governance principles

- No direct commits to `prod`
- No production deployment without passing CI
- No promotion without artifact traceability
- No secrets in source control

---

## Metrics for success

- % of prod deployments backed by passing `ci-test`
- Number of post-deploy incidents
- Mean time from test → prod
- Test coverage of critical assets

---

## Related documents
- architecture.md
- security-and-secrets.md
- runbooks/failed-ci-test.md
