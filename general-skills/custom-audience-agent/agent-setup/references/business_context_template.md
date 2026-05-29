# Business Context Template

`business_context.md` is the **agent-facing** knowledge base the audience agent reads on every request. It is NOT a customer-facing document. It contains terse direct instructions and ground-truth facts the agent treats as authoritative.

## Audience and tone

The reader is the LLM running inside the deployed audience agent. Optimize for:
- **Brevity** — every line costs context tokens on every request
- **Imperative voice** — "Use X for Y" / "Treat empty string as not-enrolled" / "Never include `email` in output"
- **Concrete facts** — column names in backticks, enum values listed explicitly, no hedging
- **No customer-collaborative voice** — drop phrases like "please confirm", "we suggest", "let us know". The customer's review already happened in Confluence; this file is the post-review ground truth.

## Three authoring stages

1. **Phase 1 (deep schema exploration → inference bundle):** the LLM runs `tdx ps desc`, distribution queries, sample queries, and `tdx sg list`. The result is an *inference bundle* covering all 9 sections of the customer requirements doc. Reviewed at the Phase 1c gate.
2. **Phase 3 (Confluence requirements doc — customer-facing):** bundle is rendered into a customer-fillable Confluence page with `[INFERRED — please confirm]` and `[NEEDS YOUR INPUT]` markers.
3. **Phase 4 (this file):** customer's reviewed Confluence doc is **distilled** into `business_context.md` — terse agent instructions. NOT a copy of the Confluence content. The customer's collaborative-voice answers become declarative facts.

## Distillation: Confluence → business_context.md

Examples of how Confluence content becomes KB content:

**Confluence §1 Business Model (customer-validated):**
> `[INFERRED — please confirm]` Multi-category retailer selling Clothing (~30% of orders), Home goods (~22%), Food (~21%), Electronics (~19%), and Beauty (~10%). Three sales channels — **Online** (~49% of transactions), **In-Store** (~26%), and **Mobile** (~25%).

**business_context.md → Business Model (distilled):**
```
## Business Model
Multi-category retailer. Categories: Clothing, Home goods, Food, Electronics, Beauty. Sales channels (`preferred_channel` values): Online, In-Store, Mobile. Online is the largest channel (~49% of transactions).
```

**Confluence §3 Key Terms:**
> `[INFERRED — please confirm or override]`
> * **"VIP"** = `customer_segment = 'VIP'` — top-tier customers (currently 100 of 1,000). NOTE: distinct from `loyalty_tier` (Bronze/Silver/Gold).
> * **"At-Risk"** = `customer_segment = 'At-Risk'` (171 customers).
> * **"Churned"** = no transaction in `behavior_transactions` for the last 180 days. *(customer confirmed)*

**business_context.md → Key Terms (distilled):**
```
## Key Terms
- "VIP" → `customer_segment = 'VIP'`. Distinct from `loyalty_tier`.
- "At-Risk" → `customer_segment = 'At-Risk'`.
- "Churned" → no row in `behavior_transactions` for last 180 days. Use `WHERE TD_INTERVAL(timestamp, '-180d')` to identify recent activity.
```

**Confluence §6 Exclusions:**
> **PII columns to exclude** `[INFERRED — please confirm]`:
> * `email` — auto-flagged
> * `first_name` — auto-flagged
> * `last_name` — auto-flagged

**business_context.md → Exclusion Rules (distilled):**
```
## Exclusion Rules
- Never surface raw values of: `email`, `first_name`, `last_name`. May use as filters/counts; output aggregates or hashes.
- Exclude `behavior_email_events.event_type = 'Unsubscribed'` recipients from any email-activation segment.
- Exclude `customers.churn_risk = 'High'` from acquisition campaigns unless the user explicitly requests them.
```

## Distillation rules

