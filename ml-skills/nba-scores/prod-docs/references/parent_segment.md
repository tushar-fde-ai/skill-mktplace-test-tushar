---
name: nba-scores-parent-segment-update
description: |
  NBA Scores Phase 6 Step 1 — specifies WHAT to add to the Parent Segment and WHAT example audiences to create in Audience Studio. Read by the calling agent alongside shared/parent_segment_update.md which handles the HOW.
---

# NBA Engagement Scores — Parent Segment Update Spec

This file is the solution-specific "what" for Phase 6 Step 1. The process (approval gates, push mechanics, section placement rules) is in `../../../shared/parent_segment_update.md`. Read both files together before making any changes.

---

## Attribute Block

**Block name:** `NBA Engagement Scores`
**Source table:** `<sink_database>.nba_combined_metrics_final`
**Join key:** `<unique_user_id>` on both sides

### Core Scores (always include)

| Column | Type | Label |
|--------|------|-------|
| `next_best_channel` | string | Next Best Channel |
| `next_best_time` | string | Next Best Time |
| `next_best_campaign` | string | Next Best Campaign |

### Campaign Flags (always include)

| Column | Type | Label |
|--------|------|-------|
| `cart_abandon_flag` | number | Cart Abandon Flag |
| `new_visitor_flag` | number | New Visitor Flag |
| `recent_purchase_flag` | number | Recent Purchase Flag |
| `recent_ad_engagement_flag` | number | Recent Ad Engagement Flag |
| `ad_engagement_flag` | number | Ad Engagement Flag |
| `no_ads_or_purchases_flag` | number | No Ads or Purchases Flag |

### Optional — Per-Channel Engagement Scores (add if customer wants segment-level channel scoring)

| Column | Type | Label |
|--------|------|-------|
| `email_engagement_score` | string | Email Engagement Score |
| `online_engagement_score` | string | Online Engagement Score |
| `organic_engagement_score` | string | Organic Engagement Score |
| `mobile_engagement_score` | string | Mobile Engagement Score |
| `social_engagement_score` | string | Social Engagement Score |
| `cpc_engagement_score` | string | CPC Engagement Score |
| `display_engagement_score` | string | Display Engagement Score |

### Optional — Per-Daypart Engagement Scores (add if customer wants daypart-level scoring beyond next_best_time)

| Column | Type | Label |
|--------|------|-------|
| `morning_engagement_score` | string | Morning Engagement Score |
| `afternoon_engagement_score` | string | Afternoon Engagement Score |
| `evening_engagement_score` | string | Evening Engagement Score |
| `overnight_engagement_score` | string | Overnight Engagement Score |

### Optional — Activity Dates (add if customer wants recency signals in Audience Studio)

| Column | Type | Label |
|--------|------|-------|
| `last_purchase_date` | string | Last Purchase Date |
| `last_visit` | string | Last Visit Date |
| `last_cart_add_date` | string | Last Cart Add Date |
| `num_visits` | number | Number of Visits |

> **Default recommendation:** present Core Scores + Campaign Flags to the customer first. Add optional groups only if the customer confirms they need them for segmentation or activation.

---

## Example Audience Folder

**Folder name:** `FDE Solutions - NBA Scores Examples`

Confirm this name with the user before creating any segments (per `shared/parent_segment_update.md` Step 3a).

---

## Example Segments

Present this full list to the user for approval before creating any YAMLs.

### Next Best Channel segments

| Segment name | Filter | Description |
|---|---|---|
| `[NBA] Top Email Channel` | `next_best_channel = 'email'` | Profiles with highest email engagement affinity — ideal for email campaign targeting |

### Next Best Time segments

| Segment name | Filter | Description |
|---|---|---|
| `[NBA] Morning Engagers` | `next_best_time = 'morning'` | Most active 06:00–11:59 — schedule sends in the morning window |
| `[NBA] Afternoon Engagers` | `next_best_time = 'afternoon'` | Most active 12:00–17:59 — schedule sends mid-day |

### Next Best Campaign segments

| Segment name | Filter | Description |
|---|---|---|
| `[NBA] Cart Abandoners` | `cart_abandon_flag = 1` | Added to cart in the lookback window without purchasing — trigger cart-abandon retargeting journey |
| `[NBA] Next Best Product Ready` | `next_best_campaign = 'Next Best Product'` | Profiles flagged for product recommendation — pair with NBP workflow output for personalized product push |

> **Note:** Not all segments will have meaningful size in every deployment. For example, `cart_abandon_flag` depends on the lookback window and traffic volume — confirm counts with the customer before presenting segments as production audiences. Run a quick `COUNT(*)` query per filter against `nba_combined_metrics_final` before finalizing the list.
