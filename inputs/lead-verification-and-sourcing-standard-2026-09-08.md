# Lead verification and sourcing standard (effective 2026-09-08)

A lead is eligible for campaign upload only when all are true: business email present; role and company captured; ICP segment assigned; source URL retained; email verifier result is `valid` or `accept_all` with risk flag; duplicate check against active campaign exports completed within 24h; suppression/unsubscribe check completed before upload.

Daily operating targets (targets, not observed): 620 raw prospects researched, 520 enrichment attempts, 485 verified-and-routable leads. Rework is required for `unknown`, bounced, missing source URL, missing company, or stale verification older than 14 days.

Exception handling: regulated Health and Finance requires manual title/market review; accept-all domains must be separately tagged; partner referrals can bypass source URL only with reviewer initials. Credits are checked in the verifier workspace by an operator; no credit-balance export was retained in this repository.
