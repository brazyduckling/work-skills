# Eval Protocol — Adding Datamart Metadata

Use this eval pack to check whether the skill protects the shared glossary while still producing useful metadata proposals.

## Why this eval exists

This skill is effectively maintaining a **data dictionary / shared glossary** for FTDNA metadata.

The quality bar is not just "did it write a description?" It must also:

1. choose the right resolution path
2. keep the shared glossary precise
3. avoid generic definitions
4. separate business meaning from metadata guidance
5. justify rename or inline-DDL exceptions with evidence

## Best-practice basis

This eval is based on standard data-dictionary and business-glossary expectations:

- a data dictionary is a centralized repository of metadata about **meaning, relationships, origin, usage, and format**
- glossary terms should provide **clear business vocabulary** and map that vocabulary to technical assets such as tables and columns

Those principles translate here into:

- **central shared glossary first**
- **precise reusable definitions**
- **clear exceptions only when reuse would mislead**
- **business meaning plus usage metadata**

## How to run the eval

For each case in `cases.jsonl`:

1. Give the case input to the skill.
2. Compare the output against `rubric.md`.
3. Record:
   - chosen path
   - whether the evidence was sufficient
   - whether the description stayed specific
   - whether the two-audience structure was clear
4. Mark the case:
   - **Pass**
   - **Borderline**
   - **Fail**

## Critical pass conditions

The skill should fail the case if it does any of these:

- claims there is no shared glossary because the current table folder lacks one
- chooses inline DDL before exhausting glossary-first options
- reuses a shared glossary key despite clear meaning mismatch
- preserves a stale table-specific shared definition when new-table evidence should trigger a revision
- writes a generic definition just to preserve reuse
- recommends a rename without evidence
- omits the two-audience structure when proposing final description text

## Suggested release gate

Treat the update as healthy only if:

- no critical fails
- all critical dimensions in `rubric.md` score at least **1**
- at least **7 of 8** core cases pass cleanly

## Maintenance note

When the skill logic changes, update:

- `cases.jsonl` if new edge cases appear
- `rubric.md` if the decision policy changes
- `SKILL.md` version in frontmatter
