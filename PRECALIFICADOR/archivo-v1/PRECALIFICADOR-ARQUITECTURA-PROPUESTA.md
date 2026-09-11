# PRECALIFICADOR — PROPUESTA DE ARQUITECTURA COMPLETA (ARCHIVADO)

**Versión:** 1.0  
**Propósito:** Definir arquitectura técnica, UX, flujos de datos y roadmap antes de escribir código  
**Fecha:** 2026-09-10  
**Estado:** SUPERADO — ver `../MASTER_BLUEPRINT.md` para la versión vigente (v3.0)

---

## I. DEFINICIÓN DEL PROYECTO

**¿Qué es el Precalificador?**

Una **microaplicación de autoservicio** que:
- Ayuda a personas a identificar qué seguro necesitan
- Conduce a través de un flujo simple: exploración → elección → personalización → revisión
- Recolecta información esencial sin parecer un formulario
- Genera un resumen precalificado para Lorena
- Agenda automáticamente una cita de ~15 minutos en Google Calendar de Lorena

**No es:**
- Un formulario de captación de leads
- Un chatbot conversacional
- Una cotización automática (no generamos precio final)
- Una "aprobación" (solo es revisión con Lorena)

**Es:**
- Un checkout asistido
- Una experiencia guiada de autoservicio
- Un recolector de inteligencia para Lorena
- Una puerta a la conversación de 15 minutos

---

## II. ARQUITECTURA TÉCNICA

### 2.1 Principio clave: Flujos como datos, no como código

**NO:** 6 apps separadas, cada una con sus preguntas hardcodeadas.

**SÍ:** UN motor genérico que lee flujos desde JSON (Supabase). Cada flujo = una configuración. Lorena puede agregar/editar flujos sin tocar código.

### 2.2 Stack Tecnológico

| Componente | Tecnología | Justificación |
|---|---|---|
| **Frontend** | React 19 + Next.js 15 | App interactiva, SSR, optimización automática |
| **Hosting** | Vercel | Deploy directo de Next.js, CDN global, serverless functions |
| **Backend** | Next.js API Routes + Supabase | Sin servidor separado, escalabilidad simple |
| **Base de datos** | Supabase (PostgreSQL) | SQL real, relaciones complejas, easy migrations |
| **Autenticación** | Supabase Auth (JWT) | Dashboard de Lorena seguro |
| **Almacenamiento** | Supabase Storage | Assets dinámicos, archivos multitenant |
| **Calendario** | Google Calendar API | Integración nativa, confirmación automática |
| **Iframe** | Integración en 10MinutesWebsite | Aislamiento CSS/JS, carga independiente |
### 2.2 Estructura de Carpetas (Propuesta)

```
precalificador/
├── app/
│   ├── (public)/
│   │   ├── layout.tsx
│   │   ├── page.tsx              # Landing inicial
│   │   ├── flujo/
│   │   │   ├── [flowId]/
│   │   │   │   ├── page.tsx      # Pantalla de pregunta
│   │   │   │   └── [stepId]/page.tsx
│   │   │   └── resumen/page.tsx
│   │   ├── contacto/page.tsx
│   │   ├── disponibilidad/page.tsx
│   │   ├── confirmacion/page.tsx
│   │   └── gracias/page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── dashboard/page.tsx    # Resumen para Lorena
│   │   ├── prospectos/page.tsx   # Lista de leads
│   │   ├── prospectos/[id]/page.tsx
│   │   ├── configuracion/page.tsx
│   │   └── settings/page.tsx
│   ├── api/
│   │   ├── flujos/
│   │   │   ├── [flowId]/route.ts
│   │   │   └── [flowId]/preguntas/route.ts
│   │   ├── sesiones/
│   │   │   ├── route.ts          # POST: nueva sesión
│   │   │   ├── [sessionId]/respuesta/route.ts
│   │   │   └── [sessionId]/contacto/route.ts
│   │   ├── prospectos/
│   │   │   ├── route.ts          # GET: lista (Lorena)
│   │   │   └── [prospectId]/route.ts
│   │   ├── calendario/
│   │   │   └── disponibilidad/route.ts
│   │   └── auth/
│   │       ├── login/route.ts
│   │       └── callback/route.ts
├── components/
│   ├── Flujo/
│   │   ├── PantallaDePregunta.tsx  # Componente genérico (el corazón)
│   │   ├── RenderizadorOpciones.tsx # Botones, rangos, opciones
│   │   ├── CampoContacto.tsx       # Nombre, email, teléfono
│   │   ├── PantallaAlternativas.tsx # Mostrar 3 opciones
│   │   ├── PantallaResumen.tsx
│   │   ├── ProgresoBarra.tsx
│   │   └── Navegacion.tsx
│   ├── Dashboard/
│   │   ├── TablaProspectos.tsx
│   │   ├── FiltrosAvanzados.tsx
│   │   ├── TarjetaProspecto.tsx
│   │   └── EditorFlujo.tsx        # Editor visual de JSON para Lorena
│   └── Compartido/
│       ├── Header.tsx
│       ├── Footer.tsx
│       └── Logo.tsx
├── lib/
│   ├── supabase/
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── admin.ts
│   ├── google-calendar/
│   │   └── scheduler.ts
│   ├── flujos/
│   │   ├── loader.ts
│   │   ├── evaluator.ts
│   │   └── validator.ts
│   ├── data-structures/
│   │   ├── flujo.ts              # TypeScript tipos
│   │   ├── sesion.ts
│   │   ├── prospecto.ts
│   │   └── respuesta.ts
│   └── utils/
│       ├── lenguaje.ts
│       └── validaciones.ts
├── public/
│   └── assets/
│       └── logos/
├── database/
│   └── migrations/               # SQL migrations
├── tests/
├── .env.local
├── next.config.ts
├── tsconfig.json
└── README.md
```
### 2.3 Base de Datos (Supabase / PostgreSQL)

