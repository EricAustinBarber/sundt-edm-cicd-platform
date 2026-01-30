--------------------------------------------------------------------------

mkdir -p .github/workflows actions databricks docs examples
git add .
git commit -m "initial setup: initialize CI/CD platform repository structure"

--------------------------------------------------------------------------

touch docs/architecture.md
touch docs/branching-and-promotion.md
touch .github/CODEOWNERS