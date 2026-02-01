# Runbook: Production Rollback / Halt (cd-prod)

## Purpose
This runbook describes how to respond to production deployment issues for Databricks deployments triggered by the `cd-prod` workflow.

Use this when:
- `cd-prod` fails during production deploy
- A production deploy succeeds but downstream checks/monitoring show issues
- You need to stop further promotions until root cause is understood
- A rollback or forward-fix decision must be made quickly

---

## 1) Immediate actions (first 10 minutes)

1. **Stop further production changes**
   - Pause merges to `prod` (temporary protection change or communication hold).
   - If automated deployment triggers on merge, stop the pipeline by:
     - Cancelling the in-flight GitHub Actions run (if still running).
     - Disabling the workflow temporarily (only if necessary, and document the change).

2. **Capture identifiers**
   - GitHub Actions Run ID (cd-prod)
   - Commit SHA and artifact version
   - Databricks job run IDs (deploy + post-deploy checks)
   - Workspace target (Prod)
   - Timestamp of deploy start/end

3. **Assess blast radius**
   - Which assets/jobs were affected?
   - Is this data correctness, availability, or performance?
   - Are downstream consumers impacted (reports, APIs, operational users)?

4. **Notify stakeholders**
   - Data Platform on-call / owner
   - Business owners of impacted domains
   - Source system owners if needed
   - If customer impact exists, follow incident communications process

---

## 2) Determine rollback type

### A) Deployment halted before changes applied
If deploy failed early and nothing material changed:
- Prefer **halt + fix** (no rollback needed)
- Re-run after correcting the issue

### B) Deployment applied changes
If bundle deploy succeeded and changes are active:
Choose one:

1. **Rollback** to previous known-good artifact (fastest risk reduction)
2. **Forward fix** (apply a new commit quickly) when rollback is risky or impossible
3. **Feature flag / disable jobs** to mitigate impact while investigating

---

## 3) Standard rollback strategy (preferred)
Goal: redeploy the last known-good artifact to restore stable production state.

### Step 1: Identify last known-good release
Sources of truth:
- GitHub Releases (if used)
- Prior successful `cd-prod` run in Actions
- Deployment audit table (if implemented)
- Databricks job history showing last successful deploy

Record:
- Prior commit SHA / tag
- Artifact ID/version
- Associated `ci-test` run that validated it

### Step 2: Deploy the prior artifact
Run the rollback deployment:
- Trigger `cd-prod` manually (workflow_dispatch) with the prior version if supported
- Or run a dedicated rollback workflow (recommended future enhancement)
- Ensure the deployment uses the prior artifact (no rebuild)

### Step 3: Execute post-rollback smoke tests
Minimum checks:
- Bundle resources exist and are correct
- Key jobs can start and complete
- Critical tables are readable
- No immediate freshness/schema regressions triggered

### Step 4: Confirm stability with monitoring
Confirm:
- Databricks job failures stop increasing
- Critical dashboards/consumers recover
- No new alerts firing

---

## 4) Databricks-specific mitigation actions

### Disable/stop scheduled jobs (containment)
If the issue is caused by scheduled jobs:
- Pause schedules in Databricks Jobs UI
- Or use CLI/API to disable triggers

### Cancel running jobs (containment)
- Cancel active runs for impacted jobs
- Avoid cancelling critical long-running jobs unless required

### Revert notebooks / bundle resources
If deployment changed notebook paths or job definitions:
- Redeploy previous bundle artifact
- Validate that the job definitions match prior version

---

## 5) Decision guidance: rollback vs forward fix

### Rollback is preferred when
- The change introduced data correctness issues
- Multiple downstream consumers are broken
- The root cause is not immediately obvious
- Time-to-restore is the priority

### Forward fix is preferred when
- Rollback is complex due to state changes (migrations, schema changes)
- The fix is well-understood and can be applied quickly
- The change is additive and reversible

### If schema changes occurred
Be cautious:
- If production schema migrations were applied, rollback may require additional migration steps.
- Consider an explicit rollback plan per migration type (additive vs destructive).

---

## 6) Data correctness validation after rollback/fix

Run at least:
- Verify critical assets freshness/watermark
- Verify schema contracts for critical tables
- Spot-check row counts for key tables
- Validate downstream reports/jobs can read data

If available:
- Execute a limited metadata-driven validation suite against Prod in “smoke mode”

---

## 7) Communication checklist

Minimum message contents:
- What happened (symptoms)
- Impact (who/what affected)
- Action taken (rollback/halt/forward fix)
- Current status (restored? monitoring?)
- Next update time
- Owner(s)

Example:
> Production deploy for commit <SHA> introduced schema mismatch on critical assets. Deploy halted and rolled back to <prior SHA>. Smoke tests passed; monitoring normalizing. Next update in 30 minutes. Owner: <name>.

---

## 8) Root cause analysis (post-stabilization)

### Required outputs
- Timeline of events (deploy start → detection → response → resolution)
- Root cause (technical + process)
- Contributing factors (testing gaps, missing approvals, insufficient checks)
- Corrective actions:
  - New test coverage
  - Updated gating policy
  - Improved rollback automation
  - Documentation updates

### Common corrective actions
- Add/strengthen post-deploy smoke tests
- Enforce stricter branch protections
- Add staged rollouts (canary/blue-green)
- Improve asset test thresholds and drift detection
- Implement automated rollback workflow

---

## 9) Preventing recurrence (recommended enhancements)
These are follow-on improvements that reduce rollback frequency:

- Maintain a “last-known-good” tag automatically after successful deploy
- Implement `workflow_dispatch` inputs for deploying a specific artifact version
- Add canary deployment in Prod:
  - deploy and validate subset of assets/jobs first
- Add automated rollback when critical smoke checks fail
- Add stronger contract tests for schema and consumers

---

## Related documents
- ../architecture.md
- ../branching-and-promotion.md
- ../security-and-secrets.md
- failed-ci-test.md
