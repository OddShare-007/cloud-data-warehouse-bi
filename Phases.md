# Implementation Phases

## Phase 1 — Redshift Cluster Setup
- Choose a low-cost development configuration.
- Create the VPC, subnet/security-group strategy, cluster, database, and parameter settings.
- Verify connectivity without exposing unnecessary public access.
- Add cost warnings and a destroy procedure.

## Phase 2 — S3 to Redshift Loading
- Create private S3 bucket and landing prefixes.
- Define file format, naming, and manifest conventions.
- Create the Redshift S3-read IAM role.
- Implement Python validation and documented `COPY` commands.
- Add load logging and retry guidance.

## Phase 3 — Table and Schema Design
- Create staging, warehouse, and reporting schemas.
- Define fact and dimension tables.
- Implement deduplication, type conversion, and quality checks.
- Add representative views for BI consumption.

## Phase 4 — BI/Dashboard Connection
- Select Amazon QuickSight as the default BI tool, subject to account availability.
- Create a read-only reporting access path.
- Connect to curated views and build KPI, trend, and category dashboards.
- Document refresh behavior and access setup.

## Phase 5 — Infrastructure as Code with Terraform
- Refactor resources into reusable modules.
- Add variables, outputs, tagging, validation, and environment separation.
- Add remote state guidance and safe lifecycle settings.
- Test plan/apply/destroy in a development account.

## Phase 6 — CI/CD with GitHub Actions
- Run Python tests, formatting, Terraform checks, SQL checks, and documentation checks on pull requests.
- Use GitHub OIDC rather than long-lived AWS keys where deployment automation is enabled.
- Keep apply gated by protected branches and explicit environment approval.

## Phase 7 — Documentation and Demo
- Complete README quickstart, architecture diagram, data dictionary, cost notes, and troubleshooting.
- Record a short demo from upload through dashboard query.
- Include sample outputs and teardown confirmation.
- Review the repository for secrets and sensitive data before publishing.