#### Tablas principales:

**1. `flujos_configuracion`** — Definición completa de cada flujo (puerta/producto)
```sql
CREATE TABLE flujos_configuracion (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  codigo VARCHAR(50) UNIQUE NOT NULL,  -- "familia_salud", "empresa_independiente", etc.
  nombre VARCHAR(200) NOT NULL,
  descripcion TEXT,
  puerta VARCHAR(200),  -- Ref: "Seguro para mi familia"
  configuracion JSONB NOT NULL,  -- Árbol COMPLETO de preguntas y lógica
  metadata JSONB,  -- Config multitenant (branding, URLs, etc.)
  estado BOOLEAN DEFAULT true,
  creado_en TIMESTAMP DEFAULT NOW(),
  actualizado_en TIMESTAMP DEFAULT NOW()
);
```

**Estructura de `configuracion` (JSON):**
```json
{
  "preguntas": [
    {
      "id": "q1",
      "texto": "¿Quién necesita cobertura?",
      "tipo": "opcion_unica",
      "explicacion": "Esto nos ayuda a entender tu situación",
      "opciones": [
        {"label": "Solo yo", "valor": "solo_yo", "siguiente": "q2"},
        {"label": "Pareja e hijos", "valor": "pareja_hijos", "siguiente": "q3"}
      ],
      "obligatorio": true
    },
    {
      "id": "q2",
      "texto": "¿Cuál es tu edad?",
      "tipo": "rango",
      "opciones": [
        {"label": "18-30", "valor": "18_30", "siguiente": "q5"},
        {"label": "31-50", "valor": "31_50", "siguiente": "q5"}
      ],
      "obligatorio": true
    },
    {
      "id": "contacto_nombre",
      "texto": "¿Cuál es tu nombre?",
      "tipo": "contacto_texto",
      "campo": "nombre",
      "siguiente": "q4"
    }
  ],
  "orden_inicial": "q1",
  "alternativas": [
    {
      "id": "alt_a",
      "titulo": "Menor costo mensual",
      "descripcion": "...",
      "precio_desde": "$150/mes"
    }
  ]
}
```

**2. `sesiones`** — Una sesión = una persona atravesando un flujo
```sql
CREATE TABLE sesiones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  flujo_codigo VARCHAR(50) NOT NULL,  -- Ref a flujos_configuracion.codigo
  codigo_sesion VARCHAR(100) UNIQUE,
  estado VARCHAR(50),  -- "en_curso", "completada", "abandonada"
  pregunta_actual_id VARCHAR(50),  -- ID de la pregunta donde está ahora
  respuestas JSONB,  -- {q1: "pareja_hijos", q2: "31_50", ...}
  datos_contacto JSONB,  -- {nombre, email, telefono, preferencia_contacto}
  iniciada_en TIMESTAMP DEFAULT NOW(),
  completada_en TIMESTAMP,
  tiempo_total_segundos INT
);
```

