# Rules for the AI Coding Assistant

## Infrastructure
- Use Terraform for every AWS resource; do not create infrastructure manually as the source of truth.
- Keep reusable resources in modules and environment-specific values in environment configuration.
- Run `terraform fmt`, `terraform validate`, and a plan before proposing infrastructure changes.
- Never commit Terraform state, plans containing secrets, generated credentials, or provider cache files.

## Data and Python
- Use Python for data loading, validation, file manifests, and orchestration utilities.
- Keep Python modules small, typed where practical, deterministic, and testable.
- Validate column names, types, nullability, and required fields before loading.
- Make loads idempotent or document the deduplication strategy.
- Use synthetic, non-sensitive data in examples and fixtures.

## Security
- Never hardcode AWS keys, passwords, tokens, endpoints containing secrets, or private connection strings.
- Prefer IAM roles, short-lived credentials, AWS Secrets Manager, and GitHub OIDC.
- Apply least-privilege IAM with resource and prefix conditions.
- Separate deployment, loading, analyst, and BI permissions.
- Do not make S3 or Redshift public merely to simplify a demo.

## SQL and Warehouse Design
- Use explicit schemas, column types, constraints where supported, and naming conventions.
- Separate staging from curated analytics tables.
- Prefer incremental, auditable transformations over destructive ad hoc edits.
- Add load timestamps, source identifiers, and data-quality checks.
- Keep SQL parameterized or safely templated; never concatenate untrusted values.

## Quality
- Add tests for Python utilities and representative data-quality rules.
- Document assumptions, cost controls, cleanup, and failure recovery.
- Preserve backward compatibility unless a migration is included.
- Explain changes in the pull request and update documentation with behavior changes.
