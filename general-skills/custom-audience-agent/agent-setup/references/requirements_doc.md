# Audience Agent — Requirements Doc Body Template

This file holds the **audience-specific content** for the customer-shareable requirements doc. The generic Confluence-page-creation flow lives in `../../../shared/requirements_doc_pattern.md` and `../../../shared/confluence_folder_setup.md`.

## Page title

`Audience Agent Requirements — <Customer>`

## Body template

Use this body when calling `createConfluencePage` — phrased for the customer to fill in directly:

````markdown
# Audience Agent Requirements — <Customer>

**Purpose:** Help us tailor the Custom Audience Agent to your business. Please fill in each section below. None of the fields are mandatory, but the more you provide, the better the agent will understand your data and respond to your team's questions.

**Target return date:** [agreed date]

---

## 1. Business Model

In 1-3 sentences, describe what you sell, your main channels, and your key customer types.

> _Your answer:_

---

## 2. Business KPIs

What are the 3-5 metrics that matter most to your team? (e.g., LTV, repeat purchase rate, email engagement)

> _Your answer:_

---

## 3. Key Terms

Are there terms or shorthand your team uses that the agent should recognize? Define each one.

Examples:
- "VIP" = Tier 3+ loyalty members
- "churned" = no purchase in 180 days

> _Your answer:_

---

## 4. Priority Attributes

Which 5-10 customer attributes (columns on the `customers` table) matter most for your analysis? Especially flag any with cryptic column names that need explanation.

Example:
- `lifetime_value_usd` — total revenue across all orders
- `signup_channel` — values: web, app, referral, partner

> _Your answer:_

---

## 5. Segment Naming Conventions

Do you have any naming patterns for segments? (e.g., "all customer-facing segments start with `CS_`", region suffixes like `_US`, `_JP`)

> _Your answer:_

---

## 6. Exclusions

What should the agent never include by default?

- **PII columns to exclude:** (e.g., email, phone, full_name)
- **Customer filters to apply:** (e.g., test accounts, opted-out users, specific countries, unmarketable customers)
- **Behavior tables to hide:** (any internal/sensitive ones)

> _Your answer:_

---

## 7. Target Users & Tone

Who will use this agent (marketing, analytics, CS, executives)? Any tone preferences? Default is "neutral, professional marketing analyst."

> _Your answer:_

---

## 8. Output Preferences

What format works best — Plotly charts, tables, summary text, or mixed? Any preferred chart types?

> _Your answer:_

---

## 9. SQL Templates *(Optional)*

If your team has recurring SQL patterns you'd like the agent to know about (e.g., "compute repeat-purchase rate by cohort"), paste them here. Each template should include:
- A short name
- A 1-line description of when to use it
- The SQL itself

The agent will load these as a separate knowledge base (`sql_templates.md`) and reference them when relevant.

> _Your templates (optional):_

---

When you're done, please notify us at [Slack channel / email] and we'll incorporate the answers into the agent.
````

## Reference: Section-to-File Mapping (used in Phase 5)

When the customer returns the doc, each section maps to a specific file. Reference `business_context_template.md` for the exact section structure. **Do not act on this in Phase 3 — Phase 5 owns the actual edits.**

| Customer answer in | Goes to |
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

## Reference: Phase 5 Validation Summary Format

Before re-pushing in Phase 5, present this summary back to the FDE engineer (not the customer):

```
Custom Audience Agent — Requirements Summary
============================================

Customer: [name]
TD-Managed Project: [exact project name, e.g., "TD-Managed: Ecommerce Audience"]
Confluence folder: [URL]
Customer requirements doc: [URL]

Business Model: [1-3 sentences from §1]

Business KPIs:
- [KPI from §2]
- ...

Priority Attributes:
- [column] — [meaning, from §4]
- ...

Key Terms:
- [term] = [definition, from §3]
- ...

Segment Naming Convention: [pattern from §5, or "none"]

Target Users: [from §7]

Exclusions:
- PII columns: [from §6]
- Customer filters: [from §6]

SQL Templates: [yes — N templates / no]

Please confirm before I update business_context.md and re-push.
```

Once confirmed, proceed to Phase 5 in the parent `SKILL.md` (update `business_context.md`, optionally add `sql_templates.md`, re-push, re-run tests).
