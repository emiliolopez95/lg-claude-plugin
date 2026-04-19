---
name: lg2-lists
description: Entry point for creating LG2 lists (lg2_list curated buckets pointed at Bison/Smartlead/li2 campaigns). Use when the user says "crea una lg2 list" or similar. Different from LG1 "lista" (which is a todo_list scraping input).
---

# LG2 Lists — Entry Point

Skill para crear y operar `lg2_lists` (listas curadas en LG2, el sistema nuevo).

**Cuando usar:** el usuario pide algo como:
- "crea una lg2 list para cliente X"
- "crea una list de LG2 para X"
- Nombra explicitamente "LG2" o usa lenguaje tipico de LG2 (discovery, ICP, persona)

**No usar para:** "crea una lista para X" a secas — eso puede ser LG1 (`todo_list`). En ese caso usa `client_locate` primero para detectar el sistema, o usa el skill `lg-lists` si ya sabes que es LG1.

---

## Diferencia LG1 vs LG2 (importante)

| Concepto | LG1 | LG2 |
|---|---|---|
| Sistema | Airtable | Postgres |
| "Lista" como input de scraping | `todo_list` (lo que llamamos "lista") | `lg2_icp_discovery` (se llama "discovery") |
| "Lista" como output curado | (no existe) | `lg2_list` (lo que llamamos "lg2 list") |
| Quien pone los contactos | El equipo configura filtros, un workflow scrapea | El pipeline auto-enrola contactos segun `persona.list_id` |

El equipo en LG2 crea el **shell** de la lista (nombre, canal, campana, merge-tag config). El pipeline hace el resto.

---

## Flujo principal

Leer el skill `lg2-list-create` y seguir ese flujo.

Resumen:
1. Resolver cliente y confirmar esta en LG2
2. Decidir canal (Bison / Smartlead / li2) y obtener `external_campaign_id`
3. (Opcional) Decidir merge-tag fields
4. Crear la lista con `lg2_list_create`
5. Confirmar al usuario

---

## Tools MCP disponibles (resumen)

**Lectura:**
- `lg2_icp_list(client_domain?)` — ICPs del cliente
- `lg2_list_list(client_domain?, status?)` — listas existentes
- `lg2_list_get(list_id)` — detalle con contadores de contactos
- `lg2_custom_field_library(client_domain?)` — merge-tag fields reusables

**Escritura:**
- `lg2_list_create(client_domain, name, channel, external_campaign_id, language?, custom_fields_config?)` — crea shell
- `lg2_list_update(list_id, {name?, status?, language?})` — edits basicos
- `lg2_list_update_config(list_id, custom_fields_config)` — edit merge-tag config

**Routing cross-sistema:**
- `client_locate(domain)` — detecta si el cliente es LG1, LG2, o ambos

---

## Skills relacionados en el plugin

- `lg2-list-create` — flujo paso a paso para crear una lista
- `lg2-operations` — referencia completa de las tools LG2
