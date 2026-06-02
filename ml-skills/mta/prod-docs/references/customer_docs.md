# Phase 1: Information Gathering

Collect all information needed to generate the customer-facing documentation for MTA Model.

## Step 1: Confluence Folder Setup

Read `../../../shared/confluence_folder_setup.md` for the full folder discovery and creation flow. Pass these MTA-specific values:

- **Solution folder name:** `MTA Journey Analytics`
- **Title variants for fuzzy matching:** `MTA`, `Journey Analysis`, `Multi-Touch Attribution`, `MTA Journey Analytics`

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

