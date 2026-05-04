# work-skills

Personal AI skills for data and product work.

---

## Skills

| Skill | Description |
|-------|-------------|
| [bq](./bq/SKILL.md) | BigQuery data exploration, querying, and optimization with mandatory confirmation gates |

---

## bq vs jet-bq — What's Different?

This repo contains my personal `bq` skill. JET's internal platform has an official `jet-bq` skill. Here's how they differ in plain terms:

### What they share

Both skills do the same core job: help an AI assistant run BigQuery queries, explore schemas, estimate costs, and interact with the `bq` CLI. Same cost table, same Standard SQL requirement, same `bq head` / `bq show` / `bq ls` patterns.

---

### Where `bq` (this skill) is different

#### 1. Mandatory confirmation before every query

`jet-bq` treats dry-run as a best practice you *should* follow. `bq` makes it a **hard rule**: show the SQL → dry-run for cost → present estimate → wait for explicit user approval → then and only then execute. No exceptions.

The confirmation request always looks like:
> **Query will scan ~X GB (~$Y). Run it?**

#### 2. Schema-First Protocol (mandatory)

Before writing *any* query, `bq` requires a fixed sequence:

1. Discover tables (`bq ls`)
2. Inspect schema (`bq show --schema`)
3. Sample actual values (`bq head -n 10`) — *because column names lie*
4. Summarize findings and validate plan with the user
5. Only then write the query

`jet-bq` has no equivalent protocol. You can jump straight to writing SQL.

#### 3. No Athena confusion

`bq` explicitly says: *"Do NOT use for Athena queries — use jet-aws-athena or jet-odl-athena instead."* Useful in a JET context where multiple query engines exist.

---

### Where `jet-bq` is more complete

| Area | `jet-bq` | `bq` (this skill) |
|------|----------|-------------------|
| Windows install | ✅ `winget install Google.CloudSDK` | ❌ macOS only |
| Query Writing Guidelines | ✅ 8 numbered rules | ❌ |
| Optimization section | ✅ 8 rules + reference links | ❌ |
| Output & Reporting section | ✅ | ❌ |
| Reference docs | 4 files (CLI, patterns, optimization, future ideas) | 3 files (CLI, optimization, patterns — no future-ideas) |

---

### TL;DR

> **`bq`** is stricter and safer — it forces you to always check the data shape first and get explicit approval before running anything that costs money.
>
> **`jet-bq`** is broader and more reference-heavy — it's a comprehensive guide with more materials, but it trusts you to exercise your own judgment on when to dry-run and how to explore schemas.

Use `bq` when you want an AI that won't run expensive queries by surprise. Use `jet-bq` as a reference when you want query patterns, optimization guides, and broader tooling coverage.
