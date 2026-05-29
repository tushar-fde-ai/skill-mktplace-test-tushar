---
name: shared-test-cases-pattern
description: |
  Reusable two-round test case lifecycle for FDE solutions. Covers TC-ID convention, Confluence test cases page, test.yml, Round 1 (empty context) → Round 2 (post customer requirements), updateConfluencePage mechanics, and iteration loop. Used by every solution under general-skills/.
---

# Shared: Test Cases Pattern

Every FDE agent solution validates its deployment with a graded set of test cases run twice — once on the freshly-pushed empty agent (Round 1), once after the customer requirements doc is incorporated (Round 2).

## TC-ID Convention

Every test case gets a stable ID: `TC-001`, `TC-002`, …. IDs persist across rounds. Failures referenced in iteration logs use the TC-ID.

Typical engagement: 10-15 cases. Solution-specific categories — see the calling SKILL's `agent-setup/references/eval.md` for what categories apply. Mix complexity (simple / medium / complex).

## When to run

| Round | When |
|---|---|
| Round 1 | After Phase 2 push, while waiting for customer requirements doc. Empty/default knowledge bases. |
| Round 2 | After Phase 5 re-push (customer requirements incorporated). |

If resuming in a new session for either round, **first action is to read `Current Project State - <Customer>`** to recover the project name and prior URLs.

Skills to load:
- `tdx-skills:agent-test` — for `tdx agent test` mechanics, `test.yml` format, output parsing
- (Solution-specific schema-discovery skill — calling SKILL specifies, e.g., `tdx-skills:parent-segment-analysis` for audience)

## Step 1: Generate Test Cases

The calling SKILL provides the categories and example prompts. Generate 10-15 cases drawn from the actual schema discovered in earlier phases. Structure:

| TC-ID | Category | Complexity | Test Prompt | Expected Behavior | Pass Criteria | Round 1 Result | Round 2 Result | Notes |
|---|---|---|---|---|---|---|---|---|
| TC-001 | <category> | <simple/medium/complex> | <prompt> | <what should happen> | <what to check in response> | | | |

## Step 2: Create the Confluence Test Cases Page

Title: `<solution> Test Cases - <Customer>`

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <solution_folder_page_id>
  title: "<solution> Test Cases - <Customer>"   # MUST be suffixed — Confluence enforces unique titles per space
  contentFormat: markdown
  body: |
    # <solution> Test Cases - <Customer>

    **Project:** <project name>
    **Round 1 run date:** <date>
    **Round 2 run date:** <date or "pending">

    ## Summary

    | Round | Pass | Fail | Pass Rate |
    |---|---|---|---|
    | Round 1 (empty knowledge bases) | X / N | Y / N | Z% |
    | Round 2 (post customer requirements) | X / N | Y / N | Z% |

    ## Test Cases

    | TC-ID | Category | Complexity | Test Prompt | Expected Behavior | Pass Criteria | Round 1 Result | Round 2 Result | Notes |
    |---|---|---|---|---|---|---|---|---|
    | TC-001 | ... | ... | ... | ... | ... | ✅/❌ | ✅/❌ | ... |
    | ...
```

## Step 3: Mirror to test.yml

Place at the project root (same level as `tdx.json`). Annotate each entry with its TC-ID:

```yaml
# TC-001
- user_input: "<prompt>"
  criteria:
    - <criterion 1>
    - <criterion 2>
```

Multi-round (Discovery → Execution) tests use the `rounds:` syntax — see `tdx-skills:agent-test`.

## Step 4: Run Round 1

```bash
cd agents/<project-dir>
tdx agent test
```

Parse pass/fail from output. Then full-page-replace the Confluence test cases page via `updateConfluencePage`:

```
updateConfluencePage:
  cloudId: treasure-data.atlassian.net
  pageId: <test_cases_page_id>
  title: "<solution> Test Cases - <Customer>"
  contentFormat: markdown
  body: <updated table with Round 1 Result column filled>
```

(Confluence updates are full-page replaces — render the entire markdown body again with the new column populated.)

Update **Current Project State**: test cases page URL, Round 1 pass rate, failing TC-IDs.

**Many failures are expected in Round 1** — that's the point. Cases that fail because the agent doesn't know customer-specific terms or business rules are signals for the requirements doc.

When updating the requirements doc Confluence page, link the customer to specific failing TC-IDs to make the request concrete: *"TC-004 fails because the agent doesn't know what 'X' means. Please define it in §<section>."*

## Step 5: Run Round 2

After Phase 5 (knowledge bases updated, re-push):

```bash
tdx agent test
```

`updateConfluencePage` again — fill the Round 2 Result column. For any case still failing, the calling SKILL's `eval.md` provides a "failure pattern → fix location" table.

After each fix:

```bash
tdx agent push -y
tdx agent test
```

Stop when pass rate is acceptable for the customer's tolerance. Document remaining failures as known limitations on the Phase 6 Eval Results page.

Update **Current Project State** with the final Round 2 pass rate and remaining failing TC-IDs.
