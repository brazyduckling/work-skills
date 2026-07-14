---
name: adding-datamart-metadata
description: Prepare shared-glossary-first metadata proposals for new or existing FTDNA datamarts. Use when the user asks to "add metadata descriptions", "add column descriptions", "document this table", "populate ftdna_macros", "review glossary reuse", or "wire DDL metadata" for a table, including nested Struct or Record fields, enum/value explanations, and DDL/config metadata wiring.
metadata:
  author: Michal Witkowski
  version: 1.8.0
  category: workflow-automation
  canonical_source: https://raw.githubusercontent.com/brazyduckling/work-skills/main/adding-datamart-metadata/
---

# Adding Datamart Metadata

Prepare FTDNA datamart metadata updates by checking shared-glossary reuse, proposing new descriptions, and mapping the schema/config work needed for the table.

This skill is for evidence-backed, analyst-friendly metadata. Prefer business meaning over SQL narration, and surface naming problems when description text alone would still leave the field misleading.

The default output is a **shared glossary recommendation** that multiple DDL files can reuse safely. Do not drift into table-local definitions unless the meaning truly depends on the table or nested path.

In the standard FTDNA layout, `ftdna_macros.jinja` is the shared glossary baseline. The skill must inspect it first and treat it as authoritative unless the user explicitly points to a different shared glossary file.

For maintainers changing this skill, use `evals/protocol.md` and `evals/cases.jsonl` to check that the decision logic still protects the shared glossary.

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
- Treat the shared glossary as the canonical metadata layer. DDL files should populate table metadata from that shared glossary unless a path-specific or inline exception is genuinely required.
- Never treat "this table folder only has SQL + config" as evidence that there is no shared glossary. In the standard FTDNA layout, the shared glossary still lives in `ftdna_macros.jinja`.
- Never let an existing local DDL pattern or nearby table convention override the glossary-first policy.
- Default to the local cloned repo on the user's laptop when the user gives only a repo name.
- If a local clone is found, confirm the chosen local path with the user before using it.
- Ask the user to refresh the selected local repo against `origin/master`, or confirm that it is already up to date, before checking glossary, config, SQL, or DDL baseline files unless the user explicitly asks for a different base branch.
- Recommend reuse only when the existing glossary entry matches the column's meaning closely enough; do not reuse based on name similarity alone.
- Aim for glossary entries that are accurate enough to cover multiple tables that truly share the same field meaning. Do not make a definition more generic just to force reuse. If an existing shared definition is too specific to one table, revise it so it stays precise for all proven tables in scope.
- If the same column name is used for materially different data across tables, do not paper over the mismatch with caveats. Recommend a different name or modeling change, and base that recommendation on repo evidence, live schema or values, and any corroborating docs or review context.
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
- Every final description should use a **minimal two-part format** inside one string so both audiences can scan it quickly:
  - **For stakeholders (`Functional usage:`):** what the column means, how it should be interpreted, and any value semantics the business reader needs. Include enumerated values here for enum/coded/flag/bounded fields.
  - **For agents and analysts (`Agent / analyst notes:`):** metadata-specific guidance. Three elements are required when applicable: `Source:` (always — use the specific BigQuery table path or SQL reference, e.g. `production_external_adyen.authorisation_events`; read from the transform `*.sql`, never the `*_ddl.sql` self-select; for in-datamart derivations write `Source: derived in-datamart`); `Formula:` (calculated metrics only — the actual derivation); grain, join keys, null behavior, and other caveats that affect correct use.
- Keep both parts concise. This is not a long two-section essay; it is a compact, skimmable structure.
- Example values are required for enum/coded/flag/bounded fields; use a format sample for high-cardinality fields. Never enumerate IDs, PII-adjacent fields (e.g. `customer_id`), or GEOGRAPHY columns.
- A `--` code comment in the DDL is acceptable only as a supplement for a formula too long to fit cleanly in a description. It is never a replacement for the description.
- For tables with `require_partition_filter = TRUE` (e.g. gold datamarts), state the required filter field in the table description — an unfiltered query will fail at runtime. Also document grain, join keys, and currency conventions (`_eur` vs `_local_currency`) to help agents form correct queries.
- Before proposing any DDL-only or inline-DDL path, the skill must first show how each field was evaluated against the shared glossary.