**3. `prospectos`** — Registro definitivo (cuando completa flujo)
```sql
CREATE TABLE prospectos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sesion_id UUID REFERENCES sesiones(id),
  nombre VARCHAR(200) NOT NULL,
  email VARCHAR(200) NOT NULL,
  telefono VARCHAR(20) NOT NULL,
  preferencia_contacto VARCHAR(50),  -- "WhatsApp", "Llamada"
  flujo_codigo VARCHAR(50),
  flujo_nombre VARCHAR(200),
  respuestas_formateadas JSONB,  -- {q1: {pregunta, respuesta}, q2: {...}, ...}
  alternativa_seleccionada VARCHAR(500),  -- "alt_a", "alt_b", etc.
  nivel_intension VARCHAR(50),  -- "Alta", "Media", "Baja"
  fecha_creado TIMESTAMP DEFAULT NOW(),
  cita_agendada BOOLEAN DEFAULT false,
  cita_confirmada BOOLEAN DEFAULT false,
  cita_timestamp TIMESTAMP,
  notas_lorena TEXT,
  estado VARCHAR(50)  -- "nuevo", "contactado", "confirmado", "no_aplica"
);
```

**4. `sugerencias_mejora_flujo`** — IA analiza respuestas y sugiere mejoras
```sql
CREATE TABLE sugerencias_mejora_flujo (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  flujo_codigo VARCHAR(50) NOT NULL,
  tipo_sugerencia VARCHAR(100),  -- "pregunta_redundante", "tasa_abandono_alta", etc.
  descripcion TEXT,
  datos_analisis JSONB,  -- Estadísticas que justifican la sugerencia
  sugerida_en TIMESTAMP DEFAULT NOW(),
  revisada_por_lorena BOOLEAN DEFAULT false,
  accion_tomada TEXT,
  fecha_accion TIMESTAMP
);
```
---

## III. FLUJOS DE USUARIO (UX)

### 3.1 Pantalla 0: Landing inicial (selector de puerta)

**Pantalla (mockup simplificado):**
```
Encuentra tu camino
¿Qué necesitas resolver hoy?

[1. Seguro para mi o mi familia]
[2. Busco una opción económica]
[3. Estoy embarazada...]
[4. Trabajo por mi cuenta]
[5. Perdí cobertura / me mudé]
[6. Dudas por mi situación...]

[Hablar directamente con Lorena] (secundario)
```

**Lógica:**
- Usuario elige 1 de 6 puertas
- Sistema carga el flujo correspondiente
- Navega a `/flujo/[flowId]/step/1`

### 3.2 Pantalla N: Pregunta cerrada (1 pregunta por pantalla)

**Pantalla (mockup simplificado):**
```
Progreso: 2 de 5

¿Cuántas personas necesitan cobertura médica?

(Esto nos ayuda a entender tu situación familiar)

[Solo yo (1)]
[Pareja e hijos (2-4)]
[Familia extendida (5+)]

[Atrás] [Continuar]
```

**Componentes:**
- Barra de progreso
- Pregunta clara
- Texto contextual ("Esto nos ayuda a...")
- 2-4 opciones grandes (botones clickeables)
- Navegación: Atrás / Continuar

### 3.3 Pantalla de datos de contacto (intercalada)

**Después de 2-3 preguntas de calificación:**

```
Antes de seguir...

¿Cuál es tu nombre?

[________________]

[Continuar]
```

**Regla:** Pedir datos de a UNO, no todos juntos. Nunca al principio.

Campos en este orden:
1. Nombre
2. (después de 2 preguntas más) Email
3. (después de 2 preguntas más) Teléfono
4. (al final) Preferencia de contacto (WhatsApp / Llamada)

### 3.4 Pantalla de alternativas (cuando aplique)

```
Estas son algunas opciones que vale la pena explorar

Selecciona la que más te atrae:

[A] MENOR COSTO MENSUAL — Desde $150/mes

[B] EQUILIBRIO (RECOMENDADO) — Desde $200/mes

[C] MAYOR PROTECCIÓN — Desde $300/mes

[Continuar]
```

### 3.5 Pantalla de resumen

```
Esto es lo que seleccionaste

Necesidad: Seguro para familia
Tamaño: Pareja + 2 hijos
Prioridad: Costo equilibrado
Presupuesto: $200-400/mes
Alternativa: B — Equilibrio

[Modificar] [Confirmar]
```

