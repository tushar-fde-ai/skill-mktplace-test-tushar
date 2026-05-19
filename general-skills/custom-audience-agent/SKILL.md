---
name: fde-custom-audience-agent
description: |
  Build and deploy a custom AI Foundry audience agent for CDP parent segment analysis. Covers data exploration, requirements gathering, agent template setup from GitHub, and eval framework creation. 
---

# Custom Audience Agent

Build and deploy a custom AI Foundry agent that analyzes CDP parent segment data — queries customer attributes, behaviors, and segment membership to answer business questions via natural language.

## Workflow Sequence

Follow these phases in order. Each phase has its own reference doc in the appropriate sub-folder.

### Phase 1: Explore Data

Discover the customer's CDP data structure before gathering requirements.

1. Ask for the **TD database name** (or parent segment name)
2. Use `tdx ps desc -o` or `tdx describe` to get the schema
3. List all tables and columns — identify customer attributes, behaviors, and key dimensions
4. Sample data to understand column values, cardinality, and data quality
5. Present findings to the user

Use **td-skills** (`tdx describe`, `tdx query`) and **parent-segment-analysis** skill for CDP-specific exploration.

### Phase 2: Gather Requirements

Read `agent-setup/references/requirements_doc.md` for the full requirements gathering workflow.

Key decisions to collect:
1. **Customer name** and Confluence folder location
2. **Which tables/columns** the agent should have access to (from Phase 1 findings)
3. **Business context** — what questions should the agent answer? What KPIs matter?
4. **Target users** — who will use this agent? (marketing, analytics, CS, executives)
5. **Output format preferences** — charts (Plotly), tables, summaries, or mixed
6. **Any data restrictions** — columns to exclude (PII, internal IDs, etc.)

### Phase 3: Setup AI Foundry Agent

Read `agent-setup/SKILL.md` for the full agent setup workflow.

1. Clone the audience agent template from the GitHub repo
2. Configure knowledge bases to point to the customer's database/tables
3. Update `business_context.md` with customer-specific context from Phase 2
4. Update `data_dictionary.md` with the actual schema from Phase 1
5. Choose project name and update `tdx.json`
6. Create the LLM project: `tdx llm project create "<name>"`
7. Present the agent structure for user confirmation
8. Push: `tdx agent push -y`

### Phase 4: Create Eval Framework

Read `agent-setup/references/eval.md` for eval framework guidelines.

1. Generate a set of **test prompts** based on:
   - The tables and columns discovered in Phase 1
   - The business questions identified in Phase 2
   - Edge cases (empty results, ambiguous questions, cross-table joins)
2. Create `test.yml` with criteria for each prompt
3. Run eval: `tdx agent test`
4. Review results, refine prompts and agent configuration
5. Document final eval results

### Phase 5: Documentation

Read `prod-docs/SKILL.md` for documentation guidelines.

1. Create Confluence pages under the customer's FDE Solutions folder:
   - Requirements summary
   - Agent architecture and table access
   - Eval results and test prompts
2. Follow the Confluence folder structure from `agent-setup/references/requirements_doc.md` Step 1

## Sub-Folder Reference

| Folder | Contents |
|--------|----------|
| `agent-setup/SKILL.md` | Agent template setup and deployment instructions |
| `agent-setup/references/requirements_doc.md` | Requirements gathering workflow (Confluence setup, initial questions) |
| `agent-setup/references/eval.md` | Eval framework and test prompt generation |
| `prod-docs/SKILL.md` | Production documentation guidelines |
| `prod-docs/references/customer_docs.md` | Confluence documentation creation workflow |
| `prod-docs/references/eval.md` | Eval documentation template |

## GitHub Repository

The audience agent template is maintained at:
```
https://github.com/treasure-data-ps/audience_agent_template
```

(Update this URL once the template repo is available. Until then, agents are built from scratch using `tdx agent` skills.)
