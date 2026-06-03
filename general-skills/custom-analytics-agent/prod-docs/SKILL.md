---
name: custom-analytics-agent-prod-docs
description: |
  Analytics-specific content for the Phase 5 customer-specific Confluence documentation page set.
---

# Custom Analytics Agent — Phase 5 Page Content

The 5-page set, create order, and keep-current rules live in `../../shared/customer_docs_pattern.md`. This file specifies the **analytics-specific content** for each page.

## Prerequisites

- All Phase 4 (Round 2 testing) work complete
- Current Project State page contains: solution folder ID, project name, requirements doc URL, Confluence test cases page URL, customer-facing Google Sheet URL
- Local KBs reflect the customer's final state — `Read` them before authoring the Behavior page:
  - `knowledge_bases/business_context.md`
  - `knowledge_bases/data_dictionary.md`
  - `knowledge_bases/sql_templates.md` (if non-empty)

## Page Content

All Confluence page titles use the convention `<Base name> - <Customer>` (regular hyphen, customer-name suffixed) — Confluence enforces unique titles per space, so suffixing is mandatory.

### 1. Architecture

Title: `[CUST-FACING] Analytics Agent Architecture - <Customer>`

Content:
- Target project: `<Customer> Analytics Agent` (fresh LLM project, not bound to a TD-Managed parent segment)
- Database: `<database>` (configured in `master_database.yml`)
- **Agents pushed by FDE:**
  - `Analytics Agent` (claude-4.5-sonnet, main analyst)
  - `data_source_finder` (claude-4.5-sonnet, schema-discovery sub-agent)
- **Knowledge bases pushed by FDE (5 total):**
  - `business_context.md` — customer-specific (link to current version)
  - `data_dictionary.md` — customer-specific (link to current version)
  - `sql_templates.md` — customer-specific (link to current version, or note "empty — customer didn't provide templates")
  - `plotly_instructions.md` — generic (TD chart standards)
  - `react_dashboard_instructions.md` — generic (dashboard build standards)
- Model: `claude-4.5-sonnet`, temperature: `0`
- Chat integration: standard FDE flow doesn't include one. If a chat integration was added as a Phase 2 add-on, document it here.

### 2. Customer-Specific Behavior Summary

Title: `Analytics Agent Behavior - <Customer>`

Plain-English narrative pulled from `business_context.md` + `data_dictionary.md` + `sql_templates.md`. Sections:

- **Business Model** — what the customer does, primary channels, customer types (from `business_context.md` Business Model)
- **Business KPIs the Agent Optimizes For** (from `business_context.md` Business KPIs)
- **Business Terms the Agent Recognizes** — list each term with its definition (from `business_context.md` Key Terms). E.g., "VIP = `customer_segment = 'VIP'`"
- **Tables and Datasets the Agent Knows** — table-by-table summary from `data_dictionary.md`. Include the most important columns + any gotchas (varchar dates, empty-string semantics).
- **Segment / Dashboard Naming Conventions** — patterns the agent follows when generating new artifacts (from `business_context.md` Segment Naming Conventions)
- **What the Agent Excludes by Default** — PII columns, customer filters, hidden tables (from `business_context.md` Exclusion Rules)
- **Custom SQL Templates** *(if applicable — from `sql_templates.md`)* — list each template name + when the agent uses it

Customer stakeholders read this to understand what their agent does. Plain English; avoid YAML/JSON dumps.

### 3. Eval Results

Title: `[CUST-FACING] Analytics Agent Eval Results - <Customer>`

Content:
- Link to `Analytics Agent Test Cases - <Customer>` Confluence page (internal)
- Link to the customer-facing Google Sheet
- Round 2 final pass rate (X / N = Z%)
- List of TC-IDs that still fail and why (known limitations)
- Date of last `tdx agent test` run
- Any test cases retired between Round 1 and Round 2

### 4. Runbook

Title: `Analytics Agent Runbook - <Customer>`

Content:

**Re-deploy after edits**
1. Edit the relevant KB(s) locally (`business_context.md`, `data_dictionary.md`, `sql_templates.md`, or main `Analytics Agent/prompt.md`)
2. Confirm `master_database.yml` still points at `<database>`
3. Push:
   ```bash
   tdx agent push -y
   ```
   No `rm -rf` step — fresh LLM project has no read-only platform agents.

**Re-run eval:** `tdx agent test` from project directory (gated — wait for FDE engineer's "run" before executing). `updateConfluencePage` to refresh the Test Cases page with new results.

**Common failures** (mirrors `agent-setup/references/eval.md` Round 2 mapping):
- Agent uses wrong table/column or hallucinates one → `data_dictionary.md`
- Agent misses business term → `business_context.md` Key Terms
- Agent surfaces PII / excluded data → `business_context.md` Exclusion Rules
- Agent ignores or rewrites a SQL template → `sql_templates.md`
- Agent emits Unix epoch literals → `Analytics Agent/prompt.md` (tighten time-function guidance)

**Project rebinding to a different database:** edit `master_database.yml` → `database:` field, re-push, and re-run Phase 1 schema exploration to refresh `data_dictionary.md`. Business context likely needs review too if the new database covers different customer types.

**Adding new tables to scope:** `tdx describe <database>.<new_table>`, then add an entry to `data_dictionary.md`, push, re-test.

### 5. Access & Ownership

Title: `Analytics Agent Access & Ownership - <Customer>`

Content:
- Agent owner (FDE team member)
- Customer stakeholders with access
- Slack channel for support / questions
- GitHub fork location if customer maintains their own copy
- Escalation path for platform-level issues
