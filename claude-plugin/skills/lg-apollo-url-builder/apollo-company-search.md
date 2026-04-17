# Apollo Company Search

Base URL: `https://app.apollo.io/#/companies`

Searches for **companies/accounts** matching organizational criteria. Does NOT return individual contacts — use People Search for that.

---

## Available Filters

- **Account HQ Location** — `organizationLocations[]`, `organizationNotLocations[]` (Region) OR `organizationLocationName` + `organizationLocationRadius` (Zip)
- **Number of Employees** — `organizationNumEmployeesRanges[]`
- **Industry** — `organizationIndustryTagIds[]`, `organizationNotIndustryTagIds[]`
- **Company Keywords** — `qOrganizationKeywordTags[]`, `qAndedOrganizationKeywordTags[]`, `qNotOrganizationKeywordTags[]`
- **Revenue** — `revenueRange[min]`, `revenueRange[max]`

---

## Typical Use

Company Search is useful for building target account lists or finding companies in specific niches before running People Search against them. Only company/account-level filters apply here — no person-level filters.

For full filter details (params, typeahead APIs, examples), see `apollo-filters-guide.md`.
