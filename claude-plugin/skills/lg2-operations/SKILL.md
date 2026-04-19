---
name: lg2-operations
description: Referencia completa de las tools MCP para operar contra LG2 (Postgres). Cubre ICPs, lists, custom field library, y routing cross-sistema.
---

# LG2 Operations — MCP Tools Reference

Referencia de las tools MCP para operar con LG2. Todas pasan por el MCP `letgrowth`, no hay acceso directo a la DB.

---

## Routing cross-sistema

### `client_locate`

**Cuando usar:** al inicio de cualquier flujo si el usuario no especifico LG1 vs LG2. Detecta donde vive el cliente.

```
args: { "domain": "cliente.com" }
```

Returns:
```json
{
  "is_lg2": boolean,                     // Postgres clients.is_lg2 flag (hard discriminator)
  "in_airtable": boolean,                // present in Airtable (LG1 onboarding); every client has this
  "recommended_system": "lg1" | "lg2" | "unknown",
  "lg2_icp_count": number,               // informational
  "lg1_client"?: { name, is_laundry, client_type }
}
```

Interpretacion:
- `"lg2"` → usar skills `lg2-*` (is_lg2=true gana siempre)
- `"lg1"` → usar skills `lg-*` (cliente activo en Airtable, no marcado como LG2)
- `"unknown"` → cliente no existe, frenar

---

## ICPs (lg2_icps)

### `lg2_icp_list`

Lista ICPs, opcional filtrado por cliente.

```
args: { "client_domain": "cliente.com" }   // opcional
```

Returns array con `id`, `client_domain`, `name`, `status`, `ai_match_criteria`, `ai_location_criteria`, `created_at`.

---

## Discoveries (lg2_icp_discoveries)

### `lg2_discovery_list`

Lista discoveries de un ICP, con `asset_hits` y `people_hits` counts.

```
args: { "icp_id": "<uuid>" }
```

### `lg2_discovery_get`

Detalle de un discovery individual.

```
args: { "discovery_id": "<id>" }
```

### `lg2_discovery_create`

Crea un discovery desde una URL de LinkedIn Sales Nav. Auto-detecta `source_type`, valida duplicados.

```
args: {
  "icp_id": "<uuid>",
  "url": "https://www.linkedin.com/sales/search/people?...",
  "name": "Enterprise USA (LinkedIn)"
}
```

Ver skill `lg2-discovery-create` para el flujo completo.

**Solo Sales Nav por ahora** — Apollo y otros sources no estan soportados todavia.

---

## Lists (lg2_lists)

### `lg2_list_list`

Lista todas las lg2_lists, opcional filtrado.

```
args: {
  "client_domain": "cliente.com",          // opcional
  "status": "draft" | "dispatching" | "done"  // opcional
}
```

### `lg2_list_get`

Detalle de una lista con contadores de contactos.

```
args: { "list_id": 123 }
```

Returns el row completo de `lg2_lists` + `contact_counts: { total, dispatched, pending, dispatch_errors }`.

### `lg2_list_create`

Crea el shell de la lista. **No agrega contactos** — eso lo hace el pipeline.

```
args: {
  "client_domain": "cliente.com",
  "name": "Acme - SMB Mexico",
  "channel": "bison" | "smartlead" | "li2",
  "external_campaign_id": "12345",
  "language": "english" | "spanish",       // opcional, default "english"
  "status": "draft" | "dispatching" | "done",  // opcional, default "draft"
  "custom_fields_config": [                // opcional
    { "library_id": 42 },
    { "name": "custom_field", "ai_instruction": "..." }
  ]
}
```

Ver skill `lg2-list-create` para el flujo completo.

### `lg2_list_update`

Edits basicos (nombre, status, language).

```
args: {
  "list_id": 123,
  "name": "...",         // opcional
  "status": "...",       // opcional
  "language": "..."      // opcional
}
```

Al menos uno de los tres debe estar.

### `lg2_list_update_config`

Reemplaza el `custom_fields_config`. Invalida cache de enrichment — contactos se re-enrichan.

```
args: {
  "list_id": 123,
  "custom_fields_config": [...] | null
}
```

---

## Custom Field Library (lg2_list_custom_field_library)

### `lg2_custom_field_library`

Lista merge-tag fields reusables. Incluye globales + client-scoped si pasas `client_domain`.

```
args: { "client_domain": "cliente.com" }   // opcional
```

Returns array con `id`, `name`, `description`, `ai_instruction`, `scope`.

**Uso tipico:** antes de crear/editar una lista con `custom_fields_config`, ver que fields reusables hay. Referencias por `library_id` en vez de inline cuando existe uno que encaja.

---

## Cosas que NO estan expuestas (por diseno)

- Crear/editar ICPs (`lg2_icps`) — se configuran desde el admin panel
- Editar buyer personas — admin panel
- Operaciones del pipeline (runner, dispatcher) — son admin/ops
- Agregar contactos a listas — el pipeline lo hace auto, no el equipo

Si necesitas alguna de esas, pediselo al equipo de plataforma.
