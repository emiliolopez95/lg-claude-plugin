# todo_list_fields

Campos para crear registros en `todo_lists`. Referenciado desde `lg-import-list.md` y `lg-search-list.md`.

---

## name

Formato: `[Cliente] - [Roles] [Ubicacion]`

Si divides por volumen, especifica la subdivision.

**Ejemplos:**
- `"Circulo Dorado - Compras/RRHH/SST Antioquia"`
- `"TechCorp - Directors/VPs California"`
- `"Cliente - RRHH Antioquia sin Medellin"`

---

## clients

Dominio del cliente.

**Ejemplos:**
- `["boconcept.com"]`
- `["starlaundromats.com"]`

---

## sl_campaign

ID numerico de Smartlead (no el record ID de Airtable).

**Ejemplos:**
- `[7368121]`
- `[128421]`

---

## campaign_language

Idioma para el outreach. Basado en el ICP: que idioma habla la gente a quien vamos a escribir?

**Valores:** `"SPANISH"`, `"ENGLISH"`

---

## manual_location

**Si is_laundry = 0, siempre dejar `manual_location` vacio (sin valor).** Airtable tiene un location default para cada cliente que funciona perfecto para los que no son laundries.

Filtro de ubicacion para AI. Despues de recolectar resultados, AI usa esto para determinar si cada empresa esta en la zona.

**Balance importante:** muy amplio = llegan resultados que no son fit; muy chico = pocos resultados.

Debe reflejar exactamente el area de servicio del cliente. Puedes combinar multiples areas especificas (barrios, ciudades, zonas). Para negocios locales (ej: laundry), se especifico con los barrios/zonas que cubren.

Si es en USA siempre poner el estado de la ubicacion ej: "Brooklyn NY, Manhattan NY", etc..

**Ejemplos:**
- `"Antioquia, Colombia"`
- `"Sweden"`
- `"Brooklyn NY, Queens NY, Manhattan NY"`
- `"Coral Gables FL, Brickell FL, Downtown Miami FL"`

---

## manual_ai_match_string

**IMPORTANTE: Este campo evalua la EMPRESA, NO el contacto.**

**Siempre dejar `manual_ai_match_string` vacio (sin valor).** Airtable tiene un prompt default para cada cliente que funciona perfecto.

---

## contact_types

Single select que define que tipo de contactos buscar.

| Valor | Uso |
|-------|-----|
| `ALL` | People + generic emails (para ICPs menos formales como vets, spas) |
| `PEOPLE_ONLY` | Solo contactos con nombre/decision-makers (para ICPs formales) |
| `GENERIC_ONLY` | Solo emails publicos/genericos como info@ (requerido para Australia/Canada por leyes de cold email) |

---

## location_type

Single select que determina a que nivel se hace el test de location match.

| Valor | Uso |
|-------|-----|
| `ANY` | Location match a nivel de empresa. El AI verifica si la empresa tiene presencia en la ubicacion objetivo. **Siempre usar para Search Lists.** |
| `COMPANY` | Igual que ANY — location match a nivel de empresa. |
| `INDIVIDUAL` | Location match por contacto, no por empresa. Salta el test de location del AI a nivel empresa (auto-pasa). **Usar solo para Import Lists** donde los contactos vienen de LinkedIn con datos de ubicacion individual. |

---

## search_roles

Titulos de trabajo separados por coma. Usar 4-5 roles amplios de decision-makers. No ser muy especifico a menos que el cliente lo requiera.

**Debe estar en el idioma del mercado objetivo** (el idioma que usa el ICP).

Se usa para buscar en Google: `"{domain} {role1} OR {role2} OR ..."`.

**Ejemplos:**
- `"Owner,CEO,Founder,Managing Director,Director"`
- `"Dueno,Gerente,Director,Propietario,Fundador"`

---

## validate_roles_ai_string (opcional)

Prompt de AI para validar si el titulo de trabajo de un contacto pasa criterios estrictos de roles. La mayoria de clientes no necesitan esto — solo usar cuando el cliente es muy especifico sobre que roles quiere (ej: "presidents pero NO vice presidents").

### Como funciona

1. Se ejecuta DESPUES de encontrar contactos y verificar emails
2. GPT recibe el job_title de cada contacto y el validation string
3. Filtra estrictamente: si el string dice "not VP", los VPs son excluidos
4. Si este campo esta vacio, todos los contactos con emails validos pasan automaticamente

### Cuando usar

- Cliente dice "solo owners y founders, no managers" → usarlo
- Cliente dice "presidents pero no vice presidents" → usarlo
- Cliente no tiene preferencias estrictas de roles → no usarlo

---

## search_workflow_process

Siempre `"NEEDS_HUMAN_REVIEW"`.

---

## search_workflow_process_status

Siempre `"QUEUED"`.

---

## agent_confidence

Score del 1 al 10 que indica que tan confiado esta el agente de que la lista esta bien construida y va a producir buenos leads en cantidad suficiente.

| Score | Significado |
|-------|-------------|
| 9-10 | Muy confiado. ICP claro, filtros validados con typeahead, volumen verificado <2,500, titulos exactos. |
| 7-8 | Confiado. Buenos filtros pero algun factor de incertidumbre (mercado nuevo, titulos aproximados). |
| 5-6 | Moderado. Algunos supuestos sin validar, o ICP menos definido. |
| 3-4 | Bajo. Experimental, filtros genericos, o volumen incierto. |
| 1-2 | Muy bajo. Lista de prueba o alta incertidumbre. |

**Ejemplos:**
- `9` — ICP super claro (healthcare compliance), titulos validados, volumen ~2K verificado
- `6` — Primera lista para cliente nuevo, asumiendo industrias sin confirmar con el cliente
- `4` — Mercado nicho, pocos resultados, titulos en espanol sin typeahead

---

## agent_notes

Notas libres del agente sobre la lista. Incluye:

- **Razonamiento de confidence** — por que el score es alto/bajo
- **Decisiones tomadas** — por que se eligieron ciertos filtros vs otros
- **Lessons learned** — que funciono/no funciono en listas similares
- **Warnings** — riesgos potenciales o cosas a monitorear
- **Ideas para mejorar** — ajustes para proximas listas

**Ejemplos:**

```
Confidence 8: ICP bien definido (healthcare compliance), pero primera lista para este cliente. Titulos de Compliance Officer/Director validados con typeahead. Volumen ~2K esta bien. Watch: ver si los leads tienen engagement en LinkedIn (posted recently).
```

```
Confidence 5: Cliente pidio "marketing agencies" pero el ICP es vago. Use filtros genericos de industry. La proxima vez pedir mas detalles sobre tamano de agencia y servicios especificos.
```
