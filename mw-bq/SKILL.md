---
name: mw-bq
description: BigQuery data exploration, querying, and optimization with mandatory confirmation gates. Use when working with BigQuery data, writing BQ SQL queries, exploring dataset schemas, profiling tables, sampling data (top 10s, head), generating reports, switching GCP projects, estimating query costs, or optimizing query performance. Triggers on tasks involving BigQuery, BQ, data analysis, SQL queries against BQ datasets, schema exploration, table profiling, cost estimation, or query optimization. Do NOT use for Athena queries (use jet-aws-athena or jet-odl-athena instead).
---

# BigQuery Skill

## Important

- **NEVER run a query without user confirmation.** Always show the query, estimate cost via `--dry_run`, present the estimate, and wait for explicit approval before executing.
- **NEVER assume column values from names alone.** When exploring unfamiliar data, always sample actual values first (`bq head` or `LIMIT 10`) before writing filters or aggregations.
- **ALWAYS run the Schema-First Protocol before writing any query.** No exceptions — even if you think you know the table. Check schema, survey values, then decide. See the Schema-First Protocol section below.
- **Always use Standard SQL** (`--nouse_legacy_sql`).
- **Always filter on partition columns** to control cost.

## Confirmation Protocol

Every query execution follows this sequence — no exceptions:

### Step 1: Draft the query

Write the SQL and show it to the user.

### Step 2: Dry-run for cost estimate

```bash
bq query --dry_run --nouse_legacy_sql 'YOUR QUERY'
```

### Step 3: Present estimate and ask

Report the bytes scanned and approximate cost using this table:

| Bytes Scanned | Approximate Cost |
| ------------- | ---------------- |
| 1 GB          | $0.005           |
| 10 GB         | $0.05            |
| 100 GB        | $0.50            |
| 1 TB          | $5.00            |
| 10 TB         | $50.00           |

Format the confirmation request as:

> **Query will scan ~X GB (~$Y).** Run it?

### Step 4: Wait for confirmation

Do NOT proceed until the user explicitly confirms. If the user says "go", "yes", "run it", "do it" — then execute. Otherwise, revise or abandon.

### Exceptions (no confirmation needed)

These operations are safe and can run without asking:

- `bq head -n N dataset.table` (free, reads no data via query engine)
- `bq show dataset.table` / `bq show --schema` (metadata only)
- `bq ls` (listing datasets/tables)
- `gcloud config` commands
- `--dry_run` queries (estimation only, no cost)

## Schema-First Protocol (Mandatory)

**This protocol is mandatory for every BQ task — no exceptions.** Even if you have worked with the table before in this session, even if the user tells you the column names, even if the query looks obvious. Always verify the actual data before deciding what to do.

Do NOT skip steps. Do NOT jump straight to writing a query. The sequence is: **schema → values → plan → query.**

### Step 1: Discover tables

If the dataset or table is not already confirmed:

```bash
bq ls dataset_name
```

### Step 2: Inspect schema

**Always run this.** Understand every column's name, type, and mode before touching the data.

```bash
bq show --schema --format=prettyjson dataset.table
```

Also check partitioning and clustering (needed for cost-efficient queries):

```bash
bq show --format=prettyjson dataset.table | grep -E '"timePartitioning"|"clustering"|"numRows"|"numBytes"'
```

### Step 3: Survey actual values

**Always run this.** Column names lie. A column called `status` might contain codes, enums, NULLs, or unexpected formats. You must look at real data before deciding how to filter, group, or aggregate.

```bash
# Free peek — no query cost, no confirmation needed
bq head -n 10 dataset.table
```

For specific columns relevant to the task:

```bash
bq head -n 10 --selected_fields=col1,col2,col3 dataset.table
```

If `bq head` doesn't give enough context (e.g., need distinct values, distributions, or filtered samples), use a small query — but still follow the confirmation protocol:

```sql
-- Sample distinct values of a key column
SELECT DISTINCT column_name, COUNT(*) AS cnt
FROM `project.dataset.table`
WHERE partition_col >= DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)
GROUP BY 1
ORDER BY 2 DESC
LIMIT 20
```

### Step 4: Summarize findings and validate assumptions

Present what you learned from schema + sampling to the user before writing any query:

> **Schema check:** Table has X columns, partitioned on `date_col`, clustered on `region`.
> **Value survey:** Column `status` contains: `'completed'` (80%), `'cancelled'` (15%), `'pending'` (5%). Column `amount` ranges from 0.01 to 9999.99.
> **Plan:** I'll filter on `status = 'completed'` and partition date >= last 7 days. Does this match what you need?

Wait for confirmation before proceeding.

### Step 5: Build the query

Only after understanding the actual data shape AND getting user confirmation on your plan, write the full query — then follow the Confirmation Protocol above (dry-run → cost estimate → approval → execute).

## When to Re-run the Schema-First Protocol

- **New table:** Always.
- **Same table, new question:** Re-run Step 3 (survey values) for any columns you haven't inspected yet.
- **Same table, same columns, follow-up query:** You may skip — the data shape is already known from this session.

## Prerequisites

Before using BigQuery, verify the environment:

```bash
bq version
gcloud config get-value project
gcloud auth list
```

If `bq` is not installed:

```bash
# macOS
brew install google-cloud-sdk

# Then authenticate
gcloud auth login
gcloud auth application-default login
gcloud config set project PROJECT_ID
```

