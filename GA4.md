# Google Analytics 4 MCP

Upstream source and credit: [Google Analytics MCP](https://github.com/googleanalytics/google-analytics-mcp), based on [commit 8ce1c8f](https://github.com/googleanalytics/google-analytics-mcp/tree/8ce1c8fd0d23375ca02787b3c31b21fccc4238f3). See the [official setup guide](https://developers.google.com/analytics/devguides/MCP).

## Setup and changes

1. Choose a Google Cloud project and enable billing for Cloud Run.
2. Enable the Analytics Admin and Analytics Data APIs in the deployment project.
3. Configure a Google OAuth web client with the Cloud Run callback and add approved reporting accounts as testers. The reporting login needs GA4 property access independently of the hosting account. Use `https://www.googleapis.com/auth/analytics.readonly`.
4. Store the OAuth client secret in Secret Manager. Deploy with a dedicated builder and a dedicated runtime service account. Grant the runtime access to its secret and the Firestore access needed by the OAuth persistence layer.
5. Deploy the source to `ga4-mcp` in `us-central1`. Set `ANALYTICS_MCP_ALLOW_WRITE=false`, `ANALYTICS_MCP_OAUTH=true` and the service base URL as `ANALYTICS_MCP_ISSUER_URL`. Supply `GOOGLE_OAUTH_CLIENT_ID` and reference `GOOGLE_OAUTH_CLIENT_SECRET` from Secret Manager.
6. The first version held OAuth state in memory. A later deployment added `ANALYTICS_MCP_OAUTH_STORE=firestore`. Its deployed `analytics_mcp/oauth_store.py` stores client registrations and refresh-token digests. It does not store raw Google refresh tokens in Firestore. Uncached access tokens are checked with Google, including expiry and client audience. The authorization handshake still uses in-memory state while it is in progress. The deployed code limits OAuth redirects to approved client callbacks and loopback.
7. With OAuth in place, allow external clients to reach the URL and confirm anonymous calls still fail. The original test returned HTTP 401 for unauthenticated MCP calls and HTTP 200 for OAuth discovery.

## Verification and rebuilding

A temporary client signed in, discovered 10 read-only tools, listed accessible properties and ran a seven-day sessions-by-channel report. We reviewed the persistence deployment separately. A deliberate restart test was not recorded.

The upstream commit is the base, not the full deployed source. To reproduce the persistence changes, retrieve the exact Cloud Build source archive for the deployed revision from the project's Cloud Run build details. Review the OAuth and Firestore differences against the pinned upstream before rebuilding.
