---
name: adding-datamart-metadata
description: Use when documenting a new or existing FTDNA datamart with column descriptions, table metadata, nested Struct or Record fields, enum/value explanations, or DDL/config metadata wiring. Triggers include metadata review feedback, misleading field names, nested field schemas, unclear allowed values, and similar FTDNA datamart metadata requests.
metadata:
  author: Michal Witkowski
  version: 1.3.0
  category: workflow-automation
---

# Adding Datamart Metadata

Prepare FTDNA datamart metadata updates by checking glossary reuse, proposing new descriptions, and mapping the schema/config work needed for the table.

This skill is for evidence-backed, analyst-friendly metadata. Prefer business meaning over SQL narration, and surface naming problems when description text alone would still leave the field misleading.

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
1. local cloned repo files on the user's laptop
2. suggest that the user refresh the local repo against `origin/master`, then use `git show` for baseline files from the refreshed remote
3. file read/search tools available in the environment
4. `jet-bq` if installed, otherwise `bq show` + `bq head` for live BigQuery schema and free sampling
5. `gh pr view` only when the user explicitly wants an example PR and GitHub CLI is available

If one of these tools is unavailable, fall back to the next available option and tell the user what evidence source is being used.

## Important

- Always inspect the latest shared glossary before proposing any new descriptions.
- Default to the local cloned repo on the user's laptop when the user gives only a repo name.
- If a local clone is found, confirm the chosen local path with the user before using it.
- Ask the user to refresh the selected local repo against `origin/master`, or confirm that it is already up to date, before checking glossary, config, SQL, or DDL baseline files unless the user explicitly asks for a different base branch.
- Recommend reuse only when the existing glossary entry matches the column's meaning closely enough; do not reuse based on name similarity alone.
- Preserve exact column order from the current SQL or BigQuery schema when preparing a DDL file.
- Keep the final decision with the user: ask for approval before any write step.
- Stop after showing the proposed changes. Do not edit target repo files unless the user explicitly asks for a follow-up implementation step.
- Treat PR review comments, thread replies, transcripts, user notes, and linked process docs as first-class business-meaning evidence when they are part of the task.
- Describe what a field means to a new analyst or AI agent, not what the SQL happens to do. Avoid generic phrasing such as `SQL maps`, vague transformation narration, or dual-meaning wording like `country or tenant` when one precise meaning can be stated.
- Never leave internal shorthand unexplained in final descriptions. Expand or replace source nicknames, system labels, and team jargon with the actual business concept or concrete source table or product name.
- For enum-like fields, flags, or codes, list only corroborated values and explain what each value means. Distinguish `NULL` from explicit values like `Unknown`, `None`, `No Data`, `false`, or `0`.
- If a clearer description still has to apologize for a misleading field name, recommend a rename or modeling fix instead of forcing awkward prose. The user still decides whether to implement the rename.
- If meaning, null behavior, source precedence, or allowed values are not safely proven, keep the proposal conservative and flag the gap.
- If the table contains nested `STRUCT` / `RECORD` / `REPEATED` fields, treat each nested field path as its own metadata target and preserve the exact nested order, type, and mode from the source schema.
- For nested schemas, describe parent groups and child fields differently: parent descriptions explain the grouped domain and grain; child descriptions explain the individual field meaning, but do not force boilerplate on obvious child fields that are already clear from the parent context.
- When nested field meaning depends on parent context, prefer a full dotted path such as `customer_payments.gross_customer_amounts.gmv_eur` over a leaf-only identifier like `gmv_eur`.
- If the repo's current macro or DDL pattern cannot express nested descriptions safely, treat that as a metadata-support gap and prepare the minimum macro or template changes needed.

## Instructions

### Step 1: Gather the target

Ask the user for:
1. The target table or datamart name
2. Whether this is a new datamart, an existing datamart, or unknown
3. The local repo path, or the repo name if they want the skill to find the local cloned checkout
4. Any example PR, review thread, process doc, transcript, or deck they want followed
5. The BigQuery table identifier if a live table already exists

If the user gives only a repo name:
- search common local clone locations on the laptop
- identify matching git checkouts
- confirm the chosen local path with the user before proceeding
- use GitHub only if no local clone is found or the user explicitly asks for GitHub evidence

If the repo layout is not the standard FTDNA layout, ask for:
- glossary file path
- table folder path
- config file path

If the user does not know whether the datamart is new or existing, determine it from the repo structure and config.

If the user gives a PR or says the work is already under review:
- inspect inline review comments and thread replies before drafting descriptions
- treat review feedback as evidence about reader confusion and missing context, not as proof of raw values by itself

### Step 2: Inspect the current state

After selecting the local repo path:
1. ask the user to refresh against `origin/master`, or confirm that the repo is already up to date
2. use the refreshed remote baseline for glossary comparison
3. use the local working tree for branch-under-change evidence

