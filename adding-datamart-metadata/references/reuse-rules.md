# Reuse Rules For Datamart Metadata

Use this file when glossary reuse is ambiguous.

The target is the **shared glossary layer**, not a table-local copy unless the meaning truly depends on a specific table or path.

## Shared glossary rule

- Prefer one shared glossary key only when one honest definition can cover every proven use.
- Do not broaden wording just to maximize reuse.
- If one name is carrying different meanings across tables, recommend a path-specific key, a rename, or a modeling split.
- A conflict with the current shared glossary does **not** justify skipping the glossary layer. It triggers a decision between revise, add, path-specific, rename, or, only at the very end, inline DDL.

## Revise the existing shared glossary definition when

- The glossary key is still the right concept
- Cross-table meaning is consistent
- The current wording is generic, incomplete, weak, or too specific to one table
- A more precise shared definition can still honestly cover the proven uses

Do not create a duplicate glossary key just because the current shared wording is poor.
If a new table changes the safest shared wording, revise the existing glossary definition instead of preserving stale table-specific phrasing.

## Reuse when all of these are true

- The column name matches exactly
- The business meaning matches
- The grain matches
- Units, currency, or timezone behavior match
- Null behavior does not materially differ
- The value domain matches (for coded/enum/flag fields)
- The formula matches (for calculated metrics)

Source and lineage are **per-instance**, not shared-glossary fields. Do not include upstream table names in a shared glossary entry when the upstream differs per table. Source belongs in the `Agent / analyst notes` section of each table's DDL description, not in the shared glossary string.

## Do not reuse when any of these are true

- Names are only similar, not exact
- The same word means something different in this table
- The column changed from raw to derived meaning
- The grain changed
- The allowed values changed in a meaningful way
- The formula differs by grain or aggregation level
- The value domain differs by type or grain (e.g. INT 0/1 in one table, BOOL in another for the same column name)
- The old description would mislead an analyst or AI agent
- The cleanest honest wording still needs caveats because the current field name is misleading
- The same column name is carrying materially different data across tables

When reuse fails:

1. Check whether the existing shared definition should be revised
2. Check whether a new shared key is the right fix
3. Check whether a path-specific shared key is safer
4. Check whether the field should be renamed or remodeled
5. Use inline DDL only if those options still reduce clarity

## Ask the user to decide when

- The old glossary key is close but not exact
- The same metric is reused with different business framing
- Null behavior is unclear
- The source SQL is not enough to explain the field safely

## Nested fields

- Use the full dotted path as the default comparison key for nested fields.
- A leaf-only match is not enough when the same leaf name appears under different parents.
- Reuse a leaf description only when the parent context does not materially change the meaning.
- Prefer path-specific or inline descriptions when nested context is part of the meaning.

Common nested-field traps:

- `status` under different parent structs
- `id`, `name`, `type`, or `amount` reused across multiple domains
- the same metric name under `gross`, `net`, `billing`, or `revenue` groups
- the same metric name at different aggregation grains (e.g. delivery-grain vs order-grain) — path-specific key, not a shared glossary entry

## Meaning-first wording

- State what the field means before describing how it is derived.
- Avoid generic phrasing such as `SQL maps`, `derived from logic`, or `populated from CASE` unless the transformation itself is the business meaning.
- Prefer one precise business concept over dual-meaning wording like `country or tenant`.
- Expand internal shorthand when the reader would not know it.

## Enum-like fields, flags, and codes

- List only corroborated values.
- Explain what each value means, not just the raw label or code.
- Distinguish `NULL` from explicit values like `Unknown`, `None`, `No Data`, `false`, or `0` when they represent different states.
- If a raw code and its human meaning are both proven, include both.

## Rename or modeling issue recommended

Use this bucket when any of these are true:

- The current name implies a different business concept than the proven meaning.
- The description has to keep apologizing for the name to stay accurate.
- One name bundles two concepts that should be separated.
- A legacy or dashboard-aligned name is still misleading for the datamart layer.
- A description-only fix would still mislead analysts or AI agents.
- A nested parent name is too generic to explain the grouped fields cleanly.
- A leaf field is too generic unless the parent path is treated as part of the name.

## Evidence standard for rename recommendations

Ground rename or modeling recommendations in confirmed evidence such as:

- repo SQL or DDL logic
- live schema or free-sampled values
- process docs, transcripts, or review comments that explain reader confusion

Do not recommend a rename from naming taste alone.

## Example values

- Example values can support a reuse or non-reuse decision.
- Example values do not override business meaning, grain, units, or null-behavior mismatches.

## Common examples

- `tenant` vs `tenant_id`: usually do not auto-reuse
- `status` vs `three_ds_outcome`: do not reuse
- `order_id` reused as the same primary order key: usually safe reuse candidate
- `status` meaning payment outcome in one table and pipeline processing state in another: usually split or rename candidate

## Output buckets

- **Reuse candidate**
- **Revise existing glossary definition**
- **New description needed**
- **Do not reuse**
- **User decision required**
- **Rename or modeling issue recommended**