- **Strip markers.** `[INFERRED — please confirm]` and `[NEEDS YOUR INPUT]` are customer-facing; remove them.
- **Strip hedge phrases.** "Please confirm", "we suggest", "currently averaging" → just give the number or fact.
- **Strip rationale-for-customer.** Confluence might say "*NOTE: distinct from loyalty_tier (Bronze/Silver/Gold), which is the rewards-program tier*" — distilled becomes "Distinct from `loyalty_tier`."
- **Strip the question text from each section header.** The Confluence doc has both the question prompt and the answer; KB has only the answer.
- **Convert lists with stats into terse rules.** `lifetime_value — total revenue per customer (USD). Range $0.04 – $49,860, mean $16,226.` becomes `\`lifetime_value\` — total revenue (USD), mean ~$16k.` Drop ranges unless they're operationally relevant.
- **Inline imperatives.** Where the inference flagged a gotcha (e.g., "stored as `varchar`, not `timestamp`. **Use** `TD_TIME_PARSE`"), keep the imperative; that's the most agent-relevant content.
- **Drop sections the customer left as `[NEEDS YOUR INPUT]`.** If the customer didn't confirm "New = first 30 days", don't put a half-defined "New" term in the KB. Better to omit than to mislead.

## File format

YAML frontmatter + markdown body. Reproduce the frontmatter exactly:

```
---
name: business_context
---
```

Then sections (verbatim — do NOT wrap this template content in code fences when writing the actual file):

`## Business Model`

1-3 sentences distilled from §1 + KPI list distilled from §2 (KPIs ride alongside as a bullet list to keep the file compact).
- LTV: `customers.lifetime_value`, mean ~$16k.
- VIP retention: track quarter-over-quarter engagement of `customer_segment = 'VIP'`.
- Email engagement: open and click rates on `behavior_email_events` per campaign.

`## Key Terms`

Customer-specific jargon as direct mappings.
- "VIP" → `customer_segment = 'VIP'`. Distinct from `loyalty_tier`.
- "Churned" → no row in `behavior_transactions` for last 180 days.

`## Priority Attributes`

5-10 columns the agent should weight most heavily. Direct facts + gotchas.
- `lifetime_value` (double) — total revenue (USD), mean ~$16k.
- `customer_segment` (varchar) — values: `Regular`, `New`, `At-Risk`, `VIP`.
- `loyalty_tier` (varchar) — values: `Bronze`, `Silver`, `Gold`, or empty string. **Empty string = not enrolled**, not missing data.
- `signup_date`, `join_date` (varchar) — stored as varchar, not timestamp. **Use** `TD_TIME_PARSE` for time arithmetic.

`## Segment Naming Conventions`

Direct rules. If the customer confirmed a convention, state it. If they left it as `[NEEDS YOUR INPUT]`, write: `Use descriptive Title Case names. No prefix/suffix convention.`

`## Exclusion Rules`

Imperative rules. Most operationally important section — agent treats these as hard constraints.
- Never surface raw values of: `email`, `first_name`, `last_name`. Output counts or hashes only.
- Exclude `behavior_email_events.event_type = 'Unsubscribed'` from email-activation segments.
- Exclude `customers.churn_risk = 'High'` from acquisition campaigns unless explicitly requested.

## Phase 3 vs Phase 4

| Phase | Source | Action |
|---|---|---|
| Phase 3 | Phase 1 inference bundle (recovered by re-reading the Confluence requirements doc — it has the same content) | Distill bundle → write `business_context.md`. Push. Round 1 tests run against this. |
| Phase 4 | Customer's edited Confluence requirements doc | Re-distill the *current* state of the doc → overwrite `business_context.md`. Customer's edits supersede Phase 1 inferences. Push. Round 2 tests run against this. |

Both phases follow the same distillation rules. The only difference is the source has more / better customer-validated content in Phase 4.

## Rules of thumb

- Keep each section to a few lines. Loaded on every request.
- Real column names in backticks, real enum values, real numeric ranges where operationally useful.
- Imperative voice for any constraint or gotcha.
- Drop the customer-facing markers (`[INFERRED]`, `[NEEDS YOUR INPUT]`). They never appear in the KB.
- Schema sample data (real customer records) must NOT end up in `business_context.md`. Only column names + inferred descriptions + aggregate stats go in.
