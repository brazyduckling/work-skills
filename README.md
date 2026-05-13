# work-skills

Personal AI skills for data and product work.

---

## Skills

| Skill | Description |
|-------|-------------|
| [mw-bq](./mw-bq/SKILL.md) | BigQuery data exploration, querying, and optimization with mandatory confirmation gates |

---

## Installation (Claude Code)

1. **Clone this repository** into your Claude Code skills directory:
   ```bash
   cd ~/.claude/skills
   git clone https://github.com/brazyduckling/work-skills.git
   ```

2. **Or link the skill directly** if you prefer:
   ```bash
   ln -s /path/to/work-skills/mw-bq ~/.claude/skills/mw-bq
   ```

3. **Verify installation** by invoking the skill in Claude Code:
   ```
   /mw-bq
   ```

The skill will be automatically discovered and available for use in Claude Code.

---

## mw-bq vs jet-bq — What's Different?

`mw-bq` is based on JET's internal `jet-bq` skill (`ai-platform/skills`). The core content is the same — same query writing guidelines, same optimization rules, same CLI patterns, same reference files. What differs is the workflow enforcement.

### What `mw-bq` adds

#### Confirmation Protocol

Every query execution requires explicit user approval — no exceptions:

1. Write and show the SQL
2. Run `--dry_run` to get bytes scanned
3. Present: **"Query will scan ~X GB (~$Y). Run it?"**
4. Wait for explicit confirmation before executing

`jet-bq` mentions dry-run as a best practice. `mw-bq` makes it a hard gate.

Free operations (metadata, `bq head`, `bq ls`, `--dry_run`) skip the gate.

#### Schema-First Protocol

Before writing any query, always:

1. `bq ls` — confirm which tables exist
2. `bq show --schema` — inspect every column
3. `bq head -n 10` — look at actual values *(column names lie)*
4. Summarize findings and confirm the plan with the user
5. Then write the query

`jet-bq` has no equivalent — you can start writing SQL immediately.

#### JOIN explanation rule

After every query, always note which JOIN type was used and why:

> *"Used LEFT JOIN — keeps all left-table rows; unmatched rows show as NULL and stay in totals."*

Prevents silent row drops from unexpected INNER JOINs going unnoticed.

#### Scope boundary

Explicitly states: *"Do NOT use for Athena queries — use jet-aws-athena or jet-odl-athena instead."*

---

### What `jet-bq` has that `bq` doesn't

| | `jet-bq` | `mw-bq` |
|---|---|---|
| Windows install | ✅ `winget install Google.CloudSDK` | ❌ macOS/Linux only |
| `future-ideas.md` reference file | ✅ | ❌ |

---

### TL;DR

`mw-bq` = `jet-bq` + three safety layers: you must always look at the data before writing a query, and the AI must always ask before running anything that costs money.
