# Reuse Rules For Datamart Metadata

Use this file when glossary reuse is ambiguous.

## Reuse when all of these are true

- The column name matches exactly
- The business meaning matches
- The grain matches
- Units, currency, or timezone behavior match
- Null behavior does not materially differ

## Do not reuse when any of these are true

- Names are only similar, not exact
- The same word means something different in this table
- The column changed from raw to derived meaning
- The grain changed
- The allowed values changed in a meaningful way
- The old description would mislead an analyst or AI agent
- The cleanest honest wording still needs caveats because the current field name is misleading

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

## Example values

- Example values can support a reuse or non-reuse decision.
- Example values do not override business meaning, grain, units, or null-behavior mismatches.

## Common examples

- `tenant` vs `tenant_id`: usually do not auto-reuse
- `status` vs `three_ds_outcome`: do not reuse
- `order_id` reused as the same primary order key: usually safe reuse candidate

## Output buckets

- **Reuse candidate**
- **New description needed**
- **Do not reuse**
- **User decision required**
- **Rename or modeling issue recommended**
