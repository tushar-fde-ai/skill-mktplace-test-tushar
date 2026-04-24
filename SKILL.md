---
name: fde-skills
description: |
  Master orchestrator guide for deploying end-to-end FDE-team ML solutions and custom Claude skills inside the Treasure Data console. It has understanding of TD architecture and helps route requests to the appropriate project folders and reference files.
compatibility:
  required_tools:
    - Bash
    - Read
    - Write
    - Edit
  dependencies:
    - td-skills (for SQL queries and workflow exploration)
---

# Treasure Data FDE Team Workflows and Custom Agents

Route user requests to the correct sub-skill by reading the appropriate SKILL.md file.

## Routing Rules

**Read `ml-skills/SKILL.md`** when the request involves:
- ML models, predictive analytics, scoring, or segmentation algorithms
- RFM, CLV, churn prediction, propensity models
- Multi-touch attribution (MTA), journey analytics
- Next Best Action (NBA), Next Best Product (NBP)
- Keywords: RFM, segment scoring, predict, model, attribution, Markov, Shapley, machine learning

**Read `general-skills/` sub-folders** when the request involves:
- Ad-hoc analytics or reporting on CDP audience data (→ `general-skills/segment-analytics/`)
- Building custom analytics agents for TD AI Foundry (→ `general-skills/custom-analytics-agent/`)
- Building custom audience/segment agents for TD AI Foundry (→ `general-skills/custom-audience-agent/`)
- Keywords: dashboard, report, analyze audience, build agent, analytics agent

**If unclear**, ask: "Are you looking to deploy an ML workflow (RFM, attribution, predictions) or build a general analytics/agent solution?"
