---
name: lg-search-list
description: Crea un SEARCH_LIST (Google/Facebook/Google Maps) en Airtable para clientes de lavanderia. Construye los campos especificos y llama a la tool MCP `lg1_list_create_search`.
---

# Let Growth Search List Creation

Crear Search Lists usando la tool `lg1_list_create_search` del MCP.

**Cuando usar:** crear una lista tipo SEARCH_LIST para un cliente de lavanderia.

**Antes de empezar:** completar el skill `lg-create-list-for-client` (obtiene client info, campaign number, ICP, contexto de listas).

**Referencia de campos comunes:** ver `references/todo-list-fields.md` en el plugin.

---

## Paso 1: Construir los args

La tool `lg1_list_create_search` recibe:

```json
{
  "name": "Cliente - Roles Ubicacion",
  "client_domain": "cliente.com",
  "sl_campaign_number": 7368121,
  "campaign_language": "SPANISH",
  "manual_location": "Brooklyn NY, Queens NY",
  "contact_types": "ALL",
  "search_roles": "Owner,CEO,Manager",
  "agent_confidence": 8,
  "agent_notes": "Confidence 8: ICP claro, primera lista para cliente nuevo.",
  "search_type": "FACEBOOK",
  "search_places": "Williamsburg,Bushwick,Park Slope,Flatbush",
  "search_keywords": "spa,gym,salon,hotel",
  "search_gl": "us",
  "search_hl": "en"
}
```

**Campos comunes** (ver `todo-list-fields.md` del plugin):
- `name`, `client_domain`, `sl_campaign_number`, `campaign_language`
- `manual_location` (ver nota abajo)
- `contact_types` (ALL / PEOPLE_ONLY / GENERIC_ONLY)
- `search_roles`, `agent_confidence`, `agent_notes`

**Campos especificos** (abajo):
- `search_type`, `search_places`, `search_keywords`, `search_gl`, `search_hl`

**Defaults automaticos** (no los pongas tu):
- `search_workflow_process = "NEEDS_HUMAN_REVIEW"`
- `search_workflow_process_status = "QUEUED"`
- `location_type = "ANY"`

---

## Campos especificos de Search List

### `search_type`

| Valor | Cuando usar |
|-------|-------------|
| `GOOGLE` | Default, busqueda general |
| `FACEBOOK` | Negocios locales con presencia en FB |
| `GOOGLE_MAPS` | Negocios fisicos con ubicacion |

Por ahora solo usemos `FACEBOOK` para todas las SEARCH_LIST.

### `search_places`

Lugares de busqueda (comma-separated). Lugares especificos dentro del area de servicio. Mas lugares = mas resultados pero mas costo.

**Para lavanderias (radio-based):**
- `"Billings MT, Laurel MT, Lockwood MT"`

**Para LATAM:**
- `"Medellin,Envigado,Bello,Itagui,Sabaneta"`

Tips:
- Usa barrios/zonas/condados, no solo el nombre de la ciudad
- Mas lugares = mas creditos de Serper (total queries = keywords x lugares)
- `manual_location` puede ser amplio (pais) o estrecho (radio) — ajustalo al area real
- `search_places` debe ser granular para encontrar resultados que no aparecerian solo con el nombre de la ciudad principal
- Nombres distintos + estado. Si comparten la palabra principal (e.g. `["Columbia", "North Columbia", "West Columbia"]`), Google dara resultados repetidos
- Siempre agregar el estado con 2 letras: `Billings MT, Laurel MT, Lockwood MT`
- Ordenar de mayor a menor potencial

### `search_keywords`

Keywords de busqueda (comma-separated). Usa el idioma local del mercado.

Para lavanderias en USA (usar TODOS siempre):

```
spa,medspa,nail spa,laser clinic,beauty clinic,massage,barbershop,salon,laser,gym,fitness,yoga,aquatics,pool,boxing,crossfit,pilates,animal shelter,wildlife center,dog daycare,veterinary,vet clinic,animal hospital,equestrian center,dentist,dental clinic,chiropractic,physical therapy,rehab center,hospital,medical center,clinic,senior living,assisted living,cleaning service,commercial cleaning,janitorial,waste management,fire restoration,restoration,vacation rental,short term rental,airbnb,property management,vrbo,school,daycare,childcare,preschool,student housing,residence hall,church,shelter,fire department,fire station,police department,detention center,correctional,hotel,motel,inn,lodge,hostel,suites,resort,casino,conference center
```

Para otros (latam): 5-10 keywords que den muchos resultados para su industria.

### `search_gl` / `search_hl`

`search_gl`: Google geolocation. ISO 3166-1 alpha-2 minusculas. Vacio para US.
`search_hl`: codigo de idioma para Google.

| Pais | search_gl | search_hl |
|------|-----------|-----------|
| USA | `us` | `en` |
| Colombia | `co` | `es` |
| Mexico | `mx` | `es` |
| Argentina | `ar` | `es` |
| Chile | `cl` | `es` |
| Peru | `pe` | `es` |

---

## Paso 2: Crear la lista

```
tool: lg1_list_create_search
args: { ...los campos de arriba... }
```

Devuelve `{ created: true, id, name, search_type, client_domain }`.

---

## `manual_location` vs `search_places`

Estos dos campos trabajan juntos pero tienen propositos diferentes:

- `search_places` = **expande la busqueda**. Lugares usados para generar queries (cada lugar x cada keyword = un query). Objetivo: maximizar resultados relevantes.
- `manual_location` = **filtra los resultados**. AI usa esto para determinar si cada empresa esta en la zona. Debe reflejar el area de servicio real del cliente.

### Ejemplo: Pais/Region (muebles en Antioquia)

- **search_places:** `"Medellin,Envigado,Bello,Rionegro,Itagui"`
- **manual_location:** `"Antioquia, Colombia"`

### Ejemplo: Radio (lavanderia en Brooklyn)

- **search_places:** `"Williamsburg,Bushwick,Greenpoint,Park Slope,Bay Ridge,Flatbush"`
- **manual_location:** `"Within 20 miles of Brooklyn, NY"`

### Ejemplo: Pais entero (disenadores en Suecia)

- **search_places:** `"Stockholm,Gothenburg,Malmo,Uppsala,Linkoping"`
- **manual_location:** `"Sweden"`
