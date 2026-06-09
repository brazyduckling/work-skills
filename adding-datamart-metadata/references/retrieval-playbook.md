# Retrieval Playbook

Use this file when the skill needs to know exactly how to retrieve the latest glossary, live schema, and current wiring state.

## Prerequisites

Required:
- access to the target repo or a copy of the relevant files
- ability to read and search files
- `git` if you need the latest shared baseline from the remote default branch

Optional:
- `bq` and `gcloud` for live BigQuery inspection
- `gh` if the user wants to inspect an example PR directly from GitHub
- `jet-bq` if the environment supports approved installable skills

If a tool is missing, do not pretend the result is complete. State the limitation and switch to the safest fallback.

If the environment supports skills:
- prefer `jet-bq` for BigQuery inspection
- do not rely on a user-specific personal `bq` skill
- if `jet-bq` cannot be found, tell the user to download `jet-bq` from the JET-approved skills source

## 1. Get the latest shared glossary

If the target repo is a git repo:

```bash
git fetch --all --prune
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@')
git show origin/$DEFAULT_BRANCH:fintech_data_analytics_datamarts/ftdna_macros.jinja
```

Use this as the default shared-glossary baseline unless the team explicitly says to use a different base branch.

If the repo does not use the standard FTDNA glossary path, ask the user for the glossary file path. Do not assume a different path.

If the repo is not available as a git checkout, read the local glossary file the user points to and clearly say that the result may not reflect the latest shared baseline.

## 2. Get the current table SQL

Inspect the table folder in the target repo and read the main SQL file:

```text
fintech_data_analytics_datamarts/<table-folder>/<table-name>.sql
```

Use the SQL output columns as the schema source of truth when there is no live BigQuery table yet.

If the repo layout differs, ask the user for the exact SQL file path.

## 3. Check for an existing DDL file

Look for:

```text
fintech_data_analytics_datamarts/<table-folder>/<table-name>_ddl.sql
```

If it exists, read it and compare its column order with the current SQL or live BigQuery schema.

If it does not exist, treat that as missing DDL wiring, not as an error.

## 4. Check whether config is already wired

Read:

```text
fintech_data_analytics_datamarts/<table-folder>/config.json
```

Treat the datamart as already wired only if the config contains a separate `*_ddl` task with:

- `is_dml: true`
- `is_common_template: true`
- dependency on the main table task

If those are missing, prepare a new DDL task block.

## 5. Get the live BigQuery schema

If `jet-bq` is installed, use it first.

Use metadata-safe commands first:

```bash
bq show --schema --format=prettyjson project:dataset.table
bq show --format=prettyjson project:dataset.table | grep -E '"timePartitioning"|"clustering"|"numRows"|"numBytes"'
```

This gives:

- live column names
- live types
- partition field
- clustering
- table size

If `bq` is unavailable, say that live schema inspection could not be done and fall back to the SQL file.

## 6. Sample real values

If `jet-bq` is installed, use it first.

Use free sampling first:

```bash
bq head -n 10 project:dataset.table
bq head -n 10 --selected_fields=col1,col2,col3 project:dataset.table
```

Use this for:

- enum-like fields
- booleans
- status fields
- payment methods
- country or tenant codes

If that is not enough, prepare a small partition-filtered query, dry-run it, show the cost, and ask before running it.

If the user does not approve a query, keep the proposal conservative and say which allowed-value fields are based only on free sampling.

## 7. Decide which source wins

Use this order:

1. Live BigQuery schema, if the table already exists
2. Current SQL file, if the table is not live yet
3. Existing DDL file, only as a comparison input, not as the default truth

For glossary reuse, use the latest shared glossary baseline from the repo base branch, not just the current local working tree.

## 8. Example PR retrieval

Only do this if the user asks for PR inspection or gives a PR link or number.

If `gh` is available:

```bash
gh pr view <PR_NUMBER> --json title,files,url
```

If `gh` is not available, ask the user to provide:
- the changed file list
- the PR diff
- or the relevant files directly