Required IAM: `roles/bigquery.user` (query), `roles/bigquery.dataViewer` (schema), `roles/bigquery.dataEditor` (create/modify).

## Discovery

### List datasets

```bash
bq ls
bq ls --project_id=OTHER_PROJECT
```

### List tables in a dataset

```bash
bq ls dataset_name
bq ls project_id:dataset_name
```

### Show table metadata

```bash
bq show dataset.table
bq show --format=prettyjson dataset.table
```

### Show table schema

```bash
bq show --schema --format=prettyjson dataset.table
```

### Show row count and size

```bash
bq show --format=prettyjson dataset.table | grep -E '"numRows"|"numBytes"'
```

## Data Sampling

### Quick top-N sample (free — no confirmation needed)

```bash
bq head -n 10 dataset.table
```

### Sample via query (follows confirmation protocol)

```bash
bq query --nouse_legacy_sql \
  'SELECT * FROM `project.dataset.table` LIMIT 10'
```

### Sample with specific columns

```bash
bq query --nouse_legacy_sql \
  'SELECT col1, col2, col3 FROM `project.dataset.table` LIMIT 10'
```

## Running Queries

Always use Standard SQL:

```bash
bq query --nouse_legacy_sql 'YOUR SQL HERE'
```

### Useful flags

| Flag                  | Purpose                                  |
| --------------------- | ---------------------------------------- |
| `--nouse_legacy_sql`  | Use Standard SQL (always include)        |
| `--format=prettyjson` | JSON output                              |
| `--format=csv`        | CSV output                               |
| `--format=sparse`     | Compact table output                     |
| `--max_rows=N`        | Limit output rows displayed              |
| `--dry_run`           | Estimate bytes scanned without executing |

### Multi-line queries

```bash
bq query --nouse_legacy_sql '
  SELECT
    DATE(created_at) AS day,
    COUNT(*) AS total
  FROM `project.dataset.table`
  WHERE created_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
  GROUP BY day
  ORDER BY day DESC
'
```

## Cost Estimation

**Always dry-run before executing:**

```bash
bq query --dry_run --nouse_legacy_sql 'YOUR QUERY'
```

If a query scans > 1 TB, warn the user prominently and suggest optimizations before even offering to run.

## Project & Dataset Switching

```bash
# Switch default project
gcloud config set project NEW_PROJECT_ID

# Query a different project without switching
bq query --project_id=OTHER_PROJECT --nouse_legacy_sql \
  'SELECT * FROM `other_project.dataset.table` LIMIT 10'

# List available projects
gcloud projects list
```

## Query Writing Guidelines

1. **Always use Standard SQL** — never legacy SQL
2. **Always filter on partition columns** — reduces cost and improves speed
3. **Select only needed columns** — avoid `SELECT *` on wide tables
4. **Use `LIMIT` during exploration** — add LIMIT when sampling data
5. **Prefer `APPROX_COUNT_DISTINCT`** over `COUNT(DISTINCT x)` for large tables
6. **Use `IF`/`COUNTIF`/`SUMIF`** instead of CASE+aggregate for conditional aggregation
7. **Qualify all table references** — use `project.dataset.table` format
8. **Use backtick quoting** — wrap table refs in backticks: `` `project.dataset.table` ``
9. **Always explain JOIN choice after writing a query.** Never silently use `JOIN` vs `LEFT JOIN` without noting which was used and why. After presenting a query, add a note like:
   > **JOIN note:** Used `LEFT JOIN` — keeps all rows from the left table; unmatched rows appear as `NULL` and are included in totals, so percentages reflect the full population. Used `INNER JOIN` — excludes unmatched rows; if N% of rows have no match, they disappear silently and all percentages are calculated over a smaller denominator.

For advanced SQL patterns (window functions, CTEs, unnesting, pivots, funnels, cohorts), see [references/query-patterns.md](references/query-patterns.md).

## Optimization

Key rules:

1. **Dry-run first** — always `--dry_run` queries that might scan large volumes
2. **Partition filtering** — always include partition column in WHERE clause
3. **Cluster column filtering** — use clustered columns in WHERE/JOIN for better pruning
4. **Avoid SELECT \*** — select only required columns
5. **Use EXISTS over IN** — `WHERE EXISTS (SELECT 1 ...)` is faster than `WHERE x IN (SELECT ...)`
6. **Reduce data before JOINs** — filter/aggregate in CTEs before joining
7. **Avoid repeated reads** — use CTEs or temp tables for data read multiple times
8. **Use approximate functions** — `APPROX_COUNT_DISTINCT`, `APPROX_QUANTILES`, `APPROX_TOP_COUNT`

For comprehensive optimization guidance, see [references/optimization-guide.md](references/optimization-guide.md).

## Output & Reporting

### Export results to CSV

```bash
bq query --nouse_legacy_sql --format=csv '...' > output.csv
```

### Save results to a destination table

```bash
bq query --nouse_legacy_sql \
  --destination_table=dataset.results_table \
  --replace \
  'SELECT ...'
```

### Export a table to GCS

```bash
bq extract --destination_format=CSV \
  dataset.table gs://bucket/path/file.csv
```

When generating reports, format results as markdown tables for readability.

## Reference Material

- **CLI commands**: [references/bq-cli-reference.md](references/bq-cli-reference.md)
- **SQL patterns**: [references/query-patterns.md](references/query-patterns.md)
- **Optimization**: [references/optimization-guide.md](references/optimization-guide.md)
