---
name: bison-campaigns
description: Crear, editar, pausar, reanudar, archivar o borrar campañas de Bison (email outreach). Cross-sistema — sirve tanto para clientes LG1 como LG2 (ambos usan Bison como canal de email).
---

# Bison Campaigns — Admin

Skill para crear y controlar campañas en **Bison** (la plataforma de email outreach que usamos). Bison es el canal de envío — tanto LG1 como LG2 pueden apuntar `lg2_list.external_campaign_id` o `todo_list` downstream a una campaña de Bison.

**Cuando usar:**
- "crea una campaña nueva en Bison para cliente X"
- "duplica la campaña 14 y ponle otro nombre"
- "pausa / reanuda / archiva la campaña Y"
- "sube el limite diario de emails de la campaña Z"

---

## Glosario rápido

| Término | Qué es |
|---|---|
| **Workspace** | Tenant de Bison — un cliente = un workspace. Cada workspace tiene su propio set de campañas, senders, tags. |
| **Campaign** | Una secuencia de emails enviada a un pool de leads. Tiene status, limites diarios, tracking settings, sequence (pasos), senders. |
| **Sender** | Email de envío (mailbox) configurado en el workspace. Una campaña puede tener varios. |
| **Sequence step** | Email individual dentro de la secuencia (steps 1, 2, 3...). Cada uno tiene subject, body, days-since-previous. |
| **Schedule** | Ventana de envío (días + horas). |
| **Status** | `draft | launching | active | stopped | completed | paused | failed | queued | archived | deleted` |

**Relación con lg2 lists:** una `lg2_list` con `channel=bison` apunta a una campaña de Bison por `external_campaign_id`. Cuando el pipeline enrola contactos en la lg2_list, el dispatcher los empuja a los leads de esa campaña Bison.

---

## Tools MCP disponibles

Todas aceptan **`client_domain` O `workspace_id`** (pasa uno, no ambos — `client_domain` es lo normal, el resolver busca el workspace).

| Tool | Qué hace |
|---|---|
| `bison_campaign_create` | Crea campaña vacía (status=draft). Solo pide `name` + `type` opcional. NO configura sequence/senders/schedule. |
| `bison_campaign_duplicate` | **Preferido** para casos reales. Clona una campaña existente (sequence + schedule + senders + settings) como draft. |
| `bison_campaign_update` | Edita settings: name, max_emails_per_day, max_new_leads_per_day, plain_text, open_tracking, reputation_building, can_unsubscribe, sequence_prioritization. No cambia status. |
| `bison_campaign_pause` | Pausa una campaña corriendo. |
| `bison_campaign_resume` | Reanuda una paused. También "lanza" una draft (si ya tiene senders + schedule + sequence). |
| `bison_campaign_archive` | Soft-archive (reversible desde UI). **Preferido sobre delete.** |
| `bison_campaign_delete` | **PERMANENTE, no reversible.** Solo usar si el usuario dice "borrar permanentemente". |

---

## Flujos típicos

### A. Crear campaña nueva (desde cero)

Raro — casi nunca empezas en blanco. Si es nueva:

```
1. bison_campaign_create(client_domain="X.com", name="...")  → devuelve campaign_id (draft)
2. [EN UI DE BISON o via otras tools no expuestas aun] configurar:
   - Attach sender emails
   - Set schedule
   - Add sequence steps
3. bison_campaign_resume(campaign_id=N)  → lanza
```

**Nota:** sequence steps, schedule, y sender attach **no estan expuestos aun** como tools MCP. Para esas configuraciones el equipo tiene que ir a la UI de Bison por ahora.

### B. Duplicar una campaña existente (común)

```
1. Averiguar el ID de la campaña plantilla (mirando en UI, o preguntando al user)
2. bison_campaign_duplicate(client_domain="X.com", campaign_id=14)  → devuelve nueva campaign_id en draft
3. bison_campaign_update(campaign_id=nueva, { name: "New Name" })  → renombra si hace falta
4. Opcional: ajustar limites diarios con update
5. [En UI] ajustar sequence si hace falta
6. bison_campaign_resume(campaign_id=nueva)  → lanza
```

### C. Cambiar limites diarios

```
tool: bison_campaign_update
args: {
  "client_domain": "kudert.com",
  "campaign_id": 8,
  "max_emails_per_day": 500,
  "max_new_leads_per_day": 100
}
```

### D. Pause/Resume temporal

```
tool: bison_campaign_pause  args: { "client_domain": "X.com", "campaign_id": N }
# ... algo ...
tool: bison_campaign_resume args: { "client_domain": "X.com", "campaign_id": N }
```

### E. Limpieza

- Campaña vieja que queres sacar de la vista → `bison_campaign_archive` (reversible)
- Campaña de test que nunca vas a necesitar → `bison_campaign_delete` (PERMANENTE, confirmar con user primero)

---

## Notas importantes

1. **Campaign ID es workspace-scoped.** El id 14 en kudert NO es el mismo 14 en aguadcapital. Siempre pasar `client_domain` para que el resolver tome el workspace correcto.

2. **No hay `bison_campaign_list` todavia** (viene en Fase 1). Si el user no sabe el ID, tiene que ir a la UI de Bison a verlo. O preguntarle al admin.

3. **`resume` en draft = launch.** Es el mismo endpoint: PATCH /api/campaigns/{id}/resume. Bison internamente lo trata como "empezar a correr". Pero si falta config (sin sender/schedule/sequence) probablemente te dara error descriptivo.

4. **Delete es irreversible.** Bison tarda en borrar (background job) pero una vez iniciado, no hay undo. Preferir archive siempre que se pueda.
