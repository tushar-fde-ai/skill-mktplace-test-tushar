---
name: fde-rfm-foundry-skill
description: |
  Deploy the RFM Customer Segmentation AI Foundry agent to a TD instance. Covers cloning the agent template from GitHub, configuring the project name, reviewing the agent structure, and pushing via tdx agent commands. Trigger when users want to set up the RFM companion agent, deploy the RFM Foundry agent, or build an AI agent for RFM analysis.
---

# RFM Customer Segmentation — Foundry Agent Setup

This skill covers the RFM-specific steps for deploying the RFM Analysis Agent to a customer's TD instance. The generic push mechanics (clone, `tdx llm project create`, `tdx agent push -y`, `:react:` patch) are in `../../../shared/push_pattern_new_llm_project.md` — read that first.

## Prerequisites

- The RFM workflow must be deployed and have run successfully (Phase 3 complete)
- `rfm_output_table`, `rfm_stats`, `rfm_combined_user_events` must exist in the `sink_database`
- `tdx` is authenticated against the customer's account

## Agent Architecture

```
RFM Analysis Agent (single agent)
├── knowledge_bases/
│   ├── rfm_tables.yml          — DB config pointing at rfm_output_table, rfm_stats, rfm_combined_user_events
│   ├── business_context.md     — Customer industry, segments, source tables used
│   ├── data_dictionary.md      — Schema reference for RFM output tables
│   ├── output_instructions.md  — Response formatting rules
│   ├── plotly_instructions.md  — TD color palette + chart standards
│   └── sql_guide.md            — Pre-built SQL templates for RFM queries
├── prompts/                    — Pre-configured analysis templates
│   ├── 1. Customer Segmentation Overview.yml
│   ├── 2. High-Value Customer Analysis.yml
│   ├── 3. At-Risk Customer Identification.yml
│   ├── 4. Segment Migration Analysis.yml
│   ├── 5. Engagement Optimization.yml
│   └── Data Validation.yml
└── chat_interfaces/
    └── RFM Segmentation Summary.yml
```

## Files to Edit Before Push

| File | What to update |
|---|---|
| `tdx.json` | `"llm_project": "<confirmed_project_name>"` — default is `RFM Customer Segmentation` |
| `knowledge_bases/rfm_tables.yml` | Update `database:` to the customer's `sink_database` if different from template default |
| `knowledge_bases/business_context.md` | Customer industry, source tables used, monetary metric column, any notable data quirks from Phases 1–3 |

## Repository

```bash
git clone https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod.git
cd ps_ml_analytics_team_solutions_prod/rfm_prod/foundry_agent
```

## Integration Check

After push, run:

```bash
tdx chat --agent "<project_name>/RFM Analysis Agent" "Show me the customer segmentation overview"
```

Verify:
1. The agent appears in the project (`tdx agents`)
2. `rfm_output_table`, `rfm_stats`, and `rfm_combined_user_events` are queryable
3. A segmentation overview response is returned with profile counts per segment

## Key Output Tables Referenced by the Agent

| Table | Purpose |
|---|---|
| `rfm_output_table` | Per-profile R/F/M scores and segment labels |
| `rfm_stats` | Distribution statistics for each metric |
| `rfm_combined_user_events` | Union activity table from all sources |
| `rfm_input_table` | Preprocessed input features |

## Customization After Deployment

- **`business_context.md`** — update with customer-specific business rules, segment definitions, and KPIs
- **`data_dictionary.md`** — update if custom columns were added to the RFM workflow
- **`sql_guide.md`** — add customer-specific query templates
- **`prompts/`** — add or modify analysis templates for customer-specific patterns

Run `tdx agent push` after any local changes to sync updates to TD.
