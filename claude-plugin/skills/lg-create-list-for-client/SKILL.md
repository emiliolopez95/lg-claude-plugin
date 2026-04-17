---
description: Flujo guiado para crear una lista de leads para un cliente. Obtiene info del cliente, campana activa, contexto de listas previas, y decide el tipo de lista (SEARCH_LIST vs IMPORT_LIST).
---

# Let Growth - Create List for Client

Punto de entrada para crear listas de leads para un cliente.

**Cuando usar:** te piden crear una lista para un cliente.

---

## Flujo completo

```
1. Obtener info del cliente
2. Definir Campaign ID
3. Obtener ultimas listas del cliente (contexto)
4. Definir tipo de lista (SEARCH_LIST o IMPORT_LIST)
5. Ir al skill correspondiente
```

---

## Paso 1: Obtener info del cliente

Usa la tool MCP `client_get`:

```
tool: client_get
args: { "domain": "cliente.com" }
```

Devuelve: `{ found, id, domain, name, icp_summary, is_laundry, is_serviced, client_type, curr_week_new_leads_needed, next_week_new_leads_needed }`.

**importante**: **SOLO USAR** la info en `icp_summary` para crear las listas. **NADA MAS**.

**Importante:** Solo crear listas para clientes con `is_serviced=true`. Si la tool devuelve `found: false` o `is_serviced=false`, avisar al usuario.

---

## Paso 2: Definir Campaign ID

```
tool: client_campaigns
args: { "domain": "cliente.com", "active_only": true }
```

**Logica:**
- Si tiene **una sola campana activa** → usar esa (`campaigns[0].name` es el numero SL a pasar como `sl_campaign_number`)
- Si tiene **varias campanas activas** → preguntar al humano cual usar (a menos que este clarisimo por el contexto)

---

## Paso 3: Obtener ultimas listas del cliente

```
tool: client_lists
args: { "domain": "cliente.com", "days": 60, "limit": 15 }
```

Devuelve listas recientes con metricas clave:
- `search_good_fit_and_location_match` — empresas que pasaron filtro
- `search_re_valid_contacts` — contactos verificados
- `search_added_to_campaign` — leads finales

Tambien chequea listas en progreso:

```
tool: client_lists
args: { "domain": "cliente.com", "only_in_progress": true }
```

**Si devuelve listas**, avisar al humano antes de continuar — ya hay listas en proceso.

**No asumas que las listas pasadas estaban bien hechas.** Usalas como referencia, no como plantilla perfecta.

---

## Paso 3.5: Estrategia de Titulos

Priorizar titulos por probabilidad de respuesta:

| Tamano empresa | Titulos a EVITAR | Titulos a PRIORIZAR |
|----------------|------------------|---------------------|
| Enterprise (1000+) | CEO, COO, CFO, CXO | Directors, Managers, VPs |
| Mid-market (200-1000) | CEO | COO, VPs, Directors, Managers |
| SMB (<200) | — | Todos los del ICP |

**Regla:** Mientras mas grande la empresa, mas bajo el rango del titulo (dentro del ICP). C-Suite en enterprise NO responde cold email.

---

## Paso 4: Definir tipo de lista

| Condicion | Tipo de lista | Skill a usar |
|-----------|---------------|--------------|
| `is_laundry = true` | SEARCH_LIST | skill `lg-search-list` |
| `is_laundry = false` | IMPORT_LIST | skill `lg-import-list` |

---

## Paso 5: Ir al skill correspondiente

- **SEARCH_LIST:** leer skill `lg-search-list`
- **IMPORT_LIST:** leer skill `lg-import-list`

Esos skills asumen que ya tienes:
- Info del cliente (paso 1) — incluyendo `icp_summary`
- Campaign ID numerico (paso 2)
- Contexto de listas anteriores (paso 3)
