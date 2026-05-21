# Business Context Template

Copy the content below into `knowledge_bases/business_context.md` and fill each section with customer-specific content. Keep it short — the LLM reads this on every request.

---

The actual file content starts here (do not include this line or anything above it):

`---`
`name: business_context`
`---`

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

`## Segment Naming Conventions`
Prefix/suffix patterns the customer uses for segments.
- All customer-facing segments start with `CS_`
- Region code suffix: `_US`, `_JP`, `_KR`

`## Exclusion Rules`
Things the agent should never include by default.
- PII columns: `email`, `phone`, `full_name`
- Test accounts: `email LIKE '%@test.treasuredata.com'`
- Unmarketable: `marketing_opt_in = false`

---

## Rules of thumb

- The file is plain markdown with a YAML frontmatter block (the `---` `name: business_context` `---` lines at the top). Reproduce that frontmatter exactly when authoring `business_context.md`.
- Keep each section to a few lines. This file is loaded on every request.
- Use concrete values (real column names, real segment prefixes) — not generic descriptions.
- Exclusion Rules is the most important section: the agent follows these as hard constraints.
- When in doubt, leave a section empty rather than guess. The user can add later.
