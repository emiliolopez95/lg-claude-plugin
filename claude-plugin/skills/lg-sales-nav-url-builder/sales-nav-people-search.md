# People Search Filters

Search path: `/sales/search/people`

## URL Wrapper

```
(spellCorrectionEnabled:true,recentSearchParam:(doLogHistory:true),filters:List(...),keywords:...)
```

---

## Dynamic Filters (require typeahead)

| Filter | `type` value | Typeahead `type` | Exclusion |
|--------|--------------|------------------|-----------|
| Lead Location | `REGION` | `BING_GEO` | Yes |
| Company HQ Location | `COMPANY_HEADQUARTERS` | `BING_GEO` | Yes |
| Job Title | `CURRENT_TITLE` | `TITLE` | Yes |
| Past Title | `PAST_TITLE` | `TITLE` | Yes |
| Industry | `INDUSTRY` | `INDUSTRY` | Yes |
| Current Company | `CURRENT_COMPANY` | `COMPANY` | Yes |
| Past Company | `PAST_COMPANY` | `COMPANY` | Yes |
| Function | `CURRENT_FUNCTION` | `FUNCTION` | Yes |
| Seniority | `SENIORITY_LEVEL` | `SENIORITY` | Yes |
| School | `SCHOOL` | `SCHOOL` | Yes |

### Title Notes

- Typeahead only works for **English** titles
- Spanish titles → text-only: `(text:Gerente%2520de%2520Compras,selectionType:INCLUDED)` (no `id`)
- Sales Nav matches profiles in **any language** regardless of filter language

---

## Static Filters

### Company Headcount (`COMPANY_HEADCOUNT`)

| Label | ID |
|-------|-----|
| Self-employed | A |
| 1-10 | B |
| 11-50 | C |
| 51-200 | D |
| 201-500 | E |
| 501-1,000 | F |
| 1,001-5,000 | G |
| 5,001-10,000 | H |
| 10,001+ | I |

Use `COMPANY_HEADCOUNT` NOT `CURRENT_COMPANY_HEADCOUNT`

### Seniority Level (`SENIORITY_LEVEL`)

| Label | ID |
|-------|-----|
| In Training | 100 |
| Entry Level Manager | 200 |
| Experienced Manager | 210 |
| Director | 220 |
| Vice President | 300 |
| CXO | 310 |
| Owner / Partner | 320 |

### Years of Experience (`YEARS_OF_EXPERIENCE`)

| Label | ID |
|-------|-----|
| Less than 1 year | 1 |
| 1 to 2 years | 2 |
| 3 to 5 years | 3 |
| 6 to 10 years | 4 |
| More than 10 years | 5 |

Same IDs for: `YEARS_AT_CURRENT_COMPANY`, `YEARS_IN_CURRENT_POSITION`

### Profile Language (`PROFILE_LANGUAGE`)

| Language | ID |
|----------|-----|
| English | en |
| Spanish | es |
| Portuguese | pt |
| French | fr |

### First/Last Name

Filter types: `FIRST_NAME`, `LAST_NAME` — text-only, no ID:
```
(type:FIRST_NAME,values:List((text:Jose,selectionType:INCLUDED)))
```

---

## `REGION` vs `COMPANY_HEADQUARTERS`

- `REGION` = where the person is (use most of the time)
- `COMPANY_HEADQUARTERS` = where company HQ is (niche use)
