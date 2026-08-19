---
name: Track keyword ranking trends
description: Pull a site's keyword ranking trends over a date range from the DemandSphere API v5.0, then drill into per-keyword performance and landing matches.
api: openapi/demandsphere-keywords-api-openapi.yml
operations:
- Sites_PropertiesList
- Keywords_RankingTrends
- Keywords_KeywordsPerformanceDetail
- Keywords_LandingMatches
---

# Track keyword ranking trends

Operating instructions for using the DemandSphere API v5.0 to report how a
site's keywords are trending in search rankings.

## Auth & conventions
- Base URL: `https://api.demandsphere.com`
- Every request needs the `api_key` query parameter (global security scheme `ApiKey`, `in: query`).
- All operations are `POST .../list` with parameters passed in the query string (no request body).
- Date range is required: `From` and `To` as `YYYY-MM-DD`.
- Pagination is offset-based: `Limit` (default 25), `Offset`, `PageNum`.
- Export with `Export=true` plus `Format=csv|xlsx`.
- Errors surface as HTTP `400` (missing/invalid required parameter). See `errors/demandsphere-problem-types.yml`.

## Steps
1. **Resolve the site** — call `Sites_PropertiesList` to list tracked properties and get the `SiteGlobalKey` for the target site.
2. **Pull ranking trends** — call `Keywords_RankingTrends` with `SiteGlobalKey`, `From`, `To`, `Granularity` (`daily|weekly|monthly`), `Order`, `SearchEngines`, and a `SortByRankingTrends_UnGrouped` (or `_Grouped` with `Grouped=true`) sort key. Page with `Limit`/`Offset`.
3. **Drill into a keyword** — for a keyword of interest, call `Keywords_KeywordsPerformanceDetail` with the same site/date scope and `SortByPerformanceDetail` to get rank, rank_change, search_volume, clicks, impressions, average_position, and CTR.
4. **Find landing pages** — call `Keywords_LandingMatches` to see which pages rank for those keywords (`SortByLandingMatches`).

## Notes
- Confirm valid `SearchEngines` enum values (e.g. `google_us`, `bing_uk`) before querying.
- The API is read-only query analytics; there is no idempotency contract because no state is mutated.
