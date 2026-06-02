---
name: shared-fde-ml-solution-patterns
description: |
  Index of reusable patterns shared across all ml-skills FDE solutions (rfm, nbp, mta etc.). Loaded by solution SKILLs to reuse Confluence folder setup, Current Project State page, push patterns, requirements doc workflow, test case lifecycle, and Phase 6 documentation page set. Not invoked directly — read individual files within this folder as called for by the solution SKILL.
---

# Shared Patterns for FDE ML Skills

Reference library of generic patterns used by every solution under `ml-skills/`. Solution SKILLs (rfm, mta, nba-scores, nbp, etc.) read individual files here to avoid duplicating the generic flow.

## Files

| File | What it contains |
|------|-----------------|
| `confluence_folder_setup.md` | Discovery + creation of the customer's Confluence folder hierarchy (CUST → Region → Customer → FDE Solutions → solution folder) |
| `current_project_state.md` | The cross-session context store concept — `Current Project State - <Customer>` Confluence page format and update rules |
| `push_pattern_td_workflows.md` | Push pattern for deploying a TD Workflow project — cloning the repo, configuring `input_params.yml`, setting secrets, and running `tdx wf push` |
| `push_pattern_new_llm_project.md` | Push pattern for deploying a companion Foundry agent into a new LLM project — `tdx llm project create`, `tdx.json` setup, and `tdx agent push` (e.g., RFM Analysis Agent, NBA Insights Agent) |
| `requirements_doc_pattern.md` | Customer-fillable Confluence page workflow + share + pause-for-customer-response pattern |
| `test_cases_pattern.md` | Two-round test case lifecycle (TC-IDs, Confluence test cases page, `tdx agent test`, iteration loop) |
| `parent_segment_update.md` | Phase 6 Step 1 — process for adding score attributes to the Parent Segment and creating example audience segments in Audience Studio. Handles the "how"; solution's `prod-docs/references/parent_segment.md` provides the "what". |
| `customer_docs_pattern.md` | Phase 6 Step 2+ — the 5 standard customer-specific Confluence pages (Architecture, Behavior, Eval Results, Runbook, Access & Ownership) |

## How solutions use this

A solution SKILL.md for a new engagement type will reference these files in its phases. For example:

```markdown
### Phase 3: Customer Requirements Doc

Read `../shared/confluence_folder_setup.md` for folder discovery + creation.
Read `../shared/current_project_state.md` for the State page setup.
Read `../shared/requirements_doc_pattern.md` for the customer-shareable page workflow.

**Solution-specific overrides:**
- Page title: `<solution> Requirements - <Customer>`
- Body template: see `workflow-setup/references/requirements_doc.md`
```

## Solution-specific configuration

These values are populated dynamically based on which solution is being set up. The calling solution SKILL provides them.

| Field | Value |
|------|-------|
| Solution folder name | `<Solution Name>` (e.g., `RFM`, `MTA`, `NBA Scores`, `NBP`) |
| Solution folder title variants | Variants the calling SKILL specifies for fuzzy Confluence title matching (e.g., `RFM`, `RFM Segmentation`, `Customer Segmentation`) |
| Push pattern | `../shared/push_pattern_td_workflows.md` for workflow deploy; `../shared/push_pattern_new_llm_project.md` for companion agent deploy |
| Companion agent | Whether a companion Foundry agent exists for this solution — see `<Solution Name>/agent-skills/foundry-agent/SKILL.md` |
| Workflow config file | Solution-specific config file path (e.g., `input_params.yml` for RFM/MTA/NBA, `params.yml` for NBP) |

## Entry Point — Where to Start

Before starting any phase, ask the user: **"Is this a new solution engagement, or are you resuming an existing one?"**

- **New engagement:** First present high-level project plan summary and what tasks will be done during each stage. Then start at Phase 1. 
- **Resume:** ask for the customer name, search Confluence for `Current Project State - <Customer>` (see `../shared/current_project_state.md`). Read it — its "Current phase" + "Next Action" fields say where to pick up.

Common phrases mapped to phases:

| User says | Likely phase |
|---|---|
| "the customer filled out their requirements" | Phase 5 |
| "let's run tests on the agent" | Phase 4 (Round 1) or Phase 5 (Round 2) — check Current Project State |
| "the agent's responses need tweaking" / "fix this failing test case" | Phase 5 iteration loop |
| "write the customer facing / handoff docs for the prod-ready project" | Phase 6 |
| "push a fresh workflow for [customer]" | Phase 1 → Phase 2 |
| "create the requirements doc" | Phase 3 (assumes Phase 2 push already done) |

