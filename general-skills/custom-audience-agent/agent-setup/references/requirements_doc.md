# Audience Agent — Requirements Doc Body Template

This file holds the **audience-specific content** for the customer-shareable requirements doc. The generic Confluence-page-creation flow lives in `../../../shared/requirements_doc_pattern.md` and `../../../shared/confluence_folder_setup.md`.

**The body is pre-filled** with the Phase 1 inference bundle — the customer reviews and edits each section rather than writing from scratch. Reference example: [Audience Agent Requirements - Test Customer Retail](https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/4981719206/).

The customer's edits in Phase 4 are **distilled** into `business_context.md` (an agent-facing KB — see `business_context_template.md` for the distillation rules; the KB is NOT a copy of the Confluence doc).

## Page title

`Audience Agent Requirements - <Customer>` (every Confluence page title must be suffixed with the customer name — Confluence enforces unique titles per space)

## Inference markers

Every pre-filled answer must be tagged with one of two markers so the customer knows what to scrutinize:

| Marker | Meaning |
|---|---|
| `[INFERRED — please confirm]` | We made a confident guess from the data (column names, distinct values, distributions). Customer should verify. |
| `[NEEDS YOUR INPUT]` | We genuinely couldn't infer this from the data. Customer must provide. |

Use `[INFERRED — please confirm or override]` / `[INFERRED — please confirm or replace]` / `[INFERRED — proposed starter templates]` etc. as natural phrasing variants of the same idea.

Do NOT use generic `<TODO>` placeholders. The customer-facing markers are part of the expected format.

## Body structure

> **CRITICAL — replace, don't preserve.** The body template below contains `[Inference from Phase 1 §X: ...]` blocks as **authoring instructions to the LLM**. Replace each bracketed *authoring instruction* with actual inferred content. Do NOT publish the authoring instructions to the customer.
>
> The `[INFERRED — please confirm]` and `[NEEDS YOUR INPUT]` markers, on the other hand, ARE customer-facing labels — those should appear verbatim in the published Confluence page next to the corresponding inferred / missing answers.
>
> Other template placeholders that should remain in the published body: `<Customer>`, `<Parent Segment>`, `[N customers across M behavior tables]`, `[agreed date]`, `[Slack channel / email]`.

### Bracket types — quick reference

The body has four categories of brackets. The LLM must handle each correctly:

| Bracket form | Category | Action when rendering |
|---|---|---|
| `[Inference from Phase 1 §X: ...]` | LLM authoring instruction | **Replace** with actual inferred content |
| `<Customer>`, `<Parent Segment>` | Template placeholder | **Replace** with real value |
| `[N customers across M behavior tables]`, `[agreed date]`, `[Slack channel / email]` | Template placeholder | **Replace** with real value |
| `[INFERRED — please confirm]`, `[NEEDS YOUR INPUT]` (and natural variants like `[INFERRED — please confirm or override]`) | Customer-facing marker | **Keep verbatim** in the published page |

### Worked example: §1 rendered for a real customer

To illustrate which brackets resolve and which stay:

**Template instruction (authoring guidance to the LLM):**

> [Inference from Phase 1 §1, prefixed with `[INFERRED — please confirm]`. 1-3 sentences citing concrete distinct values + percentages from the customers table — categories breakdown, channel breakdown, customer-segment breakdown, key averages (LTV, AOV).]

**Rendered output that gets published to Confluence:**

> `[INFERRED — please confirm]` Multi-category retailer selling Clothing (~30% of orders), Home goods (~22%), Food (~21%), Electronics (~19%), and Beauty (~10%). Three sales channels — **Online** (~49% of transactions), **In-Store** (~26%), and **Mobile** (~25%). Customer base of ~1,000 active profiles segmented into Regular shoppers (53%), New customers (20%), At-Risk customers (17%), and VIP (10%). Average order value $658, lifetime value averages $16k.

What changed:
- `[Inference from Phase 1 §1, ...]` → resolved to actual content
- `[INFERRED — please confirm]` → kept verbatim (customer marker)
- Concrete numbers (`~30%`, `$658`, `$16k`) come from Phase 1 distribution queries

Use the canonical full example as a reference when rendering the rest: [Audience Agent Requirements - Test Customer Retail](https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/4981719206/).

