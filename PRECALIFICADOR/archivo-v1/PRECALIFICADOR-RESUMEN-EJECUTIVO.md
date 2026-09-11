# PRECALIFICADOR — RESUMEN EJECUTIVO

**Versión:** 2.0 (Arquitectura dinámica)  
**Fecha:** 2026-09-10  
**Estado:** PARA APROBACIÓN INMEDIATA

---

## I. QUÉ ES (EN 30 SEGUNDOS)

Una **microapp de autoservicio** que:
- Corre en Vercel (iframe independiente)
- Hace preguntas cerradas (botones grandes, 1 por pantalla)
- NO pide SSN, documentos, datos bancarios
- Recolecta: necesidad, composición, prioridades, contacto
- Agenda automáticamente cita con Lorena (Google Calendar)
- Da a Lorena inteligencia estructurada para la llamada de 15 min

---

## II. ARQUITECTURA CLAVE: "FLUJOS COMO DATOS"

**NO:** 6 apps separadas, cada una con preguntas hardcodeadas.

**SÍ:** UN motor genérico que lee flujos desde JSON.

```
Flujo 1 (JSON) -> Motor genérico (PantallaDePregunta) -> Renderiza cualquier pregunta de cualquier flujo
Flujo 2 (JSON) -> mismo motor
Flujo 3 (JSON) -> mismo motor
...
```

**Ventaja:** Lorena agrega/edita flujos SIN código. Solo JSON en dashboard.

---

## III. STACK TÉCNICO (MÍNIMO)

| Parte | Tech |
|---|---|
| Frontend | Next.js 15 + React 19 |
| Host | Vercel |
| Backend | Next.js API Routes |
| DB | Supabase (PostgreSQL) |
| Auth | Supabase Auth |
| Calendar | Google Calendar API |
| Integración | iframe en 10MinutesWebsite |

---

## IV. BASE DE DATOS (3 TABLAS PRINCIPALES)

### 1. `flujos_configuracion`
```json
{
  "codigo": "familia_salud_01",
  "nombre": "Seguro para mí o mi familia",
  "configuracion": {
    "preguntas": [
      {
        "id": "q1",
        "texto": "¿Quién necesita cobertura?",
        "tipo": "opcion_unica",
        "opciones": [
          {"label": "Solo yo", "valor": "solo_yo", "siguiente": "q2"},
          {"label": "Pareja e hijos", "valor": "pareja_hijos", "siguiente": "q3"}
        ]
      }
    ],
    "alternativas": [
      {"id": "alt_a", "titulo": "Menor costo", "precio_desde": "$150/mes"},
      {"id": "alt_b", "titulo": "Equilibrio", "precio_desde": "$200/mes"},
      {"id": "alt_c", "titulo": "Mayor protección", "precio_desde": "$300/mes"}
    ]
  }
}
```

**¿Por qué JSON?** Lorena (o admin) puede cambiar preguntas editando este archivo en el dashboard. Sin código.

### 2. `sesiones`
Guarda: qué flujo, qué respuestas, dónde está ahora (en qué pregunta), datos de contacto.

### 3. `prospectos`
Cuando termina: nombre, email, teléfono, todas las respuestas formateadas, alternativa elegida, cita agendada.

---

## V. FLUJO DE USUARIO (TODO EN SERIE)

```
1. Usuario abre HOME
2. Elige 1 de 6 puertas ("Seguro para familia", "Económico", etc.)
3. Carga flujo correspondiente (JSON de flujos_configuracion)
4. Pantalla 1: "¿Quién necesita cobertura?"
5. Usuario elige -> Motor evalúa siguiente pregunta (JSON.siguiente)
6. Pantalla 2: Ej: "¿Cuál es tu edad?" (porque eligió "familia")
7. Intercaladas: pide nombre, luego 2 preguntas, luego email, luego 2 preguntas, luego teléfono
8. Después de ~6-8 preguntas: Resumen + "Elige alternativa"
9. Usuario elige: "B — Equilibrio"
10. "¿Contacto preferido? WhatsApp / Llamada"
11. Google Calendar: "¿Qué día/hora te va?"
12. Confirma cita
13. Pantalla de gracias + video de Lorena
14. TODO guardado en prospectos (Lorena ve en dashboard)
```

