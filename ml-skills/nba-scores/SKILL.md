---
name: fde-nba-scores
description: |
  NBA (Next Best Action) Engagement Scores for Treasure Data. Configures workflows that union customer activity, derive per-profile Next Best Channel / Next Best Time / Next Best Campaign affinity scores plus cart-abandon and new-visitor flags, and write the combined output into Audience Studio. Trigger on: NBA, Next Best Action, engagement scores, channel affinity, time of day affinity, cart abandon, new visitor, NBA dashboard, requirements gathering for NBA, NBA workflow setup, NBA configuration, NBA runbook.
---

# NBA Engagement Scores

Next Best Action (NBA) workflow that unions customer behavioral data from multiple sources, scores per-profile engagement across **Channel** (where), **Time of Day** (when), and **Campaign** (what), and combines the metrics into a single profile-level table that can be joined to a Parent Segment in Audience Studio.

## Pick the right reference for the task

| If the user wants to... | Read |
|---|---|
| Gather requirements before configuring (Confluence folder, parent segment, scoring strategy, business windows) | `workflow-setup/references/requirements_doc.md` |
| Configure the workflow / generate `input_params.yml` end-to-end | `workflow-setup/references/workflow_setup_guide.md` |
| Look up YAML parameter reference | `workflow-setup/references/yaml_structure.md` |
| Configure a specific source type (pageviews / email / sales / orders / custom) | `workflow-setup/references/table_configuration.md` |
| See a working example config | `workflow-setup/references/input_params_template.yml` |
| Clone the repo / push to GitHub / set secrets | `workflow-setup/references/github_instructions.md` |
| Production runbook — architecture, output tables, monitoring, failure modes | `prod-docs/references/runbook.md` |
| Customer-facing documentation template | `prod-docs/references/customer_docs.md` |
| Technical handoff document for ops/CSMs | `prod-docs/references/technical_handoff.md` |

## Companion agent skills (separate entry points)

These are registered as their own skills — they trigger directly from user prompts, no need to route through this file:

- **`fde-nba-scores-foundry-skill`** — deploy the NBA Insights AI Foundry agent template to a customer's TD instance
- **`fde-nba-scores-insights-agent`** — query the NBA dashboard tables (`nba_dash_*`) directly in Treasure Work to summarize runs, compare two runs, explain score distributions

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/nba_eng_scores`
- **Workflow project name**: `nba_eng_prod`
- **Workflow path**: `nba_eng_scores/td_wf/`
- **Workflow entry point**: `nba_eng_launch.dig`
- **Config file**: `nba_eng_scores/td_wf/config/input_params.yml`
- **Foundry agent project**: `NBA Engagement Scores` (single-agent: `NBA Insights Agent`)
- **Key output**: `nba_combined_metrics_final` — per-profile NBA scores joinable to Parent Segment
- **Dashboard tables**: `nba_dash_stats_summary`, `nba_dash_model_metrics`, `nba_dash_source_tables`

## How It Works

1. **Union activity**: Combine pageviews, email events, sales-rep interactions, orders into a single `nba_combined_user_events` table (sessionized by `session_length`).
2. **Next Best Channel**: Parse UTM params + apply channel regex rules → per-profile affinity score per channel (e.g. social, email, search). Scored via `percentile`, `quartile`, or `minmax` (Hivemall) — chosen by config.
3. **Next Best Time**: Bucket activity into morning / afternoon / evening / overnight → per-profile affinity score per daypart, same scoring strategy.
4. **Next Best Campaign**: Apply business-rule flags — **cart-abandon** (added to cart in lookback window without conversion) and **new-visitor** (recent first visit, low page count, not paid-referred, no purchase).
5. **Combine**: Join all metric tables on `${unique_user_id}` into `nba_combined_metrics_final`.
6. **Dashboard stats**: Append config + per-source stats + score distributions to `nba_dash_*` tables for the TI dashboard and the NBA Insights agent.

## Scoring Strategies

Set via `scoring_logic` in `input_params.yml`:

| Strategy | When to use |
|----------|-------------|
| `percentile` | Default for evenly distributed activity. Score = percentile rank within audience. |
| `quartile` | Coarse 4-bucket banding. Easier to communicate to marketers. |
| `minmax` | Hivemall min-max scaling. Best when activity distribution is heavily skewed. |

## Standard Workflow

When a user asks to set up NBA from scratch, follow this sequence:
1. Walk through `workflow-setup/references/requirements_doc.md` to gather inputs (parent segment, scoring strategy, cart-abandon / new-visitor windows, time-of-day granularity, ESP) and locate the customer's Confluence folder
2. Use `tdx-skills:tdx-basic` to explore the customer's TD database and confirm tables/columns
3. Generate `input_params.yml` using `workflow-setup/references/workflow_setup_guide.md` and the per-source guidance in `table_configuration.md`
4. Present the YAML to the user, get confirmation, then push via `tdx wf push -y`
5. After deploy, document the configuration on Confluence under the customer's NBA sub-folder
