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

## Ask the user to decide when

- The old glossary key is close but not exact
- The same metric is reused with different business framing
- Null behavior is unclear
- The source SQL is not enough to explain the field safely

## Common examples

- `tenant` vs `tenant_id`: usually do not auto-reuse
- `status` vs `three_ds_outcome`: do not reuse
- `order_id` reused as the same primary order key: usually safe reuse candidate

## Output buckets

- **Reuse candidate**
- **New description needed**
- **Do not reuse**
- **User decision required**
