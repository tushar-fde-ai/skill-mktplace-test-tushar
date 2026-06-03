---
name: shared-customer-branch-pattern
description: |
  Per-customer branching convention for FDE template repos. Every customer engagement gets a long-lived `customer/<slug>` branch off main. Phase 2 creates the branch with edited template files; Phase 3 commits the distilled KBs; Phase 4 re-distills and commits again. The branch IS the customer's deployed artifact — kept forever post-engagement. Used by every solution under general-skills/ that pulls from a template repo.
---

# Shared: Per-Customer Branch Pattern

Every FDE deployment is mirrored on a long-lived branch in the template repo so the customer's deployed state is reconstructable from git. Not a fork — a branch on the template repo itself.

## Why a branch (not a fork or a separate repo)

- **Single source of truth.** The template's main branch holds the canonical, vertical-agnostic version. Customer branches diverge with their KBs / config but rebase onto main whenever the template gets a generic improvement.
- **Visibility.** All engagements live in one repo. New FDEs onboarding can `git branch -a` to see every active customer.
- **Reuse.** A new customer engagement can `git checkout customer/<similar-prior>` as a starting point if their schema looks similar to a past customer.
- **No org sprawl.** No need to mint a new repo per engagement (which would require admin permissions every time).

## Branch naming

```
customer/<slug>
```

Where `<slug>` is the customer name lowercased, with non-alphanumerics replaced by `-`, runs of `-` collapsed, and leading/trailing `-` stripped.

| Customer name | Slug | Branch |
|---|---|---|
| `Test Customer Retail` | `test-customer-retail` | `customer/test-customer-retail` |
| `ACME Corp.` | `acme-corp` | `customer/acme-corp` |
| `Bob's Burgers (B2B)` | `bobs-burgers-b2b` | `customer/bobs-burgers-b2b` |

If the auto-derived slug collides with an existing branch (rare — implies a same-named customer or a re-engagement), append `-2`, `-3`, etc. and record the chosen slug on **Current Project State** so the engineer can resume against the right branch in later sessions.

## Lifecycle

| Phase | Branch action | What gets committed |
|---|---|---|
| **Phase 2** (first push) | `git clone <template-repo>` → `git checkout -b customer/<slug>` → solution-specific edits → first commit `Phase 2: minimal push for <Customer>` → `git push -u origin customer/<slug>` → `tdx agent push -y` | Edited `tdx.json`, edited config files (`master_database.yml`, `agent.yml` model field, deletes of read-only `TD-Managed:*` agents for audience). KBs still as shipped stubs. |
| **Phase 3** (distill from inference bundle) | Pull → edit KBs locally → `git commit -am "Phase 3: distilled KBs from Phase 1 inference bundle"` → `git push` → `tdx agent push -y` | Distilled customer-specific KBs (e.g., `business_context.md`, `data_dictionary.md`, `sql_templates.md`). |
| **Phase 4** (re-distill from customer edits) | Pull → re-distill KBs from customer's reviewed Confluence → `git commit -am "Phase 4: re-distilled KBs from customer requirements review"` → `git push` → `tdx agent push -y` | Re-distilled KB(s). Customer's edits supersede Phase 1 inferences for any section they touched. |
| **Phase 4 iteration loop** | Each fix: edit → `git commit -am "Phase 4: fix TC-<N> — <one-line reason>"` → `git push` → `tdx agent push -y` → re-test | KB or `prompt.md` adjustments per failing test case. |
| **Engagement complete** | Branch lives forever. No merge to main — customer state is intentionally divergent. | n/a |

## Commit message convention

```
Phase <N>: <one-line summary>

<optional 2-4 line body if the change is non-obvious>
```

Examples:
- `Phase 2: minimal push for Test Customer Retail`
- `Phase 3: distilled KBs from Phase 1 inference bundle`
- `Phase 4: re-distilled business_context.md from customer requirements`
- `Phase 4: fix TC-007 — agent missed VIP key term, added to Key Terms section`

## What to commit / what to .gitignore

Commit:
- `tdx.json` (with customer's project name)
- All config YAML files (`agent.yml`, `master_database.yml`, integration files)
- All knowledge bases referenced by `agent.yml` tools
- `prompt.md` files (any agent or sub-agent customizations)
- `test.yml` (so test cases version with the deployment)

Don't commit:
- `.tdx/` cache or session directories (already in template's `.gitignore`)
- Any local secrets — the customer's API key / auth context lives in `tdx auth setup`, never in repo files
- Customer-confidential raw data dumps from Phase 1 schema exploration — those go into the KBs as distilled summaries, not as raw query output files

## Resuming on an existing branch (multi-session)

Every phase update ends with `git push` to the customer branch. To resume in a new session:

1. Read `Current Project State - <Customer>` to find the branch name (the State page records `customer/<slug>` under Artifact URLs).
2. `git fetch origin` → `git checkout customer/<slug>` → `git pull`.
3. Continue with the phase indicated by State page's "Current phase" field.

If the engineer's local checkout is on the wrong branch (e.g., they've been working on a different customer), first stash or commit current work, *then* switch — never `git checkout -f` past uncommitted state.

## Rebasing onto template improvements

When the template's main branch gets a generic improvement (model upgrade, new shared tool, fixed prompt bug), customer branches can pull it in:

```bash
git checkout customer/<slug>
git fetch origin
git rebase origin/main
# resolve any conflicts (rare — customer changes are mostly KBs, template changes mostly prompts/configs)
git push --force-with-lease
tdx agent push -y
```

This is engineer-discretion, not automatic. Don't rebase mid-engagement (Phases 2-4) without notifying the FDE engineer — could introduce unrelated changes that affect the test rounds.

## Recording the branch URL

Phase 2 records the customer branch URL on **Current Project State** under Artifact URLs:

```
- Customer template branch: https://github.com/<org>/<template-repo>/tree/customer/<slug>
```

Future sessions read this first to know where to check out.
