# PRECALIFICADOR — MASTER BLUEPRINT

**Versión:** 3.0 (definitiva, post-entrevista MAGO)  
**Fecha:** 2026-09-10  
**Estado:** APROBADO PARA FASE 0 — pendiente de revisión de arquitectura antes de programar

---

## 1. Título del proyecto

**Precalificador** — Motor de Autoservicio Inteligente para Seguros de Salud y Vida

## 2. Documento maestro del proyecto

Este documento es la **fuente única de verdad** del Precalificador. Sustituye y actualiza las versiones previas (`PRECALIFICADOR-ARQUITECTURA-PROPUESTA.md` y `PRECALIFICADOR-RESUMEN-EJECUTIVO.md`, archivadas en `archivo-v1/`), que quedaron desactualizadas tras decisiones nuevas: razonamiento de IA en vivo, motor OpenAI, dashboard de auto-creación de Precalificadores por parte de Lorena.

## 3. Propósito del documento

Servir de referencia para Claude Code (o cualquier desarrollador) cada vez que se retome el trabajo en este proyecto — sin necesidad de repetir la entrevista de descubrimiento. Debe leerse completo antes de escribir o modificar código.

## 4. Rol que debe asumir Claude Code

Claude Code es el **arquitecto e implementador técnico**. No decide reglas de negocio, contenido, ni lenguaje — esas ya están definidas aquí y en los documentos del CODEX (`SEGUROS-SALUD-VIDA-CODEX/`). Ante cualquier ambigüedad no cubierta en este documento, debe **detenerse y preguntar**, no improvisar.

**No deberá escribirse una sola línea de código hasta que la Fase 0 (sección 13) haya sido revisada y aprobada.**

## 5. Filosofía del producto

> La web ORIENTA + ORGANIZA + PRECALIFICA.  
> Lorena VALIDA + EXPLICA + ASESORA + CONFIRMA + CIERRA.

El Precalificador no es un formulario ni un chatbot. Es un **autoservicio guiado** que se siente como *"ya casi termino, solo falta revisar esto con Lorena"* — nunca como *"estoy llenando papeles"* ni *"estoy chateando con un bot"*.

## 6. Visión

Que cualquier persona que llega al sitio de Seguros de Salud y Vida pueda, en menos de 5 minutos y sin sentirse interrogada, dejar clara su necesidad real — y que Lorena reciba esa información ya organizada, lista para tener una conversación útil desde el primer segundo de la llamada.

## 7. Misión

Reemplazar el "formulario de contacto" tradicional por un sistema de preguntas inteligente, adaptable por segmento de cliente, que use IA para generar y ajustar preguntas sin que eso implique riesgo de compliance ni pérdida de velocidad.

## 8. Objetivo principal

Convertir visitantes del sitio en **citas agendadas y bien preparadas** con Lorena, capturando solo la información que realmente cambia la conversación.

## 9. Principios del producto

1. **Nunca se siente como un formulario.** Botones grandes, una pregunta por pantalla.
2. **Nunca se siente como un chatbot.** No hay burbujas de chat ni conversación libre — son pantallas de selección, aunque el contenido de la pregunta sea generado por IA.
3. **La IA prepara, nunca improvisa libremente sobre temas prohibidos.** Genera preguntas — en construcción y en vivo — pero siempre dentro de barandas de seguridad fijas (sección 19).
4. **El progreso siempre es visible.** El usuario nunca se pregunta "¿cuánto falta?".
5. **Máximo 10 preguntas por Precalificador**, sin excepción, contando preestablecidas + las que la IA añada en vivo.
6. **Los datos de contacto se piden en UNA pantalla consolidada** (nombre, apellido, teléfono, correo juntos), no repartidos ni al principio.
7. **Lorena configura por objetivo, no por pregunta.** Ella describe qué necesita saber; la IA traduce eso en preguntas.
8. **Cero promesas falsas.** Nunca "aprobado", "cotización final", "ya estás asegurado".

## 10. Tipo de producto

Microapp web independiente (no un widget HTML estático), multi-pantalla, con backend propio, base de datos propia y panel de administración propio. Se integra al sitio principal (10MinutesWebsite) mediante `<iframe>`.