### 3.6 Pantalla de confirmación de cita

```
Tu caso es especial y merece atención directa.

¿Con quién prefieres hablar?

[Lorena]
[Un miembro del equipo]

(Disponibilidad Google Calendar)
[Próximos horarios disponibles]

[Agendar]
```

### 3.7 Pantalla de gracias

```
¡Casi lista tu asesoría!

Cita confirmada:
Martes, 12 sept — 14:00 - 14:15
Con Lorena Alvarez

Te enviaremos un enlace por email y WhatsApp 5 minutos antes.

Mientras tanto...
[Ver video de Lorena] (embebido)

Preguntas: hola@segurosdevidafl.com
```
---

## IV. FLUJOS INICIALES (EJEMPLOS CONFIGURABLES)

**Importante:** Estos NO son 6 apps codificadas. Son 6 ejemplos de CONFIGURACIONES JSON que cargan en `flujos_configuracion`.

Una vez creada la infraestructura, Lorena (u otro admin) puede:
- Agregar nuevos flujos editando JSON
- Cambiar el orden de preguntas
- Ajustar opciones
- Crear variantes (ej: "Seguro de Salud para familia" vs "Seguro de Salud para empresa")
- TODO sin tocar código, solo configuración en el dashboard

### Ejemplo 1: Flujo "Seguro para mí o mi familia"

**Objetivo:** Entender composición familiar, edades, prioridades de cobertura

**Configuración JSON (simplificada):**
```json
{
  "codigo": "familia_salud_01",
  "nombre": "Seguro para mí o mi familia",
  "preguntas": [
    {
      "id": "q1",
      "texto": "¿Quién necesita cobertura?",
      "tipo": "opcion_unica",
      "opciones": [
        {"label": "Solo yo", "valor": "solo_yo", "siguiente": "q2"},
        {"label": "Pareja e hijos", "valor": "pareja_hijos", "siguiente": "q3"},
        {"label": "Familia extendida", "valor": "familia_ext", "siguiente": "q3"}
      ]
    },
    {
      "id": "q2",
      "texto": "¿Cuál es tu rango de edad?",
      "tipo": "opcion_unica",
      "opciones": [
        {"label": "18-30", "valor": "18_30", "siguiente": "contacto_nombre"},
        {"label": "31-50", "valor": "31_50", "siguiente": "contacto_nombre"},
        {"label": "51+", "valor": "51plus", "siguiente": "contacto_nombre"}
      ]
    },
    {
      "id": "q3",
      "texto": "¿Cuál es la edad del principal?",
      "tipo": "opcion_unica",
      "siguiente": "q4"
    },
    {
      "id": "q4",
      "texto": "¿Cuál es tu prioridad?",
      "tipo": "opcion_unica",
      "opciones": [
        {"label": "Menor costo", "valor": "costo", "siguiente": "contacto_nombre"},
        {"label": "Cobertura amplia", "valor": "cobertura", "siguiente": "contacto_nombre"},
        {"label": "Equilibrio", "valor": "equilibrio", "siguiente": "contacto_nombre"}
      ]
    },
    {
      "id": "contacto_nombre",
      "texto": "¿Cuál es tu nombre?",
      "tipo": "contacto_texto",
      "campo": "nombre",
      "siguiente": "q5"
    },
    {
      "id": "q5",
      "texto": "¿Tienes cobertura actualmente?",
      "tipo": "opcion_unica",
      "opciones": [
        {"label": "Sí, tengo", "valor": "si_tengo", "siguiente": "q6"},
        {"label": "No, sin cobertura", "valor": "sin_cob", "siguiente": "q6"},
        {"label": "Necesito cambiar", "valor": "cambiar", "siguiente": "q6"}
      ]
    },
    {
      "id": "contacto_email",
      "texto": "¿Cuál es tu email?",
      "tipo": "contacto_texto",
      "campo": "email",
      "siguiente": "q6"
    },
    {
      "id": "q6",
      "texto": "¿Cuál es tu presupuesto aproximado mensual?",
      "tipo": "opcion_unica",
      "opciones": [
        {"label": "$0-150/mes", "valor": "0_150", "siguiente": "contacto_telefono"},
        {"label": "$150-300/mes", "valor": "150_300", "siguiente": "contacto_telefono"},
        {"label": "$300+/mes", "valor": "300plus", "siguiente": "contacto_telefono"}
      ]
    },
    {
      "id": "contacto_telefono",
      "texto": "¿Cuál es tu teléfono?",
      "tipo": "contacto_texto",
      "campo": "telefono",
      "siguiente": "contacto_preferencia"
    },
    {
      "id": "contacto_preferencia",
      "texto": "¿Cómo prefieres que te contactemos?",
      "tipo": "opcion_unica",
      "opciones": [
        {"label": "WhatsApp", "valor": "whatsapp", "siguiente": "alternativas"},
        {"label": "Llamada", "valor": "llamada", "siguiente": "alternativas"}
      ]
    }
  ],
  "alternativas": [
    {
      "id": "alt_a",
      "titulo": "Menor costo mensual",
      "descripcion": "Máxima economía",
      "precio_desde": "$150/mes"
    },
    {
      "id": "alt_b",
      "titulo": "Equilibrio (Recomendado)",
      "descripcion": "Balance entre costo y cobertura",
      "precio_desde": "$200/mes"
    },
    {
      "id": "alt_c",
      "titulo": "Mayor protección",
      "descripcion": "Cobertura más amplia",
      "precio_desde": "$300/mes"
    }
  ]
}
```

