---
name: fde-rfm-foundry-skill
description: |
  Deploy the RFM Customer Segmentation AI Foundry agent to a TD instance. Covers cloning the agent template from GitHub, configuring the project name, reviewing the agent structure, and pushing via tdx agent commands. Trigger when users want to set up the RFM companion agent, deploy the RFM Foundry agent, or build an AI agent for RFM analysis.
---

# RFM Customer Segmentation — Agent Setup

This skill guides you through deploying the RFM Customer Segmentation AI Foundry agent template to a customer's TD instance. The agent queries RFM output tables and generates visualizations for customer segmentation analysis.

## Agent Architecture

```
RFM Analysis Agent
├── knowledge_bases/ — shared reference material (SQL guide, data dictionary, chart instructions)
├── prompts/ — pre-configured analysis templates
└── chat_interfaces/ — pre-built chat entry points
```

## Prerequisites

- The RFM workflow must be deployed and run successfully first (see `../../workflow-setup/SKILL.md`)
- RFM output tables must exist in the `sink_database` (e.g., `rfm_output_table`, `rfm_stats`, `rfm_combined_user_events`)

## Setup Workflow

### Step 1: Clone the Repository

```bash
git clone https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod.git
cd ps_ml_analytics_team_solutions_prod/rfm_prod/foundry_agent
```

### Step 2: Review the Agent Structure

Present the project structure to the user:

```
foundry_agent/
├── tdx.json                              # Project name config
├── RFM Analysis Agent/
│   ├── agent.yml                         # Agent config with table access
│   └── prompt.md                         # Analysis logic, scoring rules, visualization
├── knowledge_bases/
│   ├── business_context.md               # Customer business model, segments, channels
│   ├── data_dictionary.md                # Schema reference for RFM output tables
│   ├── output_instructions.md            # Response formatting rules
│   ├── plotly_instructions.md            # TD color palette, chart standards
│   ├── sql_guide.md                      # Pre-built SQL templates for RFM queries
│   └── rfm_tables.yml                    # DB config for RFM output tables
├── prompts/                              # Pre-configured analysis templates
│   ├── 1. Customer Segmentation Overview.yml
│   ├── 2. High-Value Customer Analysis.yml
│   ├── 3. At-Risk Customer Identification.yml
│   ├── 4. Segment Migration Analysis.yml
│   ├── 5. Engagement Optimization.yml
│   └── Data Validation.yml
└── chat_interfaces/
    └── RFM Segmentation Summary.yml
```

Show the user the list of agents, knowledge bases, and prompts. Ask:

> Here is the RFM Foundry agent project structure with 1 agent, 6 knowledge bases, 6 prompt templates, and 1 chat interface. Does this look correct to deploy?

**Wait for explicit confirmation before proceeding.**

### Step 3: Configure Knowledge Bases for Customer Data

The knowledge base YAML files reference the output database. Verify they point to the correct `sink_database` from the RFM workflow config:

- `knowledge_bases/rfm_tables.yml` — should reference the database containing `rfm_output_table`, `rfm_stats`, `rfm_combined_user_events`

If the customer's `sink_database` differs from the template default, update this file.

Also review `knowledge_bases/business_context.md` — if it contains template-specific business context (e.g., references to a different customer), update it to reflect the current customer's business.

### Step 4: Choose Project Name and Update tdx.json

Ask the user:

> Would you like to push this agent with the default project name `RFM Customer Segmentation`, or would you like to provide a custom project name?

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
cd /path/to/rfm_prod/foundry_agent
tdx agent push -y
```

If the project already exists, skip the create step — `tdx agent push` will push updates to the existing project.

This pushes the agent, knowledge bases, prompts, and chat interfaces to the TD instance.

### Step 6: Verify Deployment

After push succeeds:

```bash
tdx agents                                    # List agents in the project
tdx chat --agent "<project_name>/RFM Analysis Agent" "Show me the customer segmentation overview"
```

Verify:
1. The agent appears in the project
2. Knowledge bases are accessible
3. Queries return data from the RFM output tables

## Key Output Tables Referenced by the Agent

| Table | Purpose |
|-------|---------|
| `rfm_output_table` | Per-profile R/F/M scores and segment labels |
| `rfm_stats` | Distribution statistics for each metric |
| `rfm_combined_user_events` | Union activity table from all sources |
| `rfm_input_table` | Preprocessed input features |

## Customization

After initial deployment, the agent can be customized:

- **Knowledge bases**: Update `business_context.md` with customer-specific business rules, segments, and KPIs
- **Data dictionary**: Update `data_dictionary.md` if custom columns were added to the RFM workflow
- **Prompts**: Add or modify prompt templates for customer-specific analysis patterns
- **SQL guide**: Add customer-specific SQL templates to `sql_guide.md`

Use `tdx agent push` after any local changes to sync updates to TD.

## Related Skills

- **workflow-setup** — RFM workflow configuration and deployment (must run first)
- **prod-docs** — Production documentation and operational runbook
- **tdx-skills/agent** — General `tdx agent` CLI reference
- **tdx-skills/agent-prompt** — System prompt writing best practices
