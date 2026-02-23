# Workflow Validation Runbook

Use this runbook to execute one validation run in `dev` and `test`.

## Prerequisites

- `gh` CLI installed.
- Valid GitHub authentication (`gh auth login`).
- Branches `dev` and `test` exist in the remote repository.

## Validate platform CI checks

1. Trigger and watch `ci-platform` on `dev`:
   - `gh workflow run "ci-platform" --ref dev`
   - `gh run list --workflow "ci-platform" --limit 1`
2. Trigger and watch `ci-platform` on `test`:
   - `gh workflow run "ci-platform" --ref test`
   - `gh run list --workflow "ci-platform" --limit 1`

## Success criteria

- `workflow-lint` passes.
- `yaml-lint` passes.
- `docs-guard` passes.
