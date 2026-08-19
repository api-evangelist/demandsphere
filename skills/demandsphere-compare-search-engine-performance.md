---
name: Compare search engine performance
description: Compare a site's ranking performance across search engines and markets using the DemandSphere API v5.0, then summarize the engine-level picture for a date range.
api: openapi/demandsphere-keywords-api-openapi.yml
operations:
- Sites_PropertiesList
- Keywords_SearchEngines
- SearchEngines_Summary
---

# Compare search engine performance

Operating instructions for using the DemandSphere API v5.0 to compare how a site
performs across the 200+ market/engine combinations DemandSphere tracks.

## Auth & conventions
- Base URL: `https://api.demandsphere.com`
- Every request carries the `api_key` query parameter (global security scheme `ApiKey`, `in: query`).
  The developer page advertises "OAuth 2.0", but the contract is an API key — see
  `authentication/demandsphere-authentication.yml`.
- The key travels in the query string, so strip query strings from any proxy or
  CDN access log that sits in front of these calls.
- All operations are `POST .../list` with parameters passed in the query string (no request body).
- `From` and `To` are required, `YYYY-MM-DD`.
- Pagination is offset-based: `Limit` (default 25), `Offset`, `PageNum`.
- Errors: `400` is the only declared failure, but `401`, `403`, `404`, `429` and
  `5xx` all occur — retry only `429` and `5xx`. See `errors/demandsphere-problem-types.yml`.

## Steps
1. **Resolve the site** — call `Sites_PropertiesList` to list tracked properties and
   take the `SiteGlobalKey` of the target site.
2. **Pull per-engine keyword data** — call `Keywords_SearchEngines` with
   `SiteGlobalKey`, `From`, `To`, `Granularity` (`daily|weekly|monthly`), `Order`,
   and a `SearchEngines` list (e.g. `google_us`, `bing_uk`, `yandex_ru`). Page with
   `Limit`/`Offset` until the result set is exhausted.
3. **Summarize by engine** — call `SearchEngines_Summary` over the same site and
   date scope to get the engine-level rollup rather than per-keyword rows. Use this
   for the headline comparison; use step 2 for the drill-down.
4. **Export if the result set is large** — set `Export=true` and `Format=csv` (or
   `xlsx`) rather than paging thousands of rows.

## Notes for agents
- Through the DemandSphere MCP server both of these operations are reachable only
  as views of one tool: `serp_analytics(view="engine_comparison")` maps to
  `Keywords_SearchEngines` and `serp_analytics(view="engine_summary")` maps to
  `SearchEngines_Summary`. See `mcp/demandsphere-tool-crosswalk.yml`.
- Keep date ranges under a year; the first-party client rejects ranges over 365 days.
- There are no idempotency keys, and these are read queries — a retry is safe.
