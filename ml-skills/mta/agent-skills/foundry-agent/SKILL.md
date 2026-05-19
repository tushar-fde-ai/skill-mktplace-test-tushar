---
name: fde-mta-foundry-skill
description: |
  Deploy the MTA Journey Analytics AI Foundry agent to a TD instance. Covers cloning the agent template from GitHub, configuring the project name, reviewing the agent structure, and pushing via tdx agent commands. Trigger when users want to set up the MTA companion agent, deploy the MTA Foundry agent.
---

# MTA Journey Analytics — Agent Setup

This skill guides you through deploying the MTA Journey Analytics AI Foundry agent template to a customer's TD instance. The agent is a three-agent orchestration system that queries MTA output tables and generates Plotly visualizations.

## Agent Architecture

```
Master Orchestrator Agent
├── Journey Agent (sub-agent) — path analysis, Sankey flows, touchpoint sequences
├── MTA Models Agent (sub-agent) — attribution model comparison, Markov/Shapley analysis
├── knowledge_bases/ — shared reference material (SQL guide, data dictionary, chart instructions)
├── prompts/ — pre-configured analysis templates
└── chat_interfaces/ — pre-built chat entry points
```

## Prerequisites

- The MTA workflow must be deployed and run successfully first (see `workflow-setup/SKILL.md`)
- MTA output tables must exist in the `sink_database` (e.g., `mta_journeys`, `mta_models_standard`, etc.)

## Setup Workflow

### Step 1: Clone the Repository

```bash
git clone https://github.com/treasure-data-ps/mta_journey_analysis.git
cd mta_journey_analysis/foundry_agent
```

### Step 2: Review the Agent Structure

Present the project structure to the user:

```
foundry_agent/
├── tdx.json                              # Project name config
├── Master Orchestrator Agent/
│   ├── agent.yml                         # Coordinator with Plotly output + sub-agent references
│   └── prompt.md                         # Routing logic, visualization, response formatting
├── Journey Agent/
│   ├── agent.yml                         # Journey table access (knowledge_base tools)
│   └── prompt.md                         # Journey queries, Sankey flows, drop-off analysis
├── MTA Models Agent/
│   ├── agent.yml                         # MTA model table access (knowledge_base tools)
│   └── prompt.md                         # Attribution queries (first/last touch, linear, Markov, Shapley)
├── knowledge_bases/
│   ├── business_context.md               # Customer business model, segments, channels
│   ├── data_dictionary.md                # Schema reference for all output tables
│   ├── output_instructions.md            # Response formatting rules
│   ├── plotly_instructions.md            # TD color palette, chart standards
│   ├── sql_guide.md                      # Pre-built SQL templates
│   ├── journey_tables.yml                # DB config for journey tables
│   └── mta_model_tables.yml              # DB config for attribution tables
├── prompts/                              # Pre-configured analysis templates
│   ├── 1. Attribution Model Comparison.yml
│   ├── 2. Customer Journey Flow_.yml
│   ├── 3. HLTV Customer Journey Analysis.yml
│   ├── 4. Conversion Optimization Opportunities.yml
│   ├── 5. Key Insights Summary.yml
│   ├── 6. Budget Recommendations.yml
│   ├── 7. Channel Spend and CPA _ CPB Analysis.yml
│   └── Data Validation.yml
└── chat_interfaces/
    └── MTA Model Summary.yml
```

Show the user the list of agents, knowledge bases, and prompts. Ask:

> Here is the MTA Foundry agent project structure with 3 agents, 7 knowledge bases, 8 prompt templates, and 1 chat interface. Does this look correct to deploy?

**Wait for explicit confirmation before proceeding.**

### Step 3: Configure Knowledge Bases for Customer Data

The knowledge base YAML files reference the output database. Verify they point to the correct `sink_database` from the MTA workflow config:

- `knowledge_bases/journey_tables.yml` — should reference the database containing `mta_journeys`, `mta_sankey_journeys`, `journey_src_union_summary`
- `knowledge_bases/mta_model_tables.yml` — should reference the database containing `mta_models_standard`, `mta_markov_attribution`, `mta_shapley_attribution_final`

If the customer's `sink_database` differs from the template default, update these files.

Also review `knowledge_bases/business_context.md` — if it contains template-specific business context (e.g., references to a different customer), update it to reflect the current customer's business.

### Step 4: Choose Project Name and Update tdx.json

Ask the user:

> Would you like to push this agent with the default project name `MTA Journey Analysis`, or would you like to provide a custom project name?

Once confirmed, update `tdx.json` with the chosen name:

```json
{
  "llm_project": "<confirmed_project_name>"
}
```

### Step 5: Create Project and Push

First, create the project in TD (required if it doesn't already exist):

```bash
tdx llm project create "<confirmed_project_name>"
```

Then push from the foundry_agent directory:

```bash
cd /path/to/mta_journey_analysis/foundry_agent
tdx agent push -y
```

If the project already exists, skip the create step — `tdx agent push` will push updates to the existing project.

This pushes all agents, knowledge bases, prompts, and chat interfaces to the TD instance.

### Step 6: Verify Deployment

After push succeeds:

```bash
tdx agents                                    # List agents in the project
tdx chat --agent "<project_name>/Master Orchestrator Agent" "What attribution models are available?"
```

Verify:
1. All 3 agents appear in the project
2. Knowledge bases are accessible
3. Master Orchestrator can route to sub-agents
4. Queries return data from the MTA output tables

## Key Output Tables Referenced by the Agent

| Table | Used By | Purpose |
|-------|---------|---------|
| `mta_journeys` | Journey Agent | Event-level journey data |
| `mta_sankey_journeys` | Journey Agent | Pre-aggregated Sankey flows |
| `journey_src_union_summary` | Journey Agent | Source metadata and channel distributions |
| `mta_models_standard` | MTA Models Agent | Rule-based attribution (first/last/linear/u-shaped) |
| `mta_markov_attribution` | MTA Models Agent | Markov chain attribution with removal effects |
| `mta_shapley_attribution_final` | MTA Models Agent | Shapley value attribution with daily time series |

## Customization

After initial deployment, the agent can be customized:

- **Knowledge bases**: Update `business_context.md` with customer-specific business rules, segments, and KPIs
- **Data dictionary**: Update `data_dictionary.md` if custom columns were added to the MTA workflow
- **Prompts**: Add or modify prompt templates for customer-specific analysis patterns
- **SQL guide**: Add customer-specific SQL templates to `sql_guide.md`

Use `tdx agent push` after any local changes to sync updates to TD.

## Related Skills

- **workflow-setup** — MTA workflow configuration and deployment (must run first)
- **prod-docs** — Production documentation and operational runbook
- **tdx-skills/agent** — General `tdx agent` CLI reference
- **tdx-skills/agent-prompt** — System prompt writing best practices
