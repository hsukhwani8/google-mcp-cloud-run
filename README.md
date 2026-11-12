# Google Ads and GA4 MCP on Cloud Run

This repository documents a deployment of two read-only reporting connections for Claude and other compatible MCP clients. Both services use Streamable HTTP with Google OAuth. They share a Google Cloud project and run as separate Cloud Run services so each can be deployed and rolled back independently.

| Service | Runbook | Official upstream |
| --- | --- | --- |
| GA4 | [GA4.md](GA4.md) | [Google Analytics MCP](https://github.com/googleanalytics/google-analytics-mcp) |
| Google Ads | [GOOGLE-ADS.md](GOOGLE-ADS.md) | [Google Ads MCP](https://github.com/googleads/google-ads-mcp) |

See [OPERATIONS.md](OPERATIONS.md) for the shared deployment sequence, verification, troubleshooting, costs and migration options.

## Source credit

Google maintains the upstream server code. This project covers deployment, OAuth, access controls, verification and operating notes. The GA4 deployment also includes a local OAuth persistence change described in its runbook. The repository contains runbooks and configuration examples. Retrieve the pinned server code and any local deployment changes from the sources described in the runbooks.

## Connect Claude

1. In Claude's remote connector settings, add the relevant Cloud Run service URL with `/mcp` appended. Use Streamable HTTP with OAuth where the client asks.
2. Select a Google account on the Cloud project's OAuth tester list that already has the required GA4 property or Google Ads account access.
3. Start with account or property discovery and then run a report.

Use the Cloud Run console to locate each service URL. Keep OAuth credentials, keys, tokens, private account details and report output out of GitHub.
