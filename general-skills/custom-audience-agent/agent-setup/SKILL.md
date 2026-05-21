---
name: custom-audience-agent-setup
description: |
  Audience-specific Phase 2 (first push) and Phase 5 (re-push after customer requirements) file-edit checklist. Push mechanics live in ../../shared/push_pattern_td_managed.md.
---

# Custom Audience Agent — Agent Setup

This file is the audience-specific overlay on the shared TD-Managed push pattern. The push mechanics (project discovery, integration check, `rm -rf TD-Managed:*`, `tdx agent push`) live in `../../shared/push_pattern_td_managed.md`. This file specifies *which files to edit* and *which to leave alone* for an audience-agent engagement.

## Routing

| Task | Action |
|------|--------|
| The actual push command sequence | Read `../../shared/push_pattern_td_managed.md` |
| Author the customer-shareable requirements doc on Confluence | Read `references/requirements_doc.md` |
| Fill out `business_context.md` after the customer responds | Read `references/business_context_template.md` |
| Add a customer-supplied `sql_templates.md` KB | Read `references/sql_templates_template.md` |
| Generate test cases, run `tdx agent test`, manage two-round eval | Read `references/eval.md` (and `../../shared/test_cases_pattern.md`) |

## Phase 2 — First Push

Follow `../../shared/push_pattern_td_managed.md` end-to-end. Audience-specific values to plug into that flow:

### Files to edit

| File | Change |
|------|--------|
| `tdx.json` | `llm_project` → exact name of the selected `TD-Managed: <Parent Segment>` project |
| `knowledge_bases/business_context.md` | **Phase 2: don't edit.** The shipped placeholder ("You are an expert analyst") is fine for first push. Phase 5 fills it in from `references/business_context_template.md`. |
| `integrations/chat_parent_segment.yml` | Optional: `chat_widget_label`, `chat_welcome_message` |
| `Custom Audience Agent/prompt.md` | Optional: tone tweaks, vertical-specific guardrails |

### Read-only platform agent directories to delete before push

| Directory | Why |
|-----------|-----|
| `TD-Managed: Marketing Copilot/` | Read-only in target project |
| `TD-Managed: Data Source Finder/` | Read-only in target project |
| `TD-Managed: Questions Suggester/` | Read-only in target project |

### Directories to leave alone

| Directory / File | Why |
|------------------|-----|
| `Clone Data Source Finder/` | gpt-4.1 mirror; main agent refs it via `@ref` |
| `Clone Questions Suggester/` | gpt-4.1 mirror; main agent refs it via `@ref` |
| `knowledge_bases/get_segment_draft_rules.md` | Shared segment-rule JSON schema |

### Pre-push integration check substring

`name: "Custom Audience Agent"` must appear in `integrations/chat_parent_segment.yml` `actions:` block.

## Phase 5 — Re-push After Customer Requirements

Follow `../../shared/push_pattern_td_managed.md` "Re-Pushing Later" section end-to-end. Audience-specific updates between sessions:

1. Read filled requirements doc via `getConfluencePage` (URL in Current Project State).
2. Update `knowledge_bases/business_context.md` from customer answers using `references/business_context_template.md`.
3. If customer provided SQL templates in §9 of the requirements doc, create `knowledge_bases/sql_templates.md` using `references/sql_templates_template.md`.
4. Re-confirm pre-push state: `TD-Managed: *` directories absent locally (a `git pull` may have reintroduced them — `rm -rf` again if so), integration reference still intact.
5. `tdx agent push -y`.
6. Re-run test cases per `references/eval.md` Round 2.