---

## VI. LOS 6 FLUJOS INICIALES (SON CONFIGURACIONES)

| # | Puerta | Objetivo |
|---|--------|----------|
| 1 | Seguro para mí/familia | Composición familiar, edades, prioridades |
| 2 | Busco opción económica | Persona ya tiene seguro, quiere bajar costo |
| 3 | Embarazada/cambios | Embarazo, nuevo nacimiento, cambios familiares |
| 4 | Trabajo por mi cuenta | Independientes, sin cobertura de empleador |
| 5 | Perdí cobertura/me mudé | Cambios de vida (mudanza, trabajo nuevo, etc.) |
| 6 | Dudas migración/SS | Seguro Social, ciudadanía, estatus (sin pedir docs) |

**Cada uno = 1 archivo JSON en `flujos_configuracion`**
---

## VII. COMPONENTES REACT (GENÉRICOS)

**El corazón:**

```tsx
<PantallaDePregunta 
  pregunta={preguntaActual}  // Del JSON
  onRespuesta={handleRespuesta}
  onNavegar={goToNextStep}
/>
```

Este componente renderiza TODO:
- Pregunta de opción múltiple → botones
- Pregunta de rango → slider o botones
- Campo de texto → input
- Alternativas → cards grandes

**Resto:**
- `RenderizadorOpciones` — dibuja botones según el JSON
- `CampoContacto` — guarda nombre/email/teléfono
- `PantallaAlternativas` — muestra 3 opciones
- `PantallaResumen` — resumen antes de cita
- `ProgresoBarra` — muestra dónde estás (3 de 8)

---

## VIII. DASHBOARD PARA LORENA

### Pantalla 1: Resumen
```
Nuevos prospectos hoy: 3
Citas agendadas (próx. 7d): 8
Tasa conversión (mes): 42%

Acciones pendientes:
- María López (familia) — HOY, 14:00
- Carlos Rodríguez (empresa) — MAÑANA
- [3 más...]
```

### Pantalla 2: Lista de prospectos
```
Filtros: [Todos] [Últimos 7d] [Estado]

| Nombre | Flujo | Fecha | Estado | Acción |
|--------|-------|-------|--------|--------|
| María López | Familia | 2026-09-10 | Confirmado | Ver |
| Pedro García | Económico | 2026-09-10 | Nuevo | Ver |
```

### Pantalla 3: Detalle de prospecto
```
María López
Email: maria@email.com
Teléfono: (305) 555-1234
Contacto: WhatsApp

Flujo: Seguro para familia
¿Quién necesita? → Pareja e hijos
¿Edad? → 31-50
¿Prioridad? → Costo
¿Presupuesto? → $200-400/mes
Alternativa elegida: B — Equilibrio

Cita: Martes 12 sep, 14:00 (Google Meet)
Notas internas: [editable]
```

### Pantalla 4: Editor de flujos
```
[Nuevo flujo] [Editar existente]

Código: familia_salud_01
Nombre: Seguro para mí o mi familia

Preguntas:
1. ¿Quién necesita cobertura?
   - Solo yo
   - Pareja e hijos
   - Familia extendida

[Agregar pregunta]
[Guardar cambios] [Preview]
```

---

## IX. REGLAS INMUTABLES (DEL CODEX)

**NUNCA pedir:**
- SSN (Social Security Number)
- Documentos migratorios
- Información bancaria
- Historial médico detallado

**SIEMPRE decir:**
- "Opciones que vale la pena explorar"
- "Para que Lorena revise esto contigo"
- "Una conversación de 15 minutos puede cambiar…"

**NUNCA afirmar:**
- "Cotización final"
- "Aprobado"
- "Ya estás asegurado"

**SIEMPRE usar:**
- Lenguaje humano (explorar, opciones, proteger, cobertura)
- Geografía cuando aplique (Florida, Miami, Doral)
- Idioma: en español
---

