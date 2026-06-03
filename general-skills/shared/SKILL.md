---
name: shared-fde-solution-patterns
description: |
  Index of reusable patterns shared across all general-skills FDE solutions (custom-audience-agent, custom-analytics-agent, etc.). Loaded by solution SKILLs to reuse Confluence folder setup, Current Project State page, push patterns, requirements doc workflow, test case lifecycle, and Phase 6 documentation page set. Not invoked directly — read individual files within this folder as called for by the solution SKILL.
---

# Shared Patterns for FDE General Skills

Reference library of generic patterns used by every solution under `general-skills/`. Solution SKILLs (custom-audience-agent, custom-analytics-agent, etc.) read individual files here to avoid duplicating the generic flow.

## Files

| File | What it contains |
|------|-----------------|
| `confluence_folder_setup.md` | Discovery + creation of the customer's Confluence folder hierarchy (CUST → Region → Customer → FDE Solutions → solution folder) |
| `current_project_state.md` | The cross-session context store concept — `Current Project State - <Customer>` Confluence page format and update rules |
| `push_pattern_td_managed.md` | Push pattern for solutions that customize an existing `TD-Managed: <Parent Segment>` project (e.g., custom-audience-agent) |
| `push_pattern_fresh_project.md` | Push pattern for solutions that create a new LLM project from scratch (e.g., custom-analytics-agent) |
| `customer_branch_pattern.md` | Per-customer `customer/<slug>` branch convention for template repos — slug derivation, branch lifecycle (Phase 2 create → Phase 3 distill → Phase 4 re-distill → iteration loop), commit message conventions, what to commit / .gitignore |
| `requirements_doc_pattern.md` | Customer-fillable Confluence page workflow + share + pause-for-customer-response pattern |
| `test_cases_pattern.md` | Two-round test case lifecycle (TC-IDs, Confluence test cases page, customer-facing Google Sheet, `tdx agent test`, iteration loop) |
| `audience_studio_segments.md` | Generic process for pushing segments to Audience Studio — test-segment dumps from `tdx agent test` runs (`FDE Test - <Customer>` folder) and customer-facing demo segments (`FDE Solutions - <Solution> Examples`). Approval gates, naming, JSON→YAML conversion, overwrite-by-name behavior. |
| `customer_docs_pattern.md` | Phase 5 — the 5 standard customer-specific Confluence pages (Architecture, Behavior, Eval Results, Runbook, Access & Ownership) |

## How solutions use this

A solution SKILL.md for a new engagement type will reference these files in its phases. For example:

```markdown
### Phase 3: Customer Requirements Doc

Read `../shared/confluence_folder_setup.md` for folder discovery + creation.
Read `../shared/current_project_state.md` for the State page setup.
Read `../shared/requirements_doc_pattern.md` for the customer-shareable page workflow.

**Solution-specific overrides:**
- Page title: `<solution> Requirements - <Customer>`
- Body template: see `agent-setup/references/requirements_doc.md`
```

Solution SKILLs handle the *what* (which questions, which knowledge bases, which test categories). The shared files handle the *how* (Confluence flow, push mechanics, two-round test rhythm).
