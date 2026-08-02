---
name: Order an SSL/TLS certificate
description: Order and retrieve an SSL/TLS certificate through the OpenSRS Trust Service (SSL) commands.
api: OpenSRS Domains and TLS/SSL API
docs: https://domains.opensrs.guide/docs/sw_register-trust_service.md
operations:
  - get_capabilities
  - create_token
  - sw_register (trust_service)
  - get_order_info
  - get_cert
---

# Order an SSL/TLS certificate

Uses the OpenSRS SSL Service (Trust Service) command family. Same OPS
XML-over-HTTPS transport and `X-Username` / `X-Signature` auth as the domain
commands.

## Steps
1. **Check product support** — call `get_capabilities` to confirm which
   certificate products/authorities (DigiCert, Actalis, etc.) are enabled for
   the reseller.
2. **(If required) create a token** — `create_token` to establish the order
   context for the certificate flow.
3. **Place the order** — `sw_register (trust_service)` to initiate the
   certificate order. Set the `handle` parameter: `save` keeps the order pending
   for RSP approval; `process` submits it to the certificate authority.
4. **Track it** — `get_order_info` to follow the order through validation and
   issuance.
5. **Retrieve the certificate** — once issued, `get_cert` returns the issued
   certificate for installation.

## Error rules
- Insufficient balance for a paid certificate returns code `440`.
- Permission problems (product not enabled for the reseller) return `435`.
- Order/registry communication problems surface in the 7xx range (`702`/`705`);
  `705` means "timed out — resubmit". See `errors/tucows-problem-types.yml`.
