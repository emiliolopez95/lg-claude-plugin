# Sales Navigator URL Builder - Full Reference

Build and validate LinkedIn Sales Navigator search URLs for LetGrowth lead generation.

---

## Overview

Covers **People Search** (`/sales/search/people`) and **Company Search** (`/sales/search/company`).

### Key Differences

| Aspect | People Search | Company Search |
|--------|---------------|----------------|
| Path | `/sales/search/people` | `/sales/search/company` |
| Wrapper | `spellCorrectionEnabled:true,recentSearchParam:(doLogHistory:true)` | `spellCorrectionEnabled:true` |
| Location filter | `REGION` (person) + `COMPANY_HEADQUARTERS` (HQ) | `REGION` (company location) |
| Unique filters | Title, Function, Seniority, School, Profile Language, Years | Revenue, Followers, Dept Headcount, Technologies |

---

## URL Anatomy

### People Search (before encoding)

```
(spellCorrectionEnabled:true,recentSearchParam:(doLogHistory:true),filters:List(...),keywords:...)
```

### Company Search (before encoding)

```
(spellCorrectionEnabled:true,filters:List(...),keywords:...)
```

### Filter Syntax Patterns

**Standard filter:**
```
(type:FILTER_TYPE,values:List(
  (id:ID,text:ENCODED_TEXT,selectionType:INCLUDED),
  (id:ID,text:ENCODED_TEXT,selectionType:EXCLUDED)
))
```

**Range filter:**
```
(type:FILTER_TYPE,rangeValue:(min:MIN,max:MAX),selectedSubFilter:SUB)
```

**Text-only filter:**
```
(type:FILTER_TYPE,values:List((text:ENCODED_TEXT,selectionType:INCLUDED)))
```

**Company/org filter:**
```
(type:FILTER_TYPE,values:List(
  (id:urn%3Ali%3Aorganization%3A13044068,text:Company%2520Name,selectionType:INCLUDED,parent:(id:0))
))
```

---

## Encoding Rules

| Character | In `text:` values | In structure |
|-----------|-------------------|--------------|
| Space | `%2520` | N/A |
| Comma | `%252C` | `%2C` |
| Colon | N/A | `%3A` |
| Parentheses | N/A | Literal `()` |
| n tilde | `%25C3%25B1` | N/A |
| i accent | `%25C3%25AD` | N/A |
| o accent | `%25C3%25B3` | N/A |
| e accent | `%25C3%25A9` | N/A |
| a accent | `%25C3%25A1` | N/A |

### Build Script (Python)

```python
# People Search
inner = "(spellCorrectionEnabled:true,recentSearchParam:(doLogHistory:true),filters:List(" \
"(type:REGION,values:List((id:107163507,text:Antioquia%252C%2520Colombia,selectionType:INCLUDED)))," \
"(type:CURRENT_TITLE,values:List((id:254,text:Purchasing%2520Manager,selectionType:INCLUDED)))," \
"(type:COMPANY_HEADCOUNT,values:List((id:D,text:51-200,selectionType:INCLUDED)))" \
"))"

encoded = inner.replace(':', '%3A').replace(',', '%2C')
url = f"https://www.linkedin.com/sales/search/people?query={encoded}&viewAllFilters=true"
```

---

## Boolean Keyword Search

```
keywords:(fintech OR banking) NOT ("private equity")
```

| Operator | Function | Example |
|----------|----------|---------|
| `AND` | Both required (implicit) | `sales marketing` |
| `OR` | Either term | `CEO OR founder` |
| `NOT` | Exclude | `engineer NOT software` |
| `"..."` | Exact phrase | `"vice president"` |
| `(...)` | Grouping | `(CEO OR founder) AND fintech` |

**Rules:**
- Operators **MUST be CAPITALIZED**
- **Always wrap OR groups in parentheses**
- No wildcards — list variations
- Max ~1,000 characters

---

## Validation via LetGrowth API

**API Base:** `https://letgrowth.com/api/linkedin-sales?api_key=$LETGROWTH_API_KEY`

### Typeahead

```bash
curl -sL "https://letgrowth.com/api/linkedin-sales?api_key=$LETGROWTH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"action": "typeahead", "params": { "query": "SEARCH_TERM", "type": "TYPE", "count": 5 }}' \
  | jq '.data.elements[] | {id, displayValue}'
```

**Types:** `BING_GEO`, `TITLE`, `INDUSTRY`, `COMPANY`, `FUNCTION`, `SENIORITY`, `SCHOOL`

### Test a URL

```bash
curl -sL "https://letgrowth.com/api/linkedin-sales?api_key=$LETGROWTH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"action": "searchByUrl", "params": { "url": "THE_URL", "page": 1 }}'
```

**Parse people:** `| jq '{total: .data.pagination.total, count: .data.pagination.count, first_results: [.data.data[:3][] | {name: .fullName, title: .currentPosition.title, company: .currentPosition.companyName, location: .geoRegion}]}'`

### Validate First Page

**ENDPOINT COSTOSO** — 15-20s response, 10s cooldown.

```bash
curl -sL "https://letgrowth.com/api/linkedin-sales" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $LETGROWTH_API_KEY" \
  -d '{"action": "scrapeFirstSearchPage", "params": { "url": "URL", "enrich": true }}'
```

---

## Diagnosis

| Symptom | Cause | Fix |
|---------|-------|-----|
| 500 error | Malformed URL | Check encoding, filter names, parentheses |
| 0 results | Too restrictive | Remove filters, widen geography/headcount |
| Very few (<50) | Niche combo | May be valid — present to user |

---

## Tips & Gotchas

1. **`REGION` vs `COMPANY_HEADQUARTERS`** — People search has both. `REGION` = where the person is. `COMPANY_HEADQUARTERS` = where HQ is.
2. **Company IDs use URN format** — `id:urn%3Ali%3Aorganization%3A13044068` with `parent:(id:0)`
3. **Filter logic** — All filters are AND. Multiple INCLUDED values within a filter are OR. EXCLUDED values are AND NOT.
4. **LinkedIn's geo hierarchy** — "Mexico" (country) vs "Mexico City" (metro) vs "Ciudad de Mexico" (state) are different IDs. Always typeahead.
5. **Spanish titles work** — Text-only (no ID) still filters correctly.
6. **`YEARS_IN_CURRENT_POSITION`** — NOT `YEARS_AT_CURRENT_POSITION`
