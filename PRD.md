# Product Requirements Document

## Project
Cloud Data Warehouse and BI

## Problem Statement
Small teams often have operational data in files and cloud storage but lack a reliable, queryable analytical layer. This project creates a low-cost AWS Redshift warehouse that ingests curated files from Amazon S3 and makes them available for repeatable analytics and BI dashboards.

## Goals
- Build a reproducible Redshift-based analytical warehouse.
- Load realistic business data from S3 through documented, testable Python workflows.
- Provide secure SQL access for BI analysis and dashboards.
- Demonstrate production-minded data engineering practices without unnecessary complexity.

## Target Users
- Data engineers building and maintaining ingestion and infrastructure.
- Analysts querying modeled warehouse tables.
- Business stakeholders consuming dashboards.
- Portfolio reviewers evaluating cloud, SQL, IaC, and BI skills.

## Core Features
- S3 landing zone with documented file conventions.
- Redshift cluster or serverless-compatible warehouse configuration.
- IAM role allowing Redshift to read only the required S3 prefix.
- Staging and analytics schemas.
- Python validation/loading utilities where appropriate.
- Terraform-managed infrastructure.
- BI connection and at least one dashboard.
- GitHub Actions checks and clear setup documentation.

## Non-Goals
- Real-time streaming.
- Enterprise multi-region disaster recovery.
- Production-scale workload management.
- Storage of sensitive personal or healthcare data.

## Success Criteria
- A new user can provision the environment from documented steps.
- A sample dataset can be uploaded and loaded reproducibly.
- Core analytical queries execute successfully.
- The BI tool can connect using least-privilege credentials and display useful KPIs.
- Terraform validation, Python tests, SQL checks, and documentation checks pass in CI.
- All resources can be destroyed cleanly to control cost.

## Constraints and Risks
- Redshift pricing can continue while a cluster is running; default to the smallest practical test configuration and destroy it when idle.
- Credentials must remain outside Git and code.
- Network, IAM, and cleanup steps must be documented before public release.
