---
name: custom-analytics-agent-prod-docs
description: |
  Analytics-specific content for the Phase 6 customer-specific Confluence documentation page set. STATUS: scaffold — TODO to fill in once template repo lands.
---

# Custom Analytics Agent — Phase 6 Page Content

> **STATUS — Scaffold.** Analytics-specific content TBD once template repo defines agent + KB structure.

The 5-page set, create order, and keep-current rules live in `../../shared/customer_docs_pattern.md`. This file specifies the **analytics-specific content** for each page.

## Prerequisites

- All Phase 5 (Round 2 testing) work complete
- Current Project State page contains: solution folder ID, project name, requirements doc URL, test cases page URL
- Local knowledge base files reflect customer's final state — `Read` them before authoring the Behavior page

## Page Content

All Confluence page titles use the convention `<Base name> - <Customer>` (regular hyphen, customer-name suffixed) — Confluence enforces unique titles per space, so suffixing is mandatory.

### 1. Architecture

Title: `Analytics Agent Architecture - <Customer>`

Content:
- Target project: `<Customer> Analytics Agent` (fresh LLM project)
- TODO: list of agents pushed (main + sub-agents)
- TODO: list of knowledge bases
- TODO: chat integration config
- Model: `claude-4.5-sonnet`, temperature: `0`

### 2. Customer-Specific Behavior Summary

Title: `Analytics Agent Behavior - <Customer>`

TODO: section list. Should pull from the analytics agent's knowledge bases and describe what databases/tables it queries, common analytical patterns it knows, exclusions, etc.

### 3. Eval Results

Title: `Analytics Agent Eval Results - <Customer>`

Content:
- Link to `Analytics Agent Test Cases - <Customer>` page
- Round 2 final pass rate
- Known limitations (failing TC-IDs)
- Date of last `tdx agent test` run

### 4. Runbook

Title: `Analytics Agent Runbook - <Customer>`

TODO content:
- Re-deploy procedure (no `rm -rf` — fresh project; just `tdx agent push -y`)
- Re-run eval procedure
- Common failure modes mapped to which file to edit
- Project rebinding guidance

### 5. Access & Ownership

Title: `Analytics Agent Access & Ownership - <Customer>`

Standard content:
- Agent owner (FDE team member)
- Customer stakeholders with access
- Slack channel
- GitHub fork location
- Escalation path
