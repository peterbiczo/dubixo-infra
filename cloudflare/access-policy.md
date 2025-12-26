# Cloudflare Access Policy (Staging parity + Service Token testing)

## What this file is
This documents the Cloudflare Zero Trust (Access) configuration for Dubixo **staging** so that:
- Staging is gated by Cloudflare Access for humans/browsers.
- Automated/manual curl verification is possible via a **Service Token**.
- Production (when it exists) should behave the same **without** the Access gate.

## Where to configure
Cloudflare Zero Trust dashboard:
- Access → Applications
- Access → Policies (Reusable policies)
- Access → Service Auth / Service Tokens
- Access → Logs (for debugging decisions)

---

## Current reusable policies (expected)
Access → Policies → Reusable policies

### 1) `dubixo access gate policy`
- Action: **ALLOW**
- Purpose: human access (browser login)
- Rules (include): email / identity-provider based allow-list(s)
- Notes:
  - Do NOT include service tokens in this policy.
  - Keep this policy reusable across staging apps that need human login.

### 2) `meta bypass`
- Action: **BYPASS**
- Purpose: publicly reachable endpoints on staging that must not be gated (only if needed)
- Rules: path-scoped (keep narrow)
- Notes:
  - Only use for endpoints that must be public (e.g. very specific meta endpoints, health checks for external monitors, etc.).
  - Avoid bypassing all of `/api/*` unless you truly want it public.

### 3) `staging service token auth` (required for manual tests)
- Action: **SERVICE AUTH**
- Purpose: allow non-browser calls (curl / CI) through Access gate
- Rules (include): **Service Token** = `staging-curl-tests-*`
- Notes:
  - This must be **Service Auth**, not Allow.
  - Never commit the token secret to git.

---

## Applications (expected)
Access → Applications

### App: `Dubixo – Staging`
- Type: Self-hosted
- Domain: `staging.dubixo.com/*`
- Policies in order (top to bottom):
  1) `staging service token auth` (SERVICE AUTH)  ← allows curl/CI with headers
  2) `dubixo access gate policy` (ALLOW)          ← allows humans after login
  (Optional) deny policy below if you use one

Expected behaviour:
- Unauthenticated browser / curl without token → **302 redirect** to `dubixo.cloudflareaccess.com/...` (Access login)
- Curl with service token headers → origin reachable (HTTP 200 from the app)

### App: `Dubixo Public Meta API` (optional)
- Type: Self-hosted
- Domain: `staging.dubixo.com/api/...` (path-scoped)
- Policy: `meta bypass` (BYPASS)
- Notes:
  - Ensure the path here is tighter than `/api/*` unless you want the entire API public.

### Temporary app used for debugging (should be deleted)
- `Dubixo – Token Test Health` (`staging.dubixo.com/api/health`)
- Used to isolate whether service tokens are accepted.
- Delete after confirming Service Auth works.

---

## Service Token used for manual tests
Access → Service Auth / Service Tokens

- Name: `staging-curl-tests-YYYY-MM`
- Duration: 1 year (smallest available in UI)
- Rotation: rotate at least yearly, and immediately if accidentally exposed.

### Curl header format (do not paste secrets into tickets or git)
- `CF-Access-Client-Id: <client_id>`
- `CF-Access-Client-Secret: <client_secret>`

---

## Manual verification (staging)
### 1) Access blocks unauthorised users (no token)
```bash
HOST="staging.dubixo.com"
curl -sI "https://${HOST}/api/health" | egrep -i 'HTTP/|location'

Service token: staging-curl-tests-2025-12

Duration: 1 year (smallest available in UI)

Rotation: set a calendar reminder ~30 days before expiry