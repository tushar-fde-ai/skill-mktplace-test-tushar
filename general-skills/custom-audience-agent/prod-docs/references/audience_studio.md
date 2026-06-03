---
name: custom-audience-agent-audience-studio
description: |
  Audience-agent specifics for Audience Studio segment pushes — folder names, naming conventions, when to push test segments vs demo segments, and how the agent's :segment: JSON converts into tdx sg YAML. Read alongside ../../../shared/audience_studio_segments.md which holds the generic process.
---

# Custom Audience Agent — Audience Studio Segment Spec

The HOW (process, gates, push mechanics) lives in `../../../shared/audience_studio_segments.md`. This file holds the **audience-agent-specific WHAT** — folder names, demo segment generation rules, test-segment dump details.

## Folders

| Folder name | Created in | Purpose |
|---|---|---|
| `FDE Test - <Customer>` | Phase 3 (Round 1 test run) | Mirror of `:segment:` outputs from `tdx agent test` runs. Internal-only. Round 2 push overwrites Round 1 segments by name (same TC-ID prefix). |
| `FDE Solutions - Audience Agent Examples` | Phase 5 (handoff) | Customer-facing demo segments showing what the agent can produce. Static. |

## Test-Segment Dump (Phase 3 / Phase 4)

Triggered after every `tdx agent test` run that produces `:segment:` output (test categories: Segment Draft Creation in `agent-setup/references/eval.md`).

### Which test cases dump segments

Only test cases that produced a valid `:segment:` JSON in the agent's response. Cases that failed before the agent emitted JSON (e.g., agent refused per guardrail, agent asked for clarification) are skipped — there's nothing to push.

For passing AND failing cases that *did* produce JSON, push both. The folder is for inspection — failures are part of what the engineer needs to see.

### Naming

`[Test] TC-NNN - <one-line prompt summary>`

The summary is the test prompt truncated to ~50 chars. E.g.:
- `[Test] TC-004 - VIP customers no purchase 90d`
- `[Test] TC-007 - Customers with order in last 30d`
- `[Test] TC-009 - Recent visitors who haven't purchased`

### Conversion: `:segment:` JSON → `tdx sg` YAML

The audience agent's `:segment:` output schema maps directly to `tdx sg` rule format — same operator names, same `value` / `right_value` / `unit` shape. Conversion is mechanical:

```yaml
# JSON: { "type": "Value", "attribute": {...}, "operator": "Equal", "right_value": "VIP" }
# →
- type: Value
  attribute:
    name: <attribute group name>
    column: <column>
  operator: Equal
  right_value: "VIP"
```

For nested condition groups (AND/OR blocks), preserve the nesting depth — `tdx sg` accepts the same nesting the agent emits.

If the agent's JSON references a `baseSegmentId` (named-segment reference), translate to a `BaseSegment` rule with the segment ID resolved from earlier `tdx sg list` output. If the segment ID can't be resolved, log it as a known limitation and skip the push for that case (don't fail the whole test run).

### URL recording

Add a new column to the Confluence test cases page (internal only, not on the customer-facing Google Sheet):

| TC-ID | ... | Round 1 Result | Round 2 Result | **Pushed Segment URL** | Notes |

For each TC, paste the console URL returned by `tdx sg push`. Round 2 retests refresh the same column — same URL since segments overwrite by name.

## Demo Segments (Phase 5)

**Generation:** customer-specific, derived from the customer's distilled `business_context.md` and schema. Do NOT use a static default list — the demos should look like the customer's own data.

### Generation procedure

After Phase 5 begins, before authoring the Architecture page:

1. `Read` the customer's `knowledge_bases/business_context.md` — Priority Attributes section, Key Terms section, existing-segment naming conventions.
2. `Read` the customer's local `tdx sg list` output captured in Phase 1 (their existing segment folder structure tells you the kinds of segments they actually use).
3. Propose 4-6 demo segments that:
   - Use the customer's actual attribute / behavior columns
   - Use Key Terms from `business_context.md` (e.g., if they defined `VIP = customer_segment in ('VIP', 'Premier')`, build a demo around that)
   - Mirror their existing folder taxonomy where possible (e.g., if they have `lifecycle/` folder, propose lifecycle-style demos)
   - Cover a mix: at least one attribute-only, one behavior-only, one combined attribute+behavior with time filter

4. **Approval gate** — present the proposed list to the FDE engineer with each segment's name, 1-line description, and the rule-source (which Key Term / which attribute). Wait for approval before generating any YAMLs.

### Naming

`[Demo] <Segment Name>`

Examples (customer-specific — generated, not static):
- `[Demo] VIP customers - last purchase 90+ days`
- `[Demo] Maker Haul cohort - email engaged`
- `[Demo] High-value lapsed (top quintile spend, no visits 60d)`

### Mandatory rules

Every demo segment MUST:
- Use **only** attribute / behavior columns documented in `business_context.md` Priority Attributes (no hallucinated columns)
- Use a **time filter** appropriate to the customer's data freshness (typically last 30 / 60 / 90 days — confirmed via Phase 1 distribution queries)
- Use `TIME WITHIN PAST` with `{value, unit}` format **never** Unix epoch numbers (this mirrors the agent's own time-operator rule from `eval.md` TC-005)
- Be sized > 0 (run a `tdx sg push --dry-run` to estimate; if a proposed demo returns 0 rows, drop it and propose another)

### URL recording

Demo segment URLs go into:
- The **Architecture** Phase 5 page (under "Example segments the agent can produce")
- The **Behavior** Phase 5 page (linked inline next to the Key Terms / Priority Attributes that each demo exercises)
- **Current Project State**: under Artifact URLs, "Demo segments folder" + the per-segment URLs

## Phase Integration

| Phase | Step | What happens |
|---|---|---|
| 3 | Step 6b (new) | After `tdx agent test` Round 1 runs, parse `:segment:` outputs → push to `FDE Test - <Customer>` → record URLs in Confluence test cases page (`Pushed Segment URL` column) |
| 3 | Step 7 | Update Current Project State: test folder created |
| 4 | Step 7 (Round 2 retest) | Same dump, same overwrite-by-name behavior; refresh the URL column on Confluence |
| 4 | Step 7 (iteration loop) | Every iteration retest also refreshes test segments |
| 5 | Step (new) before Architecture page | Generate + push demo segments to `FDE Solutions - Audience Agent Examples`. Three approval gates per shared SKILL. |
| 5 | Architecture / Behavior pages | Reference demo segment URLs as concrete examples |

## Cleanup

- **Test folder** at end of engagement: leave in place. The customer's PS may have many internal folders already; one more named `FDE Test` is fine. If the customer specifically asks for cleanup, delete via `tdx sg delete` per segment (no folder-level delete in `tdx sg`).
- **Demo folder** at end of engagement: leave permanent. It IS the customer's handoff artifact.

## Edge cases

- **Agent refuses to draft (PII guardrail trip):** no `:segment:` JSON, no push. Note the TC-ID in the test cases page Notes column instead.
- **Agent emits invalid JSON:** the test case fails on JSON-schema-valid criteria. Skip the push but log "Invalid :segment: JSON" in the test cases page Notes column.
- **`tdx sg push` itself fails (e.g., bad column reference):** record the error in the Notes column. Do NOT fix the agent's JSON to make it push — the failed push IS evidence the agent's output isn't deployable, which is what the test was checking.