## Instructions

### Step 0: Check for skill updates

Before proceeding with any user task, check if a newer version of this skill is available:

1. Read the local version from this skill's frontmatter (`version: X.Y.Z`) and the `canonical_source` URL.
2. If `canonical_source` is set and `curl` is available, fetch the remote version header:
   ```bash
   curl -s <canonical_source>SKILL.md | grep "^  version:"
   ```
3. Compare versions:
   - If remote version is higher: tell the user before doing anything else — _"There is a newer version of this skill (vX.Y.Z) available. I can update it now — this will replace the files in `~/.copilot/skills/adding-datamart-metadata/` (or `~/.claude/skills/` for Claude Code). Reply **update skill** to proceed or **skip** to continue with the current version."_
   - If update is approved: download each file from `canonical_source` and overwrite the local copies, then confirm. After updating, restart the user's original request with the new version.
   - If update is skipped, the check fails, or `canonical_source` is not set: proceed silently with the current version and note it at the start of the response (e.g. `Running skill v1.8.0`).

**Files to update when self-updating:**
- `SKILL.md`
- `references/description-quality-rules.md`
- `references/reuse-rules.md`
- `references/retrieval-playbook.md`
- `references/nested-field-rules.md`
- `references/installation.md`
- `evals/protocol.md`
- `evals/rubric.md`
- `evals/cases.jsonl`

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

If the standard FTDNA shared glossary file exists, not reading it is a blocking error. Do not continue to a DDL-first recommendation until the shared glossary has been inspected.

If the schema contains nested `STRUCT` / `RECORD` / `REPEATED` fields:
- build the inventory using full dotted paths
- capture parent group names, child field names, types, modes, and order
- inspect how the repo currently renders nested schemas in DDL or macros before proposing syntax changes
- see `references/nested-field-rules.md`

Build a column or field inventory with these buckets:
- **Reuse candidate**: glossary key already exists and likely matches
- **Revise existing glossary definition**: the glossary key is the right concept, but the current shared wording is too weak, too generic, or incomplete
- **New description needed**: no suitable glossary entry exists
- **Do not reuse**: similar-looking key exists, but meaning differs
- **Rename or modeling issue recommended**: the current field name is itself misleading, or the cleanest description still needs caveats to avoid misleading readers

For any field meant for the shared glossary, also ask:
- Can one accurate shared definition cover every proven use of this field across the relevant tables?
- If not, is the right fix a different field name, a path-specific key, or a table-local inline exception?

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

Use this quick funnel first to decide the resolution path for each field:

- **Glossary term with the same meaning at this grain?** → Reuse (path 1)
- **Same concept but wording is weak or too specific?** → Revise (path 2)
- **Genuinely new universal concept?** → Add to glossary (path 3)
- **Same name, different meaning or grain?** → Path-specific key (path 4). Rename only for true top-level overloads with evidence (paths 5–6). Nested pillars (`customer.reductions.amount` vs `partner.costs.amount`) are intentional namespacing — path-specific key, not rename.
- **Table-local only?** → Inline DDL (path 7, last resort)

Then confirm using the full resolution paths below.

For each field, explicitly evaluate these resolution paths:
1. **Reuse the existing shared glossary definition**
2. **Revise the existing shared glossary definition**
3. **Add a new shared glossary key**
4. **Use a path-specific glossary key**
5. **Recommend a clearer column name**
6. **Recommend a modeling split or structural fix**
7. **Keep the description inline in the DDL only as an exceptional fallback**

Choose the cleanest honest path based on evidence. Do not default to reuse just because the name matches. Treat inline DDL descriptions as the last path, not a normal outcome.

Every field must receive one explicit glossary disposition before you propose the DDL plan.

For each column:
1. Check whether an exact glossary key already exists
2. If yes, recommend reuse only if the meaning really matches
3. If yes but the wording is too generic, inaccurate, or too specific to one table, put it in **revise existing glossary definition**
4. If not, draft a new description in the shared glossary style
5. If the match is ambiguous, put it in the **user decision required** bucket
6. If the current name is misleading and the best wording still feels defensive, put it in the **rename or modeling issue recommended** bucket