**Alcance de tenant:** Un solo tenant real en producción — **Seguros de Salud y Vida / Lorena Alvarez**. La arquitectura de datos se diseña de forma que sea multitenant-ready (todo registro cuelga de un `tenant_id`), pero **no se construye UI ni lógica multitenant en esta fase** — sería sobre-ingeniería para un solo cliente real hoy.

## 11. Usuarios y permisos

| Usuario | Acceso | Permisos |
|---|---|---|
| **Prospecto** (público, sin login) | Solo el flujo del Precalificador vía enlace/iframe | Responde preguntas, deja contacto, agenda cita. No ve dashboard ni datos de otros. |
| **Lorena** (admin, único usuario interno) | Dashboard con login (usuario + contraseña, Supabase Auth) | Ve todos los leads/prospectos, agrega notas fechadas, borra prospectos, crea y configura Precalificadores, aprueba preguntas generadas por IA, obtiene el enlace de cada Precalificador para entregarlo para integración en los widgets del HOME. |

No hay rol de "equipo/asistente" con acceso al sistema — el selector "Lorena / un miembro del equipo" que ve el prospecto al cerrar el flujo es **solo de percepción de marca**; ambas opciones agendan con Lorena en la práctica (ver sección 12, paso de cierre).

## 12. Flujo general

```
1. Prospecto entra a una "puerta" en el HOME (ej. "Seguro para mi familia")
        ↓
2. Se carga el Precalificador correspondiente vía iframe (URL única por segmento)
        ↓
3. Pantalla 1..N: preguntas preestablecidas (definidas al crear el Precalificador,
   generadas por IA a partir del objetivo de Lorena, aprobadas por ella una vez)
        ↓
4. En cualquier punto, la IA puede decidir en vivo que hace falta una pregunta
   adicional para cumplir el objetivo → la agrega con tono de disculpa
   ("una pregunta más...") → nunca supera el máximo de 10 preguntas totales
        ↓
5. Una pantalla es el mini-formulario de contacto consolidado
   (nombre, apellido, teléfono, correo — una sola pantalla, no repartido)
        ↓
6. Pantalla de alternativas (hasta 3, cuando el segmento las tenga definidas)
        ↓
7. Pantalla de resumen ("esto es lo que seleccionaste")
        ↓
8. Cierre: selector "Lorena" / "un miembro del equipo" (percepción de marca;
   funcionalmente ambos agendan con Lorena)
        ↓
9. Disponibilidad → Cita ~15 min → Google Calendar de Lorena
        ↓
10. Confirmación + notificación por email a Lorena
    (enviada desde segurosdesaludyvidaahora@gmail.com)
        ↓
11. Landing de gracias + video de Lorena
        ↓
12. Prospecto queda guardado en el dashboard de Lorena, con todas sus respuestas,
    listo para que ella agregue notas o lo gestione
```

**Flujo paralelo — Lorena crea un Precalificador nuevo:**

```
1. Lorena entra al dashboard (login)
        ↓
2. Va a "Crear Precalificador"
        ↓
3. Llena: nombre (ej. "Familia"), descripción del cliente esperado,
   objetivo (qué necesita saber: "nombre, contacto, composición familiar,
   edades, presupuesto, prioridad costo/cobertura")
        ↓
4. La IA genera un borrador de preguntas base (dentro del tope de 10,
   dejando margen para las que se sumen en vivo)
        ↓
5. Lorena revisa, edita si quiere, aprueba
        ↓
6. El sistema genera un enlace único para ese Precalificador
        ↓
7. Lorena entrega ese enlace para que se inserte vía iframe
   en el widget correspondiente del HOME
```

## 13. Fase 0 obligatoria antes de programar

Antes de escribir una sola línea de código de la aplicación, debe entregarse para aprobación (este mismo documento cumple ese propósito, pero cualquier cambio de alcance debe volver a pasar por aquí):

- Arquitectura completa (sección 14).
- Stack tecnológico (sección 14.1).
- Modelo de datos (sección 14.3).
- Flujos de usuario (sección 12).
- Mapa de navegación / estructura de carpetas (sección 14.2).
- Roadmap por fases (sección 22).
- Riesgos técnicos (sección 23).
- Propuestas de mejora futuras (sección 23).

**No deberá escribirse una sola línea de código hasta que esta fase haya sido revisada y aprobada explícitamente por Milton y/o Lorena.**
## 14. Arquitectura funcional

### 14.1 Stack tecnológico

