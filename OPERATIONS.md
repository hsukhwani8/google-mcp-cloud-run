# Operating the Google MCP services

## Deployment and changes

- Example region: `us-central1`. Both services use request-based billing, zero minimum instances, one maximum instance, 1 vCPU and 512 MiB.
- Preserve separate runtime identities and secrets. Keep the shared OAuth project in Testing for the pilot and add only approved users who already have access to the relevant data. For each new service hostname, update that service's registered OAuth callbacks and client configuration.
- Before rollout, record the working revision and image digest, build the next revision from pinned source and verify its read-only configuration. Test OAuth discovery, rejection of missing and invalid tokens, tool discovery and an authenticated report through Claude. When feasible, test OAuth after a restart.
- For rollback, route traffic back to the previously verified revision and check that its OAuth configuration and secret versions still work. Cloud Run build details identify the source archive and the image digest. Avoid routing traffic to a bootstrap revision.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Browser shows HTTP 401 at `/mcp` | Open the connection from the AI client's MCP settings. OAuth discovery should respond and the client should initiate sign-in. |
| Google sign-in is refused | Confirm the Google account is on the project's OAuth tester list and the registered callback matches the deployment. |
| Sign-in succeeds but accounts are missing | Check GA4 property roles or Ads manager and customer permissions. A Cloud project role does not grant reporting access. |
| Sign-in is required again | Check for expired Google authorization in Testing mode or an interrupted login. Firestore persists supported OAuth state across Cloud Run restarts. |

## Estimating costs

Track request counts and billable instance time separately. For example, a 1 vCPU service averaging three hours of billable time per day would use around 90 vCPU-hours in a 30-day month. Compare both CPU time and memory usage with the billing account's remaining Cloud Run free allowance. Long-running requests can drive compute costs even with a low request count and zero minimum instances.

Also check Artifact Registry image storage, Firestore operations and storage, and Secret Manager usage. A separate named Firestore database can isolate OAuth state but does not get the database free quota.

See [Cloud Run pricing](https://cloud.google.com/run/pricing), [Firestore pricing](https://cloud.google.com/firestore/pricing) and [Artifact Registry pricing](https://cloud.google.com/artifact-registry/pricing).

## Hosting options discussed

- Move the services to a shared Google Cloud project. Recreate or migrate the API access, OAuth, secrets, identities and deployment settings, then validate the new URLs in Claude before retiring the current deployments.
- Host on a spare office laptop. It must stay awake and connected, and the services should start after reboot. Use a private network URL for local MCP clients or an HTTPS endpoint reachable by Claude's cloud connector with Google OAuth still protecting reports. Start with the service that uses the most billable runtime.
