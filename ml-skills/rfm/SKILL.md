---
name: fde-rfm
description: |
  RFM (Recency, Frequency, Monetary) Customer Segmentation for Treasure Data. Configures workflows that union customer behavioral data from multiple sources (pageviews, orders, email, sales), compute per-profile Recency, Frequency, and Monetary scores, and segment customers into actionable groups. Trigger on: RFM, recency frequency monetary, customer segmentation, high-value customers, at-risk customers, VIP segmentation, customer scoring, RFM workflow setup, RFM configuration, RFM runbook, RFM output tables.
---

# RFM Customer Segmentation

RFM workflow that unions customer behavioral data from multiple sources, computes per-profile Recency, Frequency, and Monetary scores using configurable binning, and segments customers into actionable groups for targeted marketing.

## Pick the right reference for the task

| If the user wants to... | Read |
|---|---|
| Gather requirements before configuring (Confluence folder, customer name, data readiness, scoring strategy) | `workflow-setup/references/requirements_doc.md` |
| Configure the workflow / generate `input_params.yml` end-to-end | `workflow-setup/references/workflow_setup_guide.md` |
| Look up YAML parameter reference | `workflow-setup/references/yaml_structure.md` |
| Configure a specific table type (pageviews / email / sales / orders) | `workflow-setup/references/table_configuration.md` |
| See a working example config | `workflow-setup/references/input_params_template.yml` |
| Clone the repo / push to GitHub | `workflow-setup/references/github_instructions.md` |
| Production runbook — architecture, output tables, monitoring, failure modes | `prod-docs/references/runbook.md` |
| Customer-facing documentation template | `prod-docs/references/customer_docs.md` |
| Technical handoff document for ops/CSMs | `prod-docs/references/technical_handoff.md` |

## Companion agent skills (separate entry points)

These are registered as their own skills — they trigger directly from user prompts, no need to route through this file:

- **`fde-rfm-foundry-skill`** — deploy the RFM Analysis AI Foundry agent template to a customer's TD instance
- **`fde-rfm-analysis-agent`** — query the RFM output tables to answer segmentation, customer value, and engagement questions with visualizations

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod/tree/main/rfm_prod`
- **Workflow path**: `ps_ml_analytics_team_solutions_prod/rfm_prod/`
- **Workflow entry point**: `rfm_launch.dig`
- **Config file**: `rfm_prod/config/input_params.yml`
- **Key outputs**: `rfm_output_table` (per-profile R/F/M quartiles + segment), `rfm_stats` (per-segment distribution stats), `rfm_combined_user_events` (union activity), plus `rfm_stats_histogram`, `rfm_stats_model_params` (dashboard tables) in the sink database
- **Scoring**: Quartile-based (1-4 scale) using 25th/50th/75th percentile boundaries

## How It Works

1. **Union**: Combine pageviews, email events, sales interactions, orders into a single `rfm_combined_user_events` table (per-profile aggregation per source)
2. **Compute metrics**: For each profile — min recency_days across sources (Recency), sum of total_touchpoints (Frequency), sum of total_spend (Monetary)
3. **Score**: Assign quartile scores (1-4) using 25th/50th/75th percentile boundaries, combine into `rfm_quartile` label (e.g., `R4F3M2`)
4. **Segment**: Assign customer segments based on R/F/M quartile combinations (Champions, Loyal Customers, Cannot lose them, Lost customers, etc.)
5. **Dashboard stats**: Generate per-segment statistics, histograms, and model parameters for the TI dashboard

## Standard Workflow

When a user asks to set up RFM from scratch, follow this sequence:
1. Walk through `workflow-setup/references/requirements_doc.md` to gather inputs and locate the customer's Confluence folder
2. Use `tdx-skills:tdx-basic` to explore the customer's TD database and confirm tables/columns
3. Generate `input_params.yml` using `workflow-setup/references/workflow_setup_guide.md` and the per-table guidance in `table_configuration.md`
4. Present the YAML to the user, get confirmation, then push via `tdx wf push -y`
5. After deploy, document the configuration on Confluence under the customer's RFM sub-folder