| Componente | Tecnología | Rol |
|---|---|---|
| Frontend | Next.js 15 + React 19 | UI del flujo público + dashboard |
| Hosting | Vercel | Deploy, CDN, serverless functions |
| Backend | Next.js API Routes | Lógica de sesiones, orquestación de IA, webhooks |
| Base de datos | Supabase (PostgreSQL) | Precalificadores, sesiones, prospectos, notas |
| Autenticación | Supabase Auth | Login exclusivo de Lorena al dashboard |
| Motor de IA | OpenAI API — modelo ligero/económico (ej. `gpt-4o-mini` o equivalente vigente) | (a) Generación de preguntas base al crear un Precalificador. (b) Razonamiento en vivo para decidir preguntas adicionales durante una sesión real |
| Calendario | Google Calendar API | Agendar cita ~15 min, generar enlace de Google Meet |
| Notificaciones | Gmail (SMTP/API) desde `segurosdesaludyvidaahora@gmail.com` | Aviso a Lorena de nuevo prospecto |
| Integración con sitio | `<iframe>` en 10MinutesWebsite | Una URL única por Precalificador |

### 14.2 Estructura de carpetas propuesta

```
precalificador/
├── app/
│   ├── (publico)/
│   │   ├── p/[codigoPrecalificador]/
│   │   │   ├── page.tsx              # Pantalla de pregunta actual (motor genérico)
│   │   │   ├── contacto/page.tsx     # Mini-formulario consolidado
│   │   │   ├── alternativas/page.tsx
│   │   │   ├── resumen/page.tsx
│   │   │   ├── cierre/page.tsx       # Selector Lorena / equipo
│   │   │   ├── disponibilidad/page.tsx
│   │   │   └── gracias/page.tsx
│   ├── (dashboard)/
│   │   ├── login/page.tsx
│   │   ├── dashboard/page.tsx        # Resumen general
│   │   ├── leads/page.tsx            # Lista de prospectos
│   │   ├── leads/[id]/page.tsx       # Detalle + notas + borrado
│   │   ├── precalificadores/page.tsx # Lista de segmentos configurados
│   │   ├── precalificadores/nuevo/page.tsx   # Crear (nombre+descripción+objetivo → IA)
│   │   └── precalificadores/[id]/editar/page.tsx
│   ├── api/
│   │   ├── precalificadores/
│   │   │   ├── route.ts                       # CRUD
│   │   │   └── [id]/generar-preguntas/route.ts # Llamada a OpenAI (build-time)
│   │   ├── sesiones/
│   │   │   ├── route.ts                       # Crear sesión
│   │   │   ├── [sessionId]/siguiente-pregunta/route.ts # Llamada a OpenAI (en vivo)
│   │   │   ├── [sessionId]/respuesta/route.ts
│   │   │   └── [sessionId]/contacto/route.ts
│   │   ├── leads/
│   │   │   ├── route.ts
│   │   │   └── [id]/route.ts                  # notas, borrado
│   │   ├── calendario/
│   │   │   ├── disponibilidad/route.ts
│   │   │   └── agendar/route.ts
│   │   ├── notificaciones/
│   │   │   └── nuevo-prospecto/route.ts       # Envío de email vía Gmail
│   │   └── auth/
│   │       └── [...supabase]/route.ts
├── components/
│   ├── Flujo/
│   │   ├── PantallaDePregunta.tsx     # Motor genérico — renderiza cualquier pregunta
│   │   ├── RenderizadorOpciones.tsx
│   │   ├── FormularioContacto.tsx     # Nombre+apellido+teléfono+correo, una pantalla
│   │   ├── PantallaAlternativas.tsx
│   │   ├── PantallaResumen.tsx
│   │   ├── SelectorCierre.tsx         # "Lorena" / "un miembro del equipo"
│   │   ├── ProgresoBarra.tsx          # Siempre visible, se adapta si la IA suma preguntas
│   │   └── Navegacion.tsx
│   ├── Dashboard/
│   │   ├── TablaLeads.tsx
│   │   ├── FiltrosLeads.tsx
│   │   ├── DetalleLead.tsx
│   │   ├── NotasFechadas.tsx
│   │   ├── EditorPrecalificador.tsx   # Nombre, descripción, objetivo
│   │   ├── RevisorPreguntasIA.tsx     # Lorena aprueba/edita el borrador de IA
│   │   └── EnlaceEmbebido.tsx         # Genera y copia el <iframe> listo
│   └── Compartido/
│       ├── Header.tsx
│       └── Logo.tsx
├── lib/
│   ├── supabase/ (client.ts, server.ts, admin.ts)
│   ├── openai/
│   │   ├── generarPreguntasBase.ts    # Build-time
│   │   ├── decidirSiguientePregunta.ts # En vivo, con guardrails
│   │   └── guardrails.ts              # Validación de contenido prohibido
│   ├── google-calendar/scheduler.ts
│   ├── gmail/enviarNotificacion.ts
│   └── tipos/ (precalificador.ts, sesion.ts, lead.ts, pregunta.ts)
├── database/migrations/
├── .env.local
└── README.md
```
### 14.3 Modelo de datos (Supabase / PostgreSQL)

