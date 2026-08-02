---
name: Find expiring domains and renew them
description: List domains nearing expiry and renew them, or update contacts and auto-renew flags.
api: OpenSRS Domains and TLS/SSL API
docs: https://domains.opensrs.guide/docs/get-domain.md
operations:
  - get_domains_by_expiredate
  - get (domain)
  - renew (domain)
  - modify (domain)
  - get_order_info
---

# Find expiring domains and renew them

## Steps
1. **List expiring domains** — call `get_domains_by_expiredate` with a start/end
   expiry date window to get every domain due for renewal. Alternatively use
   `get (domain)` with `type=list` and an expiry range.
2. **Read current state** — `get (domain)` for a specific fqdn to confirm expiry
   year, status and auto-renew flag.
3. **Renew** — call `renew (domain)` with the correct `currentexpirationyear`.
   If the year does not match the registry you get code `541`; if the domain was
   already renewed you get `555` (treat as success, do not retry).
4. **(Optional) manage** — `modify (domain)` changes contacts, nameservers, or
   sets the `auto_renew` flag (optionally across all `affected_domains`).

## Error and idempotency rules
- No Idempotency-Key: a renewal already applied returns `555`; a concurrent
  request returns `437`. Re-query `get (domain)` / `get_order_info` instead of
  blindly retrying.
- `480` = domain not owned by the user or renewals not supported for the TLD.
- Rate/connection limits surface as `300`/`310`/`350`; a connection allows max
  100 commands and lives at most 60 seconds. See `errors/tucows-problem-types.yml`.
