---
name: shared-parent-segment-update
description: |
  Phase 6 Step 1 — Add NBA/RFM/MTA/NBP score attributes to the customer's Parent Segment and create example audience segments in Audience Studio. Handles the "how" (process, approval gates, push mechanics). The "what" (which attributes and segments to create) comes from the calling solution's prod-docs/references/parent_segment.md file.
---

# Shared: Parent Segment Update & Example Audiences

Run this as the **first step of Phase 6**, after Phase 5 workflow re-run and validation are complete. Do not run before the final workflow output is validated — attributes and example segments must reflect production-ready data.

## Prerequisites

- Phase 5 complete: final `input_params.yml` pushed, workflow re-run, output tables validated
- `sink_database` and `final_nba_metrics_table` (or equivalent) confirmed from Current Project State
- Parent Segment name and ID known (from Current Project State)

---

## Step 1: Present the Plan — Wait for Approval Before Touching Anything

Before making any changes, tell the user what you are about to do:

> I'm about to make the following changes to the **[Parent Segment Name]** Parent Segment:
>
> 1. Add **[Solution Name] score attributes** from `[sink_database].[output_table]` to the Parent Segment
> 2. Create a new Audience Studio folder: **FDE Solutions - [Solution Name] Examples**
> 3. Create **[N] example segments** in that folder
>
> **Shall I proceed?**

Do not read `prod-docs/references/parent_segment.md` yet — wait for explicit user approval first. If the user wants to change the PS, folder name, or scope, incorporate their feedback before continuing.

---

## Step 2: Add Attributes to the Parent Segment

### 2a. Pull the current config

```bash
tdx ps pull <parent_segment_id> --output <local_path>/<ps-name>
```

The config is saved as a flat YAML file (not a directory).

### 2b. Read the solution's attribute spec

Read the calling solution's `prod-docs/references/parent_segment.md`. It specifies:
- Which output table columns to add
- How to group them (attribute block name)
- Column types and labels

### 2c. Present column list to user for confirmation

Show the proposed attribute block grouped by category (e.g. "Core Scores", "Campaign Flags", "Engagement Scores"). Ask:

> Here are the attributes I'll add. Would you like to include all of them, or remove any?

Wait for confirmation before modifying the file.

### 2d. Append the attribute block to the YAML

**Critical rule: append inside `attributes:` — never inside `behaviors:`.**

Verify the section boundaries before editing:

```bash
grep -n "^attributes:\|^behaviors:" <ps-file>
```

The new block must be inserted **before** the `behaviors:` line. Never append to the end of the file without checking which section it falls under.

```yaml
attributes:
  # ... existing attribute blocks ...
  - name: <Solution Name> Scores        # ← insert here, before behaviors:
    source:
      database: <sink_database>
      table: <final_output_table>
    join:
      parent_key: <unique_user_id>
      child_key: <unique_user_id>
    columns:
      - column: <col_name>
        type: string | number
        label: <Human Readable Label>
      # ...

behaviors:
  # ... existing behavior blocks — do not touch ...
```

### 2e. Push

```bash
cd <project_dir> && tdx ps push <ps-name> -y
```

Verify the diff shown by tdx confirms the new block appears under `attributes:` and that no behaviors were modified.

---

## Step 3: Create Example Audience Folder and Segments

### 3a. Confirm folder name

Default folder name: **`FDE Solutions - <Solution Name> Examples`**

Ask the user to confirm or rename before creating anything.

### 3b. Read the solution's example segment spec

Read the calling solution's `prod-docs/references/parent_segment.md` for the list of example segments to create. It specifies segment names, descriptions, filter rules, and which attribute columns each segment uses.

### 3c. Present segment list for approval

Show the full list of planned segments with a one-line description of each. Ask:

> Here are the example segments I'll create in the **[folder name]** folder. Shall I proceed, or would you like to add, remove, or rename any?

Wait for approval before creating any YAML files.

### 3d. Create the folder first — ALWAYS before pushing any segments

**`folder:` in the segment YAML does NOT control folder placement.** Folder assignment is determined entirely by the **directory structure** — the subdirectory the YAML file lives in. The `folder:` field is just metadata.

**Step 1 — Create the folder in Audience Studio:**
```bash
tdx sg folder create "<Parent Segment Name>" "<folder name>"
```

Example:
```bash
tdx sg folder create "Automotive Demo" "FDE Solutions - NBA Scores Examples"
```

**Step 2 — Place segment YAML files inside a subdirectory of the same name:**
```
segments/automotive-demo/
└── FDE Solutions - NBA Scores Examples/   ← subdirectory name must match folder name
    ├── nba_top_email_channel.yml
    ├── nba_morning_engagers.yml
    └── ...
```

The subdirectory name is what `tdx sg push` uses to resolve the folder. The push summary will confirm placement: `FDE Solutions - NBA Scores Examples/[NBA] Top Email Channel`.

### 3e. Create and push segments

Create one YAML file per segment inside the folder subdirectory. The `folder:` field in the YAML is optional metadata — placement is driven by directory structure.

```yaml
name: "[<Solution>] <Segment Name>"
description: "<one-line description>"
rules:
  - type: Value
    attribute:
      name: <Solution Name> Scores
      column: <column_name>
    operator: Equal
    value: <value>
```

Push from inside the subdirectory:
```bash
cd "segments/automotive-demo/FDE Solutions - NBA Scores Examples"
tdx sg push <segment-file>.yml -y
```

Confirm folder placement in the push summary output — it should read `<folder name>/<segment name>`, not just `<segment name>`.

### 3f. Common mistakes to avoid

- **Do not push from the parent segment root directory** — segments land at root regardless of `folder:` in the YAML.
- **Do not rely on the `folder:` YAML field for placement** — it is metadata only; directory structure controls placement.
- **Do not push segments before creating the folder** — the folder must exist server-side before the first push.
- **Do not push segments before the PS attribute block is live** — Audience Studio will reject rules referencing columns not yet in the PS schema.
- **Do not use `behaviors` columns in segment rules** — example segments should only filter on the newly added `attributes` columns.

---

## Step 4: Update Current Project State

After all pushes succeed, update the Current Project State page:

- PS attribute block: added (`<N> columns from <sink_database>.<table>`)
- Example audience folder: `FDE Solutions - <Solution Name> Examples`
- Example segment URLs: list each console URL
- Current phase: Phase 6, Step 1 complete — proceed to documentation pages
