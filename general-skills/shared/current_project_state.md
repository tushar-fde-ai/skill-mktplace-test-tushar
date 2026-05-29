---
name: shared-current-project-state
description: |
  Cross-session context store concept for FDE engagements. Defines the "Current Project State - <Customer>" Confluence page format and the rule that every resumed session reads this page first. Used by every solution under general-skills/.
---

# Shared: Current Project State

FDE engagements span multiple sessions. The **Current Project State - <Customer>** Confluence page is the cross-session context store — a running summary of what's been done and the URLs of every artifact created.

## Rules

1. **Create early.** The Current Project State page is the *first* page created under the solution folder, before any customer-shareable docs.
2. **Update at the end of every phase.** Phase completion, date, key URLs, key decisions, next action.
3. **Read first on resume.** When starting a new session, the LLM's first action is `searchConfluenceUsingCql` for `Current Project State - <Customer>` (title match). The "Current phase" + "Next Action" fields determine where to pick up.
4. **Authoritative.** When in doubt, the State page wins.

## Page Creation

Title: `Current Project State - <Customer>` (suffix is mandatory — Confluence enforces unique titles per space)

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <solution_folder_page_id>
  title: "Current Project State - <Customer>"   # MUST be suffixed — Confluence enforces unique titles per space
  contentFormat: markdown
  body: |
    # Current Project State - <Customer>

    Living context store. Update at the end of every phase. New sessions should `searchConfluenceUsingCql` for this page (title match) and read it first.

    ## Status
    - **Solution:** <solution name, e.g. "Custom Audience Agent" / "Custom Analytics Agent">
    - **Current phase:** <number + short description>
    - **Last updated:** <date>

    ## Project
    - **Customer:** <name>
    - **TD Project:** <project name — for audience: "TD-Managed: <Parent Segment>"; for analytics: the freshly created project name>
    - **Solution folder ID:** <Confluence folder page ID>

    ## Artifact URLs
    - Requirements doc: <pending or URL>
    - Test cases page: <pending or URL>
    - Architecture: <pending or URL>
    - Behavior summary: <pending or URL>
    - Eval Results: <pending or URL>
    - Runbook: <pending or URL>
    - Access & Ownership: <pending or URL>

    ## Key Decisions
    <none yet — append per phase>

    ## Next Action
    <one-sentence handoff for the next session>
```

## Updates

Use `updateConfluencePage` (full-page replace) at the end of each phase. Render the entire markdown body again with updated fields.

```
updateConfluencePage:
  cloudId: treasure-data.atlassian.net
  pageId: <state_page_id>
  title: "Current Project State - <Customer>"
  contentFormat: markdown
  body: <fully re-rendered markdown>
```

## Phase-by-phase update conventions

| Phase complete | Update fields |
|---|---|
| Phase 1d (requirements doc published) | Current phase → 2, Requirements doc URL, customer notification date in Key Decisions |
| Phase 2 push | Current phase → 3, project name + push date in Key Decisions |
| Phase 3 Round 1 tests | Current phase → 4 (waiting on customer), Test cases page URL, Round 1 pass rate, failing TC-IDs |
| Phase 4 Round 2 tests | Current phase → 5, Round 2 pass rate, remaining limitations |
| Phase 5 docs | Current phase → "complete — engagement live", all 5 doc URLs populated |
