---
name: fde-general-skills
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

Route to the correct sub-skill based on the user's request. These are non-ML solutions — custom AI agents and analytics tooling deployed for customers.

## Routing

| Keywords | Status | Action |
|----------|--------|--------|
| Audience agent, custom audience agent, CDP audience agent, segment agent, parent segment agent | In Progress | Read `custom-audience-agent/SKILL.md` |
| Analytics agent, custom analytics agent, data analytics agent, reporting agent, dashboard agent | Scaffold only | Inform user — `custom-analytics-agent/` not yet implemented |
| Segment analytics, segment reporting, CDP reporting, audience reporting, ad-hoc segment analysis | Scaffold only | Inform user — `segment-analytics/` not yet implemented |

If the user's request doesn't clearly match one solution, ask: "Are you looking to build a custom AI agent for audience/segment analysis, a general analytics agent, or set up segment reporting?"

## Available Solutions

### Custom Audience Agent (In Progress)
Build and deploy a custom AI Foundry agent that analyzes CDP parent segment data — queries customer attributes, behaviors, and segment membership to answer business questions via natural language.

**Sub-folder structure:**
- `agent-setup/` — Requirements gathering, agent configuration, deployment to TD AI Foundry
- `prod-docs/` — Production documentation, customer-facing docs, evaluation framework

### Custom Analytics Agent (Scaffold Only)
Build and deploy a general-purpose analytics agent that queries any TD database and generates dashboards/reports. Not yet implemented.

### Segment Analytics (Scaffold Only)
Ad-hoc analytics and reporting tooling for CDP segments — pre-built queries, dashboard templates, and reporting workflows. Not yet implemented.

## Sub-Folder Convention

Each solution folder follows the same pattern:
- `SKILL.md` — overview and routing within that solution
- `agent-setup/` — SKILL.md + references/ for configuring and deploying the agent
- `prod-docs/` — production documentation and references