**Duración esperada:** 4-5 minutos

**Datos que llegan a Lorena:**
```json
{
  "flujo_codigo": "familia_salud_01",
  "flujo_nombre": "Seguro para mí o mi familia",
  "respuestas": {
    "q1": "pareja_hijos",
    "q3": "31_50",
    "q4": "costo",
    "q5": "sin_cob",
    "q6": "150_300"
  },
  "datos_contacto": {
    "nombre": "María López",
    "email": "maria@email.com",
    "telefono": "(305) 555-1234",
    "preferencia_contacto": "WhatsApp"
  },
  "alternativa_seleccionada": "alt_b"
}
```

### Ejemplo 2, 3, 4, 5, 6: Flujos adicionales

Los 5 flujos restantes ("Busco económico", "Embarazada", "Trabajo por mi cuenta", "Perdí cobertura", "Dudas migratorias") siguen la MISMA estructura:
- Cada uno es un archivo JSON en `flujos_configuracion`
- Cada uno define sus propias preguntas, orden, ramificaciones
- El motor genérico `<PantallaDePregunta />` los renderiza todos sin cambios

**Cuando sea momento de crear esos 5, usaremos el mismo formato JSON.**
---

## V. DASHBOARD DE LORENA

### 5.1 Acceso y autenticación

**Login:** Email + contraseña (Supabase Auth)  
**Roles:** 
- Lorena (admin)
- Asistente (si aplica)

### 5.2 Secciones del Dashboard

#### Dashboard principal (mockup simplificado)
```
Lorena Alvarez — Dashboard

RESUMEN
- Nuevos prospectos hoy: 3
- Citas agendadas (próx. 7d): 8
- Tasa de conversión (mes): 42%
- Tiempo promedio/sesión: 4m 30s

ACCIONES PENDIENTES
- María López (familia) — HOY
- Carlos Rodríguez (empresa) — MAÑANA
- [3 más...]

NUEVOS PROSPECTOS
Tabla: Nombre | Puerta | Hora | Acción
1. María López | Familia | 14:22 | Ver
2. Pedro García | Económico | 14:10 | Ver
3. Ana Martín | Embarazo | 13:45 | Ver
```

#### Lista de prospectos (con filtros)

```
Prospectos

Filtros: [Todas las puertas] [Últimos 7d] [Estado: Nuevo]

Resultados: 24 prospectos

Nombre | Puerta | Fecha | Estado
María López | Familia | 2026-09-10 | Nuevo
Carlos R. | Empresa | 2026-09-10 | Contactado
...

[Página 1 de 3] [Siguiente]
```

#### Detalle de prospecto

```
María López

Email: maria@email.com
Teléfono: (305) 555-1234
Contacto preferido: WhatsApp

RESPUESTAS AL CUESTIONARIO

Puerta: Seguro para mi o mi familia

¿Quién necesita cobertura? → Pareja e hijos (2-4)
Rango de edad principal: → 31-50 años
Prioridad: → Costo equilibrado
Presupuesto: → $200-400/mes
Alternativa seleccionada: → B — Equilibrio

CITA AGENDADA
Martes, 12 sep 2026 — 14:00 - 14:15
[Google Meet link]

NOTAS INTERNAS
[Campo de texto para agregar notas]

[Volver] [Enviar reminder] [Cambiar estado]
```

