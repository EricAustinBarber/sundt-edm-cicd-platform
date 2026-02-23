# Security Policy

## Supported Versions

Only the default branch (`main`) is supported for security updates.

## Reporting a Vulnerability

Report vulnerabilities privately through one of these paths:

- GitHub private vulnerability reporting for this repository
- Direct notification to the Data Platform and DevOps maintainers

Include:

- Affected file/path or workflow
- Reproduction steps
- Impact assessment
- Suggested mitigation (if known)

Do not open public issues for unpatched security vulnerabilities.

## Security Expectations for This Repository

- Reusable workflows must run with least-privilege permissions.
- Secrets must be sourced from GitHub Environments or external vaults, never committed.
- Production deployments must be approval-gated through protected branches and environments.
- Third-party action and tool versions should be pinned and regularly updated.
