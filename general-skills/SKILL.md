---
name: aps-general-skills
description: |
  Routes to the correct FDE general solution skill. These are non-ML solutions — custom AI agents for audience analysis, ad-hoc analytics, and segment reporting. Trigger on: audience agent, custom agent, analytics agent, segment analytics, CDP reporting, build agent, ad-hoc analysis.
compatibility:
  required_tools:
    - Bash
    - Read
    - Write
    - Edit
  dependencies:
    - td-skills (for SQL queries, agent push, and workflow exploration)
---

# FDE General Skills

Route to the correct sub-skill based on the user's request. These are non-ML solutions — custom AI agents and analytics tooling deployed for customers. Each solution shares a common 6-phase rhythm; reusable patterns live in `shared/`.

## Routing

| Keywords | Status | Action |
|----------|--------|--------|
| Audience agent, custom audience agent, CDP audience agent, segment agent, parent segment agent | Ready | Read `custom-audience-agent/SKILL.md` |
| Analytics agent, custom analytics agent, data analytics agent, reporting agent, dashboard agent, BI agent | Scaffold (template pending) | Read `custom-analytics-agent/SKILL.md` (TODO sections need filling once template lands) |
| Segment analytics, segment reporting, CDP reporting, audience reporting, ad-hoc segment analysis | Scaffold only | Inform user — `segment-analytics/` not yet implemented |

If the user's request doesn't clearly match one solution, ask: "Are you looking to build a custom AI audience agent on top of an existing parent segment, a general analytics agent in a fresh project, or set up segment reporting?"

## Available Solutions

### Custom Audience Agent (Ready)
Customizes an existing `TD-Managed: <Parent Segment>` AI project with a parallel set of editable agents + knowledge bases. Pushes onto an already-provisioned project.

### Custom Analytics Agent (Scaffold)
Creates a fresh LLM project for general analytics + dashboard generation against any TD database. Template repo URL pending; `custom-analytics-agent/` is structurally complete but has TODO sections that need filling once the template is available.

### Segment Analytics (Scaffold Only)
Ad-hoc analytics and reporting tooling for CDP segments — pre-built queries, dashboard templates, reporting workflows. Not yet implemented.

## Shared Patterns

All solutions share a 6-phase rhythm: discover → push minimal → customer requirements doc → Round 1 tests → re-push with customer answers + Round 2 → customer-specific docs.

The generic pieces (Confluence folder hierarchy, Current Project State page, push patterns, requirements-doc workflow, two-round test lifecycle, Phase 6 page set) live in `shared/`. Each solution SKILL is the *what* (agent-specific overrides); `shared/` is the *how*.

## Sub-Folder Convention

Each solution folder follows the same pattern:
- `SKILL.md` — entry point + 6-phase routing (references `../shared/` files)
- `agent-setup/` — solution-specific Phase 2 + Phase 5 file edits and references
- `prod-docs/` — solution-specific content for the Phase 6 documentation page set

## Shared Reference Files (`shared/`)

- `confluence_folder_setup.md` — customer Confluence folder hierarchy discovery + creation
- `current_project_state.md` — cross-session context store concept + page format
- `push_pattern_td_managed.md` — push pattern for solutions customizing an existing TD-Managed project (audience)
- `push_pattern_fresh_project.md` — push pattern for solutions creating a fresh LLM project (analytics)
- `requirements_doc_pattern.md` — customer-fillable Confluence page workflow
- `test_cases_pattern.md` — TC-IDs, Confluence test cases page, two-round flow
- `customer_docs_pattern.md` — Phase 6 — the 5 customer-specific pages (Architecture, Behavior, Eval Results, Runbook, Access & Ownership)
