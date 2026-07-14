# Description Quality Rules For Datamart Metadata

Use this file when a metadata draft sounds generic, over-explains SQL, or still feels misleading after the meaning is understood.

## Core rule

Describe what the field means to a new analyst or AI agent, not what the SQL happened to do.

Use a compact two-part structure inside one string:

- `Functional usage: ...`
- `Agent / analyst notes: ...`

Keep both parts short and specific.

## Red flags

- `SQL maps ...`
- `Derived from logic ...`
- `Country or tenant ...` when one precise meaning can be stated
- Unexplained source nicknames or internal shorthand
- Raw value lists without what the values mean
- Descriptions that blur `NULL`, `Unknown`, `None`, or other explicit states together
- Descriptions that need caveats because the name itself is misleading
- Parent struct descriptions that say only `grouped fields` without naming the domain or grain
- Boilerplate child descriptions that repeat the field name but still do not explain the meaning
- `Agent / analyst notes` that contain generic filler instead of proven metadata
- A shared glossary description made vague enough to fit incompatible table uses
- No named source in `Agent / analyst notes`
- Enum, coded, or flag field with no example values
- Calculated metric with no derivation formula
- Example values listed for an ID, PII-adjacent, or GEOGRAPHY column

## Meaning-first checklist

Ask these questions in order:

1. What is the field's business meaning?
2. What is the grain?
3. Does the current name still match that meaning?
4. Does the reader need units, currency, timezone, or sensitivity notes?
5. Does the field have allowed values, flags, or codes that need explanation?
6. What is the data source? (Read from the transform `*.sql`, not the `*_ddl.sql` self-select — the DDL FROM clause reads from the table itself, not upstream.)
7. If this is a calculated metric: what is the derivation formula?
8. Is `NULL` a separate state from explicit values?
9. Is the current name so misleading that a rename should be recommended?
10. If the field is nested, does the parent context change the meaning?
11. Can the final text stay accurate across all intended table uses without becoming generic?

## Two-part micro-structure

Use this pattern:

`Functional usage: <what the field means and how to interpret it; include enumerated values here for coded/flag/bounded fields>. Agent / analyst notes: Source: <upstream table or system, from the transform SQL>. Formula: <derivation — metrics only>. <grain, join keys, null behavior, or other caveats that affect correct use>.`

Guidance:

- Put the business meaning first.
- `Source:` is always required. Use the **specific BigQuery table path or SQL reference** (e.g. `production_external_adyen.authorisation_events`, `production_je_justsaying.orderreserved_*`), not a vague system name. Read it from the transform `*.sql`, not the DDL. For a field derived purely in-datamart with no upstream table, write `Source: derived in-datamart` plus the expression. If the source is genuinely unknown, flag the gap rather than omitting the label.
- `Formula:` is required only for calculated metrics. Use the actual derivation, not a vague description of it.
- Put join keys or metadata caveats in the second part only when they are proven and useful.
- Do not turn the second part into a long derivation dump.
- If there is no meaningful agent/analyst caveat beyond source, keep that part short and factual instead of inventing filler.

## Enum and flag checklist

- Only list corroborated values.
- Explain each value's meaning in business language.
- Include raw code plus human meaning only when both are proven.
- State whether `NULL` means "no matching source record", "not applicable", "not yet enriched", or another distinct state.
- Never enumerate example values for IDs, PII-adjacent fields (e.g. `customer_id`), or GEOGRAPHY columns.

## Rename recommendation heuristic

Recommend a rename when the best honest description still feels like it is working around a bad name.

Typical signals:

- `tenant` really means `country_code`
- `is_recurring` really means `used_saved_payment_method`
- a field name matches a dashboard legacy, but not the datamart business meaning
- the same field name is overloaded across sources

## Reviewer mindset

Read the draft like a skeptical analyst who is new to the domain:

- Would I understand this without extra tribal knowledge?
- Would I know how to use the field correctly?
- Would I misread the name and make the wrong join, filter, or metric?
- Could I scan the functional meaning and the metadata caveat separately without reading a paragraph?
- If the answer is yes, gather more evidence or recommend a rename.

## Nested fields

For parent structs:

- Explain the grouped business domain, not just that fields are nested together.
- Mention grain or scope when it helps the reader interpret all children.
- Mention shared unit or currency conventions if they apply to the group.

For child fields:

- Add descriptions only where they add real meaning.
- Do not force boilerplate when the parent context and child name already make the meaning obvious.
- If the leaf field is ambiguous without the parent path, treat the full path as the real identifier.