## X. DATOS QUE LORENA RECIBE (ESTRUCTURA FIJA)

```json
{
  "prospecto_id": "uuid",
  "nombre": "María López",
  "email": "maria@email.com",
  "telefono": "(305) 555-1234",
  "preferencia_contacto": "WhatsApp",
  "flujo_codigo": "familia_salud_01",
  "flujo_nombre": "Seguro para mí o mi familia",
  "respuestas": {
    "q1": {"pregunta": "¿Quién necesita?", "respuesta": "Pareja e hijos"},
    "q2": {"pregunta": "¿Edad?", "respuesta": "31-50"},
    "q4": {"pregunta": "¿Prioridad?", "respuesta": "Costo"},
    "q6": {"pregunta": "¿Presupuesto?", "respuesta": "$200-400/mes"},
    "q5": {"pregunta": "¿Cobertura actual?", "respuesta": "Sin cobertura"}
  },
  "alternativa_seleccionada": "B — Equilibrio",
  "nivel_intension": "Alta",
  "tiempo_total_minutos": 5.3,
  "cita_agendada": true,
  "cita_timestamp": "2026-09-12T14:00:00Z",
  "cita_enlace_google_meet": "https://meet.google.com/...",
  "estado": "confirmado"
}
```

**Lorena puede leerlo en 30 segundos y empezar la llamada sabiendo exactamente qué preguntó y qué le importa.**

---

## XI. INTEGRACIÓN CON 10MinutesWebsite

En la página del HOME, donde va el Precalificador:

```html
<iframe 
  src="https://precalificador.vercel.app/?tenant=ssv" 
  width="100%" 
  height="600"
  style="border: none; border-radius: 16px;"
/>
```

**Eso es TODO.** No se ejecuta script, no hay conflicto de CSS, solo el iframe.

---

## XII. ROADMAP (¿CUÁNTO TARDA?)

### Fase 1: SETUP + FLUJO 1 (Semana 1)
- Crear proyecto Next.js + Supabase
- Base de datos (3 tablas)
- Componente `PantallaDePregunta` genérico
- Flujo 1 JSON (Familia)
- Integración Google Calendar
- Deploy en Vercel
- Testing: 5+ usuarios completan flujo sin errores

**Criterio de éxito:** Lorena recibe 5 prospectos estructurados, cada uno con inteligencia completa, agendados en su calendario.

### Fase 2: Flujos 2-6 (Semana 2)
- Crear 5 JSONs más (replicar estructura)
- Testing de cada uno
- Validación con Lorena

### Fase 3: Dashboard completo (Semana 2)
- Lista de prospectos
- Filtros
- Detalle + notas
- Editor de flujos (Lorena edita JSON sin código)

### Fase 4: Optimizaciones (Semana 3-4)
- Analizar abandono
- IA sugiere mejoras de flujos (basada en respuestas guardadas)
- A/B testing de lenguaje

---

## XIII. DECISIÓN AHORA (SUPERADA — ver MASTER_BLUEPRINT.md v3.0)

**Opción A (Recomendada): Arquitectura completa desde MVP**
- Flujos dinámicos desde inicio
- Multitenant listo
- Lorena puede agregar flujos sin técnicos
- Tarda: 3-4 semanas
- Costo: medio

**Opción B: MVP hardcodeado, refactor después**
- Flujo 1 pegado en código
- Luego refactorizar a JSON
- Tarda: 2 semanas MVP
- Riesgo: cambios después son caros

**Recomendación:** Opción A. El overhead de tener flujos como JSON NO es tan grande, y el valor para Lorena es enorme (ella controla).

---

## XIV. SIGUIENTE PASO (SUPERADO)

Este documento fue reemplazado por `MASTER_BLUEPRINT.md` tras la entrevista completa con MAGO (incluye razonamiento de IA en vivo, motor OpenAI, y dashboard de auto-creación de Precalificadores). Se conserva solo como referencia histórica.

---

**Documento versión:** 2.0 (archivado)  
**Sustituido por:** `../MASTER_BLUEPRINT.md`