Then read the current source of truth in this order:
1. The latest shared glossary file
2. The table SQL file or current BigQuery schema
3. Any existing DDL file for the table
4. The table config file
5. Any example PR, review thread, transcript, or process note the user supplied
6. Any linked documentation that explains the business concept more clearly than the SQL alone

Use `references/retrieval-playbook.md` for the exact repo and BigQuery commands to retrieve each of these safely.

If the schema contains nested `STRUCT` / `RECORD` / `REPEATED` fields:
- build the inventory using full dotted paths
- capture parent group names, child field names, types, modes, and order
- inspect how the repo currently renders nested schemas in DDL or macros before proposing syntax changes
- see `references/nested-field-rules.md`

Build a column or field inventory with these buckets:
- **Reuse candidate**: glossary key already exists and likely matches
- **New description needed**: no suitable glossary entry exists
- **Do not reuse**: similar-looking key exists, but meaning differs
- **Rename or modeling issue recommended**: the current field name is itself misleading, or the cleanest description still needs caveats to avoid misleading readers

When comparing columns, check:
- business meaning
- grain
- units, currency, or timezone
- allowed values
- null behavior
- sensitivity
- whether the current name still matches the proven meaning
- whether the proposed text would make sense to a new analyst with no internal context
- for nested fields, whether the leaf meaning is still clear without the parent path

Before drafting descriptions, run this quality gate:
- Can I explain the field without narrating SQL mechanics?
- Does the description use one clear business meaning instead of `X or Y` wording?
- Are allowed values and null states fully explained where relevant?
- Would the text still make sense if the reader does not know internal shorthand?
- For nested schemas, have I documented the parent group, the ambiguous child fields, and any repeated-element semantics that matter?
- If not, gather more evidence or move the field to **User decision required** or **Rename or modeling issue recommended**.

After inspection, always include:
- **Recommended next steps**

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

If nested fields exist:
- inspect whether the current DDL or macro pattern already supports nested descriptions
- if yes, reuse that pattern
- if no, prepare the minimum support change needed in addition to the metadata proposal

### Step 4: Draft the metadata

For each column:
1. Check whether an exact glossary key already exists
2. If yes, recommend reuse only if the meaning really matches
3. If not, draft a new description in the shared glossary style
4. If the match is ambiguous, put it in the **user decision required** bucket
5. If the current name is misleading and the best wording still feels defensive, put it in the **rename or modeling issue recommended** bucket

See `references/reuse-rules.md` when reuse is ambiguous. See `references/description-quality-rules.md` when wording is generic, null states are tricky, or the current field name may be the real problem. See `references/nested-field-rules.md` when the schema contains `STRUCT`, `RECORD`, or `REPEATED` fields.

For nested fields:
- treat the full dotted path as the default comparison key
- decide whether the parent struct needs a description
- decide which child fields need descriptions because they are ambiguous, coded, overloaded, or easy to misuse
- decide whether the description belongs in the shared glossary, a path-specific glossary key, or inline DDL schema objects

Every proposed new description should cover:
- what the field means
- the grain
- units, currency, or timezone if relevant
- allowed values if relevant
- null or edge-case behavior
- sensitivity if relevant
- an example value when one is safely observed from live sampling or an unambiguous source example
- ticket or glossary reference when useful
- why the current name is misleading, when a rename is recommended

For enum-like fields, flags, or codes:
- include only corroborated values
- explain what each value means
- state whether `NULL` is a separate state or just missing data
- include raw code plus human meaning only when both are safely proven

For nested parent groups:
- explain what domain the group represents
- explain the grain or scope of the grouped fields
- mention shared unit or currency conventions when that context helps all children

For repeated arrays:
- describe one element of the array and what repetition represents
- preserve the repeated mode explicitly in the proposed DDL

If no safe example value is available:
- omit the example instead of guessing
- say that the description is conservative where helpful

### Step 5: Prepare the file changes

Prepare, but do not apply, the exact changes for:

1. **Shared glossary**
   - keys to reuse
   - keys to add
   - keys that should not be reused and why
   - fields where a rename or modeling fix is recommended and why

2. **DDL file**
   - file path
   - full ordered schema
   - full nested ordered schema when `STRUCT` / `RECORD` fields exist
   - preserve partitioning and clustering spec from the live table or current config when present
   - keep the dataset target consistent with the intended environment
   - table-level description
   - short agent-facing guidance in the table description based only on confirmed evidence
   - whether the file is new or updated
   - any parent-group descriptions, child-field descriptions, or repeated-element notes that must live inline

3. **Config**
   - whether config changes are required
   - the exact task block to add if required
   - dependencies on the main table task
   - keep the main task target and the `*_ddl` task target aligned with the DDL file target

4. **Macro or template support**
   - whether the current repo already supports nested field descriptions
   - the exact macro or template changes needed if it does not

