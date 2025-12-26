# Cloudflare Cache Rules (Dubixo)

## What this file is
This documents the Cloudflare cache behaviour we require:
- `/api/*` is **not cached**
- `/_next/static/*` and `/admin/_next/static/*` are **cached aggressively** (immutable)
- Staging and production should behave the same for caching.

---

## Where to configure
Cloudflare dashboard (zone):
Caching → Cache Rules

After changes:
Caching → Configuration → Purge Cache (if needed)

---

## Required behaviours

### A) API must not be cached
Scope:
- `/api/*` (including `/api/health`)

Expected response characteristics:
- `CF-Cache-Status: DYNAMIC` (or BYPASS)
- No cached Age/HIT behaviour

Manual proof (staging, with service token headers if Access is enabled):
```bash
HOST="staging.dubixo.com"
curl -ksSI \
  -H "CF-Access-Client-Id: ${CF_ACCESS_CLIENT_ID}" \
  -H "CF-Access-Client-Secret: ${CF_ACCESS_CLIENT_SECRET}" \
  "https://${HOST}/api/health" \
| egrep -i 'HTTP/|cf-cache-status|cache-control|age|cf-ray'