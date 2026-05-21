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
| Write the Phase 4 schema-derived `business_context.md` draft + Phase 5 merge with customer answers | Read `references/business_context_template.md` |
| Add a customer-supplied `sql_templates.md` KB | Read `references/sql_templates_template.md` |
| Generate test cases, run `tdx agent test`, manage two-round eval | Read `references/eval.md` (and `../../shared/test_cases_pattern.md`) |

## Phase 2 — First Push

Follow `../../shared/push_pattern_td_managed.md` end-to-end. Audience-specific values to plug into that flow:

### Files to edit

| File | Change |
|------|--------|
| `tdx.json` | `llm_project` → exact name of the selected `TD-Managed: <Parent Segment>` project |
| `Custom Audience Agent/agent.yml` | `model: gpt-4.1` → `model: claude-4.5-sonnet`. Also remove the `get_segment_draft_rules` tool entry from the `tools:` list (gpt-only workaround, not needed for Claude). |
| `Clone Data Source Finder/agent.yml` | `model: gpt-4.1` → `model: claude-4.5-sonnet` |
| `Clone Questions Suggester/agent.yml` | `model: gpt-4.1` → `model: claude-4.5-sonnet` |
| `Custom Audience Agent/prompt.md` | Remove the line `Before creating a segment draft using the :segment: output, you must call get_segment_draft_rules.` from the `# Segment Draft Creation Guidelines` section (gpt-only step). Optional: tone tweaks, vertical-specific guardrails. |
| `knowledge_bases/business_context.md` | **Phase 2: don't edit.** The shipped placeholder ("You are an expert analyst") is fine for first push. Phase 4 writes a schema-derived draft (Priority Attributes + PII exclusions). Phase 5 merges customer answers into the draft. |
| `knowledge_bases/get_segment_draft_rules.md` | **Delete the file.** Was a gpt-only tool — Claude follows the schema description in the `:segment:` output's `function_description` directly. |
| `integrations/chat_parent_segment.yml` | Optional: `chat_widget_label`, `chat_welcome_message` |

### Read-only platform agent directories to delete before push

| Directory | Why |
|-----------|-----|
| `TD-Managed: Marketing Copilot/` | Read-only in target project |
| `TD-Managed: Data Source Finder/` | Read-only in target project |
| `TD-Managed: Questions Suggester/` | Read-only in target project |

### Directories to leave alone

| Directory / File | Why |
|------------------|-----|
| `Clone Data Source Finder/` (after model flip) | claude-4.5-sonnet mirror; main agent refs it via `@ref` |
| `Clone Questions Suggester/` (after model flip) | claude-4.5-sonnet mirror; main agent refs it via `@ref` |

### Pre-push integration check substring

`name: "Custom Audience Agent"` must appear in `integrations/chat_parent_segment.yml` `actions:` block.

## Phase 5 — Re-push After Customer Requirements

Follow `../../shared/push_pattern_td_managed.md` "Re-Pushing Later" section end-to-end. Audience-specific updates between sessions:

1. Read filled requirements doc via `getConfluencePage` (URL in Current Project State).
2. **Merge customer answers into the existing schema-derived `business_context.md` draft** (the draft was written in Phase 4). See `references/business_context_template.md` "Phase 5 merge rule":
   - Empty customer field → keep the schema draft.
   - Customer wrote something → customer wins, replace the draft section.
   - Customer added items the draft didn't have → append.
3. If customer provided SQL templates in §9 of the requirements doc, create `knowledge_bases/sql_templates.md` using `references/sql_templates_template.md`.
4. Re-confirm pre-push state: `TD-Managed: *` directories absent locally (a `git pull` may have reintroduced them — `rm -rf` again if so), integration reference still intact.
5. `tdx agent push -y`.
6. Re-run test cases per `references/eval.md` Round 2.
