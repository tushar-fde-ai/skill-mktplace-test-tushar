# Phase 1: Information Gathering

Collect all information needed to generate the customer-facing documentation for MTA Model.

## Step 1: Locate the Customer's Confluence Folder

The customer documentation lives in the **Customers** Confluence space (CUST), organized by region:

```
Customers (CUST, space ID: 9797636)
├── US/ROWs    (page ID: 44728439)    → customer folders alphabetically
├── Japan      (page ID: 643963288)   → customer folders
└── Korea      (page ID: 1824266587)  → customer folders
```

Ask the user to identify the customer's Confluence folder using **one of two methods**:

### Method A: Search by Customer Name (Preferred)

Ask: **What is the customer name?**

Then search for their folder in the CUST space:

```
searchConfluenceUsingCql:
  cloudId: treasure-data.atlassian.net
  cql: space = "CUST" AND type = page AND title ~ "<customer_name>"
  limit: 10
```

From the results, identify the correct page by checking:
1. The page title matches or closely matches the customer name
2. The page's `parentId` is one of the three region pages (`44728439`, `643963288`, `1824266587`) — confirming it's a top-level customer folder, not a deeply nested subpage

If multiple matches are found, show them to the user and ask which one is correct.
If no matches are found, ask the user to provide a direct link (Method B).

### Method B: User Provides a Page URL or ID

Ask: **Can you paste a link to any page in the customer's Confluence folder?**

Extract the page ID from the URL. Confluence URLs look like:
- `https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/<pageId>/Page+Title`
- `https://treasure-data.atlassian.net/wiki/x/<tinyId>` (tiny link — pass the `tinyId` to `getConfluencePage` as the `pageId`)

Once you have the page ID, read the page to get its `parentId`. If the page itself is the customer folder (i.e., its parent is a region page), use its ID. Otherwise, walk up the tree until you find the customer-level folder.

### Step 1b: Locate or Create the FDE Solutions Sub-Folder

Documentation pages should live under an **FDE Solutions** sub-folder within the customer folder — not directly under the customer root.

**Search for an existing sub-folder**:

Get the direct children of the customer folder and look for a match:

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <customer_folder_page_id>
  depth: 1
  limit: 50
```

Scan the results for a page whose title matches any of these patterns (case-insensitive):
- `FDE Solutions`
- `ML & Analytics Solutions`
- `ML & Analytics Projects`
- `ML Solutions`
- `ML Projects`
- `Analytics Solutions`
- `Analytics Projects`
- `FDE`

Also match titles that include the customer name as a suffix (e.g., `ML & Analytics Projects - SCI`).

If a match is found, use that page's ID.

**If no match is found**, create the sub-folder:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <customer_folder_page_id>
  title: "FDE Solutions"
  contentFormat: markdown
  body: "Landing page for Forward Deployed Engineering solutions deployed for this customer."
```

### Step 1c: Locate or Create the MTA Sub-Folder

Within the ML/FDE sub-folder, create (or find) a folder specific to the MTA project.

**Search for an existing MTA folder**:

Get the direct children of the ML/FDE sub-folder:

```
getConfluencePageDescendants:
  cloudId: treasure-data.atlassian.net
  pageId: <ml_fde_folder_page_id>
  depth: 1
  limit: 50
```

Scan the results for a page whose title matches any of these patterns (case-insensitive):
- `Journey Analysis`
- `MTA`
- `Multi-Touch Attribution`
- `MTA Journey Analytics`

If a match is found, use that page's ID.

**If no match is found**, create the sub-folder:

```
createConfluencePage:
  cloudId: treasure-data.atlassian.net
  spaceId: 9797636
  parentId: <ml_fde_folder_page_id>
  title: "MTA Journey Analytics"
  contentFormat: markdown
  body: "Multi-Touch Attribution journey analytics workflow documentation for this customer."
```

### Store the Folder IDs

Save both:
- **ML/FDE sub-folder page ID**
- **MTA sub-folder page ID** — you'll use this as the `parentId` when creating documentation pages in Phase 5

The final page hierarchy will be:
```
[Customer Folder]
└── FDE Solutions (or ML & Analytics Projects, etc.)
    └── MTA Journey Analytics          ← parentId for Phase 5
        ├── MTA Configuration Summary
        ├── MTA Architecture & Output Schema
        └── MTA Runbook & Maintenance
```

## Step 2: Check for Existing MTA Doc

Ask the user:

> Do you have an existing filled-out customer-facing handoff docs? If yes, paste the Confluence link.

If provided, read the page content using `getConfluencePage` and extract whatever configuration details are available (database, tables, columns, conversion definition, etc.). Use the extracted values to pre-fill later steps, but still validate everything through auto-discovery.

The standard MTA requirements template lives at:
`https://treasure-data.atlassian.net/wiki/spaces/PS/pages/2684977527/MTA+Model+-+Requirements+Gathering+Template`

## Step 3: Initial Questions

Collect answers to these questions before exploring any data. These determine the information you will use in your hand-off.

### 3a: Data Information

Ask: **What is the name of the database where the mta tables live?**

