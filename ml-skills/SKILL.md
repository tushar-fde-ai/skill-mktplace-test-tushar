---
name: aps-ml-wf-skills
description: |
  Routes to the correct APS ML workflow skill. Trigger on: RFM, CLV, churn, MTA, attribution, NBA, NBP, customer segmentation, predictive model, ML workflow.
compatibility:
  required_tools:
    - Bash
    - Read
    - Write
    - Edit
  dependencies:
    - td-skills (for SQL queries and workflow exploration)
---

# APS ML Workflows

Route to the correct sub-skill based on keywords. Each workflow has its own folder with a SKILL.md, workflow-setup/, agent-setup/, and prod-docs/.

## Routing

| Keywords | Status | Action |
|----------|--------|--------|
| RFM, recency, frequency, monetary, customer segmentation, segment scoring, value tiers | Ready | Read `rfm/SKILL.md` |
| MTA, multi-touch attribution, Markov, Shapley, channel attribution, journey analytics | Ready | Read `mta-journey-analytics/SKILL.md` (then route to workflow-setup/ or agent-setup/) |
| NBA, next best action, propensity, recommendation engine | Scaffold only | Inform user — not yet implemented |
| NBP, next best product, product recommendation | Scaffold only | Inform user — not yet implemented |
| CLV, lifetime value, CLTV, customer worth | Planned | Inform user — not yet implemented |

If the user's request doesn't clearly match one workflow, ask: "Which solution are you looking for — customer segmentation (RFM), channel attribution (MTA), or something else?"

## Shared Workflow Pattern

All ML workflows follow the same steps:
1. Ask for TD database name
2. Explore tables via Trino SQL (use td-skills)
3. Identify columns (time, user_id, amounts, statuses)
4. Generate `input_params.yml` from the workflow's reference templates
5. Clone the GitHub repo for that workflow
6. Validate config, present for approval, deploy

## Sub-Folder Convention

Each workflow folder contains:
- `SKILL.md` — overview and routing within that workflow
- `workflow-setup/` — SKILL.md + references/ for configuring the TD workflow
- `agent-setup/` or `agent-skills/` — references for the companion LLM agent
- `prod-docs/` — production documentation and references
