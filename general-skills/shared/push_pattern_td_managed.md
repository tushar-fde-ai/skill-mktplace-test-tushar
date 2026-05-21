---
name: shared-push-pattern-td-managed
description: |
  Reusable push pattern for FDE solutions that customize an EXISTING TD-Managed project (auto-provisioned alongside a parent segment). Covers project discovery, the rm -rf TD-Managed:* read-only-dirs step, integration check, and tdx agent push. Used by custom-audience-agent.
---

# Shared: Push Pattern — TD-Managed Project

Use this pattern when the solution pushes custom agents/KBs **into an existing TD-Managed project** that the platform auto-provisioned alongside a parent segment. The TD-Managed project ships with read-only platform agents that the FDE deploy must work around.

**Other pattern:** `push_pattern_fresh_project.md` for solutions that need a fresh `tdx llm project create`.

## Mental Model

When a parent segment is set up in TD, the platform auto-provisions an AI project named `TD-Managed: <Parent Segment Name>` containing read-only platform agents (e.g., Marketing Copilot, Data Source Finder, Questions Suggester for audience use cases). Those cannot be edited.

To customize, push parallel agents/KBs into the same project. The template repo's `TD-Managed: *` agent directories are reference copies only — they must be deleted locally before push, because `tdx agent push` errors on read-only agents.

## Step 1: Discover Target Project

Load `tdx-skills:tdx-basic` (auth + general `tdx` syntax). Then:

```bash
tdx llm project list 2>&1 | grep -i "TD-Managed:" || echo "COMMAND_NOT_FOUND"
```

- If output lists projects: present them, let the user pick.
- If `COMMAND_NOT_FOUND` or any error: fall back to constructing `TD-Managed: <parent-segment-name>` and ask the user to confirm.

Record the exact project name (e.g., `TD-Managed: Ecommerce Audience`).

## Step 2: Clone the Template

The calling SKILL provides the template repo URL. Clone, then `cd` into the (single) project directory under `agents/`:

```bash
git clone <template-repo-url>
cd <repo-name>/agents
cd "$(ls -d */ | head -n 1)"   # there's only one project dir; future-proofs against template renames
```

## Step 3: Set tdx.json

Set `tdx.json` `llm_project` to the exact `TD-Managed: <Parent Segment>` name from Step 1.

## Step 4: Pre-Push Integration Check

`Read` `integrations/chat_parent_segment.yml` and confirm a substring matching the calling SKILL's expected agent reference appears in the `actions:` block. (Calling SKILL specifies the substring — e.g., for audience: `name: "Custom Audience Agent"`.)

If missing, restore it before proceeding. Without it the chat UI won't surface the custom agent after push.

## Step 5: Delete Read-Only Platform Agent Directories

The calling SKILL provides the list of `TD-Managed: *` directories to remove. For audience agent:

```bash
rm -rf "TD-Managed: Marketing Copilot" "TD-Managed: Data Source Finder" "TD-Managed: Questions Suggester"
```

These exist as reference only. Pushing them errors because the platform versions are read-only.

## Step 6: Push

Load `tdx-skills:agent` for `tdx agent push` mechanics. From inside the directory containing `tdx.json`:

```bash
tdx agent push -y
```

**No `tdx llm project create`** — the project already exists. Push uploads the custom agents, KBs, prompts, and integration into it.

## Step 7: Verify

Ask the user to verify in the TD UI that the chat integration surfaces the new custom agent prompt.

## Re-Pushing Later (Phase 5 / iterations)

A later session may have re-cloned or `git pull`-ed and reintroduced the `TD-Managed: *` directories. Always re-confirm Steps 4 + 5 before re-pushing:

```bash
rm -rf "TD-Managed: Marketing Copilot" "TD-Managed: Data Source Finder" "TD-Managed: Questions Suggester"
tdx agent push -y
```
