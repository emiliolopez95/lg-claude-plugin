---
name: lg2-discovery-create
description: Crear un discovery (import source) en LG2 desde una URL de LinkedIn Sales Navigator. Equivalente al lg-import-list de LG1, pero para LG2 (lg2_icp_discoveries en Postgres). Valida URL y evita duplicados.
---

# LG2 — Create Discovery (LinkedIn Sales Nav)

Crear un discovery tipo LinkedIn Sales Nav para un ICP en LG2.

**Cuando usar:** te piden crear un discovery, o crear una "import list" para un cliente LG2, o cargar una URL de Sales Nav a LG2.

**No usar para:** LG1. Si el cliente esta en LG1 usar `lg-import-list` en vez.

---

## Diferencia con LG1

| | LG1 | LG2 |
|---|---|---|
| Concepto | `todo_list` con `search_type="IMPORT_CSV"` | `lg2_icp_discovery` |
| Sistema | Airtable | Postgres |
| Skill equivalente | `lg-import-list` | este skill |
| Tool de creacion | `list_create_import` | `lg2_discovery_create` |
| Corre | workflow de scraping | discover stage del pipeline |

---

## Flujo

```
1. Confirmar que el cliente esta en LG2
2. Resolver el ICP
3. Revisar discoveries existentes (no duplicar)
4. Construir y validar URL de Sales Navigator
5. Validar volumen (<2,500) y calidad (>=50% fit)
6. Crear discovery con lg2_discovery_create
7. Confirmar al usuario
```

---

## Paso 1: Confirmar LG2

```
tool: client_locate
args: { "domain": "cliente.com" }
```

- `recommended_system = "lg2"` o `"both"` → continuar
- `recommended_system = "lg1"` → usar `lg-import-list` en vez, no este skill
- `recommended_system = "unknown"` → cliente no existe, frenar

---

## Paso 2: Resolver el ICP

```
tool: lg2_icp_list
args: { "client_domain": "cliente.com" }
```

Logica:
- 1 ICP `status: "active"` → usar ese
- >1 ICPs activos → preguntar al usuario cual usar (salvo que el contexto lo deje claro)
- 0 ICPs activos → frenar, avisar al usuario

---

## Paso 3: Ver discoveries existentes

```
tool: lg2_discovery_list
args: { "icp_id": "<icp_uuid>" }
```

Para:
1. Ver que URLs ya existen (no duplicar)
2. Entender el patron de discoveries del ICP (nombres, `source_type`, stats pasados en `last_run_stats`, hit counts)

Si tratas de crear con una URL que ya existe, `lg2_discovery_create` va a fallar. Mejor detectarlo aqui.

---

## Paso 4: Construir la URL de Sales Nav

Mismo proceso que LG1:
- Skill de referencia: `lg-sales-nav-url-builder` (en este plugin)
- Tools:
  - `sales_nav_typeahead` — resolver IDs de TITLE, COMPANY, INDUSTRY, GEOGRAPHY, etc.
  - Construir la URL con el encoding correcto (ver skill arriba)

Maximizar cobertura del ICP en una sola URL antes de dividir.

---

## Paso 5: Validar

### Volumen

```
tool: sales_nav_search_by_url
args: { "url": "<sales-nav-url>", "start": 0, "count": 25 }
```

Criterios:
- `data.pagination.total` < 2,500 (limite tecnico)
- Ideal > 1,500

Si > 2,500, dividir (ver `lg-sales-nav-url-builder` — mismas reglas de split).

### Calidad

```
tool: scrape_first_page
args: { "url": "<sales-nav-url>", "count": 25 }
```

Validar **>= 50% good fit** para el ICP del cliente (leer el `ai_match_criteria` del ICP como referencia).

**Rate limited: 1 call / 10s.** Usar con moderacion.

---

## Paso 6: Crear el discovery

```
tool: lg2_discovery_create
args: {
  "icp_id": "<icp_uuid>",
  "url": "https://www.linkedin.com/sales/search/people?query=...",
  "name": "Enterprise USA (LinkedIn)"
}
```

El tool:
1. Verifica que el ICP existe
2. Detecta `source_type` de la URL (people/lead → `linkedin_sales_leads`; company/account → `linkedin_sales_accounts`)
3. Verifica que no haya duplicado con la misma URL
4. Inserta con `run_status: "idle"` (el discover stage lo recoge automatico)

Devuelve:
```json
{
  "created": true,
  "detected_source": "LinkedIn Sales Nav - People",
  "discovery": { id, name, source_type, run_status, config, ... },
  "icp": { id, name, client_domain, status }
}
```

---

## Paso 7: Confirmar al usuario

Decirle:
- ID del discovery creado
- `source_type` detectado
- Que el pipeline `discover` lo va a recoger automaticamente cuando el ICP este `active`
- Para ver progreso: `lg2_discovery_get` con el id

---

## Notas importantes

1. **Solo URLs de Sales Nav hoy.** Apollo y otros sources no estan aun en LG2 — si el usuario pega URL de Apollo, avisarle que use LG1 (`lg-import-list`) o espere a que agreguemos soporte.
2. **El runner del `discover` stage requiere que el ICP este `active`.** Si esta `draft` o `paused`, el discovery se crea pero no corre hasta que alguien lo active.
3. **Config del discovery** solo tiene `{ import_url }` por ahora. No hay filtros adicionales — todo vive en la URL misma.
