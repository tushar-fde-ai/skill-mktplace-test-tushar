---
name: shared-requirements-doc-pattern
description: |
  Reusable workflow for creating a customer-fillable requirements doc on Confluence. Handles the customer-shareable page creation, share-with-FDE-engineer step, and session-end pause-for-customer-response pattern. Used by every solution under general-skills/.
---

# Shared: Customer Requirements Doc Pattern

Every FDE solution that customizes an agent for a customer needs to gather customer requirements before filling in the agent's knowledge bases. This is done via a customer-fillable Confluence page.

## When to use

After the agent is pushed (Phase 2) but before the agent's knowledge bases are customized (Phase 5).

## Prerequisites

- Solution folder parentId from `confluence_folder_setup.md`
- Current Project State page already created (see `current_project_state.md`)

## Step 1: Check for Existing Doc

Ask:
> Do you have an existing filled-out requirements doc? If yes, paste the Confluence link.

If provided, read with `getConfluencePage` and skip ahead to Phase 5 (use existing answers to fill knowledge bases).

## Step 2: Create the Customer-Shareable Page

The calling SKILL provides:
- The page title (e.g., `Audience Agent Requirements - <Customer>`, `Analytics Agent Requirements - <Customer>`)
- The body template (the actual customer-fillable questions — solution-specific)

Create:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <solution_folder_page_id>
  title: "<solution> Requirements - <Customer>"   # MUST be suffixed — Confluence enforces unique titles per space
  contentFormat: markdown
  body: <calling-SKILL-provided body template>
```

The body template is **solution-specific** — see the calling SKILL's `agent-setup/references/requirements_doc.md` for the actual sections. The body should be phrased for the customer to fill in directly (use `> _Your answer:_` placeholders, etc.).

Body templates should typically include:
- A short purpose statement
- A target return date placeholder
- N solution-specific sections, each with example answers
- A close-out instruction telling the customer how to notify the FDE engineer when done

## Step 3: Hand Off to FDE Engineer

After creating the page:
1. Give the URL to the FDE engineer to share with the customer (the LLM cannot send messages directly).
2. Update **Current Project State**: phase complete, requirements doc URL written into Artifact URLs, Next Action set to whatever the calling SKILL specifies for the wait period (typically Phase 4 test cases on empty agent).
3. **End the session.** The customer will fill out the doc out-of-band — typically days or weeks. Phase 5 resumes in a new session when the customer returns the doc.

## Reading the Filled Doc (Phase 5 entry)

When the FDE engineer returns:
1. Read the State page first to recover URLs.
2. `getConfluencePage` on the requirements doc URL.
3. Map customer answers to the agent's knowledge base sections per the calling SKILL's section-to-file mapping table.
4. Present a summary back to the FDE engineer for confirmation before re-pushing.
