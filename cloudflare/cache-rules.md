# Cloudflare Cache Rules (Dubixo)

## Notes
This document describes the Cloudflare **Cache Rules** we expect for Dubixo so that:
- `/api/*` is never cached (avoid stale JSON/auth issues).
- Next.js static assets are cached aggressively (performance).
- Staging and production behave the same.

This file contains:
- Required rules (what, where, why)
- Exact Cloudflare UI clicks
- Manual verification commands (curl)

## Where to configure in Cloudflare UI
Cloudflare Dashboard (zone-level):
1) Select the `dubixo.com` zone
2) Go to **Caching → Cache Rules**
3) Create / edit rules listed below

Reference: Cloudflare “Create a rule in the dashboard” (Cache Rules).

## Rule 1 — Bypass cache for API
**Name:** `bypass-api-cache`  
**When:** `URI Path` **starts with** `/api/`  
**Then:** **Bypass cache**

**Why:** APIs must not serve cached responses. This also avoids Cloudflare caching status-code responses by default behaviour.

### Expected outcome (headers)
- `CF-Cache-Status: BYPASS` (or `DYNAMIC`)
- No meaningful `Age` header

## Rule 2 — Cache Next.js static assets (immutable behaviour)
**Name:** `cache-next-static`  
**When (either):**
- `URI Path` **starts with** `/_next/static/`
- `URI Path` **starts with** `/admin/_next/static/`  (admin lives under `/admin`)

**Then (recommended):**
- Set cache eligibility to **Eligible for cache**
- Set **Edge TTL** to something long (e.g. 1 year) OR “Respect origin” if you already send `Cache-Control: public, max-age=31536000, immutable` from Next/Nginx
- Do not cache HTML pages via this rule (path match is static only)

**Why:** Next chunk files are content-hashed. Long TTL is safe and should behave “immutable”.

### Expected outcome (headers)
First request typically:
- `CF-Cache-Status: MISS`

Second request (or after some traffic):
- `CF-Cache-Status: HIT`
- `Age: <seconds>` increases over time
- Often `Cache-Control: public, max-age=31536000, immutable` (depends on origin)

## Purge behaviour after changing rules
After saving rules:
- Go to **Caching → Configuration → Purge Cache**
- Purge relevant paths if needed (e.g. `/_next/static/*`), or purge everything for staging if you’re unsure.

## Manual verification (copy/paste)
See the project ticket “INF-003 manual tests” section in the main runbook / ticket comments.