#### Configuración (multitenant)

```
Configuración

BRANDING
Logo: [Cambiar]
Color principal: #0076DF [Cambiar]

PRODUCTOS DISPONIBLES
[x] Seguro de Salud
[x] Seguro de Vida
[x] Seguros de Empresa
[x] Planificación Financiera

CALENDARIO
Email de Google Calendar: [...]
Duración de citas: 15 minutos
Horario de atención: 9 AM - 6 PM

INTEGRACIONES
Google Calendar: Conectado
Supabase: Activo

[Guardar cambios]
```
---

## VI. ESTRUCTURA DE DATOS QUE LLEGA A LORENA

### Ejemplo completo de prospecto

```json
{
  "id": "uuid-12345",
  "nombre": "María López",
  "email": "maria@email.com",
  "telefono": "(305) 555-1234",
  "preferencia_contacto": "WhatsApp",
  "puerta": "Seguro para mí o mi familia",
  "flujo_codigo": "familia_salud_01",
  "fecha_inicio": "2026-09-10T14:15:30Z",
  "fecha_completado": "2026-09-10T14:22:45Z",
  "tiempo_total_minutos": 7.25,
  "respuestas_formateadas": {
    "composicion_familiar": {
      "pregunta": "¿Quién necesita cobertura?",
      "respuesta": "Pareja e hijos (2-4)"
    },
    "edad_principal": {
      "pregunta": "Rango de edad del principal",
      "respuesta": "31-50 años"
    },
    "prioridad": {
      "pregunta": "¿Cuál es tu prioridad?",
      "respuesta": "Costo equilibrado"
    },
    "presupuesto": {
      "pregunta": "¿Cuál es tu presupuesto?",
      "respuesta": "$200-400/mes"
    },
    "situacion_actual": {
      "pregunta": "¿Tienes cobertura actualmente?",
      "respuesta": "No, sin cobertura"
    }
  },
  "alternativa_seleccionada": "B — Equilibrio",
  "nivel_intension": "Alta",
  "cita_agendada": true,
  "cita_confirmada": true,
  "cita_timestamp": "2026-09-12T14:00:00Z",
  "cita_enlace_google_meet": "https://meet.google.com/...",
  "estado": "confirmado",
  "notas_lorena": ""
}
```

---

## VII. INTEGRACIONES EXTERNAS

### 7.1 Google Calendar

- **Acción:** Cuando prospecto agenda cita, se crea automáticamente en Google Calendar de Lorena
- **Datos:** Nombre prospecto, tiempo, tipo de producto, enlace Google Meet
- **Confirmación:** Email automático al prospecto
- **Recordatorio:** 15 minutos antes (propietario=Lorena, asistente=prospecto)

### 7.2 Email (notificaciones)

- **Prospecto recibe:**
  - Confirmación de precalificación
  - Enlace Google Meet 5 min antes
  - Resumen de lo que seleccionó

- **Lorena recibe:**
  - Notificación cuando llega nuevo prospecto
  - Resumen en dashboard
  - Recordatorio antes de cita

---

## VIII. REGLAS DE CONTENIDO (NO NEGOCIABLES)

### 8.1 Prohibiciones absolutas

**JAMÁS decir:**
- "Cotización final"
- "Precio garantizado"
- "Aprobado"
- "Ya estás asegurado"
- "Garantía de cobertura"

**SIEMPRE decir:**
- "Opciones que vale la pena explorar"
- "Para que Lorena tenga toda la información"
- "Vamos a revisar esto con Lorena"
- "Una conversación de 15 minutos puede cambiar…"

### 8.2 Datos que NUNCA se preguntan

- Social Security Number (SSN)
- Documentos migratorios
- Información bancaria
- Historial médico detallado
- Condiciones preexistentes (solo en contexto especial con Lorena)

### 8.3 Lenguaje obligatorio (según 02-LENGUAJE-CLAVE-GSC-GA.md)

**Usar:** explorar, opciones, entender, proteger, cobertura, acompañamiento, según tu situación, en español, Florida/Miami/Doral, económico

**Evitar:** aprobado, calificas, precio final, compra ahora, cotización instantánea, lenguaje técnico excesivo
---

## IX. ROADMAP Y FASES (SUPERADO — ver ROADMAP.md vigente)

