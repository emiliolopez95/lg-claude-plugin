# Company Search Filters

Search path: `/sales/search/company`

## URL Wrapper

```
(spellCorrectionEnabled:true,filters:List(...),keywords:...)
```

---

## Dynamic Filters

| Filter | `type` value | Typeahead `type` | Exclusion |
|--------|--------------|------------------|-----------|
| HQ Location | `REGION` | `BING_GEO` | Yes |
| Industry | `INDUSTRY` | `INDUSTRY` | Yes |
| Technologies Used | `TECHNOLOGIES_USED` | — | No |

---

## Static/Range Filters

### Number of Followers (`NUM_OF_FOLLOWERS`)

| Label | ID |
|-------|-----|
| 1-50 | NFR1 |
| 51-100 | NFR2 |
| 101-1,000 | NFR3 |
| 1,001-5,000 | NFR4 |
| 5,001-10,000 | NFR5 |
| 10,001+ | NFR6 |

### Annual Revenue (`ANNUAL_REVENUE`)

```
(type:ANNUAL_REVENUE,rangeValue:(min:1,max:20),selectedSubFilter:USD)
```
- `min`/`max`: revenue in millions
- `selectedSubFilter`: currency code

### Department Headcount (`DEPARTMENT_HEADCOUNT`)

```
(type:DEPARTMENT_HEADCOUNT,rangeValue:(min:1,max:100),selectedSubFilter:15)
```
- `min`/`max`: employee count
- `selectedSubFilter`: department ID
