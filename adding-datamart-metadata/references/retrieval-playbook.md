# Retrieval Playbook

Use this file when the skill needs to know exactly how to retrieve the latest glossary, live schema, and current wiring state.

## 0. Resolve the local checkout first

If the user gives only a repo name:

- search common local clone locations on the laptop
- keep only matching git checkouts
- confirm the chosen path with the user before using it

Use GitHub only if:

- no local clone is found
- or the user explicitly asks for GitHub or PR inspection

## 0a. Gather contextual evidence

If the user supplies any of these, collect them before drafting descriptions:

- PR links or review threads
- transcripts or meeting notes
- process docs, decks, or Confluence pages

Use them for business meaning and reader-context evidence.

- Review comments tell you where wording is confusing or misleading.
- Thread replies and user notes often explain the intended meaning in plain language.
- Docs and transcripts can explain value semantics or source-system context that SQL alone does not.

Do not let these override schema or raw value evidence on their own. Use them to explain the field, not to guess the data shape.

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
- if `jet-bq` cannot be found, tell the user to download `jet-bq` from the JET-approved skills source

## 1. Get the latest shared glossary

If the target repo is a local git checkout:

```bash
git show origin/master:fintech_data_analytics_datamarts/ftdna_macros.jinja
```

First, ask the user to refresh the local repo against `origin/master`, or confirm that it is already up to date.

Then use this `origin/master` file as the default shared-glossary baseline unless the team explicitly says to use a different base branch.

Read the local working-tree glossary separately only when you need branch-under-change evidence. Do not treat the local working tree as the shared baseline.

If the repo does not use the standard FTDNA glossary path, ask the user for the glossary file path. Do not assume a different path.

If the repo is not available as a git checkout, read the local glossary file the user points to and clearly say that the result may not reflect the latest shared baseline.

## 2. Get the current table SQL

Inspect the table folder in the target repo and read the main SQL file:

```text
fintech_data_analytics_datamarts/<table-folder>/<table-name>.sql
```

Use the SQL output columns as the schema source of truth when there is no live BigQuery table yet.

If the repo layout differs, ask the user for the exact SQL file path.

If the table emits nested `STRUCT` / `RECORD` fields:

- inventory the full dotted paths
- keep the parent-child order
- note which groups are plain `RECORD` and which are `REPEATED`
- do not flatten the schema mentally while drafting metadata

## 3. Check for an existing DDL file

Look for:

```text
fintech_data_analytics_datamarts/<table-folder>/<table-name>_ddl.sql
```

If it exists, read it and compare its column order with the current SQL or live BigQuery schema.

If it does not exist, treat that as missing DDL wiring, not as an error.

If the DDL contains schema objects with nested `fields` arrays:

- inspect the whole nested object, not just the top-level field names
- note whether descriptions live inline on parent groups, child fields, or both
- inspect any helper macro the DDL uses to render nested schema definitions

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
- nested `fields` arrays when the table contains `RECORD` columns

When preparing a DDL file, preserve the existing partition field and clustering spec when they are present. If the table already exists and is partitioned, make sure the DDL keeps the partitioning spec instead of recreating the table without it.

If the live schema contains nested fields:

- preserve the full nested shape
- preserve each child field's type, mode, and order
- use the schema JSON as the winning source for nested structure

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
- safe example values in proposed descriptions
- sample child-field values inside nested structures when the table is already live

If that is not enough, prepare a small partition-filtered query, dry-run it, show the cost, and ask before running it.

If the user does not approve a query, keep the proposal conservative and say which allowed-value fields and example values are based only on free sampling.

## 7. Decide which source wins

Use this order for schema and value shape:

1. Live BigQuery schema, if the table already exists
2. Current SQL file, if the table is not live yet
3. Existing DDL file, only as a comparison input, not as the default truth

Use this order for business wording and reviewer context:

1. Linked process docs, Confluence pages, or transcripts that explicitly explain the concept
2. PR review comments and thread replies that show where readers were confused or where names are misleading
3. Local working-tree wording only as a draft under review, not as the source of truth

For glossary reuse, use the refreshed `origin/master` glossary baseline, not just the current local working tree.

Always label evidence as one of:

- refreshed `origin/master` baseline
- local working tree
- live BigQuery schema
- live BigQuery free sample
- user-supplied PR or document

When drafting a table description, use confirmed facts from SQL, config, schema, or live sampling. Include short agent-facing guidance only when it is supported by those sources.

When drafting column descriptions:

- state what the field means before mentioning how it is derived
- avoid unexplained internal shorthand if a new analyst would not understand it
- check whether the field name itself is misleading before trying to rescue it with caveats
- distinguish explicit values from `NULL` when the SQL or live data shows they are different states

When drafting nested field descriptions:

- use the full dotted path as the default comparison key
- write parent-group descriptions for grouped business domains
- add child descriptions selectively where leaf names are ambiguous, coded, overloaded, or easy to misuse
- do not assume a leaf description is reusable across different parents

When drafting a DDL file, preserve partitioning and clustering from the winning source. If the live table is already partitioned and the DDL uses `CREATE OR REPLACE TABLE`, include the matching `PARTITION BY` clause.

Keep the dataset target consistent for the environment you are preparing:

- for production PR work, use the intended production dataset target
- for Hub or sandbox DAG validation, use the staging mirror target when that is how the environment is wired

For the chosen environment, keep all four references aligned:

1. the main task `destination_dataset_table`
2. the `*_ddl` task `destination_dataset_table`
3. the `CREATE OR REPLACE TABLE` target in the DDL file
4. the `SELECT * FROM` source in the DDL file

## 8. Example PR retrieval

Only do this if the user asks for PR inspection or gives a PR link/number.

If `gh` is available:

```bash
gh pr view <PR_NUMBER> --json title,files,url
gh api repos/<OWNER>/<REPO>/pulls/<PR_NUMBER>/comments --paginate
```

If `gh` is not available, ask the user to provide:
- the changed file list
- the PR diff
- or the relevant files directly

If the user provides a PR:

- inspect the changed files first
- then inspect inline review comments and thread replies
- use review comments as evidence of ambiguity, unexplained jargon, or misleading naming
- do not treat reviewer guesses about raw values as authoritative without SQL, docs, or live-data corroboration
- if the PR introduces nested field support, inspect the DDL shape and any macro changes as first-class evidence for how the repo expects nested metadata to be expressed
