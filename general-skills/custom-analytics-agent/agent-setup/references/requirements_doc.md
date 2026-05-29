# Analytics Agent — Requirements Doc Body Template

This file holds the **analytics-specific content** for the customer-shareable requirements doc. The generic Confluence-page-creation flow lives in `../../../shared/requirements_doc_pattern.md` and `../../../shared/confluence_folder_setup.md`.

**The body is pre-filled** with the Phase 1 inference bundle — the customer reviews and edits each section rather than writing from scratch. Reference example (audience-agent format, same conventions): [Audience Agent Requirements - Test Customer Retail](https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/4981719206/).

The customer's edits in Phase 4 are **distilled** into the 3 analytics-agent KBs:
- §1, §2, §3, §5, §7 → `knowledge_bases/business_context.md`
- §4 → `knowledge_bases/data_dictionary.md`
- §6 → `knowledge_bases/business_context.md` (Exclusion Rules section)
- §8 → `Analytics Agent/prompt.md` (optional tone/output tweak; only if substantially different from defaults)
- §9 → `knowledge_bases/sql_templates.md` (only if non-empty)

Distillation rules mirror the audience agent's `business_context_template.md` — strip markers, hedge phrases, customer-collaborative voice; convert to terse direct agent instructions.

## Page title

`Analytics Agent Requirements - <Customer>` (every Confluence page title must be suffixed with the customer name — Confluence enforces unique titles per space)

## Inference markers

Every pre-filled answer must be tagged with one of two markers:

| Marker | Meaning |
|---|---|
| `[INFERRED — please confirm]` | We made a confident guess from the database (table names, column names, sample values, distributions). Customer should verify. |
| `[NEEDS YOUR INPUT]` | We genuinely couldn't infer this from the data. Customer must provide. |

Use `[INFERRED — please confirm or override]` / `[INFERRED — please confirm or replace]` as natural variants.

Do NOT use `<TODO>` placeholders.

## Body structure

> **CRITICAL — replace, don't preserve.** The body template below contains `[Inference from Phase 1 §X: ...]` blocks as **authoring instructions to the LLM**. Replace each bracketed *authoring instruction* with actual inferred content. Do NOT publish the authoring instructions to the customer.
>
> The `[INFERRED — please confirm]` and `[NEEDS YOUR INPUT]` markers ARE customer-facing labels — they appear verbatim in the published page.
>
> Other template placeholders that should remain in the published body: `<Customer>`, `<database>`, `[N tables across <database>]`, `[agreed date]`, `[Slack channel / email]`.

### Bracket types — quick reference

| Bracket form | Category | Action when rendering |
|---|---|---|
| `[Inference from Phase 1 §X: ...]` | LLM authoring instruction | **Replace** with actual inferred content |
| `<Customer>`, `<database>` | Template placeholder | **Replace** with real value |
| `[N tables across <database>]`, `[agreed date]`, `[Slack channel / email]` | Template placeholder | **Replace** with real value |
| `[INFERRED — please confirm]`, `[NEEDS YOUR INPUT]` (and natural variants) | Customer-facing marker | **Keep verbatim** in the published page |

Use the canonical full example as a reference when rendering: [Audience Agent Requirements - Test Customer Retail](https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/4981719206/) (same format, audience-agent context).

