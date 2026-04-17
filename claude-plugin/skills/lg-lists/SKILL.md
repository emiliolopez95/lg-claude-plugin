---
name: lg-lists
description: Punto de entrada para crear listas de leads para clientes de Let Growth. Orquesta el flujo completo (info cliente, campana, lista anterior, crear lista).
---

# Let Growth - Crear Lista para Cliente

Skill para crear listas de leads para clientes de Let Growth.

**Cuando usar:** el usuario pide crear una lista para un cliente (ej: "crea una lista para starlaundromats.com").

---

## Prerequisitos

Este skill usa el **MCP `letgrowth`** (viene dentro de este plugin). Todas las operaciones contra Airtable, Apollo y Sales Navigator se hacen via las tools del MCP — no hay que manejar API keys manualmente.

**Si las tools `mcp__letgrowth__*` no aparecen**, el MCP no esta conectado. Verificar en `/plugin` que el plugin letgrowth este enabled y la API key configurada.

---

## Flujo

1. Leer el skill `lg-create-list-for-client` y seguir ese flujo completo
2. El argumento es el **dominio del cliente** (ej: `starlaundromats.com`)
3. Si no se paso argumento, preguntar al usuario que cliente quiere

El skill principal te dirige segun el tipo de lista:
- Lavanderias (`is_laundry=true`) → skill `lg-search-list`
- Otros clientes → skill `lg-import-list`

---

## Skills disponibles (dentro de este plugin)

| Skill | Proposito |
|-------|-----------|
| `lg-create-list-for-client` | Obtener info del cliente y decidir tipo de lista |
| `lg-search-list` | Crear listas para lavanderias (Google/Facebook search) |
| `lg-import-list` | Crear listas con URLs de Apollo/Sales Nav |
| `lg-airtable-operations` | Referencia de tools MCP para Airtable |
| `lg-apollo-url-builder` | Construir URLs de busqueda en Apollo |
| `lg-sales-nav-url-builder` | Construir URLs de Sales Navigator |

## Tools MCP disponibles (resumen)

**Airtable (lectura):** `client_get`, `clients_needing_leads`, `client_campaigns`, `client_lists`
**Airtable (escritura):** `list_create_search`, `list_create_import`
**Apollo:** `apollo_search_tags`, `apollo_search_organizations`, `apollo_search_by_url`
**Sales Nav:** `sales_nav_typeahead`, `sales_nav_search_by_url`, `sales_nav_enrich_company`
**Scraping compartido:** `scrape_first_page`

Endpoint: `https://app.letgrowth.com/api/mcp` (auth con `LETGROWTH_API_KEY` configurada en el plugin).
