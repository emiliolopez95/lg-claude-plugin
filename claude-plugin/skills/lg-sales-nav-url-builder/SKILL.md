---
name: lg-sales-nav-url-builder
description: Construir y validar URLs de LinkedIn Sales Navigator. Resolver IDs via typeahead, encoding correcto, validar volumen y calidad antes de crear IMPORT_LIST.
---

# Sales Navigator URL Builder

Construir y validar URLs de LinkedIn Sales Navigator para listas IMPORT_LIST.

**Cuando usar:** construyendo scraping_links de Sales Nav, resolviendo IDs via typeahead, validando volumen/calidad.

**Referencias** (archivos en este skill):
- `sales-nav-people-search.md` — people search filters
- `sales-nav-company-search.md` — company search filters
- `sales-nav-full.md` — full reference

---

## URL Anatomy

### Filter Syntax Patterns

**Standard filter:**
```
(type:FILTER_TYPE,values:List(
  (id:ID,text:ENCODED_TEXT,selectionType:INCLUDED),
  (id:ID,text:ENCODED_TEXT,selectionType:EXCLUDED)
))
```

**Range filter** (revenue, department headcount):
```
(type:FILTER_TYPE,rangeValue:(min:MIN,max:MAX),selectedSubFilter:SUB)
```

**Text-only filter** (names, Spanish titles):
```
(type:FILTER_TYPE,values:List((text:ENCODED_TEXT,selectionType:INCLUDED)))
```

**Company/org filter** (usa URN IDs):
```
(type:FILTER_TYPE,values:List(
  (id:urn%3Ali%3Aorganization%3A13044068,text:Company%2520Name,selectionType:INCLUDED,parent:(id:0))
))
```

---

## Boolean Keyword Search

Ambos search types soportan `keywords` con logica boolean:

```
keywords:(fintech OR banking) NOT ("private equity")
```

| Operator | Function | Example |
|----------|----------|---------|
| `AND` | Ambos (implicito) | `sales marketing` |
| `OR` | Cualquiera | `CEO OR founder` |
| `NOT` | Excluir | `engineer NOT software` |
| `"..."` | Exacta | `"vice president"` |
| `(...)` | Grouping | `(CEO OR founder) AND fintech` |

- Operadores **MAYUSCULAS**
- **Siempre envuelve OR groups en parentesis**
- Sin wildcards — lista variantes
- Max ~1,000 caracteres

---

## Encoding Rules

| Character | En `text:` values | En structure |
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
inner = "(spellCorrectionEnabled:true,recentSearchParam:(doLogHistory:true),filters:List(" \
"(type:REGION,values:List((id:107163507,text:Antioquia%252C%2520Colombia,selectionType:INCLUDED)))," \
"(type:CURRENT_TITLE,values:List((id:254,text:Purchasing%2520Manager,selectionType:INCLUDED)))," \
"(type:COMPANY_HEADCOUNT,values:List((id:D,text:51-200,selectionType:INCLUDED)))" \
"))"

encoded = inner.replace(':', '%3A').replace(',', '%2C')
url = f"https://www.linkedin.com/sales/search/people?query={encoded}&viewAllFilters=true"
```

---

## Tools MCP disponibles

### Typeahead: resolver IDs

```
tool: sales_nav_typeahead
args: { "query": "Antioquia", "type": "GEOGRAPHY", "count": 5 }
```

**Types:** `BING_GEO`, `TITLE`, `INDUSTRY`, `COMPANY`, `FUNCTION`, `SENIORITY`, `SCHOOL`, `KEYWORD`, `GEOGRAPHY`

**Siempre validar cada filtro dinamico. Nunca adivines IDs.**

### Test de URL (volumen)

```
tool: sales_nav_search_by_url
args: { "url": "https://www.linkedin.com/sales/search/people?query=...", "start": 0, "count": 25 }
```

El total viene en `data.pagination.total` (o `totalDisplayCount`).

### Enriquecer empresa

```
tool: sales_nav_enrich_company
args: { "company_id": "123456", "sources": ["sales-company", "voyager-entities"] }
```

### Validar calidad (primera pagina)

```
tool: scrape_first_page
args: { "url": "...", "count": 25, "enrich": true }
```

**Rate limited: 1 call / 10s.** Validar **>= 50% good fit**.

---

## List Acceptance Criteria

| Criterio | Requerido |
|----------|-----------|
| < 2,500 resultados | Obligatorio |
| > 1,500 resultados | Ideal |
| >= 50% good fit en primera pagina | Obligatorio |

---

## 2,500 Result Limit

Si `totalDisplayCount` >= 2,500, dividir. **Split strategies (en orden):** Geography → Headcount → Title/industry → Combinations.

Reglas: no overlap, validar cada una bajo 2,500, cada una es su propia Import List.

---

## Tips & Gotchas

1. **Filter logic** — AND entre filtros. INCLUDED dentro de un filtro es OR. EXCLUDED es AND NOT.
2. **Geo hierarchy** — "Mexico" no es igual a "Mexico City" no es igual a "Ciudad de Mexico" — siempre typeahead.
3. **Spanish titles funcionan** — Text-only (no ID) filtra correctamente.
4. **`YEARS_IN_CURRENT_POSITION`** — NO `YEARS_AT_CURRENT_POSITION`
5. **Company IDs usan URN format** — `id:urn%3Ali%3Aorganization%3A13044068` con `parent:(id:0)`
