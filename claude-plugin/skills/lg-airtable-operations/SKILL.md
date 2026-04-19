---
name: lg-airtable-operations
description: Referencia de las tools MCP para operar contra Airtable de Let Growth (clients, campaigns, todo_lists). Todas las tools pasan por el MCP `letgrowth`.
---

# Let Growth Airtable Operations

Referencia de las tools MCP para operar con Airtable de Let Growth. Todas las llamadas pasan por el MCP `letgrowth`, no se accede a Airtable directo.

**Cuando usar:** necesitas consultar clientes, listas, o cualquier dato de Airtable de Let Growth.

---

## Flujo de aprobacion de listas

**Todas las listas nuevas se crean con `search_workflow_process = "NEEDS_HUMAN_REVIEW"`.** Esto lo hacen automaticamente las tools `lg1_list_create_search` y `lg1_list_create_import`.

El humano revisa en Airtable y cambia el status cuando aprueba. No hay que notificar ni esperar confirmacion en chat.

Al crear listas: usa las tools (setean los workflow fields auto) y luego informa al humano cuantas listas se crearon y para que cliente.

---

## Tools de lectura

### `lg1_client_get`

Info de un cliente por dominio.

```
args: { "domain": "cliente.com" }
```

Devuelve:
```json
{
  "found": true,
  "id": "recXXX",
  "domain": "cliente.com",
  "name": "Cliente",
  "icp_summary": "...",
  "is_laundry": false,
  "is_serviced": true,
  "client_type": "...",
  "curr_week_new_leads_needed": 50,
  "next_week_new_leads_needed": 100
}
```

- `is_laundry=true` → usar `lg1_list_create_search`
- `is_laundry=false` → usar `lg1_list_create_import`
- **Siempre** verificar `is_serviced=true` antes de crear listas

### `lg1_clients_needing_leads`

Clientes activos con leads pendientes.

```
args: { "week": "current" }   // o "next"
```

Filtra: `is_serviced=1`, leads_needed>0, no `USED_FOR_OPS`.

### `lg1_client_campaigns`

Campanas de Smartlead del cliente.

```
args: { "domain": "cliente.com", "active_only": true }
```

El `name` de la campana es el **numero SL** que se pasa como `sl_campaign_number` en `list_create_*`.

### `lg1_client_lists`

Listas recientes del cliente con metricas.

```
args: { "domain": "cliente.com", "days": 60, "limit": 15, "only_in_progress": false }
```

Metricas clave: `search_good_fit_and_location_match`, `search_re_valid_contacts`, `search_added_to_campaign`.

Para chequear listas en progreso: `only_in_progress: true`.

---

## Tools de escritura

### `lg1_list_create_search`

Crea SEARCH_LIST (Google/Facebook/Google Maps). Para lavanderias y similar. Ver skill `lg-search-list` para los campos especificos.

### `lg1_list_create_import`

Crea IMPORT_LIST (IMPORT_CSV) desde URL Apollo o Sales Nav. Ver skill `lg-import-list`.

---

## Tablas principales (referencia)

| Tabla | Proposito |
|-------|-----------|
| `clients` | Clientes de Let Growth |
| `todo_lists` | Listas de leads (IMPORT_LIST, SEARCH_LIST, POOL_LIST) |
| `sl_campaigns` | Campanas de Smartlead |
| `contacts` | Contactos/leads |

Acceso directo a Airtable **NO** esta disponible para el equipo — todo pasa por las tools MCP. Si necesitas una operacion que no esta expuesta, pedirle al equipo de plataforma que la agregue al MCP.

---

## Notas importantes

1. **Las tools devuelven texto JSON.** Parsea con cuidado.
2. **Record IDs** empiezan con `rec` — no los necesitas normalmente porque las tools resuelven por dominio/nombre.
3. **Rate limits**: Airtable tiene 5 req/s por base. Las tools hacen retry automatico.
4. **Errores**: si una tool falla, el resultado tendra `isError: true` con un mensaje descriptivo. Leelo antes de reintentar.
