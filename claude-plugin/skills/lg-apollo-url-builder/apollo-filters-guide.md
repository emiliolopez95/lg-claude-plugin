# Apollo Filters Guide

_Last updated: February 20, 2026_

Guide for building Apollo search URLs with the correct filters. Each filter documents its input type, API params, and how to query valid values.

---

## People Search

### URL Structure

```
https://app.apollo.io/#/people?page=1&personTitles[]=ceo&personTitles[]=president&...
```

- Base: `https://app.apollo.io/#/people`
- Arrays use `paramName[]=value` (one per value)
- Nested objects use `paramName[key]=value` (e.g. `revenueRange[min]=2000000`)
- Values are URL-encoded (spaces become `%20`, `[]` becomes `%5B%5D`)
- `page=1` is always present
- `sortAscending=false&sortByField=[none]` is the default sort

---

### Job Titles

**Input type:** Free text (any language). Also has a typeahead API for predefined English values.

**URL params:**

| Param | Type | Description |
|-------|------|-------------|
| `personTitles[]` | array, free text | Titles to include. One per value. |
| `personNotTitles[]` | array, free text | Titles to exclude. One per value. |
| `includeSimilarTitles` | `true` / `false` | Expands results to include similar job titles. |

**URL example:**

```
...&personTitles[]=ceo&personTitles[]=president&personTitles[]=gerente%20comercial&personNotTitles[]=vice%20president&personNotTitles[]=viceprecidente&includeSimilarTitles=true
```

**Typeahead API** (optional — to get predefined values in English):

```bash
curl -X POST https://app.letgrowth.com/api/apollo?api_key=$LETGROWTH_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"action": "searchTags", "params": {"query": "ceo", "kind": "person_title"}}'
```

Returns tags with `cleaned_name` — use that as the value. But any free text works too.

---

### Company

**Input type:** Selection only (no free text). Use the typeahead API to find companies by name and get their IDs.

**URL params:**

| Param | Type | Description |
|-------|------|-------------|
| `organizationIds[]` | array, IDs | Current company — is any of. |
| `notOrganizationIds[]` | array, IDs | Current company — is not any of. |
| `personPastOrganizationIds[]` | array, IDs | Past company — include. |
| `personNotPastOrganizationIds[]` | array, IDs | Past company — exclude. |

**Typeahead API** (required — to get company IDs):

```bash
curl -X POST https://app.letgrowth.com/api/apollo?api_key=$LETGROWTH_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"action": "searchOrganizations", "params": {"query": "letgrowth"}}'
```

Use the `id` field as the value.

---

### Company Lookalikes

**Input type:** Selection only. Same typeahead API as Company filter.

**URL params:**

| Param | Type | Description |
|-------|------|-------------|
| `lookalikeOrganizationIds[]` | array, IDs | Find people at companies similar to these. |

---

### Contact Location

Has two modes: **Region** or **Zip Code Radius**.

#### Mode 1: Region

| Param | Type | Description |
|-------|------|-------------|
| `personLocations[]` | array, text | Locations to include. |
| `personNotLocations[]` | array, text | Locations to exclude. |

**Typeahead API** (optional):

```bash
curl -X POST https://app.letgrowth.com/api/apollo?api_key=$LETGROWTH_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"action": "searchTags", "params": {"query": "panama", "kind": "location"}}'
```

#### Mode 2: Zip Code Radius

| Param | Type | Description |
|-------|------|-------------|
| `personPostalCodes[]` | array, zip codes | Zip codes to search around. |
| `personPostalCodesRadius` | number | Radius in miles. |

> Region and Zip Code Radius are mutually exclusive.

---

### Account HQ Location

#### Mode 1: Region

| Param | Type | Description |
|-------|------|-------------|
| `organizationLocations[]` | array, text | Company locations to include. |
| `organizationNotLocations[]` | array, text | Company locations to exclude. |

#### Mode 2: Zip Code Radius

| Param | Type | Description |
|-------|------|-------------|
| `organizationLocationName` | string, single zip code | Zip code to search around. |
| `organizationLocationRadius` | number | Radius in miles. |

> Unlike Contact Location (multiple zip codes as array), Account HQ only takes a single zip code.

---

### Number of Employees

**Input type:** Predefined ranges. Value format is `min,max`.

| Param | Type | Description |
|-------|------|-------------|
| `organizationNumEmployeesRanges[]` | array, range strings | Employee count ranges. Format: `min,max`. |

**No API needed** — values are static ranges.

---

### Industry & Keywords

#### Industry

**Input type:** Selection only. Use typeahead API.

| Param | Type | Description |
|-------|------|-------------|
| `organizationIndustryTagIds[]` | array, IDs | Industries to include. |
| `organizationNotIndustryTagIds[]` | array, IDs | Industries to exclude. |

**Typeahead API** (required):

```bash
curl -X POST https://app.letgrowth.com/api/apollo?api_key=$LETGROWTH_API_KEY \
  -H "Content-Type: application/json" \
  -d '{"action": "searchTags", "params": {"query": "building", "kind": "linkedin_industry"}}'
```

#### Company Keywords

| Param | Type | Description |
|-------|------|-------------|
| `qOrganizationKeywordTags[]` | array, free text | Keywords to include (OR). |
| `qAndedOrganizationKeywordTags[]` | array, free text | Keywords to include (AND). |
| `qNotOrganizationKeywordTags[]` | array, free text | Keywords to exclude. |
| `includedOrganizationKeywordFields[]` | array, field names | Which fields to search in. |
| `excludedOrganizationKeywordFields[]` | array, field names | Which fields to exclude. |

Field values: `tags`, `name`, `social_media_description`.

---

### Email Status

| Param | Type | Description |
|-------|------|-------------|
| `contactEmailStatusV2[]` | array, predefined | `verified`, `unverified`, `unavailable` |

---

### Revenue

| Param | Type | Description |
|-------|------|-------------|
| `revenueRange[min]` | number | Minimum revenue in USD. |
| `revenueRange[max]` | number | Maximum revenue in USD. |