See `references/reuse-rules.md` when reuse is ambiguous. See `references/description-quality-rules.md` when wording is generic, null states are tricky, or the current field name may be the real problem. See `references/nested-field-rules.md` when the schema contains `STRUCT`, `RECORD`, or `REPEATED` fields.

For nested fields:
- treat the full dotted path as the default comparison key
- decide whether the parent struct needs a description
- decide which child fields need descriptions because they are ambiguous, coded, overloaded, or easy to misuse
- decide whether the description belongs in the shared glossary, a path-specific glossary key, or inline DDL schema objects

Every proposed new description should:
- use the minimal two-part format:
  - `For stakeholders: ...`
  - `For agents and analysts: ...`
- keep the first part focused on what the field means and how it should be interpreted
- keep the second part focused on grain, join keys, derivation caveats, source precedence, null behavior, allowed values, and other metadata details that affect correct usage
- include units, currency, timezone, or sensitivity notes only when they matter
- include an example value only when one is safely observed from live sampling or an unambiguous source example
- include rename reasoning only in the recommendation notes, not by turning the description into an apology for the field name

When drafting the two-part format:
- do not let the `For agents and analysts` part become generic filler
- if there are no confirmed join keys or derivation caveats, say only the confirmed metadata detail that matters
- if the functional meaning is still too broad to be honest across tables, do not weaken it just to preserve a shared key

Only use inline DDL descriptions when:
- the meaning is genuinely table-local
- or the meaning depends on a nested/table path that the shared glossary cannot express cleanly
- and reuse, revise, add, path-specific, and rename/modeling options were all considered first

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
   - confirm that this is the reusable cross-table glossary layer, not a table-local dictionary
   - show the glossary disposition for every field before moving to the DDL plan
   - keys to reuse
   - keys whose current shared definition should be revised because it is weak, inaccurate, or too specific to one table
   - keys to add
   - keys that should not be reused and why
   - fields where a rename or modeling fix is recommended and why
   - where a path-specific shared key is safer than a flat shared key

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
   - which descriptions are inline only as a last-resort exception and why

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

Present every proposed change as a **numbered recommendation list**. Each item must use this exact format:

```
[N] ACTION — `field_name` (or `glossary_key` / `parent.child` for nested)
    Why: one sentence explaining the evidence behind this change.
    Proposed: "<the exact new description string>"
    Status: <see status levels below>
```

**Status levels — always written in full, never represented by emoji alone:**

- 🟢 **Status: NEW TABLE / NEW FIELD** — This description is being added to a column or table that does not yet exist in production BigQuery. There are no existing consumers of this column or this glossary key. It is safe to merge once the wording is approved. No downstream check is required.

- 🟡 **Status: EXISTING TABLE — DESCRIPTION ONLY** — This change updates metadata text only. The column name, type, and schema structure are unchanged. Risk is low: the only impact is what analysts and AI agents see when they read the BigQuery schema or call `bq show`. Before merging, verify the new wording is still accurate for every DDL file and table that shares this glossary key — a shared glossary string is used by multiple tables, so a revision to it affects all of them.

- 🔴 **Status: EXISTING PRODUCTION TABLE — STRUCTURAL CHANGE** — This change modifies a column name, DDL schema structure, or a shared glossary key that is currently live in BigQuery and may have active consumers. Risk is HIGH. Any SQL query, Looker explore, downstream DDL task, BI dashboard, or data pipeline that references this column by name will break silently or throw an error if the change is applied without coordination. Before approving this item you must: (1) run `grep -r "<column_name>" <repo_path>/fintech_data_analytics_datamarts/` to find all references in the repo; (2) search for dependent BigQuery views using `bq show --format=prettyjson <project>:<dataset>.<table>`; (3) check all config.json files for downstream tasks that depend on this table; (4) confirm with the owners of any downstream tables or dashboards that they can absorb the change. I will not implement this item until you explicitly confirm in this conversation that the downstream check is complete.

After listing all recommendations, close with:

