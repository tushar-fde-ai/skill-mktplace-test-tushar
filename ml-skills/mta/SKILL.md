---
name: fde-mta
description: |
  MTA (Multi-Touch Attribution) Journey Analytics for Treasure Data. Configures workflows that build unified customer journeys from pageviews, email, sales, and orders, then runs attribution models (Markov, Shapley, linear, time-decay) to measure channel contribution to conversions. Trigger on: MTA, multi-touch attribution, channel attribution, conversion paths, journey analytics, Markov, Shapley, marketing attribution, channel mix, CPA, CPB, requirements gathering for MTA, MTA workflow setup, MTA configuration, MTA runbook, MTA output tables.
---

# MTA Journey Analytics

Multi-Touch Attribution workflow that builds unified customer journeys from multiple touchpoint sources and runs attribution models to measure channel contribution to conversions.

## Pick the right reference for the task

| If the user wants to... | Read |
|---|---|
| Gather requirements before configuring (Confluence folder, customer name, conversion definition, touchpoint inventory) | `workflow-setup/references/requirements_doc.md` |
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

- **`fde-mta-foundry-skill`** — deploy the MTA Journey Analytics AI Foundry agent template to a customer's TD instance
- **`fde-mta-journey-agent`** — query the MTA output tables to answer attribution and journey questions (Sankey, path analysis, model comparison)

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/mta_journey_analysis`
- **Workflow path**: `mta_journey_analysis/td_wf/mta_journey_agent/`
- **Config file**: `mta_journey_agent/config/input_params.yml`
- **Key outputs**: `mta_attribution_results`, `mta_top_conversion_journeys`, `mta_channel_summary` in the sink database
- **Models**: Markov, Shapley, Linear, Time-Decay

## How It Works

1. **Union**: Combine pageviews, email, sales, orders into a single touchpoint table
2. **Sessionize**: Group touchpoints into sessions based on inactivity gap
3. **Build journeys**: Create per-customer journey sequences with conversion flags
4. **Attribute**: Run Markov, Shapley, linear, time-decay models
5. **Output**: Channel attribution scores, top conversion paths, spend efficiency

## Standard Workflow

When a user asks to set up MTA from scratch, follow this sequence:
1. Walk through `workflow-setup/references/requirements_doc.md` to gather inputs and locate the customer's Confluence folder
2. Use `tdx-skills:tdx-basic` to explore the customer's TD database and confirm tables/columns
3. Generate `input_params.yml` using `workflow-setup/references/workflow_setup_guide.md` and the per-table guidance in `table_configuration.md`
4. Present the YAML to the user, get confirmation, then push via `tdx wf push`
5. After deploy, document the configuration on Confluence under the customer's MTA sub-folder
