# Branch Protection Checklist (SUNDT_DW_CICD_PIPELINE)

Use this checklist to configure production-grade repository governance.

## Global repository settings

- Enable branch protection for `dev`, `test`, `prod`, and `main`.
- Require pull requests before merge.
- Require at least 1 approving review (2 for `prod`/`main`).
- Require CODEOWNERS review (`.github/CODEOWNERS`).
- Dismiss stale approvals on new commits.
- Block force pushes and branch deletions.
- Enable secret scanning and push protection.

## Required status checks

Configure these required checks:
- `workflow-lint` (from `ci-platform`)
- `yaml-lint` (from `ci-platform`)
- `docs-guard` (from `ci-platform`)

For `prod`/`main`, also require:
- Branch up to date before merge.

## Environment approvals

For any consuming repo that deploys using this platform workflow:
- `DataBricks-Dev`: no manual approval required.
- `DataBricks-Test`: optional approvals by data platform lead.
- `DataBricks-Prod`: required manual approval by production approvers.

## Release / hotfix controls

- Allow merge to `prod` only from approved `release/*` or `hotfix/*`.
- Require linked change ticket in PR template (recommended).
- Require rollback runbook link for `prod` changes.
