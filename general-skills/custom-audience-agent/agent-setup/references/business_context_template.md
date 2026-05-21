# Business Context Template

`business_context.md` is the customer-specific knowledge base the audience agent reads on every request. It's authored in **two stages**:

1. **Phase 4 (schema-derived draft):** the FDE engineer (LLM) writes the schema-derivable sections from `tdx ps desc -o` output — Priority Attributes and Exclusion Rules → PII columns. Other sections stay as empty placeholders.
2. **Phase 5 (customer merge):** when the customer's filled requirements doc returns, customer answers merge into the draft — customer wins on conflict; schema defaults fill gaps.

## Field-by-field: schema-derivable vs customer-required

| Section | Schema-derivable in Phase 4? | Source in Phase 5 |
|---|---|---|
| Business Model | ❌ | Customer requirements §1 |
| Key Terms | ❌ | Customer requirements §3 |
| Priority Attributes | ✅ Top ~10 columns by inferred usefulness, with descriptions inferred from column names | Customer §4 (overrides + adds) |
| Segment Naming Conventions | ❌ | Customer requirements §5 |
| Exclusion Rules → PII columns | ✅ Auto-flagged by name pattern (`email`, `phone`, `mobile`, `ssn`, `dob`, `birth`, `name`, `address`, `zip`, `postal`) | Customer §6 (overrides + adds) |
| Exclusion Rules → Customer filters | ❌ | Customer requirements §6 |
| Exclusion Rules → Hidden behavior tables | ❌ | Customer requirements §6 |

When inferring Priority Attribute descriptions: if the column name is cryptic and you can't be confident in its meaning, write `<TODO: confirm with customer>` rather than guess. The customer can clarify in §4 of the requirements doc.

## The actual file content

The file is plain markdown with a YAML frontmatter block. Reproduce the frontmatter exactly:

```
---
name: business_context
---
```

Then sections (verbatim — do NOT wrap this template content in code fences when writing the actual file):

`## Business Model`
1-3 sentences on what the customer sells, channels, and key customer types.

`## Key Terms`
Customer-specific jargon the agent should recognize.
- "VIP" = Tier 3+ loyalty members
- "churned" = no purchase in 180 days

`## Priority Attributes`
5-10 most important columns on the `customers` table and what they mean (especially cryptic names).
- `lifetime_value_usd` — total revenue across all orders
- `signup_channel` — values: `web`, `app`, `referral`, `partner`
- `last_seen_ts` — `<TODO: confirm with customer>` — likely last login/visit timestamp

`## Segment Naming Conventions`
Prefix/suffix patterns the customer uses for segments.
- All customer-facing segments start with `CS_`
- Region code suffix: `_US`, `_JP`, `_KR`

`## Exclusion Rules`
Things the agent should never include by default.
- PII columns: `email`, `phone`, `full_name`
- Test accounts: `email LIKE '%@test.treasuredata.com'`
- Unmarketable: `marketing_opt_in = false`

## Phase 4 — Writing the schema-derived draft

After running `tdx ps desc <parent_segment> -o`:

1. **Priority Attributes:** scan the `customers` table columns. Pick the top ~10 by likely usefulness — monetary fields (`*_value_*`, `*_revenue`, `*_amount`), dates (`*_ts`, `*_at`, `*_date`), channel/source enums (`*_channel`, `*_source`, `*_origin`), status enums, demographic non-PII (age, gender, region, country, language, segment tier). Skip obvious PII. For each, write a 1-line inferred description; mark uncertain ones `<TODO: confirm with customer>`.
2. **Exclusion Rules → PII columns:** auto-flag any column matching the PII name patterns above. Conservative bias — false positives cost less than false negatives.
3. **All other sections:** write the section header + a short placeholder line like `_To be filled in from customer requirements doc §X._`

Present the draft to the FDE engineer for review (the **confirmation gate** in Phase 4 Step 3 of the parent SKILL). The engineer can edit any obviously wrong inferences before push.

## Phase 5 — Merge rule

When the customer's filled requirements doc returns, merge section-by-section:

| Customer field state | Action |
|---|---|
| Empty / blank | Keep the schema-derived draft as-is |
| Customer wrote a replacement (e.g., new Priority Attributes list) | Customer wins — replace the draft section entirely |
| Customer added items (e.g., one extra PII column the schema didn't catch) | Append the customer's additions to the existing draft list |

After merge, the file is **customer-validated**. Push and run Round 2 tests.

## Rules of thumb

- Keep each section to a few lines. The file is loaded on every request.
- Use concrete values (real column names, real segment prefixes) — not generic descriptions.
- Exclusion Rules is the most important section: the agent follows these as hard constraints.
- `<TODO: confirm with customer>` is acceptable in Phase 4 drafts; it should be resolved by Phase 5.
- The schema sample data the LLM reads (real customer records) must NOT end up in `business_context.md`. Only column names + inferred descriptions go in.
