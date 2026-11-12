# Google Ads MCP

Upstream source and credit: [Google Ads MCP](https://github.com/googleads/google-ads-mcp), deployed from [commit cda8527](https://github.com/googleads/google-ads-mcp/tree/cda852701fb5787dc4d79281f37da795ac09b97d). See the [official deployment guide](https://developers.google.com/google-ads/api/docs/developer-toolkit/mcp-server).

## Setup and changes

1. Enable `googleads.googleapis.com` in the hosting project. Cloud Console initially showed Test access. We applied for and verified Explorer access, which allows queries against live accounts.
2. Use the relevant manager account as the default `GOOGLE_ADS_LOGIN_CUSTOMER_ID` (digits only). Reports target a linked client's own customer ID. The Google user who signs in needs access to the relevant Ads accounts.
3. Build the pinned upstream with the Firestore extra (`.[firestore]`) and include the reporting tool configuration below at `/app/tools_config.yaml`. Set `GOOGLE_ADS_MCP_TOOLS_CONFIG=/app/tools_config.yaml`.
4. Deploy `google-ads-mcp` privately in `us-central1` to establish its HTTPS callback address. Use its dedicated runtime identity and the existing Cloud Run builder. Create a separate Ads OAuth web client with `https://<ads-service-host>/auth/callback`, and keep the consent configuration in Testing with approved accounts.
5. Set `GOOGLE_ADS_MCP_BASE_URL` to the HTTPS service origin and configure `GOOGLE_ADS_MCP_OAUTH_CLIENT_ID`. Reference `GOOGLE_ADS_MCP_OAUTH_CLIENT_SECRET` and `GOOGLE_ADS_MCP_JWT_SIGNING_KEY` from separate Secret Manager entries. Replace any bootstrap placeholder settings before exposing the service.
6. Set `GOOGLE_ADS_MCP_STORAGE_TYPE=firestore`, `GOOGLE_ADS_MCP_STORAGE_FIRESTORE_PROJECT=<project-id>` and `GOOGLE_ADS_MCP_STORAGE_FIRESTORE_DATABASE=google-ads-mcp-oauth`. The runtime's database access is scoped to the dedicated Ads OAuth database. The signing key supports stable OAuth state across deployments and the server uses encrypted storage. This named database is billed by usage.
7. Verify OAuth discovery and unauthorized rejection while the service is private. After approving external access, disable the Cloud Run invoker IAM check for this service so Claude can reach the OAuth endpoints. Repeat the checks on the public URL. Discovery returned HTTP 200 and MCP requests with missing or invalid tokens returned HTTP 401.

## Reporting allowlist

```yaml
namespaces:
  customers:
    enabled_tools: [list_accessible_customers]
  search:
    enabled_tools: [search]
  metadata:
    enabled_tools: [get_resource_metadata]
  labels: false
  assets: false
```

Google Ads uses its standard `adwords` OAuth scope. The allowlist makes the MCP reporting-only by exposing account discovery, metadata and search. Google account permissions still determine which accounts a user can read.

## Verification

Local tool discovery returned only the three reporting tools. We also matched the active Cloud Run image digest to the Cloud Build output and verified the allowlist in that build's source archive. The first local OAuth smoke test timed out before user sign-in. The connection was later reported working in Claude. A deliberate restart test was not recorded.
