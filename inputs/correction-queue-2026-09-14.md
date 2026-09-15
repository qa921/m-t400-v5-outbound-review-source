# Held corrections / known discrepancies (updated 2026-09-14)

These are investigation inputs, not resolutions.

1. **Sender roster is stale:** `legacy-west@outreach.example` was reportedly retired on 2026-09-10. Do not count its planned cap unless it appears in live Instantly account inventory. `helio@outreach.example` was reportedly added on 2026-09-13; confirm live status/cap before counting.
2. **Campaign assignment links are incomplete:** campaign export was created before the sender migration. The mapping below is expected, but live `email_list` / account records override it. Several campaigns may have no sender attachment.
3. **Plan vs settings gap:** the 2026-09-01 plan uses an 80% usable-capacity assumption; it is not an observed send volume. Live campaign `daily_limit` and `daily_max_leads` are configuration maxima only.
4. **Lead pipeline lag:** verifier queue shows 96 records needing manual review and 41 records missing a source URL in an operator note dated 2026-09-14. These counts were not independently reconciled and must be labeled as repository claims unless a live source is available.
5. **Credit balance unknown:** no authoritative real-time verifier credit balance is available in source artifacts. State this as unknown, not zero.
6. **Activity export:** some prior exports used timestamp + email for dedupe. The weekly CSV must instead dedupe using stable activity/message IDs together with campaign ID; preserve events with absent stable IDs as exceptions.

## Expected campaign-to-sender matrix (prior state, intentionally incomplete)

| campaign family | expected sender aliases | source state |
|---|---|---|
| Northstar / Orbit | sender-01, sender-02, sender-03 | expected active; verify attachment |
| Atlas / Ember | sender-07, sender-10 | risk-managed; verify warmup |
| Meridian / Lumen | sender-08, sender-09 | regulated; Lumen expected paused |
| Vale / Canyon / Harbor | sender-04, sender-05, sender-06 | weekday only |
| Prism / Helio | sender-10, sender-11 | migration candidates |

The source file contains no confirmed live account IDs. Resolve only through Instantly account inventory; do not infer IDs from aliases.
