# Weekly outbound review — 2026-09-09 to 2026-09-15 (UTC)

Generated 2026-09-15 ~06:00 UTC from live Instantly workspace (org b1bb6f63-ae14-4fcd-bc6e-85e31ea3f9e3) plus repository inputs. Lead supply/sourcing and sender capacity are reported separately, per request.

## Sources read

- `inputs/prior-capacity-plan-2026-09-01.md` — planning snapshot for 2026-09-14..20
- `inputs/lead-verification-and-sourcing-standard-2026-09-08.md` — eligibility gates and daily sourcing targets
- `inputs/sender-operating-rules-2026-09-05.csv` — 12-alias sender roster with planned caps
- `inputs/campaign-prior-state-2026-09-14.csv` — expected lifecycle/assignment per campaign
- `inputs/correction-queue-2026-09-14.md` — held corrections and known discrepancies
- Live Instantly: full campaign inventory (25 campaigns, cursor paginated to exhaustion), per-campaign analytics for all 20 M-T400 campaigns, account inventory, email/activity listing, sending-status diagnostics

## Activity export

File: `outputs/weekly-activity-2026-09-09_2026-09-15.csv` — **zero data rows**.

Collection method: email/activity listing probed unfiltered, with `email_type=sent`, and per campaign (incl. Prism Analytics Wave 02, flagged in prior state for "legacy activity expected"); all returned zero items with no pagination cursors. Per-campaign analytics for the window are zero across all metrics (sent, contacted, bounced, opened, replied, unsubscribed). Campaign inventory was fully paginated (25 items, cursor exhausted). The required dedupe rule (stable activity/message ID + campaign ID, exceptions preserved) had no rows to apply to. **An empty export here means no observed activity, not uncollected pages.**

## 1. Lead supply / sourcing

Plan/targets (not observations): 620 raw prospects researched, 520 enrichment attempts, 485 verified-and-routable leads per day; eligibility requires verified `valid`/`accept_all` result, source URL, segment, and 24h duplicate/suppression checks.

Observed in live Instantly:

- 18 of 20 M-T400 campaigns hold 3 leads each (54 total); `M-T400-V4 | Helio Systems | Wave 01` and `M-T400-V4 | Prism Analytics | Wave 02` hold **0 leads**. Uploaded leads carry `seed_label`/`source_lifecycle` custom variables, consistent with synthetic seed data rather than production sourcing.
- No verified-lead throughput is observable: with zero sends, the 485/day verified-routable target cannot be confirmed or measured from Instantly.
- Prior-state lifecycle labels do not match live state: campaigns marked "active", "paused", or "completed" in the 2026-09-14 export are all in **draft** in the live workspace.
- Correction-queue item 4 counts (96 records needing manual review, 41 missing source URL, operator note 2026-09-14) remain **repository claims**; no live verifier source was available to reconcile them.
- Credit balance: **unknown** (not zero). No authoritative verifier credit export exists in source artifacts, and none was observable live.

## 2. Sender account capacity

Plan (snapshot 2026-09-01): 13 planned active senders, planned caps 25–45/day by segment, planning total 495/day, with an 80% usable-capacity assumption (explicitly an assumption, not an observation).

Observed in live Instantly:

- **Live sending-account inventory is empty (0 accounts).** Unfiltered list and domain search both returned zero. The correction queue expected a stale roster (legacy-west retired 2026-09-10, helio added 2026-09-13); in fact *no* planned sender exists live, so planned capacity of 495/day is not backed by any live account.
- Sampled campaign configs (e.g., Northstar SaaS Wave 01) contain no `email_list`/account attachments, matching correction-queue item 2 (incomplete assignment links; campaigns may have no sender attached).
- Sending-status diagnostics for 5 campaigns sampled across families (Northstar, Vale, Lumen, Prism) all report `campaign_draft — "Campaign is in draft mode and has not been launched"`, with `last_healthy_send_at: null`.
- Configured maxima across the 18 M-T400-V1 campaigns: `daily_limit` sums to **749/day**, `daily_max_leads` to **644/day** — configuration maxima only. With zero attached accounts and all campaigns in draft, **observed usable sending capacity is 0**.
- Schedules are weekday-only windows (CT/MT/ET, 08:30–16:00 variants), consistent with the "do not send weekends" operating rules; this constrains the window to 5 sending days once launched.

## Plan vs observed summary

| Dimension | Prior plan/target | Observed live (2026-09-09..15) |
|---|---|---|
| Verified-routable leads/day | 485 | Not measurable; 54 seed leads uploaded total, 2 campaigns empty |
| Active senders | 13 planned | 0 live accounts |
| Sending capacity/day | 495 planned (80% usable assumed) | 749/day configured maxima; 0 usable (no accounts, all draft) |
| Emails sent | — | 0 |
| Replies / bounces / opens | — | 0 / 0 / 0 |
| Campaigns in review scope | 20 expected active/paused | 20 present, all draft; 5 unrelated QA/legacy campaigns excluded |

