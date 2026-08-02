---
name: Register a domain with OpenSRS
description: Check availability, price a domain, and register it through the OpenSRS reseller API.
api: OpenSRS Domains and TLS/SSL API
docs: https://domains.opensrs.guide/docs/quickstart
operations:
  - lookup (domain)
  - get_price
  - name_suggest
  - sw_register (domain)
  - get (domain)
---

# Register a domain with OpenSRS

OpenSRS is an XML-over-HTTPS protocol, not REST. Every request is an OPS XML
envelope POSTed to the reseller endpoint. Test against the horizon host before
going live.

## Setup
- Live endpoint: `https://rr-n1-tor.opensrs.net:55443`
- Test endpoint: `https://horizon.opensrs.net:55443`
- Headers on every request: `X-Username: <reseller_username>` and
  `X-Signature: md5( md5(xml_body + api_key) + api_key )`.
- The calling server IP must be whitelisted in the Reseller Control Panel
  (`https://manage.opensrs.com`), or you get code 400 "Access denied: invalid IP".

## Steps
1. **Check availability** — send `lookup (domain)` for the requested fqdn.
   Response code `210` = available, `211` = taken.
2. **(Optional) suggest alternatives** — if taken, call `name_suggest` to get
   available similar names across gTLDs/ccTLDs.
3. **Price it** — call `get_price` to confirm the registration cost for the TLD
   and verify the reseller balance covers it (`get_balance`); insufficient funds
   surface as code `440`.
4. **Register** — call `sw_register (domain)`. Use the `handle` parameter:
   `process` completes the order now; `save` leaves it pending for later RSP
   approval. Supply the full `contact_set` (owner/admin/billing/tech) and
   nameservers.
5. **Confirm** — call `get (domain)` to read back status, expiry and nameservers.

## Error and idempotency rules
- Success is `is_success=1` with `response_code=200`; every response carries a
  numeric `response_code` + `response_text` (see `errors/tucows-problem-types.yml`).
- There is no Idempotency-Key. If a duplicate register is in flight you get code
  `486` (entity in processing state) or `437` (concurrent request) — wait a few
  seconds and re-query with `get (domain)` rather than re-sending the register.
- Validation failures return `460`/`465`/`420`; fix the field and resubmit.
- After registration the registrant may need to verify contact info — watch for
  the `registrant_verification_status_change` event and poll
  `get_registrant_verification_status`.
