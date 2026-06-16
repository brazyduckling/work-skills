# Nested Field Rules For Datamart Metadata

Use this file when the schema contains `STRUCT`, `RECORD`, or `REPEATED` fields.

## Core rule

Nested fields are documented at path level, not leaf-name level.

Treat `customer_payments.gross_customer_amounts.gmv_eur` as a different metadata target from another `gmv_eur` under a different parent.

## Canonical identifiers

- Top-level field: `country`
- Nested field: `order_attributes.order_status`
- Nested grandchild: `corporate.revenue.jet_pay_eur`

Use the full dotted path as the default comparison key when checking reuse, drafting descriptions, or explaining ambiguity.

## Description placement

Choose the storage location that matches the meaning:

1. **Shared glossary leaf key**
   - Use when the leaf meaning is globally reusable and does not depend on parent context.

2. **Path-specific glossary key**
   - Use when the exact nested path is reusable and the repo supports path lookups.

3. **Inline DDL description**
   - Use when the meaning depends on the parent struct, the field is table-local, or the parent group itself needs explanation.

Parent struct descriptions usually belong inline in the DDL schema object.

## Parent struct policy

Add a parent description when the struct groups a coherent business domain such as:

- order attributes
- vouchers
- partner billing
- corporate revenue

The parent description should explain:

- what domain the group represents
- the grain or scope of the grouped fields
- any shared unit, currency, or domain convention that helps all children

## Child field policy

Describe child fields when they are:

- ambiguous outside the parent context
- coded or value-driven
- easy to misuse
- operationally important
- using internal shorthand

Do not force boilerplate descriptions for every child field if the parent context and child name already make the meaning obvious.

## Repeated fields

For `REPEATED` fields:

- preserve repeated mode in the DDL proposal
- describe one element of the array
- explain what one repeated element represents if that is not obvious

## Macro and template support

Before proposing nested metadata edits:

- inspect the current macro or DDL pattern
- confirm it can render nested `fields` arrays and nested descriptions
- if it cannot, prepare the minimum support change needed

Do not invent one-off nested syntax that the repo cannot render.

## Common mistakes

- Reusing a leaf description only because the leaf name matches
- Flattening nested paths mentally and losing the parent context
- Adding generic parent descriptions like `Grouped fields`
- Writing every child description even when the result is repetitive and unhelpful
- Forgetting to preserve child order, type, or mode in the DDL proposal
