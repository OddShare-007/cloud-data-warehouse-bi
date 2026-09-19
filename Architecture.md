# Architecture

## Overview
The platform uses Amazon S3 as the landing layer, Amazon Redshift as the analytical warehouse, and a BI tool such as Amazon QuickSight or Power BI for exploration and dashboards.

## Data Flow
1. A source CSV or Parquet file is validated locally or by a Python ingestion utility.
2. The file is uploaded to an S3 landing prefix such as `raw/orders/`.
3. Redshift assumes a dedicated IAM role with read access to the required bucket prefix.
4. A `COPY` command loads the file into staging tables.
5. SQL transformations validate, deduplicate, and model data into analytics tables.
6. The BI tool connects to Redshift through a read-only reporting user or approved IAM-based access.
7. Dashboards query curated tables or views rather than raw files.

## Repository Structure
```text
cloud-data-warehouse-bi/
├── .github/workflows/ci.yml
├── data/sample/                 # Small, synthetic fixtures only
├── docs/
├── infra/
│   ├── modules/{s3,iam,redshift}/
│   └── environments/dev/
├── scripts/                    # Python loading and validation utilities
├── sql/
│   ├── ddl/
│   ├── staging/
│   ├── transformations/
│   └── analytics/
├── tests/
│   ├── python/
│   └── sql/
├── .gitignore
├── README.md
├── requirements.txt
└── Makefile
```

## Technology Stack
- Cloud: AWS.
- Storage: Amazon S3.
- Warehouse: Amazon Redshift provisioned cluster for the initial implementation; the design should allow later migration to Redshift Serverless.
- Infrastructure as code: Terraform with remote state recommended for shared environments.
- Data loading: Redshift `COPY`, with Python for validation, orchestration, and utilities.
- SQL: Redshift SQL.
- BI: Amazon QuickSight is the AWS-aligned default; Power BI or Apache Superset may be substituted if access is available.
- CI/CD: GitHub Actions.
- Testing: pytest, Terraform `fmt`/`validate`/plan, SQL linting or assertions.

## Security Boundaries
S3 is private. Redshift is not publicly accessible by default where feasible. Secrets are held in AWS Secrets Manager or environment-provided GitHub secrets. BI access is read-only. Synthetic data is required for the public repository.

## Operational Principles
Keep raw data immutable, make transformations repeatable, record load metadata, use explicit schemas, and make teardown straightforward.