```sql
-- Un Precalificador = un segmento configurado por Lorena
CREATE TABLE precalificadores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL DEFAULT '00000000-0000-0000-0000-000000000001', -- preparado a futuro, un solo valor hoy
  codigo VARCHAR(80) UNIQUE NOT NULL,        -- usado en la URL pública: /p/familia
  nombre VARCHAR(200) NOT NULL,              -- "Familia"
  descripcion TEXT NOT NULL,                 -- tipo de cliente esperado
  objetivo TEXT NOT NULL,                    -- qué debe averiguar el flujo
  preguntas_base JSONB NOT NULL,             -- borrador aprobado por Lorena
  alternativas JSONB,                        -- hasta 3 alternativas, si aplica
  max_preguntas INT NOT NULL DEFAULT 10,     -- tope duro
  estado VARCHAR(20) NOT NULL DEFAULT 'activo', -- activo / pausado
  creado_en TIMESTAMP DEFAULT NOW(),
  actualizado_en TIMESTAMP DEFAULT NOW()
);

-- Una sesión = un prospecto atravesando un Precalificador en tiempo real
CREATE TABLE sesiones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  precalificador_id UUID REFERENCES precalificadores(id),
  estado VARCHAR(20) DEFAULT 'en_curso',     -- en_curso / completada / abandonada
  preguntas_realizadas JSONB DEFAULT '[]',   -- historial ordenado: [{id, texto, tipo, opciones, origen: 'base'|'ia_en_vivo', respuesta}]
  numero_preguntas_hechas INT DEFAULT 0,     -- control del tope de 10
  datos_contacto JSONB,                      -- {nombre, apellido, telefono, correo}
  alternativa_seleccionada VARCHAR(100),
  destino_cierre VARCHAR(50),                -- "lorena" | "equipo" (ambos → Lorena)
  iniciada_en TIMESTAMP DEFAULT NOW(),
  completada_en TIMESTAMP
);

-- Un lead = el registro definitivo que Lorena gestiona (micro-CRM)
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sesion_id UUID REFERENCES sesiones(id),
  precalificador_nombre VARCHAR(200),
  nombre VARCHAR(200) NOT NULL,
  apellido VARCHAR(200),
  correo VARCHAR(200) NOT NULL,
  telefono VARCHAR(30) NOT NULL,
  respuestas_resumen JSONB NOT NULL,         -- todo el historial de preguntas/respuestas, legible
  alternativa_seleccionada VARCHAR(100),
  cita_timestamp TIMESTAMP,
  cita_enlace_meet TEXT,
  fecha_creado TIMESTAMP DEFAULT NOW(),
  estado VARCHAR(30) DEFAULT 'nuevo'          -- nuevo / contactado / confirmado / descartado
);

-- Notas fechadas de Lorena sobre un lead (micro-CRM)
CREATE TABLE notas_lead (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID REFERENCES leads(id) ON DELETE CASCADE,
  texto TEXT NOT NULL,
  creada_en TIMESTAMP DEFAULT NOW()
);
```

**Nota sobre borrado:** Lorena puede borrar un `lead` completo (y sus `notas_lead` en cascada) desde el dashboard. No hay borrado automático por tiempo — retención indefinida por decisión explícita.

## 15. Módulos principales

