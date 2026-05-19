---
name: fde-nba-scores-foundry-skill
description: |
  Deploy the NBA Insights AI Foundry agent to a TD instance. Covers cloning the agent template from GitHub, configuring the project name, reviewing the agent structure, updating knowledge-base table references for the customer's sink database, and pushing via tdx agent commands. Trigger when users want to set up the NBA companion agent or deploy the NBA Foundry agent.
---

# NBA Engagement Scores — Agent Setup

This skill walks you through deploying the NBA Insights AI Foundry agent template to a customer's TD instance. The agent is a **single-agent** project — one orchestrator (`NBA Insights Agent`) backed by five knowledge bases. It reads three dashboard tables and explains the latest NBA run.

## Agent Architecture

```
NBA Insights Agent (single agent)
├── knowledge_bases/
│   ├── NBA Dashboard Tables    — DB config for the 3 dashboard tables
│   ├── business_context        — How the NBA workflow defines its scores/flags
│   ├── data_dictionary         — Schema reference for the 3 dashboard tables
│   ├── plotly_instructions     — TD color palette + Plotly chart standards
│   ├── react_instructions      — React rendering rules (Recharts + Tailwind)
└── outputs/
    ├── :plotly: (newPlot)      — Plotly.js chart rendering
    └── :react:  (renderReactApp) — React/Recharts dashboards (only when asked)
```

This is a meaningful structural difference from the MTA Foundry agent (which has a Master + Journey + Models orchestration). The NBA repo ships a single agent — do NOT try to split it into sub-agents.

## Prerequisites

- The NBA workflow must be deployed and run successfully first (see `workflow-setup/SKILL.md`)
- `nba_dash_stats_summary`, `nba_dash_model_metrics`, `nba_dash_source_tables` must exist in the `sink_database` and have at least one populated `session_id`
- The user has `tdx` configured and authenticated against the customer's account (`tdx use <profile>`)

## Setup Workflow

### Step 1: Clone the Repository

```bash
git clone https://github.com/treasure-data-ps/nba_eng_scores.git
cd nba_eng_scores/foundry_agent
```

### Step 2: Review the Agent Structure

Show the user the project layout:

```
foundry_agent/
├── tdx.json                                  # Project name config
├── NBA Insights Agent/
│   ├── agent.yml                             # Agent definition (model, tools, outputs)
│   ├── prompt.md                             # System prompt with routing logic + answer patterns
│   └── starter_message.md                    # Greeting shown when user opens chat
└── knowledge_bases/
    ├── NBA Dashboard Tables.yml              # DB config — points at the 3 dashboard tables
    ├── business_context.md                   # How NBA scores are derived (the "background" KB)
    ├── data_dictionary.md                    # Schema for the 3 tables
    ├── plotly_instructions.md                # Chart standards (TD palette + layout rules)
    ├── react_instructions.md                 # React render rules (only when explicitly requested)
```

Then ask:

> Here is the NBA Insights Foundry agent project structure — 1 agent, 5 knowledge bases, 1 starter message. Does this look correct to deploy?

**Wait for explicit confirmation before proceeding.**

### Step 3: Configure Knowledge Bases for Customer Data

The agent reads from three dashboard tables. By default `knowledge_bases/NBA Dashboard Tables.yml` points at `database: td_agents`. If the customer's `sink_database` is different, update the YAML before pushing.

Open `knowledge_bases/NBA Dashboard Tables.yml` and verify:

```yaml
name: NBA Dashboard Tables
type: database
database: <customer_sink_database>            # ← update if not td_agents
tables:
  - name: nba_dash_stats_summary
    td_query: SELECT session_id, metric_name, metric_value, profile_count, converted_users, time FROM nba_dash_stats_summary
  - name: nba_dash_model_metrics
    td_query: SELECT * FROM nba_dash_model_metrics
  - name: nba_dash_source_tables
    td_query: SELECT session_id, source_table, unique_profiles, num_events, total_conversions, total_spend, day_range, min_date, max_date, time FROM nba_dash_source_tables
```

Also review `knowledge_bases/business_context.md`:
- If it contains template-specific business context (e.g., references to a different customer or industry), update it to reflect the current customer's business model and channel taxonomy.
- Generic deployments can ship as-is — `business_context.md` describes what the NBA workflow logic does, which is customer-agnostic.

### Step 4: Choose Project Name and Update `tdx.json`

Ask the user:

> Would you like to push this agent with the default project name `NBA Engagement Scores`, or would you like to provide a custom project name?

Once confirmed, update `foundry_agent/tdx.json`:

```json
{
  "llm_project": "<confirmed_project_name>"
}
```

### Step 5: Create Project and Push

Create the LLM project in TD if it doesn't already exist:

```bash
tdx llm project create "<confirmed_project_name>"
```

Then push from the `foundry_agent/` directory:

```bash
cd /path/to/nba_eng_scores/foundry_agent
tdx agent push -y
```

If the project already exists, skip the create step — `tdx agent push` will push updates to the existing project.

This pushes the `NBA Insights Agent` definition, all 5 knowledge bases, and the starter message to the TD instance.

### Step 6: Verify Deployment

After push succeeds:

```bash
tdx agents                                                          # list agents in the project
tdx chat --agent "<project_name>/NBA Insights Agent" "Summarize the latest NBA run"
```

Verify:
1. The `NBA Insights Agent` appears in the project
2. All 5 knowledge bases are accessible
3. The agent can read `nba_dash_*` tables and return a summary
4. Plotly charts render when the agent uses the `:plotly:` output

If the chat returns "no data" or "table not found", recheck `knowledge_bases/NBA Dashboard Tables.yml` — the `database:` field is the most common source of error.

## Key Tables Referenced by the Agent

All three live in the customer's `sink_database`:

| Table | Purpose |
|-------|---------|
| `nba_dash_stats_summary` | One row per (session_id, metric_name, metric_value). Score/flag distributions per run. |
| `nba_dash_model_metrics` | One row per run. Config snapshot — scoring strategy, lookback windows, source-table parsing rules. |
| `nba_dash_source_tables` | One row per (session_id, source_table). Per-source data volumes. |

The agent always defaults to the **latest `session_id`** unless the user asks for historical comparison.

## Customization

After initial deployment, the agent can be customized:

- **`knowledge_bases/business_context.md`** — Update with customer-specific business model details, channel naming conventions, KPIs, or how scores feed into specific marketing campaigns.
- **`knowledge_bases/data_dictionary.md`** — Update if the customer's NBA workflow has custom score columns beyond the standard set.
- **`NBA Insights Agent/prompt.md`** — Add customer-specific answer templates or business rules.
- **`NBA Insights Agent/starter_message.md`** — Personalize the chat greeting.

Use `tdx agent push` after any local changes to sync updates to TD.

## Related Skills

- **workflow-setup** — NBA workflow configuration and deployment (must run first, populates the dashboard tables)
- **prod-docs** — Production documentation and operational runbook
- **nba-insights** — Local skill version of the same agent, for direct use inside Treasure Work without a Foundry deployment
- **tdx-skills:agent** — General `tdx agent` CLI reference
- **tdx-skills:agent-prompt** — System prompt writing best practices
