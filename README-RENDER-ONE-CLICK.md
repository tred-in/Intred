# TredIN — easiest Render deployment

1. Push this repository to GitHub.
2. In Render: New → Blueprint → select the GitHub repository.
3. Render reads the root `render.yaml` and creates USER, ADMIN, Core API, Market Adapter and PostgreSQL.
4. During the first Blueprint setup, enter these secrets when prompted:
   - `ADMIN_USER_ID` — your admin login ID
   - `ADMIN_PASSWORD` — minimum 14 characters
   - `TRUEDATA_USER`
   - `TRUEDATA_PASSWORD`
   - `TREDIN_SYMBOL_MAP` — JSON symbol mapping supplied by TrueData
5. Deploy. The Core API initializes the PostgreSQL schema, seeds instruments when `TREDIN_SYMBOL_MAP` is supplied, provisions the `SUPER_ADMIN`, and then starts the API. These initialization steps are idempotent and are intentionally in the Free-compatible start sequence rather than a paid-only pre-deploy hook.
6. Open the generated `tredin-user-live` URL for the user app and `tredin-admin-live` URL for admin.

## Important
Free Render services are for testing/preview. Free web services can sleep, and free Postgres has important availability/backup limitations. For continuous live market operation, upgrade the Core API, Market Adapter and database to paid plans.


## CORS / frontend domains
The Blueprint intentionally contains only the two default Render static-site origins:
- https://tredin-user-live.onrender.com
- https://tredin-admin-live.onrender.com

The old GitHub Pages and Netlify placeholder origins have been removed. After your actual USER/ADMIN frontend deployment is known, open the `tredin-core-live` service in Render → Environment and set `CORS_ORIGIN` to a comma-separated list of the exact browser origins, for example:
`https://your-user-site.netlify.app,https://your-admin-site.netlify.app`
Do not include paths or trailing slashes. If you keep the Render static sites as the production frontends, no CORS change is needed for those two default origins.

## Market Adapter networking
The Market Adapter is configured as a Render Private Service (`tredin-market-live`) on the Starter plan because Free Render web services can send private-network requests but cannot receive them. Core API receives `MARKET_ADAPTER_HOSTPORT` from Render's `fromService` wiring and talks to the adapter over Render's private network. This replaces the old public `.onrender.com` market URL. The private service therefore no longer needs a public endpoint.