1. **Motor de flujo público** — renderiza preguntas, gestiona progreso, aplica el tope de 10.
2. **Motor de IA — construcción** — genera el borrador de preguntas al crear un Precalificador, a partir de nombre + descripción + objetivo.
3. **Motor de IA — en vivo** — durante una sesión real, decide si hace falta una pregunta adicional (dentro del tope), respetando guardrails.
4. **Módulo de contacto** — mini-formulario consolidado (una pantalla, cuatro campos).
5. **Módulo de cierre y agenda** — selector de percepción + integración real con Google Calendar.
6. **Módulo de notificaciones** — email a Lorena vía Gmail al completarse un lead.
7. **Dashboard / Panel de Lorena** — login, leads, notas, creación y edición de Precalificadores, generación de enlaces embebibles.

## 16. Integraciones

- **OpenAI API** — modelo ligero/económico. Dos usos distintos: generación build-time (sin presión de tiempo) y decisión en vivo (debe responder en <2-3 segundos para no romper el ritmo del autoservicio).
- **Google Calendar API** — cuenta de Lorena, para crear el evento y el enlace de Google Meet automáticamente.
- **Gmail (SMTP o Gmail API)** — cuenta `segurosdesaludyvidaahora@gmail.com` como remitente de notificaciones a Lorena.
- **Supabase** — base de datos + autenticación del dashboard.
- **10MinutesWebsite** — únicamente consumidor pasivo vía `<iframe src="https://precalificador.vercel.app/p/[codigo]">`.
## 17. Motor de IA o automatización

### 17.1 Generación en construcción (build-time)

- Input: nombre, descripción, objetivo (texto libre escrito por Lorena).
- Output: borrador de preguntas cerradas (opción única, rango, sí/no), cada una con sus opciones y a qué pregunta lleva cada opción.
- Lorena revisa y aprueba antes de publicar el Precalificador. Puede editar cualquier pregunta u opción manualmente.
- Este borrador se guarda en `precalificadores.preguntas_base` y es lo que arranca cada sesión real.

### 17.2 Razonamiento en vivo (durante la sesión del prospecto)

- Después de agotar las preguntas base (o en cualquier punto que el modelo lo considere valioso), el motor puede llamar a OpenAI con: el objetivo del Precalificador + las respuestas ya dadas + el conteo de preguntas ya hechas.
- Si el modelo determina que una pregunta adicional ayudaría a cumplir el objetivo, la genera con un tono de transición ("una pregunta más...") y la presenta como una pantalla normal (mismo componente `PantallaDePregunta`).
- **Tope duro:** nunca se supera `max_preguntas` (10 por defecto) contando base + en vivo. Al llegar al tope, el flujo pasa automáticamente a contacto/alternativas/cierre aunque la IA "quisiera" preguntar más.
- **Guardrails obligatorios** (ver sección 19) se aplican a toda pregunta generada en vivo antes de mostrarla — si una pregunta generada viola alguna regla, se descarta silenciosamente y el flujo avanza sin ella (nunca se le muestra al prospecto una pregunta sin validar).

### 17.3 Aprendizaje diferido (no en MVP, documentado para fase futura)

- Periódicamente, un proceso puede analizar las respuestas guardadas en `sesiones.preguntas_realizadas` y sugerir a Lorena ajustes a `preguntas_base` (ej. "esta pregunta casi nadie la responde distinto"). Esto es un backlog de Fase 3+, no parte del MVP.

## 18. Reglas de negocio

Heredadas directamente de `SEGUROS-SALUD-VIDA-CODEX/01-strategy/03-WIDGET-AUTOSERVICIO-MASTER.md` y `08-AUTOSERVICIO-ARQUITECTURA-ACORDADA.md` — **no negociables**:

1. No es un formulario de leads. Debe sentirse como explorar → elegir → personalizar → revisar → avanzar.
2. 1 pantalla = 1 pregunta. Nunca mostrar el cuestionario completo.
3. Nunca afirmar compra, aprobación, emisión de póliza o "cotización final".
4. Datos de contacto se piden después de que la persona ya avanzó en el flujo (nunca al inicio), consolidados en una sola pantalla (regla nueva de esta versión — sustituye la idea original de "pedirlos de a uno intercalados").
5. Máximo 3 alternativas al mostrar opciones de producto.
6. Máximo 10 preguntas totales por Precalificador.
7. Nunca preguntar: SSN, documentos migratorios, información bancaria, historial médico detallado, condiciones preexistentes.
8. El resultado que llega a Lorena debe ser legible y accionable en menos de 30 segundos.
9. Lenguaje según `02-LENGUAJE-CLAVE-GSC-GA.md`: explorar, opciones, entender, comparar, proteger, cobertura, según tu situación, en español, económico/asequible, Florida/Miami/Doral. Evitar: aprobado, calificas, precio final, compra ahora, cotización instantánea falsa, lenguaje migratorio categórico.
10. El selector de cierre "Lorena / un miembro del equipo" se mantiene por percepción de marca aunque ambas rutas terminen en Lorena.

