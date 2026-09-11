# TredIN COMPLETE FINAL

This package contains the two separate frontend deployments plus the Render backend required for real market/API operation.

## Folders
- USER/ — customer frontend; can be deployed as a Render Static Site or Netlify site.
- ADMIN/ — separate admin frontend; can be deployed as a Render Static Site or Netlify site.
- BACKEND/ — Render Core API + TrueData Market Adapter + PostgreSQL.

## Required Render environment
Core API:
- DATABASE_URL (Render PostgreSQL)
- MARKET_ADAPTER_HOSTPORT (Render private-network host:port injected by `fromService`)
- CORS_ORIGIN = comma-separated USER and ADMIN production origins (Render URLs are included by default; existing Netlify origins are retained).
- JWT_ISSUER = tredin
- JWT_SECRET = strong random secret
- ADMIN_USER_ID
- ADMIN_PASSWORD

Market Adapter:
- TRUEDATA_WS_URL
- TRUEDATA_USER
- TRUEDATA_PASSWORD
- TREDIN_SYMBOL_MAP (exact symbols from the subscribed TrueData symbol master)

Do not put TrueData credentials in either USER or ADMIN HTML.

## Important
The package is deployment-configured, but this environment cannot log into or deploy to the user's GitHub/Render/Netlify accounts. Live market data becomes available only after the Render services are deployed and valid TrueData credentials/symbol map are configured.

## Final micro-audit fixes applied (2026-09-11)
- Corrected Render service `rootDir` paths to `BACKEND/core-api` and `BACKEND/market-adapter`.
- Changed Render Core API and Market Adapter builds to `npm ci --omit=dev` so the committed lockfiles are used deterministically.
- Made Core API CORS allow-list environment-driven and support comma-separated origins/subdomain patterns.
- Disabled bundled frontend test credentials in both USER and ADMIN builds; production authentication is backend-authoritative.
- ADMIN portal now enforces authorized admin roles after live login and opens into a dedicated Admin Control Center.
- Added admin views for Users, Orders, KYC, Finance, Risk, Support, Audit and Reconciliation, using server APIs.
- Expanded ADMIN sync to all available admin data endpoints.
- Removed the unreachable duplicate `/positions` route.
- Made finance approval/rejection transaction-safe with PostgreSQL `BEGIN/COMMIT/ROLLBACK` and row locking.
- Refreshed the API contract map so implemented admin endpoints are no longer listed as missing.

## Deployment configuration fixes (2026-09-12)
- Removed the Render `preDeployCommand` from the Free Core API service because Render documents pre-deploy commands as unavailable on Free web services.
- Moved the idempotent database initialization, instrument seed and admin provisioning into the Core API start sequence so the Free deployment can boot with the existing database.
- Added Render SPA fallback rewrites for USER and ADMIN static sites.
- Expanded CORS to include the two Render frontend service origins while retaining the previously configured origins.
- Regenerated `GITHUB_FILE_LIST.txt` from the actual filesystem so `.cjs` engine filenames and package lockfiles are accurately represented.


## Deployment notes — v6 audit hardening
- `CORS_ORIGIN` contains only the two default Render USER/ADMIN origins. The previous GitHub Pages and Netlify placeholders were removed. Add your real Netlify/custom frontend origins to the `tredin-core-live` Render service's `CORS_ORIGIN` environment variable after deployment, comma-separated, with exact origins only.
- `tredin-market-live` is a Render Private Service (`pserv`, Starter plan), not a public Free web service. Core API receives its internal `host:port` through `fromService` as `MARKET_ADAPTER_HOSTPORT`. This avoids routing market traffic over the public internet.
- Free Render web services cannot receive private-network traffic, so making the market adapter private requires a paid Private Service. Do not change it back to Free if private service-to-service routing is required.