## 3. Prioritized next steps

### Priority call

**Sending readiness comes before lead-sourcing scale-up.** With 0 live sending accounts, all campaigns in draft, and no sender attachments, additional verified leads have no route to send. Increasing sourcing volume now would only grow inventory that cannot be used and would age past the 14-day verification-staleness rework threshold in the sourcing standard. Lead-sourcing work continues in parallel only as *reconciliation* (queue, source URLs, credits), not volume growth.

### Track A — Sending readiness (blocking; do in order)

| # | Action | Prerequisites | Live evidence the owner must verify before marking done |
|---|---|---|---|
| A1 | Re-provision/reconnect sending accounts in the Instantly workspace. Resolve the roster first: confirm legacy-west retirement (2026-09-10) and helio onboarding (2026-09-13) with the mailbox/DNS owner before creating anything. | Domain/DNS ownership confirmed; workspace admin access; migration roster finalized | `LIST_ACCOUNTS` (unfiltered) returns each expected account with `status=1` (Active) and a defined `daily_limit`; legacy-west absent; helio present if onboarding confirmed. Account count and caps reconcile against `sender-operating-rules-2026-09-05.csv` |
| A2 | Attach accounts to campaigns per segment rules (max 2 campaigns per sender; segment restrictions; no weekend senders on weekday-only campaigns). | A1 complete; segment-to-sender matrix approved against operating rules | `GET_CAMPAIGN` per campaign shows a non-empty account/`email_list` attachment; per-account mappings (`GET_ACCOUNT_CAMPAIGN_MAPPINGS`) show no sender on more than 2 campaigns |
| A3 | Launch campaigns (exit draft) in family order, starting with the smallest caps (Health/HR, then Security/Finance, then volume segments) to limit ramp risk. Respect weekday-only schedule windows. | A2 complete; leads present in the campaign (Helio and Prism currently have 0 — decide populate vs defer); schedule dates current | `GET_CAMPAIGN_SENDING_STATUS` no longer reports `campaign_draft`; `last_healthy_send_at` becomes non-null; first send events appear in the activity listing and campaign analytics |
| A4 | Re-baseline capacity from live settings, replacing the 495/day planning figure and the 80% usable assumption with observed values. | A1–A3 complete; at least one full sending day elapsed | Sum of live per-account `daily_limit` values; observed sends/day from analytics after ≥1 sending day; updated figure labeled as observed, not assumed |

### Track B — Lead sourcing reconciliation (parallel, in this order)

| # | Action | Prerequisites | Live evidence the owner must verify |
|---|---|---|---|
| B1 | Establish the verifier **credit balance** first — it gates whether any verification/enrichment can run at all. Current state is *unknown*, not zero. | Access to the verifier workspace (operator-held, per the sourcing standard) | An authoritative balance readout/export from the verifier workspace, dated, attached to this repo or referenced by location |
| B2 | Triage the **96-record manual-review queue** (regulated Health/Finance title/market review; accept-all tagging). Re-verify any record whose verification is older than 14 days before clearing it. | B1 confirms sufficient credits for re-verification | Live queue count from the verifier workspace reconciles with the 96 figure (or replaces it as the corrected count); cleared records show `valid`/`accept_all` results within 14 days |
| B3 | Resolve the **41 records missing a source URL**: backfill the URL, or apply the partner-referral bypass with reviewer initials, or disqualify the record. Records without a source URL are ineligible for upload. | B2 triage started (URL-less records surface during review); reviewer available for bypass sign-off | Each of the 41 records ends in one of three auditable states: URL backfilled, documented referral bypass, or disqualified; none uploaded without one |

Sequencing rationale: credits (B1) determine whether the queue can be worked at all; queue triage (B2) clears the largest block of near-eligible leads and is where URL gaps surface; URL resolution (B3) is the final eligibility gate before any upload. Only after Track A reaches A3 should sourcing volume scale toward the 620/520/485 daily targets.

## Limitations

- Analytics are workspace-reported zeros; draft diagnostics corroborate, but historical activity outside the window was not audited beyond the Prism legacy probe.
- Verifier-side metrics (credit balance, review queue counts) are not observable from available sources; reported as unknown/repository claims respectively.
- `M-T400-V4` campaigns are marked "synthetic source configuration; do not activate" — included in inventory, excluded from capacity totals.
- Sender-to-campaign mapping from the 2026-09-14 export was not used for live conclusions, per correction-queue guidance; no live account IDs exist to resolve aliases against.
