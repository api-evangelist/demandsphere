---
name: Audit landing pages against ranking keywords
description: Find which pages of a site rank for which keywords and how those landing pages have changed over time, using the DemandSphere API v5.0.
api: openapi/demandsphere-pages-api-openapi.yml
operations:
- Sites_HierarchyList
- Sites_PropertiesList
- Keywords_LandingMatches
- Pages_LandingsHistory
---

# Audit landing pages against ranking keywords

Operating instructions for using the DemandSphere API v5.0 to audit a site's
landing pages: which URLs the search engines actually surface for tracked
keywords, and how that has shifted.

## Auth & conventions
- Base URL: `https://api.demandsphere.com`
- Every request carries the `api_key` query parameter.
- All operations are `POST .../list`; parameters go in the query string.
- `From` / `To` (`YYYY-MM-DD`) are required; `Granularity` is `daily|weekly|monthly`.
- Pagination: `Limit` (default 25), `Offset`, `PageNum`.
- No rate limit is published ("no artificial rate limits"), but `429` is returned
  and is retryable. See `rate-limits/demandsphere-rate-limits.yml`.

## Steps
1. **Resolve the site** — call `Sites_PropertiesList` for the `SiteGlobalKey`. If
   you need the section/subdomain structure of the property rather than a flat
   list, call `Sites_HierarchyList` instead.
2. **Match keywords to pages** — call `Keywords_LandingMatches` with
   `SiteGlobalKey`, the date range, `SearchEngines`, and `SortByLandingMatches`.
   Each row ties a tracked keyword to the landing page currently ranking for it.
3. **Trace the page over time** — call `Pages_LandingsHistory` for the same site and
   date range to see how landing pages entered, moved, or dropped out of the result
   set. Compare against step 2 to spot keyword cannibalization (two pages
   alternating for one keyword) and lost pages.
4. **Export** — for a full-site audit set `Export=true` with `Format=csv`.

## Notes for agents
- Through the MCP server these are `get_landing_matches` and `get_landings_history`
  — a 1:1 binding to the two operations above (`mcp/demandsphere-tool-crosswalk.yml`).
- Responses have no `request-id` header to correlate on; when reporting a problem,
  quote the operation, site key and date range instead.
- Sort keys are operation-specific; `SortByLandingMatches` is not interchangeable
  with the `SortBy*` parameter of another operation.