### Fase 1: MVP (Piloto con 1 puerta)
- Arquitectura base (Next.js + Supabase)
- Flujo Puerta 1 ("Seguro para mí o mi familia")
- Dashboard básico para Lorena
- Integración Google Calendar
- Testing completo (desktop + mobile)
- Deploy en Vercel

**Duración estimada:** 2 semanas  
**Criterio de éxito:** 5+ prospectos completando flujo sin errores

### Fase 2: Replicar 5 puertas restantes
- Puerta 2-6 (igualar configuración de Puerta 1)
- Testing con cada nueva puerta
- Validación con Lorena

**Duración:** 1 semana

### Fase 3: Dashboard completo
- Filtros avanzados
- Reportes de conversión
- Exportar datos

**Duración:** 1 semana

### Fase 4: Refinamientos y optimización
- A/B testing de preguntas
- Mejorar tiempo de sesión
- Análisis de abandono

**Duración:** 2 semanas (continuo)

---

## X. CRITERIOS DE ACEPTACIÓN

### Frontend
- [ ] 1 pantalla = 1 pregunta (nunca más)
- [ ] Opciones grandes, fáciles de tocar (mobile-first)
- [ ] No se ve como formulario
- [ ] Progreso visible (barra o contador)
- [ ] Navegación clara (Atrás/Continuar)
- [ ] Responsive (375px - 1920px)
- [ ] Sin dependencias externas innecesarias (React + Next.js máximo)

### Backend / Base de datos
- [ ] Datos guardados en tiempo real (por cada respuesta)
- [ ] Estructura JSON limpia y extensible
- [ ] Sesiones no se pierden (refresh = continúan)
- [ ] Google Calendar integrado y funcionando
- [ ] Emails enviados correctamente

### UX / Contenido
- [ ] Lenguaje respeta 02-LENGUAJE-CLAVE-GSC-GA.md
- [ ] Nunca se pide SSN, documentos migratorios, datos bancarios
- [ ] No se promete "cotización final" ni "aprobación"
- [ ] Datos esenciales llegan a Lorena estructurados
- [ ] Lorena puede leer el prospecto en 30 segundos y saber qué hacer

### Dashboard Lorena
- [ ] Lista de prospectos accesible
- [ ] Ver respuestas completas por prospecto
- [ ] Filtros funcionales (puerta, fecha, estado)
- [ ] Cita agendada y confirmada automáticamente
- [ ] Notas internas guardables
---

## XI. DECISIONES TÉCNICAS PENDIENTES (RESUELTAS EN MASTER_BLUEPRINT.md v3.0)

### 11.1 ¿Cómo se generan las preguntas de cada flujo?

**Decisión final (v3.0):** híbrido — preguntas base generadas por IA en construcción (aprobadas por Lorena una vez) MÁS razonamiento de IA en vivo durante la sesión real (con tope duro de 10 preguntas y guardrails). Ver `MASTER_BLUEPRINT.md` secciones 17 y 19.

### 11.2 ¿Cuándo se hace el cierre de cita?

**Decisión final (v3.0):** Después del resumen — Flujo completa → Resumen → Selector (Lorena/Equipo, ambos van a Lorena) → Google Calendar → Confirmación.

### 11.3 ¿Iframe o embed directo?

**Decisión:** IFRAME (por recomendación de `08-AUTOSERVICIO-ARQUITECTURA-ACORDADA.md`)
- Scripts pegados en 10MinutesWebsite nunca se ejecutan
- iframe = aislamiento total de CSS/JS
- URL en Vercel = una sola línea de embed en 10MinutesWebsite

```html
<iframe src="https://precalificador.vercel.app/?tenant=ssv" width="100%" height="600"></iframe>
```

---

## XII. APROBACIÓN Y PRÓXIMOS PASOS (HISTÓRICO)

Esta propuesta fue revisada mediante una entrevista completa con la skill MAGO, cuyas respuestas dieron lugar a decisiones nuevas (razonamiento de IA en vivo, motor OpenAI, dashboard de auto-creación de Precalificadores, mini-formulario de contacto consolidado, tope de 10 preguntas, retención indefinida con notas fechadas). Todo eso está incorporado en `../MASTER_BLUEPRINT.md`, que es el documento vigente.

---

**Documento versión:** 1.0 (archivado)  
**Fecha de propuesta:** 2026-09-10  
**Estado:** Sustituido por `../MASTER_BLUEPRINT.md`
