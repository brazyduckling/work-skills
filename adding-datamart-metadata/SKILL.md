---
name: adding-datamart-metadata
description: Use when documenting a new or existing FTDNA datamart with column descriptions, table metadata, glossary reuse decisions, or schema/config metadata wiring. Triggers include "add column descriptions", "add table metadata", "inject column descriptions", and similar FTDNA datamart metadata requests.
metadata:
  author: Michal Witkowski
  version: 1.0.0
  category: workflow-automation
---

# Adding Datamart Metadata

Prepare FTDNA datamart metadata updates by checking glossary reuse, proposing new descriptions, and mapping the schema/config work needed for the table.

## Tooling and Dependencies

This skill is meant to work on a different machine, not just this repo checkout.

**Installation:** See `references/installation.md` for Copilot and Claude Code install steps.

**Required capabilities:**
- Ability to read repo files
- Ability to search repo files
- Git access to the target repo

**Optional but strongly recommended:**
- `bq` CLI for live schema and live value checks
- `gcloud` for BigQuery auth and project context
- `gh` CLI only if the user wants an example PR fetched from GitHub instead of supplying files directly

**Approved external skill dependency:**
- If the environment supports installable skills, use `jet-bq` for BigQuery exploration.
- If `jet-bq` is not installed, tell the user to download it from the JET-approved skills source before treating it as a skill dependency.

**No hidden dependency on other skills.** This skill should still work even if no other custom skills are installed.

**Default inspection tools:**
1. file read/search tools available in the environment
2. `git fetch` + `git show` for baseline files from the repo default branch
3. `jet-bq` if installed, otherwise `bq show` + `bq head` for live BigQuery schema and free sampling
4. `gh pr view` only when the user explicitly wants an example PR and GitHub CLI is available

If one of these tools is unavailable, fall back to the next available option and tell the user what evidence source is being used.

## Important

- Always inspect the latest shared glossary before proposing any new descriptions.
- Recommend reuse only when the existing glossary entry matches the column's meaning closely enough; do not reuse based on name similarity alone.
- Preserve exact column order from the current SQL or BigQuery schema when preparing a DDL file.
- Keep the final decision with the user: ask for approval before any write step.
- Stop after showing the proposed changes. Do not edit target repo files unless the user explicitly asks for a follow-up implementation step.

## Instructions

### Step 1: Gather the target

Ask the user for:
1. The target table or datamart name
2. Whether this is a new datamart, an existing datamart, or unknown
3. The code repo or folder that contains the SQL and config
4. Any example PR, process doc, or deck they want followed
5. The BigQuery table identifier if a live table already exists

If the repo layout is not the standard FTDNA layout, ask for:
- glossary file path
- table folder path
- config file path

If the user does not know whether the datamart is new or existing, determine it from the repo structure and config.

### Step 2: Inspect the current state

Read the current source of truth in this order:
1. The latest shared glossary file
2. The table SQL file or current BigQuery schema
3. Any existing DDL file for the table
4. The table config file
5. Any example PR or process note the user supplied

Use `references/retrieval-playbook.md` for the exact repo and BigQuery commands to retrieve each of these safely.

Build a column inventory with these buckets:
- **Reuse candidate**: glossary key already exists and likely matches
- **New description needed**: no suitable glossary entry exists
- **Do not reuse**: similar-looking key exists, but meaning differs

When comparing columns, check:
- business meaning
- grain
- units, currency, or timezone
- allowed values
- null behavior
- sensitivity

### Step 3: Decide the workflow path

Use this decision tree:

1. **New datamart**
   - prepare glossary additions
   - prepare a new table DDL file
   - prepare a new DDL task entry in config

2. **Existing datamart with DDL already wired**
   - prepare glossary additions or reuse decisions
   - prepare DDL updates
   - do not change config

3. **Existing datamart without DDL wiring**
   - treat this as a metadata-wiring gap
   - prepare glossary additions or reuse decisions
   - prepare a new DDL file
   - prepare a new DDL task entry in config

Treat "DDL already wired" as true only if the config already contains a separate `*_ddl` task with schema-step flags such as `is_dml` and `is_common_template`.

