---
name: shared-push-pattern-fresh-project
description: |
  Reusable push pattern for FDE solutions that create a NEW LLM project (not bound to an existing TD-Managed project). Covers tdx llm project create, tdx.json setup, and tdx agent push. Used by custom-analytics-agent.
---

# Shared: Push Pattern — Fresh Project

Use this pattern when the solution creates a **new LLM project from scratch**, not customizing an existing TD-Managed one.

**Other pattern:** `push_pattern_td_managed.md` for solutions that customize an existing `TD-Managed: <Parent Segment>` project.

## Mental Model

The customer's TD instance does not auto-provision a project for this solution. The FDE engineer creates a new LLM project, sets `tdx.json` to that project name, and pushes the full agent template into it. There are no read-only platform agents to delete.

## Step 1: Choose Project Name

**Always use `AskUserQuestion` to ask for the project name before touching any files.** Never assume the default — always present it as an option and wait for explicit confirmation.

```
AskUserQuestion:
  question: "What should the Foundry LLM project be named for <Customer>?"
  header: "Project name"
  options:
    - label: "<Default Solution Name> (Recommended)"
      description: "Use the standard default project name for this solution"
    - label: "Custom name"
      description: "I'll type a custom project name below"
```

The calling SKILL provides the default name (e.g., `NBA Engagement Scores`, `MTA Journey Analysis`). Convention for custom names: `<Customer> <Solution Name>` (e.g., `ACME MTA Journey Analysis`). Avoid the `TD-Managed:` prefix — that's reserved for platform-provisioned projects.

Record the exact project name confirmed by the user. Use it in Step 3 (tdx.json) and Step 5 (project create).

## Step 2: Clone the Template

The calling SKILL provides the template repo URL. Clone, then `cd` into the project directory under `agents/`:

```bash
git clone <template-repo-url>
cd <repo-name>/agents
cd "$(ls -d */ | head -n 1)"
```

## Step 3: Set tdx.json

Set `tdx.json` `llm_project` to the project name from Step 1.

## Step 4: Customize (calling SKILL specifies which files)

The calling SKILL provides the list of files to edit (typically: knowledge bases, integrations config, optional prompt tweaks). Refer to the solution's `agent-setup/SKILL.md` for the per-customer file list.

## Step 5: Create the Project

Load `tdx-skills:agent`. Create the LLM project:

```bash
tdx llm project create "<project-name>"
```

If the project already exists (e.g., re-pushing in a later session), this errors — skip to Step 6.

## Step 6: Push

From inside the directory containing `tdx.json`:

```bash
tdx agent push -y
```

No `rm -rf` step — there are no read-only platform agents in a fresh project.

## Step 7: Verify

Ask the user to verify in the TD UI that the project exists and the agent + integration surfaced as expected.

## Re-Pushing Later (Phase 5 / iterations)

The project already exists, so skip Step 5. Just `tdx agent push -y` from the project directory.