> _Reply with the numbers you approve (e.g. "approve 1 3 4"), numbers you want adjusted, or "approve all". For any item marked EXISTING PRODUCTION TABLE — STRUCTURAL CHANGE I will not implement until you confirm the downstream check is done._

**Additional rules:**

- Never mix the recommendation list with implementation. Show the list and stop.
- If a column requires a rename: always include a separate 🟡 description-only recommendation for the current name as a safe fallback — the user may want the description now and the rename later.
- If open questions remain, add them as `[?] UNCLEAR — field_name` items at the end of the list.
- For every rename or non-reuse recommendation, quote the specific evidence (SQL line, PR comment, live value) that supports it.

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

### Example 3b: Same name, different data across tables

User says: "These three tables all have `status`, but reviewers say the glossary reuse feels wrong."

Actions:
1. Compare each `status` field using SQL, live values, docs, and existing reviewer context
2. Check whether one honest shared definition can cover all three
3. If not, recommend more specific names or path-specific glossary keys
4. Explain the evidence for the split instead of weakening the wording into something generic

Result: The skill protects the shared glossary from becoming vague or misleading.

### Example 4: Existing glossary concept is right, but the shared definition is weak or too specific to one table

User says: "The field should stay shared, but the current glossary wording is too generic."

Actions:
1. Confirm that the same field really means the same thing across the target tables
2. Keep the shared key
3. Rewrite the existing glossary definition so it is more precise, less generic, and not overfit to one table
4. Show which DDLs would benefit from the revised shared wording

Result: The skill improves the shared glossary itself instead of creating duplicate or table-local definitions.

### Example 5: Review feedback says the wording sounds generic

User says: "These descriptions sound generic. The reviewer wants the real business meaning and clearer value explanations."

Actions:
1. Read the PR comments, thread replies, and any linked docs or transcripts
2. Identify where the current wording describes SQL logic instead of field meaning
3. Rewrite meaning-first descriptions and explain corroborated enum values
4. Recommend a rename if the field name itself is misleading

Result: Review feedback is translated into precise metadata instead of vague AI-generated wording.

### Example 6: Nested struct fields in a gold datamart

User says: "This datamart has nested STRUCT fields. Use this PR as the pattern and help me add metadata."

Actions:
1. Inspect the nested schema and inventory full dotted paths
2. Check how the repo currently renders nested field descriptions in DDL or macros
3. Write parent-group descriptions for domain structs and selective child descriptions for ambiguous or coded leaf fields
4. Recommend path-specific or inline DDL descriptions when leaf-only glossary reuse would be misleading

Result: The user gets a nested-field metadata proposal that preserves structure and avoids boilerplate.

### Example 7: Truly table-local modeling artifact

User says: "This field only exists because this table branches source precedence in a one-off way. Should it live in the shared glossary?"

Actions:
1. Check whether the concept exists elsewhere or is only a product of this table's modeling
2. Confirm that reuse, glossary revision, new glossary key, path-specific key, and rename options would all reduce clarity
3. Keep the description inline in the DDL only if it is genuinely the last clean option
4. Explain why the field should remain an exception rather than a shared glossary concept

Result: The shared glossary stays clean, and inline DDL remains a controlled exception.

## Troubleshooting

### The glossary has a similar name but not the same meaning

Cause: Name similarity is being mistaken for semantic equivalence.

Solution: Put the column in the **do not reuse** or **user decision required** bucket and explain the mismatch.

### The skill says there is no glossary file

Cause: It looked only at the current table folder, or it over-weighted nearby DDL convention instead of checking the shared glossary baseline.

Solution:
1. Re-open the shared glossary baseline first
2. In the standard FTDNA layout, treat `ftdna_macros.jinja` as mandatory evidence
3. Re-run the field-by-field glossary decision before allowing any inline-DDL outcome

### The same field name exists across tables, but the data is different

Cause: A shared glossary key is being stretched beyond one real business meaning.

Solution:
1. Compare the field across the relevant tables using schema, live values, and business docs
2. Decide whether a path-specific shared key is enough
3. If not, recommend a clearer field name or modeling split
4. Keep the shared glossary precise instead of broadening the definition until it becomes generic

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
