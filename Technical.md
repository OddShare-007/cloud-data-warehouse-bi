# Technical Specification

## Initial Deployment Profile
This is a learning and portfolio project, not a permanently running production warehouse. For the lowest-cost provisioned-cluster experiment, use a single-node `dc2.large` only if it is available in the chosen AWS region and current pricing is acceptable. Redshift pricing and free-tier eligibility change; verify current AWS pricing before deployment. Stop or destroy the cluster when not actively testing. If the account supports it, Redshift Serverless with a strict usage limit may be a better cost-control alternative, but the initial project remains Redshift-compatible.

Suggested development settings:
- Node count: 1.
- Node type: `dc2.large` for small synthetic datasets and basic BI queries.
- Database name: `analytics`.
- Port: 5439 unless an alternative is required.
- Encryption: enabled.
- Public accessibility: disabled where connectivity permits.
- Automated snapshots: minimal development retention.
- Tags: `Project=cloud-data-warehouse-bi`, `Environment=dev`, `ManagedBy=terraform`.

Do not present the cluster as free by default. Confirm regional pricing, account credits, and current AWS terms before creating it.

## IAM Roles and Access
### Redshift S3 Read Role
Trusted by `redshift.amazonaws.com`. Allow only `s3:GetObject` and `s3:ListBucket` for the project bucket and required prefixes. Add a bucket condition for the allowed prefix where supported.

### Deployment Role
Used by Terraform or GitHub OIDC. Grant only the resources and actions required for the environment, preferably through a dedicated deployment account or role. Avoid administrator access for routine CI.

### BI/Reporting Identity
Use a read-only database user or approved IAM-based database authentication. Grant `USAGE` on reporting schemas and `SELECT` on approved views/tables only.

### Loading Role
If Python runs outside Redshift, use a separate short-lived identity with S3 upload permission limited to the landing prefix and database permissions limited to loading staging tables.

## S3 Layout
```text
s3://<bucket>/raw/orders/orders-YYYY-MM-DD.csv
s3://<bucket>/processed/orders/load_date=YYYY-MM-DD/...
s3://<bucket>/manifests/orders/<load-id>.manifest
```
Keep raw objects immutable and use server-side encryption. Never upload real personal or healthcare data.

## Schemas
- `staging`: files as received, with load metadata.
- `warehouse`: conformed dimensions and facts.
- `reporting`: stable views designed for analysts and BI.

## Sample Business Dataset
The sample domain is online retail order analytics.

### `warehouse.dim_customer`
| Column | Type | Notes |
|---|---|---|
| customer_key | BIGINT | Surrogate key |
| customer_id | VARCHAR(50) | Source identifier |
| customer_segment | VARCHAR(50) | Segment label |
| city | VARCHAR(100) | Synthetic location |
| state | VARCHAR(100) | Synthetic location |
| signup_date | DATE | Customer signup date |
| is_current | BOOLEAN | Current dimension record |

### `warehouse.dim_product`
| Column | Type | Notes |
|---|---|---|
| product_key | BIGINT | Surrogate key |
| product_id | VARCHAR(50) | Source identifier |
| product_name | VARCHAR(200) | Product name |
| category | VARCHAR(100) | Product category |
| unit_cost | DECIMAL(12,2) | Cost basis |
| list_price | DECIMAL(12,2) | List price |

### `warehouse.fact_order`
| Column | Type | Notes |
|---|---|---|
| order_key | BIGINT | Warehouse key |
| order_id | VARCHAR(50) | Source order identifier |
| customer_key | BIGINT | Customer dimension key |
| product_key | BIGINT | Product dimension key |
| order_date | DATE | Order date |
| quantity | INTEGER | Units ordered |
| discount_amount | DECIMAL(12,2) | Discount |
| net_sales | DECIMAL(12,2) | Sales after discount |
| order_status | VARCHAR(30) | Completed, cancelled, returned |
| loaded_at | TIMESTAMP | Warehouse load time |

## Example Load SQL
```sql
COPY staging.orders
FROM 's3://<bucket>/raw/orders/orders-YYYY-MM-DD.csv'
IAM_ROLE '<redshift-s3-read-role-arn>'
CSV
IGNOREHEADER 1
TIMEFORMAT 'auto'
DATEFORMAT 'auto'
TRUNCATECOLUMNS
EMPTYASNULL
BLANKSASNULL;
```
Use a manifest for multi-file loads and validate row counts before publishing data.

## BI Queries
### Monthly sales
```sql
SELECT DATE_TRUNC('month', order_date) AS month,
       SUM(net_sales) AS sales,
       SUM(quantity) AS units
FROM reporting.order_sales
WHERE order_status = 'Completed'
GROUP BY 1
ORDER BY 1;
```

### Sales by category
```sql
SELECT p.category,
       SUM(f.net_sales) AS sales,
       COUNT(DISTINCT f.order_id) AS orders
FROM warehouse.fact_order f
JOIN warehouse.dim_product p ON p.product_key = f.product_key
WHERE f.order_status = 'Completed'
GROUP BY p.category
ORDER BY sales DESC;
```

### Customer repeat rate
```sql
WITH customer_orders AS (
  SELECT customer_key, COUNT(DISTINCT order_id) AS order_count
  FROM warehouse.fact_order
  WHERE order_status = 'Completed'
  GROUP BY customer_key
)
SELECT AVG(CASE WHEN order_count > 1 THEN 1.0 ELSE 0.0 END) AS repeat_customer_rate
FROM customer_orders;
```

### Data-quality check
```sql
SELECT COUNT(*) AS invalid_rows
FROM warehouse.fact_order
WHERE quantity <= 0 OR net_sales < 0 OR order_date IS NULL;
```

## Performance and Operations
Start with simple sort keys on date-oriented fact tables and distribution choices appropriate to the small dataset. Revisit sort/distribution design only after measuring query plans and data volume. Record load IDs, source paths, row counts, rejected rows, and timestamps. Schedule cleanup and cluster teardown to prevent unexpected charges.
