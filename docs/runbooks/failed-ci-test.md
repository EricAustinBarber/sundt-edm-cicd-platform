# Runbook: Failed ci-test

## Purpose
This runbook describes how to triage and remediate failures in the `ci-test` workflow (integration + metadata-driven asset validation).

Use this when:
- A commit to `test` fails `ci-test`
- A PR to `prod` is blocked by required `ci-test` checks
- A deployment to Test workspace completes but asset validation fails

---

## Quick decision tree

```
ci-test failed
  |
  +-- Did the workflow fail before Databricks deploy?
  |      |
  |      +-- YES → Build/Packaging issue (Section 2)
  |      |
  |      +-- NO  → Proceed
  |
  +-- Did bundle deploy fail?
  |      |
  |      +-- YES → Deployment issue (Section 3)
  |      |
  |      +-- NO  → Proceed
  |
  +-- Did asset validation fail?
         |
         +-- YES → Data/Connectivity/Schema/Freshness issue (Section 4)
         |
         +-- NO  → Flaky/Infra issue (Section 5)
```

---

## 1) First response checklist (do this immediately)

1. Identify the failing job and step in GitHub Actions:
   - Workflow: `ci-test`
   - Job: `ci-test / build`, `ci-test / bundle-deploy-test`, or `ci-test / asset-validation`

2. Capture identifiers for audit and correlation:
   - GitHub Run ID
   - Commit SHA
   - Branch (`test`)
   - Databricks Job Run ID (if applicable)

3. Check whether this failure blocks production promotion:
   - If a PR to `prod` is waiting, notify PR owner with failure category and next action.

4. Determine whether the failure is:
   - Code/config regression
   - Credential/secret issue
   - Source system outage
   - Databricks infrastructure/transient issue
   - Metadata configuration issue (bad sample query, wrong columns, etc.)

---

## 2) Build/Packaging failures (before Databricks)

### Symptoms
- Zip/archive step fails
- Artifact upload/download fails
- Unit tests or lint fail

### Actions
1. Check repo paths referenced in workflow:
   - Confirm expected folders exist (e.g., `notebooks/` or bundle definitions)
2. Re-run locally (fast):
   - Run lint/unit tests locally if applicable
3. Confirm artifact outputs:
   - Ensure expected build output exists and is included in upload step
4. Fix and push:
   - A new commit to `test` will re-run `ci-test`

### Common root causes
- Path changes without workflow update
- Dependency lockfile drift
- Runner image toolchain mismatch

---

## 3) Bundle deploy failures (Databricks deploy step)

### Symptoms
- `databricks bundle validate` fails
- `databricks bundle deploy` fails
- Permission denied errors in workspace

### Actions
1. Confirm the target workspace and environment:
   - Ensure workflow is deploying to Test workspace, not Dev/Prod
2. Check authentication:
   - Verify GitHub Environment secrets exist for Test
   - If using OIDC, confirm federation is valid and not misconfigured
3. Validate bundle locally (if possible):
   - `databricks bundle validate`
4. Check Databricks permissions:
   - Workspace-level: deploy identity must have rights to create/modify jobs, upload notebooks, create resources
5. Re-run deploy step after corrections

### Common root causes
- Expired or revoked token
- Missing permissions (jobs, workspace, UC catalog)
- Invalid bundle config change

---

## 4) Asset validation failures (data quality / source issues)

Asset validation failures are usually one of:
- Connectivity/authentication
- Schema drift
- Freshness/watermark failure
- Sample query failure
- Null/uniqueness threshold failure

### 4.1 Connectivity/Auth failures

#### Symptoms
- JDBC connection failures
- Authentication errors
- Timeout / network unreachable

#### Actions
1. Determine secret source:
   - Is the credential stored in Key Vault or GitHub Environment?
2. Validate secret reference in metadata:
   - Metadata should contain secret name (e.g., `sec-e1-connection-string`), not the value
3. Validate Key Vault access:
   - Confirm the CI identity has read access to the secret
4. Confirm source system availability:
   - Check known maintenance windows/outages
5. Retry once if transient; escalate if persistent

#### Escalate to
- Source system owners
- Network/infrastructure team (VNET/firewall/DNS)
- Security team (credential access)

---

### 4.2 Schema drift failures

#### Symptoms
- Missing expected columns
- Column type incompatibilities
- Unexpected columns (if enforced)

#### Actions
1. Confirm whether schema drift is expected:
   - Source system change?
   - New columns added?
2. Validate metadata entry:
   - Update expected columns list if drift is approved
3. If drift is not expected:
   - Block promotion and escalate to source owners
4. For urgent issues:
   - Use a time-bound waiver (see Section 6) with documented justification

---

### 4.3 Freshness/watermark failures

#### Symptoms
- Max timestamp older than threshold
- Watermark column null or not advancing

#### Actions
1. Confirm schedule expectations:
   - Is the asset supposed to be daily/hourly?
2. Confirm watermark column correctness:
   - Metadata may reference wrong column
3. Check upstream ingestion:
   - Did the upstream job complete?
4. If ingestion delayed:
   - Coordinate with upstream owners
   - Consider extending threshold temporarily (time-bound)

---

### 4.4 Sample query failures

#### Symptoms
- SQL syntax errors
- Query returns 0 rows unexpectedly
- Permission/lock errors

#### Actions
1. Validate query text in metadata:
   - Check for environment-specific references
   - Ensure query is compatible with source type
2. Run query manually in source (or via Databricks with same credential)
3. Fix query or adjust expectations
4. Re-run validation

---

### 4.5 Null/Uniqueness threshold failures

#### Symptoms
- Null percentage exceeds threshold
- Duplicate keys found
- Counts fall outside expected ranges

#### Actions
1. Confirm if data is legitimately degraded
2. Validate thresholds are realistic
3. If data regression:
   - Block promotion
   - Open incident/issue with source owner
4. If threshold too strict:
   - Adjust with review
   - Record change and rationale

---

## 5) Flaky / infrastructure failures

### Symptoms
- Databricks job cancelled
- Cluster failed to start
- Intermittent network timeouts
- CI runner transient errors

### Actions
1. Re-run failed workflow once
2. If repeated:
   - Check Databricks cluster event logs
   - Check workspace status page / known incidents
3. Reduce concurrency if source is overloaded
4. Add bounded retry with backoff for known transient steps

---

## 6) Waivers and overrides (use sparingly)

Waivers are allowed only when:
- Issue is understood
- Risk is accepted by appropriate approver
- Waiver is time-bound and documented

Waiver must include:
- Asset IDs affected
- Failure category (schema/freshness/etc.)
- Business justification
- Expiration date and owner

Implementation options:
- Allowlist table for waived assets with expiry
- GitHub Environment manual approval step for exception deployments

---

## 7) Communication template (internal)

Include:
- Commit SHA
- GitHub Run ID link
- Failure category
- Affected assets (IDs + names)
- Next action and ETA
- Escalation owner

Example message:
> ci-test failed for commit <SHA> (Run <ID>) due to schema drift on asset IDs 204, 395. Promotion to prod is blocked. Investigating source change in PRODDTA.F0111; next update in 1 hour. Owner: <name>.

---

## 8) Post-incident follow-up

For repeated failures:
- Create or update a backlog item for:
  - Better validation thresholds
  - Improved retries / timeouts
  - Reduced scan costs (sampling)
  - Metadata hygiene checks
- Add regression tests to prevent recurrence

---

## Related documents
- ../architecture.md
- ../branching-and-promotion.md
- ../security-and-secrets.md
