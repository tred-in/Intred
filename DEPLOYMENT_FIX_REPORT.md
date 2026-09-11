# TredIN Deployment Fix Report — V6

This package is prepared as a clean GitHub-root deployment bundle. No credentials were changed and no GitHub or Render deployment action was performed.

## Fixed in V6
- Removed the extra `tredin_audit_extract/` archive wrapper. ZIP root now directly contains `USER/`, `ADMIN/`, `BACKEND/`, `render.yaml`, and project docs.
- Added `POST /me/profile` to the Core API and wired both USER and ADMIN profile forms to it.
- Added persistent, hashed, rotating refresh-token storage in PostgreSQL (`refresh_tokens`). Access tokens remain short-lived; refresh tokens expire after 30 days and are revoked on rotation/logout.
- Wired USER and ADMIN refresh calls to send the stored refresh token.
- Added `profile_data` storage for profile fields not represented by the base user columns.
- Updated API maps and endpoint test manifests for the new profile endpoint and unauthenticated refresh endpoint.
- Regenerated `GITHUB_FILE_LIST.txt`.

## Deliberate architecture notes
- Core API bootstrap remains in the Render `startCommand` because the configured Core service is on Render Free; `preDeployCommand` is not used. The bootstrap SQL and seed/provision scripts are idempotent.
- `MARKET_ADAPTER_HOSTPORT` is now injected by Render `fromService` wiring to the private Market Adapter service. The previous public `MARKET_ADAPTER_URL` wiring was removed.
- TrueData credentials remain environment-only.

## Validation
- Node syntax checks passed for Core API and Market Adapter.
- JSON parsing passed for both backend package manifests.
- YAML parsing passed for `render.yaml`.
- ZIP root was verified to contain the project directly.
- No GitHub/Render mutation or deployment was performed.


## Re-audit hardening pass
- Removed stale GitHub Pages and Netlify placeholder CORS origins from `render.yaml`.
- Replaced public Market Adapter URL wiring with Render Private Service `fromService` hostport wiring.
- Cleaned redundant `requireAuth()` call in `/me/profile`; the route now uses the single authenticated context already established for all protected routes.
- Added strict, identical `TREDIN_SYMBOL_MAP` validation to Core API seeding and Market Adapter startup: JSON array, required string fields, and unique id/symbol/TrueData symbol.
- Verified all SQL statements used by boot-time DB initialization are idempotent (`CREATE ... IF NOT EXISTS`, `ALTER ... ADD COLUMN IF NOT EXISTS`, and idempotent indexes).
- Removed internal Market Adapter address disclosure from the public Core `/health` response.
- Revalidated JavaScript/CJS syntax, package manifest/lock pairs, YAML, SQL safety patterns, and archive structure.