Use this body when calling `createConfluencePage` (everything between the outer ```` ``` ```` lines below — peel the wrapper off when passing to the API):

````markdown
# Audience Agent Requirements - <Customer>

**Purpose:** Help us tailor the Custom Audience Agent to your business. Please fill in each section below. None of the fields are mandatory, but the more you provide, the better the agent will understand your data and respond to your team's questions.

**Target return date:** [agreed date]

> **Note from FDE:** Sections below are pre-populated with **best-guess answers inferred from the data** (column names, distinct values, distributions in the parent segment). Treat these as a starting draft — please correct, expand, or replace anything that doesn't match your business. Items marked `[INFERRED — please confirm]` are our guesses; items marked `[NEEDS YOUR INPUT]` we couldn't infer at all.

---

## 1. Business Model

In 1-3 sentences, describe what you sell, your main channels, and your key customer types.

> _Your answer:_
>
> [Inference from Phase 1 §1, prefixed with `[INFERRED — please confirm]`. 1-3 sentences citing concrete distinct values + percentages from the customers table — categories breakdown, channel breakdown, customer-segment breakdown, key averages (LTV, AOV). Example shape: "`[INFERRED — please confirm]` Multi-category retailer selling Clothing (~30% of orders), Home goods (~22%)... Three sales channels — **Online** (~49%), **In-Store** (~26%), and **Mobile** (~25%). Customer base of ~N active profiles segmented into Regular shoppers (X%), New customers (Y%), At-Risk customers (Z%), and VIP (W%). Average order value $X, lifetime value averages $Y."]

---

## 2. Business KPIs

What are the 3-5 metrics that matter most to your team? (e.g., LTV, repeat purchase rate, email engagement)

> _Your answer:_
>
> [Inference from Phase 1 §2, prefixed with `[INFERRED — please confirm]`. 4-6 KPIs as a bulleted list. Each bullet: bold KPI name — column/table reference — current value computed from the data. Example shape:
>
> * **Lifetime value (LTV)** — `customers.lifetime_value`, currently averaging $X across N customers
> * **VIP retention** — % of `customer_segment = 'VIP'` (currently N customers, X% of base) that stay engaged quarter-over-quarter
> * **Email engagement rate** — opens (N) and clicks (M) divided by sends (P) per `behavior_email_events` — currently ~X% open, ~Y% click
> ]

---

## 3. Key Terms

Are there terms or shorthand your team uses that the agent should recognize? Define each one.

Examples:

* "VIP" = Tier 3+ loyalty members
* "churned" = no purchase in 180 days

> _Your answer:_
>
> [Inference from Phase 1 §3, prefixed with `[INFERRED — please confirm or override]`. Include enum values from `customer_segment` / `churn_risk` / `loyalty_tier` with their counts. Use `[NEEDS YOUR INPUT]` inline for any term where the data suggests a definition but we genuinely don't know the customer's cutoff (e.g., "New" = first 30 vs 90 days, "Churned" = 90 vs 180 days, "High-value" threshold). Example shape:
>
> * **"VIP"** = `customer_segment = 'VIP'` — top-tier customers (currently N of M). NOTE: distinct from `loyalty_tier` (Bronze/Silver/Gold), which is the rewards-program tier.
> * **"At-Risk"** = `customer_segment = 'At-Risk'` — customers flagged as likely to churn (N customers).
> * **"New"** = `customer_segment = 'New'` (N customers). `[NEEDS YOUR INPUT]` What's the cutoff for "New" — first 30 days? 90 days?
> * **"Churned"** = `[NEEDS YOUR INPUT]` — Default would be no transaction in `behavior_transactions` for the last 180 days. Please confirm or specify your team's definition.
> ]

---

## 4. Priority Attributes

Which 5-10 customer attributes (columns on the `customers` table) matter most for your analysis? Especially flag any with cryptic column names that need explanation.

Example:

* `lifetime_value_usd` — total revenue across all orders
* `signup_channel` — values: web, app, referral, partner

> _Your answer:_
>
> [Inference from Phase 1 §4, prefixed with `[INFERRED — please confirm or replace]`. Bullet list of top ~10 columns from `customers` with concrete distribution stats (range, mean, top values, gotchas). Flag column-type traps inline — e.g., a `varchar` date column needs `TD_TIME_PARSE`. Example shape:
>
> * `lifetime_value` — total revenue per customer (USD). Range $X – $Y, mean $Z.
> * `customer_segment` — strategic segment label. Values: `Regular`, `New`, `At-Risk`, `VIP`.
> * `loyalty_tier` — rewards-program tier. Values: `Bronze`, `Silver`, `Gold`, or empty string for non-members. **Treat empty string as "not enrolled"**, not as missing data.
> * `signup_date`, `join_date`, `tier_expiry_date` — all stored as `varchar`, not `timestamp`. **Use** `TD_TIME_PARSE` before any time arithmetic.
>
> Close with a `[NEEDS YOUR INPUT]` line asking the customer to confirm semantic ambiguities (e.g., "Please confirm whether `signup_date` reflects account creation or first purchase. Also flag any other attributes that are most important to your team that we should highlight.")]

---

## 5. Segment Naming Conventions

Do you have any naming patterns for segments? (e.g., "all customer-facing segments start with `CS_`", region suffixes like `_US`, `_JP`)

> _Your answer:_
>
> [Inference from Phase 1 §5. If `tdx sg list` returned existing folder structure, use `[INFERRED — please confirm]` and list the folders + sample segment titles. If no existing segments / no clear pattern, use `[NEEDS YOUR INPUT]` and a 1-2 sentence prompt asking the customer to share any prefix/suffix conventions, with a fallback statement that the agent will use descriptive titles otherwise. Example fallback shape: "`[NEEDS YOUR INPUT]` We don't have visibility into your existing segment naming conventions. If your team uses prefixes (e.g., `CAMP_`, `LIFECYCLE_`, `EXP_` for experimental), suffixes for regions or languages, or any consistent pattern, please describe so the agent can suggest segment titles that match. If no convention exists, leave blank and the agent will use descriptive titles like 'VIP Customers - 90 Day No Purchase'."]

---

## 6. Exclusions

What should the agent never include by default?

* **PII columns to exclude:** (e.g., email, phone, full_name)
* **Customer filters to apply:** (e.g., test accounts, opted-out users, specific countries, unmarketable customers)
* **Behavior tables to hide:** (any internal/sensitive ones)

> _Your answer:_
>
> [Inference from Phase 1 §6. Three sub-blocks, each with its own marker:
>
> **PII columns to exclude** `[INFERRED — please confirm]`:
> * `email` — auto-flagged
> * `first_name` — auto-flagged
> * `last_name` — auto-flagged
>
> **Customer filters to apply** `[NEEDS YOUR INPUT]`:
> * Most common patterns: exclude test accounts (`email LIKE '%@test.%'`), exclude unsubscribed customers (typically derived from `behavior_email_events.event_type = 'Unsubscribed'` — currently N events), exclude customers with empty `email` or any explicit opt-out flag. Please specify which apply to your business.
> * If applicable, ask the customer about specific signals: "We see N `Unsubscribed` events in `behavior_email_events` — should we treat any customer with this event as 'do not email'?"
>
> **Behavior tables to hide** `[INFERRED — please confirm]`:
> * Flag any behavior tables that may contain sensitive content (support tickets, complaints, internal flags). Default: "all N behavior tables in scope" if nothing looks sensitive.]

---

## 7. Target Users & Tone

Who will use this agent (marketing, analytics, CS, executives)? Any tone preferences? Default is "neutral, professional marketing analyst."

> _Your answer:_
>
> [Inference from Phase 1 §7, prefixed with `[INFERRED — please confirm]`. Conservative defaults: marketing + analytics primary users; tone defaulted to "neutral, professional marketing analyst — concise, data-grounded, declines off-topic questions". Mention that if executives or CS will also use the agent, customer should flag for an alternative tone option.]

---

## 8. Output Preferences

What format works best — Plotly charts, tables, summary text, or mixed? Any preferred chart types?

> _Your answer:_
>
> [Inference from Phase 1 §8, prefixed with `[INFERRED — please confirm]`. Conservative defaults:
>
> * Default: **mixed** — Plotly charts for distributions/trends, tables for top-N lists, 1-sentence summary up top.
> * Preferred chart types based on data shape: bar / line / pie depending on what the underlying columns suggest.
> * Color palette: TD default. Note that customer can override with brand palette.]

---

## 9. SQL Templates *(Optional)*

If your team has recurring SQL patterns you'd like the agent to know about (e.g., "compute repeat-purchase rate by cohort"), paste them here. Each template should include:

* A short name
* A 1-line description of when to use it
* The SQL itself

The agent will load these as a separate knowledge base (`sql_templates.md`) and reference them when relevant.

> _Your templates (optional):_
>
> [Inference from Phase 1 §9, prefixed with `[INFERRED — proposed starter templates based on common <vertical> patterns]`. Up to 3 templates derived from schema. Each template uses real column names + real TD functions (TD_INTERVAL, TD_TIME_RANGE, TD_TIME_PARSE, TD_SCHEDULED_TIME). Format:
>
> **<Template name>** — Use when <plain-English description>:
>
> ```sql
> <concrete SQL using this customer's tables and columns>
> ```
>
> Close with: "Please add, remove, or replace these. They are placeholders — your team's actual recurring patterns are more valuable."]

---

When you're done, please notify us at [Slack channel / email] and we'll incorporate the answers into the agent.
````

## Authoring rules

- Every pre-filled answer must be prefixed with `[INFERRED — please confirm]` (or a natural variant) **or** `[NEEDS YOUR INPUT]`. No exceptions.
- Pre-filled content uses concrete values from the parent segment — real column names, real distinct values, real counts, real distribution stats from Phase 1 queries.
- Use blockquote (`> ...`) for the customer-facing answer block under `> _Your answer:_`. The blockquote rendering visually separates the inference from the question text.
- Numeric stats must come from actual queries run during Phase 1, not estimates.
- Do not include raw PII values in the body — use counts, ranges, or hashes for any column matching the PII patterns.
- The original question text under each `## N. Section Name` heading is preserved verbatim — the customer sees both the question they're being asked AND our inferred draft answer.

## Reference: Section-to-File Mapping (used in Phase 4)

When the customer returns the edited doc, each section maps to a specific file. **Important: `business_context.md` is agent-facing — it's distilled into terse direct instructions, not copied verbatim from Confluence.** See `business_context_template.md` for the distillation rules.

| Customer answer in | Distilled into |
|---|---|
| §1 Business Model | `business_context.md` → Business Model |
| §2 Business KPIs | `business_context.md` → Business Model (or its own KPI section) |
| §3 Key Terms | `business_context.md` → Key Terms |
| §4 Priority Attributes | `business_context.md` → Priority Attributes |
| §5 Segment Naming | `business_context.md` → Segment Naming Conventions |
| §6 Exclusions | `business_context.md` → Exclusion Rules |
| §7 Target Users & Tone | `Custom Audience Agent/prompt.md` (optional) |
| §8 Output Preferences | `Custom Audience Agent/prompt.md` (optional) + `chat_parent_segment.yml` welcome message |
| §9 SQL Templates *(optional)* | New KB: `knowledge_bases/sql_templates.md` |

## Reference: Phase 4 Validation Summary Format

Before re-pushing in Phase 4, present this summary back to the FDE engineer (not the customer):

```
Custom Audience Agent — Requirements Summary
============================================

Customer: [name]
TD-Managed Project: [exact project name, e.g., "TD-Managed: Ecommerce Audience"]
Confluence folder: [URL]
Customer requirements doc: [URL]

Business Model (distilled): [1-2 sentences of direct instruction]

Business KPIs (distilled): [bulleted list, terse]

Priority Attributes (distilled):
- [column] — [meaning]
- ...

Key Terms (distilled):
- [term] = [definition]
- ...

Segment Naming Convention: [pattern from §5, or "no convention — use descriptive titles"]

Target Users: [from §7]

Exclusions:
- PII columns: [from §6]
- Customer filters: [from §6]

SQL Templates: [yes — N templates / no]

Customer feedback notes: [Any [NEEDS YOUR INPUT] items the customer left unfilled, or sections they edited substantially vs lightly]

Please confirm before I update business_context.md and re-push.
```

Once confirmed, proceed to Phase 4 in the parent `SKILL.md` (distill into `business_context.md`, optionally add `sql_templates.md`, re-push, re-run tests after the FDE engineer approves).
