---
name: shared-audience-studio-segments
description: |
  Reusable HOW for any general-skills solution that pushes segments into Audience Studio. Covers `tdx sg push` mechanics, folder auto-creation, naming conventions, approval gates, and ordering rules. The WHAT (folder names, segment specs, attribute references) comes from the calling solution's `prod-docs/references/audience_studio.md`. Used in two contexts: dumping test-run :segment: outputs (Phase 3/4) and creating customer-facing demo segments (Phase 5).
---

# Shared: Audience Studio Segment Pushes

Reusable process for pushing segments to Audience Studio from a general-skills solution. Two contexts use it:

1. **Test segment dumps** (Phase 3 / Phase 4) — every `:segment:` output from `tdx agent test` runs gets pushed to a `FDE Test - <Customer>` folder so the FDE engineer can inspect them visually.
2. **Demo segments** (Phase 5) — customer-facing example segments dropped into `FDE Solutions - <Solution> Examples` for the handoff.

The HOW lives here. The WHAT (folder names, naming conventions for segments, which attribute columns each demo uses) lives in the calling solution's `prod-docs/references/audience_studio.md`.

## Skills to Load

- `tdx-skills:segment` — for `tdx sg push` syntax, YAML rule format, and operator reference
- `tdx-skills:tdx-basic` — for `tdx use` context-setting if needed

## Two Folders, Two Lifetimes

| Folder | Created in | Purpose | Lifetime |
|---|---|---|---|
| `FDE Test - <Customer>` | Phase 3 (Round 1) | Mirror of test-run `:segment:` outputs. Engineer-internal. | Round 2 push **overwrites Round 1 segments by name** — folder always reflects the latest run. |
| `FDE Solutions - <Solution> Examples` | Phase 5 | Customer-facing demos. Handoff artifact, linked from Architecture page. | Permanent. |

## Approval Gates

Before any push touches Audience Studio, the LLM presents a plan and waits for explicit approval. There are **three** gate moments:

1. **Folder gate** — confirm folder name before any `tdx sg push` runs.
2. **Segment list gate** — show the proposed segment list (names + filter rules + 1-line description) before creating any YAML.
3. **Re-push gate** — for test segments on Round 2, present the diff (what's being overwritten) before executing.

Wait for explicit approval at each gate.

## Naming Conventions

| Context | Pattern | Example |
|---|---|---|
| Test segments | `[Test] TC-NNN - <one-line prompt summary>` | `[Test] TC-004 - VIP customers no purchase 90d` |
| Demo segments | `[Demo] <Segment Name>` | `[Demo] Recent purchasers (last 30 days)` |

The TC-ID prefix on test segments lets the engineer map back to the test case.

## Push Mechanics

### Step 1: Generate the YAML

One file per segment. Set `folder:` to the target folder name — Audience Studio creates the folder automatically on the first push if it doesn't already exist.

```yaml
name: "[Test] TC-004 - VIP customers no purchase 90d"
description: "Generated from agent's :segment: output during Round 1 test run on <date>"
folder: "FDE Test - <Customer>"
rules:
  # ... rules block converted from the agent's :segment: JSON output
```

For test segments: parse the agent's `:segment:` JSON from the test-run output, convert to `tdx sg`-compatible YAML rules. The rule block maps 1:1 from the agent's JSON conditions.

For demo segments: the calling solution's `audience_studio.md` specifies the rules per demo segment.

### Step 2: Push

```bash
tdx sg push <segment-file>.yml -y
```

Collect the console URL returned after each push.

### Step 3: Test segments — overwrite-by-name on Round 2

Round 2 retests push the same TC-ID-prefixed segment names. `tdx sg push` updates an existing segment if a segment with the same `name` exists in the folder, so the folder always reflects the latest run. No cleanup needed between rounds.

If the agent's `:segment:` output for a TC changes meaningfully between Round 1 and Round 2 (different rules, different size), the segment in AS updates accordingly — the engineer sees the latest run.

### Step 4: Iteration retests during Round 2

Each `tdx agent push -y` → `tdx agent test` cycle in the Phase 4 iteration loop also updates the test segments. Same overwrite-by-name behavior.

## Common Mistakes to Avoid

- **Don't push segments before the referenced attributes / behaviors are live in the parent segment.** Audience Studio rejects rules referencing columns not in the PS schema. (Audience-agent doesn't add new columns, but if a future solution does, that solution's prerequisites apply.)
- **Don't create the folder manually** — the `folder:` field handles it on first push.
- **Don't skip the approval gates** — folder name, segment list, and (Round 2) overwrite preview must each be confirmed.
- **Don't share the test folder URL with the customer** — it contains test artifacts including failures. Only the demo folder is customer-facing.

## URL Recording

Where to record the resulting segment URLs:

| Context | Location |
|---|---|
| Test segments (per TC-ID) | New `Pushed Segment URL` column on the Confluence test cases page (internal-only) — **NOT** on the customer-facing Google Sheet |
| Demo segments | Architecture / Behavior Confluence pages (Phase 5), linked inline as concrete examples |

Update **Current Project State** with the test folder name, demo folder name, and demo segment URLs after Phase 5.

## Reference Index

| Need | Read |
|---|---|
| Process (this file) | `general-skills/shared/audience_studio_segments.md` |
| Audience-agent specifics (folder names, demo seg list, rule patterns) | `general-skills/custom-audience-agent/prod-docs/references/audience_studio.md` |
| `tdx sg push` syntax + operator reference | `tdx-skills:segment` |
| `:segment:` JSON → `tdx sg` YAML conversion | The audience-agent's prompt + the test runner's output parser |
