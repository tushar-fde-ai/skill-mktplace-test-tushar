---
name: shared-customer-docs-pattern
description: |
  Reusable Phase 6 customer-specific documentation page set. Defines the 5 standard Confluence pages (Architecture, Behavior, Eval Results, Runbook, Access & Ownership), their create order, and the "what changes when X" rules. Used by every solution under general-skills/.
---

# Shared: Customer-Specific Documentation Pattern

After Round 2 testing passes (Phase 5), every FDE engagement creates 5 customer-specific Confluence pages under the solution folder. The page set is the same across solutions; the *content* of each page is solution-specific (provided by the calling SKILL's `prod-docs/SKILL.md`).

## When to use

Phase 6, after Round 2 testing passes.

## Prerequisites

- Solution folder parentId (from `confluence_folder_setup.md`, recorded in Current Project State)
- Pages already in the folder from earlier phases:
  - `Current Project State - <Customer>` (internal)
  - `[CUST-FACING] <solution> Requirements - <Customer>`
  - `<solution> Test Cases - <Customer>` (internal — customer sees the Google Sheet mirror instead)

If resuming in a new session, **first action: read `Current Project State`** via `searchConfluenceUsingCql` to recover the project name, folder parentId, and URLs of the requirements doc and test cases page.

Before authoring the Behavior page, `Read` the local knowledge base files (e.g., `business_context.md`, `sql_templates.md` for audience) so the doc reflects the agent's current knowledge.

## The 5 Pages

Each `createConfluencePage` call uses:
```
cloudId: treasure-data.atlassian.net
spaceId: 9797636
parentId: <solution_folder_page_id>
contentFormat: markdown
```

**Page title convention:** every title uses the format `<Base name> - <Customer>` (regular hyphen, single spaces, customer-name suffixed). Suffixing is mandatory — Confluence enforces unique titles per space.

**Customer-facing prefix.** Three pages in the engagement are customer-facing and get a `[CUST-FACING]` prefix:
- `[CUST-FACING] <solution> Requirements - <Customer>` (Phase 1d)
- `[CUST-FACING] <solution> Architecture - <Customer>` (Phase 5)
- `[CUST-FACING] <solution> Eval Results - <Customer>` (Phase 5)

The remaining pages — Test Cases, Behavior, Runbook, Access & Ownership, Current Project State — stay internal-only (no prefix). The prefix signals which pages the customer is meant to read; it doesn't change Confluence permissions, so the parent solution folder should be share-link-ready but customers are directed to the prefixed pages by name. Internal pages remain visible if the customer navigates the folder, but their absence of prefix makes the intent clear.

### 1. Architecture

Title: `[CUST-FACING] <solution> Architecture - <Customer>`

Content (calling SKILL provides specifics):
- Target project name (TD-Managed or fresh)
- Platform agents in the project (read-only) — solution-specific list
- Custom agents pushed by FDE — solution-specific list
- Custom knowledge bases pushed — solution-specific list
- Chat integration config (widget label, welcome message)
- Model + temperature defaults

### 2. Customer-Specific Behavior Summary

Title: `<solution> Behavior - <Customer>`

Plain-English narrative of what the agent knows for THIS customer. Pulled from the agent's knowledge base files. Solution-specific section list — see calling SKILL's `prod-docs/SKILL.md`. Keep it accessible to non-technical stakeholders; avoid YAML/JSON.

### 3. Eval Results

Title: `[CUST-FACING] <solution> Eval Results - <Customer>`

Content:
- Link to the test cases page
- Round 2 final pass rate
- List of TC-IDs that still fail and why (known limitations)
- Date of last `tdx agent test` run
- Any test cases retired between Round 1 and Round 2

### 4. Runbook

Title: `<solution> Runbook - <Customer>`

Content (calling SKILL provides specifics):
- Re-deploy procedure (push pattern — TD-Managed or fresh)
- Re-run eval procedure
- Common failure modes mapped to which file to edit
- Project rebinding instructions (if applicable)

### 5. Access & Ownership

Title: `<solution> Access & Ownership - <Customer>`

Content:
- Agent owner (FDE team member)
- Customer stakeholders with access
- Slack channel for support/questions
- GitHub fork location (if customer maintains their own copy)
- Escalation path for platform-level issues

## Create Order

1. Create all 5 pages under the solution folder
2. On each page, link to the Requirements doc and Test Cases page so navigation is bidirectional
3. Update **Current Project State** via `updateConfluencePage`: mark Phase 6 complete, populate the Artifact URLs section with all 5 new page URLs, set status to "engagement live"
4. Share the folder URL with customer stakeholders

## Keeping Docs Current

| Page | When to regenerate |
|---|---|
| Eval Results | After each `tdx agent test` run that produces a different pass rate |
| Customer-Specific Behavior | Whenever the agent's knowledge bases change materially |
| Architecture | When the agent moves to a different project, or agent/KB list changes |
| Runbook | When the deployment procedure changes |
| Access & Ownership | When ownership or access changes |
