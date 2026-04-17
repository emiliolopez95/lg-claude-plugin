---
name: lg-apollo-url-builder
description: Construir y validar URLs de busqueda de Apollo. Resolver filtros via typeahead (titulos, ubicaciones, industrias), validar volumen y calidad antes de crear IMPORT_LIST.
---

# Apollo URL Builder

Construir y validar URLs de Apollo para listas IMPORT_LIST.

**Cuando usar:** construyendo URLs de Apollo, resolviendo filtros via typeahead, validando volumen/calidad.

**Referencias** (archivos en este skill):
- `apollo-people-search.md` — people search filters
- `apollo-company-search.md` — company search filters
- `apollo-filters-guide.md` — full reference

---

## URL Structure

```
https://app.apollo.io/#/people?page=1&personTitles[]=ceo&...
https://app.apollo.io/#/companies?page=1&organizationLocations[]=...
```

- Arrays: `paramName[]=value` (uno por valor)
- Nested objects: `paramName[key]=value` (ej: `revenueRange[min]=2000000`)
- `page=1` siempre presente
- Default sort: `sortAscending=false&sortByField=[none]`

---

## Encoding Rules

| Character | Encoding | Notes |
|-----------|----------|-------|
| Space | `+` | **NOT** `%20` — Apollo interpreta `%20` como texto literal |
| Comma | `,` | En rangos (ej: `11,50`) |
| `[]` | Literal | En nombres: `personTitles[]=` |

**Ejemplo:**
```
Si:  personTitles[]=gerente+general
No:  personTitles[]=gerente%20general  (Apollo lo muestra literal)
```

---

## Tools MCP disponibles

Todo se hace via el MCP `letgrowth` — no se construyen curls manualmente.

### Typeahead: resolver filtros

```
tool: apollo_search_tags
args: {
  "query": "ceo",
  "kind": "person_title"
}
```

`kind` options: `person_title`, `location`, `industry_tag`, `keywords`, `technology`, `person_seniority`, `person_department`, `organization`.

**Para locations** (excluir estados US):
```
args: { "query": "mexico", "kind": "location", "exclude_categories": ["US State"] }
```

### Typeahead de empresas

```
tool: apollo_search_organizations
args: { "query": "letgrowth" }
```

Devuelve IDs de empresas para `organizationIds[]=` del URL.

---

## Filter Logic

- Todos los filtros son **AND** entre si
- Multiples valores dentro de un filtro son **OR**
- Exclusion params (`personNotTitles[]`) son **AND NOT**

---

## Validacion

### Test de URL (volumen)

```
tool: apollo_search_by_url
args: { "url": "https://app.apollo.io/#/people?..." }
```

El total viene en `data.pagination.total_entries`.

### Validar calidad (primera pagina)

```
tool: scrape_first_page
args: { "url": "https://app.apollo.io/#/people?...", "count": 25 }
```

**Rate limited: 1 call / 10s.** Usar con moderacion.

Validar **>= 50% de resultados sean buen fit** para el ICP del cliente.

---

## Validated Email Filter — Common Pitfall

Apollo UI a veces tiene activado el filtro `Verified Email` por default. Cuando validas volumen, **el conteo debe ser de la MISMA URL exacta que vas a guardar en `scraping_link`**.

- Si tu URL final NO tiene `emailStatusV2[]=verified`, NO lo uses al validar volumen
- Si lo incluyes al contar pero no al guardar, el conteo real sera ~2x mayor y rompe el limite de 2,500
- **Regla:** siempre valida con la URL final, sin filtros extras

---

## List Acceptance Criteria

| Criterio | Requerido |
|----------|-----------|
| < 2,500 resultados | Obligatorio (limite tecnico) |
| > 1,500 resultados | Ideal |
| >= 50% good fit en primera pagina | Obligatorio |

---

## 2,500 Result Limit

Si total >= 2,500, dividir. **Split strategies (en orden):** Geography → Headcount → Title/industry → Combinations.

Reglas: no overlap, validar cada una bajo 2,500, cada una es su propia Import List.
