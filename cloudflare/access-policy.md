# Cloudflare Access Policy (Staging only)

## Notes
This document describes Cloudflare Access configuration to ensure:
- **Staging is gated** by Cloudflare Access.
- **Production is NOT gated** by Access.
- Staging and prod otherwise behave the same (routing, caching, etc).

This file contains:
- Where to click in Cloudflare Zero Trust
- Recommended app + policy structure
- Manual tests to prove staging is gated and prod is not

## Where to configure in Cloudflare UI
Cloudflare **Zero Trust** dashboard (account-level):
1) Open **Zero Trust** (Cloudflare One)
2) Go to **Access → Applications**
3) Configure the staging application

Reference: Cloudflare Access policies docs.

## Application: `dubixo-staging`
**Type:** Self-hosted application  
**Domain:** `staging.dubixo.com`  
(Optionally include additional staging hostnames if you have them, but keep prod out.)

### Policies (in order)
1) **Allow** — your allowlist
   - Include: specific email addresses and/or email domains you trust
   - Auth method: whatever you use (Google / OTP / etc)
2) **Deny** — everyone else
   - Deny: `Everyone`

## Important: webhooks / machine access (Stripe etc.)
If you have endpoints that must be callable by Stripe or other services on staging, Access can block them.
Two common approaches:
- Add a **Bypass** policy for specific paths that must be publicly reachable (narrow scope).
- Or use **Service Tokens** for machine-to-machine access (preferred when possible).

Use path-based policies if needed (Cloudflare supports app paths / policy inheritance).

## Manual verification (copy/paste)
See the project ticket “INF-003 manual tests” section in the main runbook / ticket comments.

Service token: staging-curl-tests-2025-12

Duration: 1 year (smallest available in UI)

Rotation: set a calendar reminder ~30 days before expiry