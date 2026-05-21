---
name: custom-analytics-agent
description: |
  Customize and deploy a custom AI Foundry analytics agent in a fresh LLM project. Trigger on: analytics agent, custom analytics agent, data analytics agent, reporting agent, dashboard agent, BI agent. Push-first workflow → customer requirements gathering on Confluence → two-round test cases → customer-specific documentation. STATUS: scaffold — TODO sections must be filled in once template repo lands.
---

# Custom Analytics Agent

> **STATUS — Scaffold.** Structural scaffold copying the audience-agent pattern. Phase-specific overrides (template repo, files-to-edit, requirements questions, test categories, page content) are marked **TODO** below and must be filled in once the analytics-agent template repo is available. The shared patterns (`../shared/`) are fully usable as-is.

Customize and deploy a custom AI Foundry analytics agent — queries any TD database the project is bound to, generates dashboards, and answers analytical questions across customer data.

## Mental Model

Unlike the audience agent (which customizes an existing `TD-Managed: <Parent Segment>` project), the analytics agent **runs in a fresh LLM project** created via `tdx llm project create`. There are no read-only platform agents to delete locally — the full template gets pushed as-is.

**Default model:** `claude-4.5-sonnet` (matches the platform default and the audience-agent convention). All `agent.yml` files in the analytics template should use `model: claude-4.5-sonnet, temperature: 0`.

## Template Repo

```
TODO: paste template repo URL here once available
```

## Solution-specific configuration

These values are passed into the shared patterns:

| Field | Value |
|------|-------|
| Solution folder name | `Custom Analytics Agent` |
| Solution folder title variants | `Analytics Agent`, `Custom Analytics Agent`, `Reporting Agent`, `Dashboard Agent` |
| Push pattern | `../shared/push_pattern_fresh_project.md` |
| Read-only platform agents to delete locally | None (fresh project) |
| Integration check substring | TODO |

## Entry Point — Where to Start

Before starting any phase, ask the user: **"Is this a new analytics agent engagement, or are you resuming an existing one?"**

- **New engagement:** start at Phase 1.
- **Resume:** ask for the customer name, search Confluence for `Current Project State - <Customer>` (see `../shared/current_project_state.md`). Read it — its "Current phase" + "Next Action" fields say where to pick up.

Common phrases mapped to phases:

| User says | Likely phase |
|---|---|
| "the customer filled out their requirements" | Phase 5 |
| "let's run tests on the analytics agent" | Phase 4 (Round 1) or Phase 5 (Round 2) — check Current Project State |
| "the agent's responses need tweaking" / "fix this failing test case" | Phase 5 iteration loop |
| "write the docs for the analytics agent" | Phase 6 |
| "push a fresh analytics agent for [customer]" | Phase 1 → Phase 2 |
| "create the requirements doc" | Phase 3 (assumes Phase 2 push already done) |

When in doubt, **read Current Project State first**.

## Workflow Sequence

The flow mirrors the audience agent: push first → gather requirements → two-round test cases → docs.

### Phase 1: Lightweight Project Setup

Read `../shared/push_pattern_fresh_project.md` Step 1 for project naming convention.

Analytics-specific:
1. Ask for the **customer name**.
2. Choose a project name following `<Customer> Analytics Agent` convention.
3. TODO: any pre-push schema discovery the analytics agent needs? Probably yes if it queries a specific database — list the relevant skills and commands here.

### Phase 2: Push Minimal Custom Analytics Agent

Read `../shared/push_pattern_fresh_project.md` for the full push flow (clone, set tdx.json, `tdx llm project create`, `tdx agent push -y`). Read `agent-setup/SKILL.md` for the analytics-specific files-to-edit table.

Analytics-specific:
- Template repo: TODO
- Don't edit knowledge bases yet (Phase 5 fills them in)
- Integration check substring: TODO
- No `rm -rf` step (fresh project, no read-only platform agents)

After push, update **Current Project State**: Phase 2 complete, project name, push date.

### Phase 3: Create Customer Requirements Doc on Confluence

Read `../shared/confluence_folder_setup.md` for folder discovery + creation (the `Custom Analytics Agent` folder under `<Customer>/FDE Solutions/`).

Read `../shared/current_project_state.md` for the State page setup — create it now, before the requirements doc.

Read `../shared/requirements_doc_pattern.md` for the customer-shareable page workflow.

Analytics-specific:
- Page title: `Analytics Agent Requirements - <Customer>` (suffix is mandatory — Confluence enforces unique titles per space)
- Body template: see `agent-setup/references/requirements_doc.md` (TODO — needs analytics-specific section list)

### Phase 4: Generate Test Cases (Round 1 — Empty Context)

Read `../shared/test_cases_pattern.md` for the full test-case lifecycle.

If resuming in a new session, **first read Current Project State**.

Analytics-specific:
- Schema discovery skill: TODO (likely `tdx-skills:tdx-basic` or a database-specific exploration skill)
- Test categories: TODO — likely covers query writing, dashboard generation, multi-step analytics flows, ambiguous/guardrail. Full list in `agent-setup/references/eval.md` (also TODO).

### Phase 5: Update Agent with Customer Requirements (Round 2)

Almost always a new session. **First action: read Current Project State.**

Read `../shared/test_cases_pattern.md` for Round 2 + iteration loop. Read `agent-setup/SKILL.md` Phase 5 for the analytics-specific re-push checklist.

Analytics-specific updates:
1. Read filled requirements doc.
2. Update knowledge bases — TODO list which files.
3. `tdx agent push -y` (no `rm -rf` step needed).
4. Re-run test cases (Round 2).

### Phase 6: Customer-Specific Documentation

Read `../shared/customer_docs_pattern.md` for the 5-page set + create order + keep-current rules.

If resuming in a new session, **first read Current Project State**.

Analytics-specific page content: see `prod-docs/SKILL.md` (TODO).

## Sub-Folder Reference

| Folder / File | Contents |
|---------------|----------|
| `agent-setup/SKILL.md` | Analytics-specific Phase 2 + Phase 5 file edits — TODO |
| `agent-setup/references/requirements_doc.md` | Analytics-specific customer-fillable body template — TODO |
| `agent-setup/references/eval.md` | Analytics-specific test categories + example prompts — TODO |
| `prod-docs/SKILL.md` | Analytics-specific content for the Phase 6 documentation page set — TODO |
| `../shared/*.md` | Generic patterns reused across all general-skills solutions |
