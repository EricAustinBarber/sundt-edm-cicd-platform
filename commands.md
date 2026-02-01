--------------------------------------------------------------------------

mkdir -p .github/workflows actions databricks docs examples
git add .
git commit -m "initial setup: initialize CI/CD platform repository structure"

--------------------------------------------------------------------------

touch docs/architecture.md
touch docs/branching-and-promotion.md
ECHO "@sundt-data-platform @sundt-devops" > .github/CODEOWNERS

git add .
git commit -m "initial setup: architecture.md branching-and-promoting.md CODEOWNERS"

--------------------------------------------------------------------------

git add docs/architecture.md
git commit -m "docs: add CI/CD architecture for GitHub and Databricks"

--------------------------------------------------------------------------

git add docs/security-and-secrets.md
git commit -m "docs: define security and secrets management model"

--------------------------------------------------------------------------

git add docs/branching-and-promotion.md
git commit -m "docs: define branching and promotion rules for CI/CD"

--------------------------------------------------------------------------

git add docs/branching-and-promotion.md
git commit -m "docs: define branching and promotion rules for CI/CD"

--------------------------------------------------------------------------

git add docs/runbooks/prod-rollback.md
git commit -m "docs: add production rollback runbook"

--------------------------------------------------------------------------

git add docs/onboarding.md
git commit -m "docs: add onboarding guide for CI/CD platform adoption"

--------------------------------------------------------------------------

git add .github/workflows/reusable-databricks.yml
git commit -m "ci: add reusable workflow for Databricks bundle deploy + asset validation"

--------------------------------------------------------------------------

mkdir -p examples/consuming-repo/.github/workflows
git add examples/consuming-repo/.github/workflows/ci.yml
git commit -m "examples: add consuming repo workflow calling reusable databricks workflow"