Always state which evidence source was used for each proposal:
- refreshed `origin/master` repo file
- local working tree file
- live BigQuery schema
- live BigQuery free sample
- user-supplied example PR or review thread
- user-supplied transcript or meeting note
- linked process doc or Confluence page

When a table description references sources or architecture:
- prefer concrete source table names, products, or domains over unexplained shorthand
- mention source-system names only when that context helps the reader understand the field or table meaning

### Step 6: Ask for approval

Show the user:
1. Reuse recommendations
2. New descriptions
3. Rename or modeling recommendations
4. DDL plan
5. Config plan
6. Any open questions or ambiguous columns
7. **Recommended next steps**

Ask the user to approve or adjust the proposal.

If the user approves:
- summarize the exact files and changes that should be made
- separate description-only updates from any rename recommendations that are not yet approved for implementation
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

### Example 4: Review feedback says the wording sounds generic

User says: "These descriptions sound generic. The reviewer wants the real business meaning and clearer value explanations."

Actions:
1. Read the PR comments, thread replies, and any linked docs or transcripts
2. Identify where the current wording describes SQL logic instead of field meaning
3. Rewrite meaning-first descriptions and explain corroborated enum values
4. Recommend a rename if the field name itself is misleading

Result: Review feedback is translated into precise metadata instead of vague AI-generated wording.

### Example 5: Nested struct fields in a gold datamart

User says: "This datamart has nested STRUCT fields. Use this PR as the pattern and help me add metadata."

Actions:
1. Inspect the nested schema and inventory full dotted paths
2. Check how the repo currently renders nested field descriptions in DDL or macros
3. Write parent-group descriptions for domain structs and selective child descriptions for ambiguous or coded leaf fields
4. Recommend path-specific or inline DDL descriptions when leaf-only glossary reuse would be misleading

Result: The user gets a nested-field metadata proposal that preserves structure and avoids boilerplate.

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

### The description sounds like SQL commentary

Cause: The wording explains the transformation instead of the business meaning.

Solution:
1. Rewrite from the reader's perspective: what the field means, not how the CASE statement works
2. Keep transformation detail only when it is necessary to explain the business meaning
3. If the field still feels hard to explain, check whether the name itself is misleading

### The field name is misleading

Cause: The current name implies a different concept than the proven business meaning.

Solution:
1. Try writing the cleanest honest description
2. If the description still needs caveats to avoid misleading the reader, put the field in **Rename or modeling issue recommended**
3. Provide a conservative description-only fallback only if the user wants to keep rename out of scope

### The values are known but their meaning is not

Cause: SQL, live data, or docs expose raw values, but not what those values mean in business terms.

Solution:
1. Do not list raw values without meaning
2. Search docs, PR threads, transcripts, or linked process notes for the meaning
3. If still unresolved, keep the value list conservative and flag the ambiguity

### `NULL`, `Unknown`, and `None` are being treated as the same thing

Cause: The draft metadata collapses distinct states into a single "no data" explanation.

Solution:
1. Check SQL and live samples separately
2. Explain each explicit state only if corroborated
3. State `NULL` behavior explicitly when it differs from named values like `Unknown` or `None`

### The table has nested `STRUCT` / `RECORD` fields

Cause: The workflow was written for flat columns, but the datamart groups fields into nested domains.

Solution:
1. Inventory full dotted paths instead of leaf names only
2. Describe parent groups and child fields separately
3. Preserve nested type, mode, and order in the DDL proposal
4. Use inline DDL descriptions or path-specific keys when leaf-only reuse would be misleading

### The repo cannot render nested descriptions yet

Cause: The current macro or DDL pattern only supports flat columns.

Solution:
1. Inspect the existing macro or schema rendering pattern
2. Prepare the minimum support change needed for nested fields
3. Keep the metadata proposal aligned with the repo's actual templating approach instead of inventing one-off syntax

### The DDL task fails in Hub but the prod target is correct

Cause: Hub validation may run against a staging mirror dataset while the production PR should still point to the datamarts dataset.

Solution:
1. Confirm which dataset should be used for the current environment
2. Keep the target consistent across:
   - the main task `destination_dataset_table`
   - the `*_ddl` task `destination_dataset_table`
   - the `CREATE OR REPLACE TABLE` target in the DDL file
   - the `SELECT * FROM` source in the DDL file
3. If the existing table is partitioned, preserve the same `PARTITION BY` clause in the DDL file

### The user wants implementation immediately

Cause: The workflow reached proposal-ready state, but this skill is scoped to stop after showing the changes.

Solution: Summarize the exact edits and wait for an explicit follow-up request to implement them.

### Required tool is missing

Cause: The environment does not have a needed tool such as `git`, `bq`, `gcloud`, or `gh`.

Solution:
1. Say exactly which tool is missing
2. Fall back if possible
3. If there is no safe fallback, ask the user to provide the needed file or schema evidence manually