## 19. Seguridad y privacidad

- **Guardrails de contenido para la IA (build-time y en vivo):** antes de persistir o mostrar cualquier pregunta generada por IA, validar contra una lista de patrones prohibidos (SSN / número de seguro social, pasaporte, licencia, número de cuenta bancaria, tarjeta de crédito, diagnóstico médico específico, estatus migratorio detallado más allá de lo ya cubierto por las puertas). Si la pregunta generada coincide con algún patrón, se descarta automáticamente y no se le muestra al prospecto.
- **Autenticación del dashboard:** Supabase Auth, un único usuario (Lorena), contraseña gestionada por ella. Sin registro público de nuevos usuarios admin.
- **Datos de prospectos:** retención indefinida por decisión de negocio (micro-CRM). Lorena puede agregar notas fechadas y borrar un lead completo manualmente en cualquier momento (borrado en cascada de sus notas).
- **Credenciales sensibles:** API keys de OpenAI, Google Calendar y Gmail se manejan como variables de entorno en Vercel, nunca expuestas al frontend.
- **Transporte:** todo tráfico sobre HTTPS (nativo de Vercel).
- **Aislamiento del iframe:** el Precalificador no comparte cookies ni estado con 10MinutesWebsite; es un origen independiente.

## 20. Historial, logs y recuperación

- `sesiones.preguntas_realizadas` guarda el historial completo y ordenado de cada sesión (qué se preguntó, si fue pregunta base o generada en vivo, y qué respondió el prospecto) — sirve tanto de auditoría como de insumo para el aprendizaje diferido futuro (sección 17.3).
- `notas_lead` funciona como bitácora manual de Lorena sobre cada prospecto (con fecha).
- No se requieren backups especiales más allá de los que Supabase ofrece por defecto en este MVP.
## 21. Dashboard o interfaz

**Login:** usuario/contraseña, exclusivo de Lorena.

**Secciones:**

1. **Resumen** — nuevos leads recientes, próximas citas.
2. **Leads** — tabla con filtros (por Precalificador, por fecha, por estado); detalle de cada uno con todas las respuestas, notas fechadas (agregar/ver/borrar), y botón de borrado del lead completo.
3. **Precalificadores** — lista de segmentos configurados; botón "Crear nuevo" (nombre + descripción + objetivo → la IA arma el borrador → Lorena revisa/edita/aprueba); cada Precalificador muestra su enlace único listo para copiar y entregar para el `<iframe>`.

No se requieren notificaciones push dentro del dashboard para el MVP — el aviso de nuevo prospecto llega por correo (sección 16).

## 22. Roadmap sugerido

### Fase 1 — MVP funcional completo (piloto: 1 Precalificador, ej. "Familia")
- Setup Next.js + Supabase + Vercel.
- Modelo de datos completo (sección 14.3).
- Motor de flujo público (`PantallaDePregunta` genérico + progreso + tope de 10).
- Generación de preguntas base vía OpenAI (build-time) + pantalla de revisión/aprobación para Lorena.
- Razonamiento en vivo vía OpenAI con guardrails aplicados.
- Formulario de contacto consolidado.
- Cierre con selector + integración real con Google Calendar.
- Notificación por email vía Gmail a Lorena.
- Dashboard: login, leads (con notas y borrado), creación de Precalificadores, enlace embebible.
- Deploy en Vercel, prueba con el Precalificador "Familia" end-to-end.

**Criterio de éxito:** Lorena crea el Precalificador "Familia" ella misma desde el dashboard, un prospecto de prueba lo completa (incluyendo al menos una pregunta generada en vivo), Lorena recibe el email, ve el lead completo en el dashboard, y la cita queda en su Google Calendar real.

