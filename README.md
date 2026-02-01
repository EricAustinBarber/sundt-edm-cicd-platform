# SUNDT_DW_CICD_PIPELINE

Centralized CI/CD platform for Sundt Data Warehouse repositories.

This repository provides:
- Standard GitHub Actions workflows for CI and CD
- Reusable composite actions for Databricks deployments
- Metadata-driven automated testing for Databricks Asset Bundles
- Branching, promotion, and security standards for data pipelines

## Goals
- Enable safe, repeatable, and auditable deployments to Databricks
- Enforce automated testing before promotion to production
- Reduce manual effort and production data incidents
- Standardize CI/CD practices across all data warehouse repositories

## What lives here
- GitHub Actions workflows (`.github/workflows`)
- Reusable composite actions (`/actions`)
- Databricks asset testing framework (`/databricks/asset-testing`)
- Documentation and runbooks (`/docs`)

## What does NOT live here
- Business logic or transformation code
- Databricks notebooks
- Environment-specific configuration values

## Branching & Promotion Model (High Level)
- Feature branches → PR → `dev`
- `test` branch runs full integration + data asset validation
- PRs to `prod` require `ci-test` to pass
- Production deployments use promoted artifacts only

See `docs/branching-and-promotion.md` for details.

## Adoption
Other repositories consume this repo by:
- Referencing reusable workflows via `workflow_call`
- Using composite actions for Databricks deployment and testing
- Following documented branching and promotion standards

## Status
🚧 Initial bootstrap – workflows and asset testing framework under active development.