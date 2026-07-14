# Eval Rubric — Adding Datamart Metadata

Score each dimension as:

- **2 = strong**
- **1 = acceptable**
- **0 = fail**

## 0. Shared Glossary Baseline

Did the skill actually start from the shared glossary baseline?

- **2** — Explicitly inspected `ftdna_macros.jinja` and treated it as mandatory baseline
- **1** — Mentions the shared glossary, but the baseline use is not fully clear
- **0** — Skips the shared glossary, or acts as if the table folder defines whether a glossary exists

## 1. Resolution Path Choice

Did the skill choose the right path?

- **2** — Correctly chose among reuse, revise, add, path-specific, rename/modeling, or inline-DDL exception
- **1** — Plausible but not clearly best path
- **0** — Wrong path, especially if it weakens the glossary

## 2. Shared Glossary Protection

Did the skill keep `ftdna_macros.jinja` precise and reusable?

- **2** — Protects the glossary from vague or table-local dumping
- **2** — Protects the glossary from vague or table-local dumping, and revises stale overfit definitions when needed
- **1** — Mostly correct, but leaves some ambiguity
- **0** — Pollutes the glossary or avoids it without good reason

## 3. Evidence Discipline

Did the skill ground its decision in real evidence?

- **2** — Uses repo/schema/value/doc/review evidence and labels the basis clearly
- **1** — Uses some evidence, but gaps remain
- **0** — Hand-wavy or naming-taste-driven

## 4. Description Quality

Is the proposed wording specific, accurate, and non-generic?

- **2** — Meaning-first, precise, no filler, no SQL narration unless necessary
- **1** — Usable, but still a bit broad or defensive
- **0** — Generic, misleading, or mostly SQL commentary

## 5. Two-Audience Structure

Does the proposal clearly separate the two minimal parts?

- **2** — Uses clear labels like `For stakeholders:` and `For agents and analysts:` and both parts add value
- **1** — Structure exists but one part is weak
- **0** — No clear split, or the second part is empty filler

## 6. Exception Handling

If the skill used a path-specific key, rename, modeling fix, or inline DDL, was that justified?

- **2** — Exception is clearly the cleanest option and well explained
- **1** — Exception may be right, but explanation is thin
- **0** — Exception is premature or unsupported

## Critical dimensions

These must never score **0**:

- Shared Glossary Baseline
- Resolution Path Choice
- Shared Glossary Protection
- Evidence Discipline
- Description Quality

## Common fail patterns

- "Status of the record" style generic definitions
- one shared definition forced across incompatible tables
- concluding "no glossary file in this family" without inspecting `ftdna_macros.jinja`
- keeping a stale shared definition that is clearly overfit to one older table
- inline DDL chosen just because the field came from one table
- rename suggested without showing why the old name is misleading
- two-audience split collapsed into one paragraph
