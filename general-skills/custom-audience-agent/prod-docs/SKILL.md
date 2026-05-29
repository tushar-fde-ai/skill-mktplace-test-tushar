---
name: custom-audience-agent-prod-docs
description: |
  Audience-specific content for the Phase 5 customer-specific Confluence documentation page set. The 5-page structure + create order + keep-current rules live in ../../shared/customer_docs_pattern.md; this file specifies what goes in each page for an audience-agent engagement.
---

# Custom Audience Agent — Phase 5 Page Content

The 5-page set, create order, and keep-current rules live in `../../shared/customer_docs_pattern.md`. This file specifies the **audience-specific content** for each page.

## Prerequisites

- All Phase 4 (Round 2 testing) work complete
- Current Project State page contains: solution folder ID, project name, requirements doc URL, test cases page URL
- Local `knowledge_bases/business_context.md` (and optionally `sql_templates.md`) reflects the customer's final state — `Read` these before authoring the Behavior page

## Page Content

All Confluence page titles use the convention `<Base name> - <Customer>` (regular hyphen, customer-name suffixed) — Confluence enforces unique titles per space, so suffixing is mandatory.

### 1. Architecture

Title: `Audience Agent Architecture - <Customer>`

Content:
- Target project: `TD-Managed: <Parent Segment Name>`
- **Platform agents in the project** (read-only, not customized):
  - `TD-Managed: Marketing Copilot`
  - `TD-Managed: Data Source Finder`
  - `TD-Managed: Questions Suggester`
- **Custom agents pushed by FDE:**
  - `Custom Audience Agent` (claude-4.5-sonnet, main orchestrator)
  - `Clone Data Source Finder` (claude-4.5-sonnet mirror)
  - `Clone Questions Suggester` (claude-4.5-sonnet mirror)
- **Custom knowledge bases pushed by FDE:**
  - `business_context.md` — schema-derived draft authored in Phase 3, merged with customer answers in Phase 4; link to current version
  - `sql_templates.md` *(only if customer provided in requirements §9)*
- Chat integration: widget label + welcome message from `chat_parent_segment.yml`
- Model: `claude-4.5-sonnet`, temperature: `0`

### 2. Customer-Specific Behavior Summary

Title: `Audience Agent Behavior - <Customer>`

Plain-English narrative pulled from `business_context.md`. Sections:

- **Business Model** — what the customer sells, channels (from §1)
- **Business KPIs the Agent Optimizes For** (from §2)
- **Business Terms the Agent Recognizes** — list each term + definition (from §3). E.g., "VIP = Tier 3+ loyalty members"
- **Priority Attributes** — table of column → business meaning (from §4)
- **Segment Naming Conventions** — patterns the agent follows when drafting (from §5)
- **What the Agent Excludes by Default** — PII columns, customer filters, hidden behavior tables (from §6)
- **Custom SQL Templates** *(if applicable — from §9)* — list each template name + when used

Customer stakeholders read this to understand what their agent does. Plain English; avoid YAML/JSON.

### 3. Eval Results

Title: `Audience Agent Eval Results - <Customer>`

Content:
- Link to `Audience Agent Test Cases - <Customer>` page
- Round 2 final pass rate (X / N = Z%)
- List of TC-IDs that still fail and why (known limitations)
- Date of last `tdx agent test` run
- Any test cases retired between Round 1 and Round 2

### 4. Runbook

Title: `Audience Agent Runbook - <Customer>`

Content:

**Re-deploy after edits**
1. Edit `business_context.md` (or other custom KB, or `Custom Audience Agent/prompt.md`) locally
2. Confirm the three `TD-Managed: *` agent directories are absent locally — if a `git pull` reintroduced them:
   ```bash
   rm -rf "TD-Managed: Marketing Copilot" "TD-Managed: Data Source Finder" "TD-Managed: Questions Suggester"
   ```
3. Confirm `integrations/chat_parent_segment.yml` still references `name: "Custom Audience Agent"`
4. Push:
   ```bash
   tdx agent push -y
   ```

**Re-run eval:** `tdx agent test` from project directory. `updateConfluencePage` to refresh the Test Cases page.

**Common failures** (mirrors `agent-setup/references/eval.md` Round 2 mapping):
- Agent misses business-specific term → `business_context.md` Key Terms
- Agent skips size check before drafting → `Custom Audience Agent/prompt.md` Segment Draft section
- Agent includes excluded PII → `business_context.md` Exclusion Rules
- Agent rewrites or ignores a customer SQL template → `sql_templates.md`

**Project rebinding:** the audience agent is bound to a specific parent segment via `TD-Managed: *` project name. Rebinding means deploying to a different `TD-Managed: *` project — re-run the full flow with a new `tdx.json` `llm_project` value.

### 5. Access & Ownership

Title: `Audience Agent Access & Ownership - <Customer>`

Content:
- Agent owner (FDE team member)
- Customer stakeholders with access
- Slack channel for support/questions
- GitHub fork location if customer maintains their own copy
- Escalation path for platform-level issues (anything touching the `TD-Managed: *` read-only agents)