When in doubt, **read Current Project State first**.

## Workflow Sequence

The flow mirrors the audience agent: push first → gather requirements → two-round test cases → docs.

### Phase 1: Data Exploration

Read `../shared/push_pattern_new_llm_project.md` Step 1 for project naming convention.

Analytics-specific:
1. Ask for the **customer name**.
2. Choose a project name following `<Customer> <Solution Name>` convention.
3. TODO: any pre-push schema discovery the analytics agent needs? Probably yes if it queries a specific database — list the relevant skills and commands here.

### Phase 2: Create Customer Requirements Doc on Confluence

Read `../shared/confluence_folder_setup.md` for folder discovery + creation (the `Custom Analytics Agent` folder under `<Customer>/FDE Solutions/`).

Read `../shared/current_project_state.md` for the State page setup — create it now, before the requirements doc.

Read `../shared/requirements_doc_pattern.md` for the customer-shareable page workflow.

Solution-specific:
- Page title: `<Solution Name> Requirements - <Customer>` (suffix is mandatory — Confluence enforces unique titles per space)
- Body template: see `workflow-setup/references/requirements_doc.md` (TODO — needs solution-specific section list)

### Phase 3: Push Minimal Version of Workflow

Read `../shared/push_pattern_td_workflows.md` for the full push flow. Read `<Solution Name>/SKILL.md` for the solution-specific instructions on what reference files to use for initial workflow setup.

After workflow is done running perform quick validation of expected output tables. Read `<Solution Name>/workflow-setup/references/eval.md`

After validation, update **Current Project State**: Phase 3 complete, project name, push date.

### Phase 4: Minimal Agent Setup and Validation

Read `../shared/push_pattern_new_llm_project.md` for the full push flow (clone, set tdx.json, `tdx llm project create`, `tdx agent push -y`). Read `<Solution Name>/agent-skills/foundry-agent/SKILL.md` for the solution-specific files-to-edit table.

Solution-specific:
- Template repo: TODO
- populate business_context.md knowledge base with your inferred knowledge from Phases 1 and 2
- Integration check substring: TODO
- No `rm -rf` step (fresh project, no read-only platform agents)

After Foundry agent setup perform quick summary of the validation process performed as instructed by the `<Solution Name>/agent-skills/foundry-agent/SKILL.md` file.

After validation, update **Current Project State**: Phase 4 complete, project name, push date.

### Phase 5: Update Workflow with Customer Requirements (Round 2)

Almost always a new session. **First action: read Current Project State.**

Solution-specific updates:
1. Read filled requirements doc.
2. Present summary of updates you plan to push to existing workflow and ask customer to provide any additional context if needed
2. Update existing workflow with new params based on customer feedback — TODO list which files.
3. Re-run wf and perform output table validation (Round 2).

### Phase 6: Update Parent Segment and Create Handoff Customer Documentation

If resuming in a new session, **first read Current Project State**.

**Step 1 — Parent Segment attributes + example audiences.**
Read `../shared/parent_segment_update.md` for the process (approval gates, push mechanics, `attributes:` placement rules).
Read the solution's `prod-docs/references/parent_segment.md` for the "what" (which attribute columns and which example segments to create). Do not read this file until the user has approved the plan in Step 1 of `parent_segment_update.md`.

**Step 2+ — Confluence documentation pages.**
Read `../shared/customer_docs_pattern.md` for the 5-page set + create order + keep-current rules.

Analytics-specific page content: see `prod-docs/references/` (TODO).

## Sub-Folder Reference

| Folder / File | Contents |
|---------------|----------|
| `agent-skills/foundry-agent/SKILL.md` | solution-specific Phase 2 + Phase 5 file edits — TODO |
| `workflow-setup/references/requirements_doc.md` | solution-specific customer-fillable body template — TODO |
| `workflow-setup/references/eval.md` | solution-specific test categories + example prompts — TODO |
| `prod-docs/references/` | solution-specific content for the Phase 6 documentation page set — TODO |
| `../shared/*.md` | Generic patterns reused across all ml-skills solutions |


Solution SKILLs handle the *what* (which tables, which YAML parameters, which output tables to validate). The shared files handle the *how* (Confluence flow, push mechanics, validation rhythm).
