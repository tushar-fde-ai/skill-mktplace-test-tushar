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

Ask the user for the project name. Convention: `<Customer> <Solution Name>` (e.g., `ACME Analytics Agent`). Avoid the `TD-Managed:` prefix — that's reserved for platform-provisioned projects.

Record the exact project name.

## Step 2: Clone the Template + Create Customer Branch

The calling SKILL provides the template repo URL. Clone, create the customer branch, then `cd` into the project directory under `agents/`:

```bash
git clone <template-repo-url>
cd <repo-name>
git checkout -b customer/<slug>
cd agents
cd "$(ls -d */ | head -n 1)"
```

The `<slug>` is auto-derived from the customer name. See `customer_branch_pattern.md` for slug rules + branch lifecycle. The customer branch is the long-lived deployed-state-of-record — every Phase 2/3/4 edit lands as a commit here.

After Step 6 push, push the branch to origin:

```bash
git push -u origin customer/<slug>
```

Record the customer branch URL on **Current Project State** under Artifact URLs.

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

## Re-Pushing Later (Phase 3 distill / Phase 4 / iterations)

Later sessions resume on the customer branch. First action: read `Current Project State` to recover the branch URL, then:

```bash
git fetch origin
git checkout customer/<slug>
git pull
```

The project already exists, so skip Step 5. Make edits, then:

```bash
# ... edit KBs / prompt.md per phase ...
git commit -am "Phase <N>: <one-line summary>"
git push
tdx agent push -y
```

See `customer_branch_pattern.md` for the per-phase commit message conventions and what to commit / .gitignore.
