---
name: custom-audience-agent-setup
description: |
  Audience-specific Phase 2 (first push) and Phase 4 (re-push after customer reviews) file-edit checklist. Push mechanics live in ../../shared/push_pattern_td_managed.md.
---

# Custom Audience Agent — Agent Setup

This file is the audience-specific overlay on the shared TD-Managed push pattern. The push mechanics (project discovery, integration check, `rm -rf TD-Managed:*`, `tdx agent push`) live in `../../shared/push_pattern_td_managed.md`. This file specifies *which files to edit* and *which to leave alone* for an audience-agent engagement.

## Routing

| Task | Action |
|------|--------|
| The actual push command sequence | Read `../../shared/push_pattern_td_managed.md` |
| Render the Phase 1d Confluence requirements doc body | Read `references/requirements_doc.md` |
| Distill the Phase 3 `business_context.md` from the inference bundle + Phase 4 re-distill from customer-edited doc | Read `references/business_context_template.md` |
| Add a customer-supplied `sql_templates.md` KB | Read `references/sql_templates_template.md` |
| Generate test cases, run `tdx agent test`, manage two-round eval | Read `references/eval.md` (and `../../shared/test_cases_pattern.md`) |

## Phase 2 — First Push

Follow `../../shared/push_pattern_td_managed.md` end-to-end. **Important:** apply the audience-specific file edits below **between Step 3 (set tdx.json) and Step 5 (delete TD-Managed dirs) of the shared push pattern.** Otherwise the FDE engineer risks pushing gpt-4.1 agents to the customer.

### Files to edit (between shared Step 3 and Step 5)

| File | Change |
|------|--------|
| `tdx.json` | `llm_project` → exact name of the selected `TD-Managed: <Parent Segment>` project |
| `Custom Audience Agent/agent.yml` | `model: gpt-4.1` → `model: claude-4.5-sonnet`. Also remove the `get_segment_draft_rules` tool entry from the `tools:` list (gpt-only workaround, not needed for Claude). |
| `Clone Data Source Finder/agent.yml` | `model: gpt-4.1` → `model: claude-4.5-sonnet` |
| `Clone Questions Suggester/agent.yml` | `model: gpt-4.1` → `model: claude-4.5-sonnet` |
| `Custom Audience Agent/prompt.md` | Three edits to this file: (a) **Remove** the line `Before creating a segment draft using the :segment: output, you must call get_segment_draft_rules.` from the `# Segment Draft Creation Guidelines` section (gpt-only step). (b) **Add** under `# Segment Draft Creation Guidelines`: *"For any time-based segment condition, use `TIME WITHIN PAST` with `{value, unit}` (units: day/week/month/year) or `BETWEEN` with ISO 8601 dates. Never emit Unix timestamps — they are not in the segment-draft schema."* (c) Optional: tone tweaks, vertical-specific guardrails. |
| `knowledge_bases/business_context.md` | **Phase 2: don't edit.** The shipped placeholder ("You are an expert analyst") is fine for first push. Phase 3 distills the inference bundle into this file. Phase 4 re-distills from the customer's reviewed Confluence doc. |
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

## Phase 4 — Re-push After Customer Reviews

Follow `../../shared/push_pattern_td_managed.md` "Re-Pushing Later" section end-to-end. Audience-specific updates between sessions:

1. Read the customer's reviewed requirements doc via `getConfluencePage` (URL in Current Project State).
2. **Re-distill `business_context.md` from the customer's reviewed doc** — same distillation rules as Phase 3 (strip markers, hedge phrases, customer-collaborative voice; convert to terse direct agent instructions). See `references/business_context_template.md` "Phase 4 — re-distill" section for examples.
3. If §9 of the doc has SQL templates (whether the customer kept the inference or replaced), distill into `knowledge_bases/sql_templates.md` using `references/sql_templates_template.md`. Drop SQL templates the customer marked `[NEEDS YOUR INPUT]` and didn't fill in.
4. Re-confirm pre-push state: `TD-Managed: *` directories absent locally (a `git pull` may have reintroduced them — `rm -rf` again if so), integration reference still intact.
5. `tdx agent push -y`.
6. Re-run test cases (Round 2) per `references/eval.md`.