### Fase 2 — Réplica a las 6 puertas
- Lorena crea, con el mismo dashboard, los 5 Precalificadores restantes (Económico, Embarazo, Independiente, Mudanza, Dudas migratorias).
- Ningún código nuevo debería ser necesario — es prueba de que el motor genérico funciona.

### Fase 3 — Aprendizaje diferido y refinamiento
- Análisis periódico de `sesiones.preguntas_realizadas` para sugerir ajustes a Lorena.
- Ajustes de copy y tono según datos reales de uso.

### Fase 4 (futuro, no planificada aún) — Multitenant real
- Solo si aparece un segundo negocio/cliente real. Hoy el modelo de datos ya lo permite (`tenant_id`), pero no se construye UI para eso ahora.

## 23. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| La IA en vivo pregunta algo indebido | Guardrails de validación (sección 19) antes de mostrar cualquier pregunta generada; descarte silencioso si falla la validación |
| Latencia de la llamada a OpenAI en vivo rompe el ritmo del autoservicio | Usar modelo ligero/económico; mostrar un estado de transición breve ("un momento...") en vez de dejar la pantalla congelada |
| Costo variable de OpenAI por sesión | Tope duro de 10 preguntas limita el máximo de llamadas por sesión |
| La IA se equivoca al generar preguntas base | Lorena revisa y aprueba manualmente antes de publicar cada Precalificador — nunca corre sin aprobación humana previa |
| Pérdida de una sesión a medio completar (usuario cierra el navegador) | La sesión queda guardada en Supabase con su progreso; se puede reanudar si vuelve a entrar con el mismo enlace (mejora deseable, no bloqueante para MVP) |
| Iframe no se ve bien en móvil dentro de 10MinutesWebsite | Diseño mobile-first del Precalificador, probado independientemente antes de integrarlo |
| Único usuario admin (Lorena) pierde su contraseña | Flujo estándar de recuperación de contraseña de Supabase Auth |

## 24. Criterios de aceptación

- [ ] Cada pantalla del flujo público muestra exactamente 1 pregunta.
- [ ] El progreso es visible en todo momento, incluso cuando la IA agrega una pregunta en vivo.
- [ ] Nunca se superan 10 preguntas totales en ninguna sesión.
- [ ] Ninguna pregunta generada por IA (build-time o en vivo) pide SSN, documentos migratorios, datos bancarios o historial médico detallado — verificado por los guardrails.
- [ ] El formulario de contacto aparece en una sola pantalla consolidada.
- [ ] El cierre siempre agenda con Lorena en Google Calendar real, sin importar qué opción del selector se elija.
- [ ] Lorena recibe un correo desde `segurosdesaludyvidaahora@gmail.com` por cada nuevo lead.
- [ ] Lorena puede crear un Precalificador nuevo desde el dashboard sin ayuda técnica (nombre + descripción + objetivo → revisa preguntas de IA → aprueba → obtiene enlace).
- [ ] Lorena puede agregar notas fechadas a un lead y borrarlo por completo si lo desea.
- [ ] Ningún texto del flujo usa lenguaje prohibido (aprobado, cotización final, garantizado, etc.).
- [ ] El Precalificador funciona correctamente embebido vía iframe, en desktop y en móvil.

## 25. Instrucciones finales antes de escribir código

1. Este documento debe leerse completo — junto con `SEGUROS-SALUD-VIDA-CODEX/01-strategy/03-WIDGET-AUTOSERVICIO-MASTER.md` y `08-AUTOSERVICIO-ARQUITECTURA-ACORDADA.md` — antes de tocar código.
2. No programar las 6 puertas a la vez. Construir y validar completo el piloto ("Familia") primero.
3. Ante cualquier decisión de negocio no cubierta aquí (copy exacto, tono de una pregunta, qué alternativas mostrar en un segmento específico), **detenerse y preguntar** — no improvisar sobre reglas de negocio.
4. Las decisiones técnicas de implementación (nombres de funciones, estructura interna de componentes, librerías auxiliares menores) sí quedan a criterio de Claude Code, siempre dentro del stack aprobado en la sección 14.1.
5. Cualquier cambio de alcance respecto a este documento debe reflejarse aquí mismo (actualizar este archivo), no vivir solo en el historial de chat.

---

**Documento aprobado para iniciar Fase 0 de implementación.**  
**Próximo paso:** Setup del proyecto Next.js + Supabase y construcción del Precalificador piloto ("Familia").
