---
name: custom-analytics-agent-setup
description: |
  Analytics-specific Phase 2 (first push) and Phase 5 (re-push) file-edit checklist. STATUS: scaffold — TODOs to fill in once template repo lands. Push mechanics live in ../../shared/push_pattern_fresh_project.md.
---

# Custom Analytics Agent — Agent Setup

> **STATUS — Scaffold.** TODO sections to fill in once template repo lands.

Analytics-specific overlay on the shared fresh-project push pattern. The push mechanics (`tdx llm project create`, `tdx agent push -y`) live in `../../shared/push_pattern_fresh_project.md`. This file specifies *which files to edit* for an analytics-agent engagement.

## Routing

| Task | Action |
|------|--------|
| The actual push command sequence | Read `../../shared/push_pattern_fresh_project.md` |
| Author the customer-shareable requirements doc on Confluence | Read `references/requirements_doc.md` |
| Generate test cases, run `tdx agent test`, manage two-round eval | Read `references/eval.md` (and `../../shared/test_cases_pattern.md`) |

## Phase 2 — First Push

Follow `../../shared/push_pattern_fresh_project.md` end-to-end. Analytics-specific values to plug into that flow:

### Files to edit

TODO once template repo lands. Default model for any `agent.yml` files in the analytics template should be `model: claude-4.5-sonnet` (matches platform default).

| File | Change |
|------|--------|
| `tdx.json` | `llm_project` → `<Customer> Analytics Agent` |
| Each `*/agent.yml` in the template | Confirm `model: claude-4.5-sonnet`. If template ships with `gpt-4.1`, flip to Claude before push. |
| TODO: knowledge base file path | TODO: leave empty for first push, Phase 5 fills it in |
| TODO: integration config | TODO |
| TODO: main agent prompt | TODO: optional tone tweaks |

### Pre-push integration check substring

TODO: identify the substring in the integration YAML that confirms the main analytics agent is wired to the chat UI.

## Phase 5 — Re-push After Customer Requirements

Follow `../../shared/push_pattern_fresh_project.md` "Re-Pushing Later" section. Analytics-specific updates between sessions:

1. Read filled requirements doc via `getConfluencePage`.
2. TODO: which knowledge base files to update from the customer's answers.
3. TODO: any optional secondary KBs the customer may have provided.
4. `tdx agent push -y`.
5. Re-run test cases per `references/eval.md` Round 2.
