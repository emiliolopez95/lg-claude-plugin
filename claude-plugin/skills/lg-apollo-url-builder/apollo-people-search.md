# Apollo People Search

Base URL: `https://app.apollo.io/#/people`

Searches for **individual contacts** matching person + company criteria.

---

## Available Filters

### Person-level
- **Job Titles** — `personTitles[]`, `personNotTitles[]`, `includeSimilarTitles`
- **Contact Location** — `personLocations[]`, `personNotLocations[]` (Region mode) OR `personPostalCodes[]` + `personPostalCodesRadius` (Zip mode)
- **Email Status** — `contactEmailStatusV2[]` (`verified`, `unverified`, `unavailable`)

### Company/Account-level
- **Company** — `organizationIds[]`, `notOrganizationIds[]`, `personPastOrganizationIds[]`, `personNotPastOrganizationIds[]`
- **Company Lookalikes** — `lookalikeOrganizationIds[]`
- **Account HQ Location** — `organizationLocations[]`, `organizationNotLocations[]` (Region) OR `organizationLocationName` + `organizationLocationRadius` (Zip)
- **Number of Employees** — `organizationNumEmployeesRanges[]`
- **Industry** — `organizationIndustryTagIds[]`, `organizationNotIndustryTagIds[]`
- **Company Keywords** — `qOrganizationKeywordTags[]`, `qAndedOrganizationKeywordTags[]`, `qNotOrganizationKeywordTags[]`
- **Revenue** — `revenueRange[min]`, `revenueRange[max]`

---

## Typical Use

People Search is the main search type for lead generation. Combine person filters (title + location) with company filters (industry + headcount + HQ) to target decision makers at the right companies.

For full filter details (params, typeahead APIs, examples), see `apollo-filters-guide.md`.
