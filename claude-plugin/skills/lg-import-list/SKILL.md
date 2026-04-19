---
name: lg-import-list
description: Crea un IMPORT_LIST (IMPORT_CSV) en Airtable desde URLs de Apollo o LinkedIn Sales Navigator. Valida volumen y calidad antes de crear, luego llama `lg1_list_create_import`.
---

# Let Growth Import List Creation

Crear Import Lists usando la tool `lg1_list_create_import` del MCP, con URLs de LinkedIn Sales Navigator o Apollo.

**Cuando usar:** crear una lista tipo IMPORT_LIST para un cliente (todos excepto lavanderias).

**Antes de empezar:** completar el skill `lg-create-list-for-client` (obtiene client info, campaign number, ICP, contexto de listas).

**Prerequisitos:**
- Skills `lg-apollo-url-builder` o `lg-sales-nav-url-builder` para construir la URL

## Plataforma de busqueda

- **Default: Apollo** — usar a menos que el humano indique lo contrario
- **Sales Navigator** — usar solo si el humano lo pide explicitamente

---

## Paso 0: Planificacion de Volumen

De los resultados de Sales Nav o Apollo, solo 20-30% se convierten en leads.

---

## Paso 1: Construir la lista mas completa posible

Maximizar cobertura del ICP en una sola busqueda antes de dividir.

### 1.1 Incluir todos los filtros del ICP

- **Titulos:** todos los posibles del ICP. Para LATAM, incluir en espanol e ingles (ej: "Gerente de Marketing", "Marketing Manager"). Para titulos muy especificos (ej: Compliance), usar solo la palabra clave para maximizar resultados.
- **Locations:** TODAS las relevantes al ICP.
- **Industrias:** TODAS las relevantes al ICP.
- **Keywords:** TODAS las que apliquen.
- **Headcount:** el rango completo que pide el cliente.

### 1.2 Verificar que los filtros "hagan sentido"

Revisar que **todas las combinaciones** produzcan resultados del ICP. Si alguna combinacion queda fuera del ICP → dividir en listas separadas con combinaciones coherentes.

**Ejemplo:** ICP busca "Director de IT" en tecnologia y manufactura, pero en manufactura los titulos son diferentes → hacer una lista por industria.

### 1.3 Construir la URL

**Apollo (default):** ver skill `lg-apollo-url-builder`. Tools MCP:
- `apollo_search_tags` — typeahead (titles, locations, industries, keywords)
- `apollo_search_organizations` — typeahead de empresas (IDs)
- `apollo_search_by_url` — validar URL y obtener volumen

**Sales Navigator:** ver skill `lg-sales-nav-url-builder`. Tools MCP:
- `sales_nav_typeahead` — typeahead (TITLE, COMPANY, INDUSTRY, GEOGRAPHY...)
- `sales_nav_search_by_url` — validar URL y obtener volumen
- `sales_nav_enrich_company` — enriquecer empresa

---

## Paso 2: Validar URL

### 2.1 Validar volumen

**Apollo:**
```
tool: apollo_search_by_url
args: { "url": "https://app.apollo.io/#/people?..." }
```
Total en `pagination.total_entries`.

**Sales Nav:**
```
tool: sales_nav_search_by_url
args: { "url": "https://www.linkedin.com/sales/search/people?query=...", "start": 0, "count": 25 }
```
Total en `pagination.total`.

**Si total >= 2,500:** Dividir en orden:
1. **Headcount** — sub-listas por rangos de empleados (20-50, 51-200, 201-1000)
2. **Headcount + Seniority** — dentro de cada rango, dividir por seniority
3. **Seguir combinando** hasta que cada lista tenga <2,500

### 2.2 Validar la URL FINAL

**REGLA CRITICA:** El conteo que reportas DEBE ser de la URL EXACTA que vas a guardar en `scraping_link`. Si iteraste, **vuelve a llamar la tool con la URL final** antes de crear el record.

### 2.3 Validar calidad

```
tool: scrape_first_page
args: { "url": "...", "count": 25 }
```

Verificar **>=50% de los resultados de la primera pagina sean buen fit** para el ICP. Si no pasan, ajustar filtros y re-validar.

---

## Paso 3: Crear la lista

```
tool: lg1_list_create_import
args: {
  "name": "Cliente - Roles Ubicacion",
  "client_domain": "cliente.com",
  "sl_campaign_number": 7368121,
  "campaign_language": "SPANISH",
  "contact_types": "PEOPLE_ONLY",
  "search_roles": "Gerente,Director,VP",
  "agent_confidence": 8,
  "agent_notes": "ICP claro, volumen 1,800 validado, 60% good fit en primera pagina.",
  "scraping_link": "https://app.apollo.io/#/people?..."
}
```

**Defaults automaticos** (no los pongas tu):
- `search_type = "IMPORT_CSV"`
- `search_workflow_process = "NEEDS_HUMAN_REVIEW"`
- `search_workflow_process_status = "QUEUED"`
- `location_type = "INDIVIDUAL"`
- `scraping_status = "READY_TO_SCRAPE"`

**Campos opcionales importantes:**
- `location` — usualmente dejar vacio (el cliente tiene default)
- `manual_ai_match_string` — usualmente dejar vacio (default del cliente)
- `validate_roles_ai_string` — solo cuando el cliente es estricto con roles

Devuelve `{ created: true, id, name, search_type: "IMPORT_CSV", client_domain }`.

Informar al humano cuantas listas se crearon.