Use this body when calling `createConfluencePage` (everything between the outer ```` ``` ```` lines below — peel the wrapper off when passing to the API):

````markdown
# Analytics Agent Requirements - <Customer>

**Purpose:** Help us tailor the Custom Analytics Agent to your business. Answers below were drafted from `<database>` ([N tables]) — please review, correct, or expand.

**Target return date:** [agreed date]

> **Note from FDE:** Sections below are pre-populated with **best-guess answers inferred from the data** (table names, column names, distinct values, distributions). Treat these as a starting draft — please correct, expand, or replace anything that doesn't match your business. Items marked `[INFERRED — please confirm]` are our guesses; items marked `[NEEDS YOUR INPUT]` we couldn't infer at all.

---

## 1. Business Model

In 1-3 sentences, describe what your business does, your primary channels, and your customer types.

> _Your answer:_
>
> [Inference from Phase 1 §1, prefixed with `[INFERRED — please confirm]`. 1-3 sentences inferred from table names + sample values (e.g., a database with `transactions`, `products`, `customers`, `orders` tables → "Multi-channel retailer..." or "B2B SaaS company..." depending on signals). Cite concrete table evidence: "Inferred from `<table>` containing fields like `<column>`."]

---

## 2. Business KPIs

What are the 3-5 metrics that matter most to your team? (e.g., revenue, customer count, retention rate, engagement rate)

> _Your answer:_
>
> [Inference from Phase 1 §2, prefixed with `[INFERRED — please confirm]`. 4-6 KPIs derived from numeric columns + table semantics. Each as a bullet with the underlying table.column reference and current value where computable:
>
> * **<KPI 1>** — `<table>.<column>` (currently <value> over the last <window>)
> * **<KPI 2>** — derived from joining `<table_a>` and `<table_b>` on `<key>`
> ]

---

## 3. Key Terms

Are there terms or shorthand your team uses that the agent should recognize? Define each one.

Examples:
* "VIP" = top-tier customers
* "churned" = no purchase in 180 days

> _Your answer:_
>
> [Inference from Phase 1 §3, prefixed with `[INFERRED — please confirm or override]`. Use enum values from categorical columns + behavior-derived terms. Use `[NEEDS YOUR INPUT]` inline for terms where the data suggests a definition but we don't know the customer's cutoff.
>
> Example shape:
> * **"<term>"** = `<table>.<column> = '<value>'` — [meaning]. Currently N rows.
> * **"<term>"** = `[NEEDS YOUR INPUT]` — Default would be [reasonable default based on data]. Please confirm.
> ]

---

## 4. Priority Tables / Datasets

Which tables in `<database>` matter most for your analysis? Especially flag any with cryptic column names that need explanation.

> _Your answer:_
>
> [Inference from Phase 1 §4, prefixed with `[INFERRED — please confirm or replace]`. Top 5-10 tables by likely usefulness, each with a 1-line purpose + the most-important columns.
>
> Example shape:
> * **`<database>.<table>`** — [purpose]. Key columns: `<col1>` ([type], [description]), `<col2>` ([type], [description]). Row count: ~N.
> * **`<database>.<table>`** — ...
>
> Flag column-type traps inline (e.g., a `varchar` date column needs `TD_TIME_PARSE`). Close with a `[NEEDS YOUR INPUT]` line asking the customer to confirm any semantic ambiguities or flag tables we missed.]

---

## 5. Common Analytical Questions / Patterns

What kinds of questions should the agent answer well? (e.g., "weekly revenue by region", "top products by margin", "cohort retention", "churn prediction")

> _Your answer:_
>
> [Inference from Phase 1 §5. Suggest 4-6 question patterns the schema supports, prefixed with `[INFERRED — please confirm or extend]`. Each as a bullet with the rationale.
>
> Example shape:
> * **Time-series of <metric>** — supported by `<table>.<time_column>` + `<table>.<metric_column>`.
> * **Top-N by <dimension>** — supported by joining `<table_a>` and `<table_b>` on `<key>`.
> * **Cohort retention by signup month** — supported by `<table>.signup_date` + `<table>.transaction_date`.
> ]

---

## 6. Exclusions

What should the agent never query or surface? (PII columns, sensitive tables, test environments, opted-out customers)

> _Your answer:_
>
> [Inference from Phase 1 §6. Three sub-blocks, each with its own marker:
>
> **PII columns to exclude from query output** `[INFERRED — please confirm]`:
> * `<table>.email` — auto-flagged
> * `<table>.first_name`, `<table>.last_name` — auto-flagged
> * `<table>.phone` — auto-flagged
>
> **Customer / row filters to apply** `[NEEDS YOUR INPUT]`:
> * Common patterns: exclude test accounts, exclude unsubscribed customers, exclude refunded orders. We see `<table>.<column>` with values like `<sample>` — should any of these signal "exclude by default"?
>
> **Tables to hide** `[INFERRED — please confirm]`:
> * If any table looks sensitive (HR, support tickets, complaints, internal flags), list here. Default: "all N tables in `<database>` are in scope" if nothing looks sensitive.
> ]

---

## 7. Target Users & Tone

Who will use this agent (analysts, marketing, executives, CS)? Any tone preferences? Default is "neutral, professional analyst — concise, data-grounded."

> _Your answer:_
>
> [Inference from Phase 1 §7, prefixed with `[INFERRED — please confirm]`. Conservative defaults:
> "Primary users assumed to be **analysts and operations teams**. Tone defaulted to neutral, professional analyst — concise, data-grounded, declines off-topic questions. If executives or non-technical stakeholders will also use this, please flag for an alternative accessible tone."]

---

## 8. Output Preferences

What format works best — Plotly charts, React dashboards, summary text, tables, or mixed? Any preferred chart types?

> _Your answer:_
>
> [Inference from Phase 1 §8, prefixed with `[INFERRED — please confirm]`. Conservative defaults:
> "* **Default to a mix:** 1-2 sentence summary + chart (Plotly for single charts, React for multi-chart dashboards) + small results table.
> * **Preferred chart types** based on data shape: bar charts for breakdowns by category/dimension, line charts for time-series trends, pie charts only when ≤5 categories, heatmaps for cross-tabs.
> * **Color palette:** TD default. Update if you have a brand palette to match."]

---

## 9. SQL Templates *(Optional)*

If your team has recurring SQL patterns you'd like the agent to know about (e.g., "compute retention by cohort", "top products by margin"), paste them here. Each template should include:
* A short name
* A 1-line description of when to use it
* The SQL itself

The agent will load these as a separate knowledge base (`sql_templates.md`) and reference them when relevant.

> _Your templates (optional):_
>
> [Inference from Phase 1 §9, prefixed with `[INFERRED — proposed starter templates based on common patterns for this kind of database]`. Up to 3 templates derived from schema. Each template uses real column names from `<database>` and TD-Trino-correct functions. Format:
>
> **<Template name>** — Use when <plain-English description>:
>
> ```sql
> <concrete SQL using this customer's tables and columns; uses TD_TIME_PARSE / TD_INTERVAL / TD_TIME_RANGE; uses {{N}} placeholders for parameterized values>
> ```
>
> If schema doesn't suggest meaningful templates, write: `_(No SQL templates inferred from schema. Add any recurring patterns your team uses below.)_`]

---

When you're done, please notify us at [Slack channel / email] and we'll incorporate the answers into the agent.
````

## Authoring rules

- Every pre-filled answer is prefixed with `[INFERRED — please confirm]` (or natural variant) **or** `[NEEDS YOUR INPUT]`. No exceptions.
- Pre-filled content uses concrete values from `<database>` — real table names, real column names, real distinct values, real counts from Phase 1 queries.
- Use blockquote (`> ...`) for the customer-facing answer block under `> _Your answer:_`.
- Numeric stats must come from actual queries during Phase 1, not estimates.
- Do not include raw PII values in the body — use counts, ranges, or hashes for any column matching PII patterns.
- The original question text under each `## N. Section Name` heading is preserved verbatim — the customer sees both the question they're being asked AND our inferred draft answer.

## Reference: Section-to-File Mapping (used in Phase 4)

When the customer returns the edited doc, each section maps to a specific KB. **Important: the 3 customer-specific KBs are agent-facing — distill into terse direct instructions, don't copy verbatim from Confluence.** Distillation rules mirror audience-agent's `business_context_template.md`.

| Customer answer in | Distilled into |
|---|---|
| §1 Business Model | `business_context.md` → Business Model |
| §2 Business KPIs | `business_context.md` → Business KPIs (or its own KPI section) |
| §3 Key Terms | `business_context.md` → Key Terms |
| §4 Priority Tables / Datasets | `data_dictionary.md` (table list, columns, types, gotchas) |
| §5 Common Analytical Questions | `business_context.md` → Common Patterns (informs how the agent reasons about analytical questions) |
| §6 Exclusions | `business_context.md` → Exclusion Rules |
| §7 Target Users & Tone | `Analytics Agent/prompt.md` (optional — only edit if substantially different from defaults) |
| §8 Output Preferences | `Analytics Agent/prompt.md` (optional) |
| §9 SQL Templates *(optional)* | `sql_templates.md` (only if non-empty) |

## Reference: Phase 4 Validation Summary Format

Before re-pushing in Phase 4, present this summary back to the FDE engineer (not the customer):

```
Custom Analytics Agent — Requirements Summary
=============================================

Customer: [name]
Project: [exact project name, e.g., "ACME Analytics Agent"]
Database: <database>
Confluence folder: [URL]
Customer requirements doc: [URL]

Business Model (distilled): [final, post-customer-edits, 1-3 sentences]

Business KPIs (distilled): [bulleted list, terse]

Priority Tables (distilled into data_dictionary.md):
- <table> — [purpose, key columns, gotchas]
- ...

Key Terms (distilled):
- [term] = [definition]
- ...

Common Analytical Patterns: [brief list]

Exclusions:
- PII columns: [from §6]
- Customer filters: [from §6]
- Tables hidden: [from §6]

SQL Templates: [yes — N templates / no]

Customer feedback notes: [Any [NEEDS YOUR INPUT] items the customer left unfilled, or sections they edited substantially vs lightly]

Please confirm before I update the KBs and re-push.
```

Once confirmed, proceed to Phase 4 in the parent `SKILL.md` (distill into the 3 KBs, re-push, re-run tests after the FDE engineer approves).
