---
name: shared-confluence-folder-setup
description: |
  Reusable Confluence folder discovery + creation flow for any FDE solution that documents under the Customers (CUST) space. Handles customer-folder lookup by name or URL, FDE Solutions sub-folder creation, and a solution-specific sub-folder (e.g., "Custom Audience Agent"). Used by every solution under general-skills/.
---

# Shared: Confluence Folder Setup

This is the **generic Confluence folder discovery + creation flow** every FDE general-skills solution uses to set up its documentation hierarchy. The calling SKILL passes in a `<solution-folder-name>` (e.g., `Custom Audience Agent`, `Custom Analytics Agent`) and gets back a `parentId` to use for all subsequent page creation.

## Page Naming Convention

**Every Confluence page title in this hierarchy must be suffixed with the customer name** in the format `<Base name> - <Customer Name>` (regular hyphen, single spaces). Confluence enforces unique titles per space, so suffixing is mandatory.

Examples:
- `FDE Solutions - Treasure Bikes`
- `Custom Audience Agent - Treasure Bikes`
- `Audience Agent Requirements - Treasure Bikes`

When matching existing pages by title, accept both bare names (legacy) and suffixed names (current convention).

## Hierarchy

```
Customers (CUST, space ID: 9797636)
├── US/ROWs    (page ID: 44728439)    → customer folders alphabetically
├── Japan      (page ID: 643963288)   → customer folders
└── Korea      (page ID: 1824266587)  → customer folders
    └── <Customer>
        └── FDE Solutions - <Customer>          (or legacy: ML & Analytics Projects - <Customer>)
            └── <solution-folder-name> - <Customer>      ← parentId for all FDE pages
```

## Step 1: Locate the Customer's Confluence Folder

Use one of two methods.

### Method A: Search by Customer Name (Preferred)

Ask: **What is the customer name?**

Then:

```
searchConfluenceUsingCql:
  cloudId: treasure-data.atlassian.net
  cql: space = "CUST" AND type = page AND title ~ "<customer_name>"
  limit: 10
```

Validate hits by checking:
1. Title matches or closely matches the customer name
2. `parentId` is one of the three region pages (`44728439`, `643963288`, `1824266587`) — confirming top-level customer folder

If multiple matches, show user. If no matches, fall back to Method B.

### Method B: User Provides a Page URL or ID

Ask: **Can you paste a link to any page in the customer's Confluence folder?**

Confluence URL formats:
- `https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/<pageId>/Page+Title`
- `https://treasure-data.atlassian.net/wiki/x/<tinyId>` (pass `tinyId` as `pageId`)

If the page itself is the customer folder (parent is a region page), use its ID. Otherwise walk up the tree.

## Step 2: Locate or Create the FDE Solutions Sub-Folder

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <customer_folder_page_id>
  depth: 1
  limit: 50
```

Match (case-insensitive): `FDE Solutions`, `ML & Analytics Solutions`, `ML & Analytics Projects`, `ML Solutions`, `ML Projects`, `Analytics Solutions`, `Analytics Projects`, `FDE`. Customer-name suffixes (e.g., `FDE Solutions - SCI`, `ML & Analytics Projects - SCI`) also match.

If none match, create:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <customer_folder_page_id>
  title: "FDE Solutions - <Customer Name>"   # MUST be suffixed — Confluence enforces unique titles per space
  contentFormat: markdown
  body: "Landing page for Forward Deployed Engineering solutions deployed for <Customer Name>."
```

## Step 3: Locate or Create the Solution Sub-Folder

The calling SKILL provides `<solution-folder-name>`. Search direct children of the FDE Solutions folder:

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <fde_folder_page_id>
  depth: 1
  limit: 50
```

Match titles against the solution name + reasonable variants, with or without the ` - <Customer Name>` suffix. The calling SKILL specifies the variants — e.g., audience-agent matches `Audience Agent`, `Custom Agent`, `Custom Audience Agent`.

If none match:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <fde_folder_page_id>
  title: "<solution-folder-name> - <Customer Name>"   # MUST be suffixed — Confluence enforces unique titles per space
  contentFormat: markdown
  body: "<solution-folder-name> documentation for <Customer Name>."
```

## Output

Save:
- **FDE Solutions folder page ID** (page is titled `FDE Solutions - <Customer Name>`)
- **`<solution-folder-name>` folder page ID** — this is the `parentId` for all subsequent page creation in the engagement (Current Project State, requirements doc, test cases, Phase 6 docs). Page is titled `<solution-folder-name> - <Customer Name>`.

Both go into the Current Project State page (see `current_project_state.md`).
