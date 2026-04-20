---
name: lg2-lists
description: Entry point para crear lg2_lists (buckets curados en LG2 / Postgres, apuntados a campañas de Bison/Smartlead/li2). Usar cuando el usuario dice "crea una lg2 list" o similar. Diferente a LG1 "lista" (que es un todo_list de scraping).
---

# LG2 Lists — Entry Point

Skill para crear y operar **`lg2_lists`** en LG2 (sistema nuevo, datos en Postgres).

**Cuando usar:** el usuario pide algo como:
- "crea una lg2 list para cliente X"
- "crea una list de LG2 para X"
- Nombra explícitamente "LG2" o usa lenguaje típico de LG2 (discovery, ICP, persona)

**No usar para:** "crea una lista para X" a secas — puede ser LG1 (`todo_list`). Usa `client_locate` primero para detectar el sistema.

---

## Glosario rápido (LG2)

| Término | Qué es |
|---|---|
| **Cliente** | Empresa. Misma identificación que LG1 (`domain`), más el flag `clients.is_lg2=true` en Postgres. |
| **ICP** | Ideal Customer Profile — template de targeting del cliente. Tiene `ai_match_criteria` (quién califica) y `ai_location_criteria` (dónde). Un cliente puede tener varios ICPs. |
| **Discovery** | **Input de scraping** (hoy: URL de LinkedIn Sales Nav). Cuelga de un ICP. El pipeline la corre para encontrar empresas + gente. **Equivalente LG1:** IMPORT_LIST. |
| **Persona** | Sub-target dentro de un ICP — qué títulos/roles acepta (ej. "CEO" vs "Ops Leadership"). Cada persona tiene su propio `list_id` de destino. |
| **lg2_list** | **Output curado.** Bucket de contactos ya calificados apuntado a una campaña (Bison/Smartlead/li2). El equipo crea el shell (nombre + canal + campaign_id + merge-tag config); el pipeline lo llena vía enrollment. **OJO:** esto NO es lo que llamamos "lista" en LG1. |

### Flujo LG2 de un vistazo

```
ICP (template)
  ├─ Discovery (URL) ──► pipeline scrapes ──► Companies + People
  │                        │
  │                        └──► evaluates against ai_match_criteria
  │                                 │
  │                                 └──► enrollment gate (per persona)
  │                                           │
  └─ Persona (title criteria, list_id ─────────► lg2_list ──► Bison campaign
```

---

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

- `lg2-list-create` — flujo paso a paso para crear un `lg2_list` (output curado)
- `lg2-discovery-create` — flujo paso a paso para crear un discovery (input de scraping via Sales Nav URL)
- `lg2-operations` — referencia completa de las tools LG2
