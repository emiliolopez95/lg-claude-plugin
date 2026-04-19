---
name: lg2-list-create
description: Flujo paso a paso para crear un lg2_list (curated list) apuntado a una campana de Bison/Smartlead/li2. Incluye eleccion de canal, external_campaign_id, merge-tag config.
---

# LG2 — Create List (shell)

Crear un `lg2_list` (la "cascara") que el pipeline va a llenar de contactos calificados.

**Cuando usar:** el usuario quiere crear una nueva lista LG2 para un cliente.

**Antes de empezar:** tener el `client_domain` del cliente.

---

## Paso 1: Verificar que el cliente esta en LG2

```
tool: client_locate
args: { "domain": "cliente.com" }
```

- `recommended_system = "lg2"` → continuar
- `recommended_system = "lg1"` → este cliente esta en LG1, el usuario quizas se confundio. Avisarle antes de continuar.
- `recommended_system = "unknown"` → cliente no existe, frenar.

Si esta en LG2, ver los ICPs:

```
tool: lg2_icp_list
args: { "client_domain": "cliente.com" }
```

(Informacional; no lo necesitas para crear la lista, pero ayuda a entender el contexto.)

---

## Paso 2: Ver listas existentes (evitar duplicados)

```
tool: lg2_list_list
args: { "client_domain": "cliente.com" }
```

Si ya hay una lista activa (`status = "draft"` o `"dispatching"`) con nombre similar, avisar al usuario antes de crear otra.

---

## Paso 3: Decidir los parametros

### `name`
Formato sugerido: `"[Cliente] - [Segmento] [Canal/Idioma]"`.
- `"Acme - SMB Mexico (Bison ES)"`
- `"Acme - Enterprise USA (Smartlead EN)"`

### `channel`

| Valor | Cuando usar |
|-------|-------------|
| `bison` | Bison (cold email nuevo) |
| `smartlead` | Smartlead (cold email viejo) |
| `li2` | LinkedIn outreach via li2 |

**Preguntar al usuario** si no especifica. Cada canal requiere su propio `external_campaign_id`.

### `external_campaign_id`
Es el ID de la campana en la plataforma nativa (no recId de Airtable). 

**Preguntar al usuario** — no hay tool para listar campanas disponibles (se trae desde el admin panel del canal respectivo). El usuario debe pegar el ID que ya creo en Bison/Smartlead/li2.

### `language`
Default: `"english"`. Otros: `"spanish"`, etc. El AI de enrichment lo usa para traducir merge tags.

### `status`
Default: `"draft"`. Dejar en draft hasta que el equipo apruebe el dispatch.

### `custom_fields_config` (opcional)

Merge-tag fields que el AI genera per-contacto. Cada entrada puede ser:

**A. Referencia a la library** (preferido, edits centralizados):
```json
{ "library_id": 42 }
```

**B. Inline (ad-hoc)**:
```json
{ "name": "icebreaker_custom", "ai_instruction": "Write a 2-line personalized opener based on..." }
```

**Antes de construir el array**, ver que hay en la library:

```
tool: lg2_custom_field_library
args: { "client_domain": "cliente.com" }
```

Devuelve los fields globales + client-scoped. Si hay uno que encaja, usalo por `library_id`.

---

## Paso 4: Crear la lista

```
tool: lg2_list_create
args: {
  "client_domain": "cliente.com",
  "name": "Acme - SMB Mexico (Bison ES)",
  "channel": "bison",
  "external_campaign_id": "12345",
  "language": "spanish",
  "custom_fields_config": [
    { "library_id": 42 },
    { "name": "custom_hook", "ai_instruction": "..." }
  ]
}
```

Devuelve `{ created: true, id, name, client_domain, channel, status: "draft", ... }`.

---

## Paso 5: Confirmar al usuario

Decirle:
- Que la lista se creo con id X
- Que esta en `draft` (el pipeline empezara a enrollar cuando este lista)
- Si tiene que cambiar algo (nombre, language, config de merge tags), usar `lg2_list_update` o `lg2_list_update_config`
- Para ver progreso: `lg2_list_get` con el id

---

## Notas importantes

1. **El equipo NO agrega contactos a mano.** El pipeline los auto-enrola segun `buyer_persona.list_id`. Crear la lista = crear el shell. Llenarla = pipeline.
2. **`status = "done"`** significa que ya se dispatcho todo lo que se iba a dispatchar. No es un toggle para activar — el flujo natural es `draft → dispatching → done`.
3. **`custom_fields_config`** que cambia invalida el cache de enrichment. Los contactos se re-enriquen con los nuevos fields en el siguiente run.