If the config or repo layout cannot be checked directly, do not guess. Ask the user for the missing file path or repo location.

### Step 4: Draft the metadata

For each column:
1. Check whether an exact glossary key already exists
2. If yes, recommend reuse only if the meaning really matches
3. If not, draft a new description in the shared glossary style
4. If the match is ambiguous, put it in the **user decision required** bucket

See `references/reuse-rules.md` when reuse is ambiguous.

Every proposed new description should cover:
- what the field means
- the grain
- units, currency, or timezone if relevant
- allowed values if relevant
- null or edge-case behavior
- sensitivity if relevant
- ticket or glossary reference when useful

### Step 5: Prepare the file changes

Prepare, but do not apply, the exact changes for:

1. **Shared glossary**
   - keys to reuse
   - keys to add
   - keys that should not be reused and why

2. **DDL file**
   - file path
   - full ordered schema
   - table-level description
   - whether the file is new or updated

3. **Config**
   - whether config changes are required
   - the exact task block to add if required
   - dependencies on the main table task

Always state which evidence source was used for each proposal:
- default branch repo file
- local working tree file
- live BigQuery schema
- user-supplied example PR or document

### Step 6: Ask for approval

Show the user:
1. Reuse recommendations
2. New descriptions
3. DDL plan
4. Config plan
5. Any open questions or ambiguous columns

Ask the user to approve or adjust the proposal.

If the user approves:
- summarize the exact files and changes that should be made
- stop here unless the user explicitly asks to implement them

If the user requests changes:
- revise only the affected parts
- re-present the updated proposal

### Step 7: Guide validation

After the proposal is accepted, walk the user through:
1. Creating a PR with the metadata files
2. Merging the PR
3. Triggering the Airflow task manually for one region
4. Checking BigQuery schema metadata
5. Confirming the descriptions appear on the table columns

## Examples

### Example 1: New datamart

User says: "Add column descriptions for my new FTDNA datamart."

Actions:
1. Inspect glossary, SQL, and config
2. Identify reusable glossary keys and missing ones
3. Draft new glossary entries
4. Prepare a new DDL file
5. Prepare the new `*_ddl` config block
6. Show all proposed changes for approval

Result: The user gets a push-ready metadata package for a new datamart without blind glossary reuse.

### Example 2: Existing datamart with partial metadata

User says: "Add table metadata to this existing datamart."

Actions:
1. Inspect glossary, SQL, existing DDL, and config
2. Detect whether a DDL task already exists
3. Reuse only matching glossary entries
4. Draft only the missing descriptions
5. Prepare DDL updates and config changes only if wiring is missing
6. Show the proposal and stop for approval

Result: The user gets the minimum safe metadata update path for the table's current state.

### Example 3: Edge case with misleading names

User says: "Inject column descriptions for this table; I think most names already exist in the glossary."

Actions:
1. Compare exact names and meanings
2. Flag columns like `tenant` versus `tenant_id` as ambiguous, not auto-reuse
3. Explain why some names should not reuse existing descriptions
4. Ask the user to decide on the flagged cases

Result: Similar-looking names do not create misleading metadata.

## Troubleshooting

### The glossary has a similar name but not the same meaning

Cause: Name similarity is being mistaken for semantic equivalence.

Solution: Put the column in the **do not reuse** or **user decision required** bucket and explain the mismatch.

### The table has no DDL file

Cause: The datamart exists, but the metadata schema step was never added.

Solution: Treat it as an existing datamart without DDL wiring. Prepare both a new DDL file and a new `*_ddl` config task.

### The config rule is unclear

Cause: The process note says existing datamarts may not need config changes, but that only applies when the DDL task already exists.

Solution: Check the config directly. If no `*_ddl` task exists, prepare one.

### The user wants implementation immediately

Cause: The workflow reached proposal-ready state, but this skill is scoped to stop after showing the changes.

Solution: Summarize the exact edits and wait for an explicit follow-up request to implement them.

### Required tool is missing

Cause: The environment does not have a needed tool such as `git`, `bq`, `gcloud`, or `gh`.

Solution:
1. Say exactly which tool is missing
2. Fall back if possible
3. If there is no safe fallback, ask the user to provide the needed file or schema evidence manually
