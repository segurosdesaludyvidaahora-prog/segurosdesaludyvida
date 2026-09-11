# PLAN DE FASES DE DESARROLLO — Precalificador (SaaS multi-negocio)

**Estado:** VIGENTE — define cómo se ejecuta la construcción, dividida en 3 etapas para que cada una la pueda tomar un desarrollador (o una sesión de Claude Code) distinta, sin necesitar el historial de conversación previo.

**Cómo leer este documento:** junto con `MASTER_BLUEPRINT.md` (fuente única de verdad del producto). Este documento no repite reglas de negocio ni modelo de datos completo — solo dice **qué se construye en qué orden, quién lo hace, y qué debe dejar listo para la siguiente etapa**.

---

## 1. Por qué existe este documento

Dos decisiones tomadas el 2026-09-11 cambian cómo se ejecuta el proyecto:

1. **Pivote de producto:** el Precalificador deja de ser una herramienta exclusiva de Seguros de Salud y Vida y pasa a ser un **SaaS multi-negocio genérico**, pensado para venderse a cualquier negocio que necesite precalificar prospectos y agendarlos con un asesor humano (agencias de seguros, clínicas, abogados, inmobiliarias, etc.). Seguros de Salud y Vida es el **negocio piloto (tenant #1)** — el primero en usarlo, no el único cliente para el que se diseña.
2. **Ejecución por etapas independientes:** para que cada desarrollador (o sesión de Claude Code) trabaje con el mínimo de contexto necesario, el trabajo se divide en 3 etapas secuenciales. Cada etapa tiene su propio checklist de entrada/salida — quien la ejecute NO necesita leer esta conversación, solo este documento + `MASTER_BLUEPRINT.md` + el estado real del repositorio.

---

## 2. Qué cambia respecto a `MASTER_BLUEPRINT.md` v3.0 (deltas del pivote SaaS)

`MASTER_BLUEPRINT.md` sigue siendo la fuente de verdad del **producto** (flujo, reglas de negocio, UX, seguridad). Estos son los ajustes de **arquitectura** que introduce el pivote a multi-negocio, y que cada etapa debe respetar:

| Tema | v3.0 (single-tenant) | Ahora (multi-negocio real) |
|---|---|---|
| Tenant | `tenant_id` fijo, un solo valor, "preparado a futuro" | Tabla `negocios` de primer nivel (ver sección 4). Todo dato cuelga de un `negocio_id` real, no de un valor por defecto. |
| Aislamiento de datos | No implementado (no hacía falta con 1 solo cliente) | **Row Level Security (RLS) de Supabase por `negocio_id`**, obligatorio desde el Bloque 1 — es lo que hace que sea vendible sin riesgo de fuga de datos entre clientes. |
| Autenticación del dashboard | "Login exclusivo de Lorena" | Login por negocio — cada negocio tiene su(s) propio(s) usuario(s) admin, ve solo sus propios datos. Lorena es el admin del negocio "Seguros de Salud y Vida", no un caso especial en el código. |
| Google Calendar | Cuenta de Lorena hardcodeada | **Cada negocio conecta su propia cuenta de Google Calendar** vía OAuth desde su dashboard. Sin esto, el producto no es vendible a un segundo cliente. |
| Notificaciones por email | Hardcodeado a `segurosdesaludyvidaahora@gmail.com` | Cada negocio configura su propio correo de notificación (o se usa un remitente transaccional propio del SaaS con "reply-to" del negocio — decisión técnica de la Etapa 3, no bloqueante ahora). |
| Branding en el flujo público | No contemplado | Cada negocio tiene nombre, logo y color primario propios, aplicados al flujo público que ve el prospecto (mínimo viable — no es un rediseño completo por cliente). |
| Alta de un negocio nuevo | No aplica | **Manual por ahora**: un superadmin (Milton) crea el registro en `negocios` directamente en Supabase o vía un panel interno simple. Auto-registro (signup) y cobro/planes quedan fuera de alcance por ahora — es un paso posterior, no bloqueante para vender de forma asistida. |

**Regla de decisión para cualquier ambigüedad nueva que surja durante la construcción:** si una decisión de arquitectura solo tiene sentido para "un cliente" (ej. hardcodear un nombre, un correo, un color), es señal de que se está rompiendo el pivote — detenerse y resolverlo de forma genérica antes de continuar.

---
## 3. Las 3 etapas

Cada etapa produce artefactos concretos en el repositorio (código, migraciones, tipos) que son la única "memoria" que la siguiente etapa necesita. Ninguna etapa depende de leer esta conversación.

### ETAPA 1 — Fundación multi-negocio + motor de flujo genérico

**Objetivo:** dejar corriendo el esqueleto técnico completo, aislado por negocio, con un flujo público funcional usando preguntas de prueba (sin IA todavía).

**Tareas:**
- Setup del proyecto Next.js 15 + React 19, repo propio (`precalificador/`, separado del scaffold `vinext`/Cloudflare que usa el HOME), deploy en Vercel.
- Proyecto Supabase nuevo. Modelo de datos completo (ver `MASTER_BLUEPRINT.md` sección 14.3) **más la tabla nueva `negocios`**:
  ```sql
  CREATE TABLE negocios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    slug VARCHAR(80) UNIQUE NOT NULL,           -- usado en URLs: /p/[slug]/[codigoPrecalificador]
    nombre VARCHAR(200) NOT NULL,
    logo_url TEXT,
    color_primario VARCHAR(20),                 -- hex, aplicado al flujo público
    correo_notificacion VARCHAR(200),            -- a dónde llegan los avisos de nuevo lead
    google_calendar_conectado BOOLEAN DEFAULT FALSE,
    google_calendar_refresh_token TEXT,          -- cifrado / variable de entorno gestionada, nunca en claro en la tabla si Supabase no lo permite cifrado a nivel de columna — usar Supabase Vault o equivalente
    estado VARCHAR(20) NOT NULL DEFAULT 'activo',
    creado_en TIMESTAMP DEFAULT NOW()
  );
  ```
  - `precalificadores.tenant_id` pasa a ser `precalificadores.negocio_id UUID REFERENCES negocios(id)` (sin default fijo — cada fila debe apuntar a un negocio real).
  - Agregar `negocio_id` también a `sesiones` y `leads` de forma denormalizada (no solo vía join con `precalificadores`), para que las políticas RLS sean simples y rápidas.
- **RLS obligatorio:** políticas de Supabase que garanticen que un usuario autenticado solo puede leer/escribir filas de su propio `negocio_id`. Esto es un criterio de aceptación de esta etapa, no una mejora opcional.
- Supabase Auth: relación usuario ↔ negocio (tabla `usuarios_negocio` o campo `negocio_id` en metadata del usuario). Al crear el negocio piloto, crear también el usuario de Lorena asociado a él.
- Motor de flujo público genérico: `PantallaDePregunta`, `ProgresoBarra`, `RenderizadorOpciones`, navegación — corriendo con un Precalificador de prueba con preguntas fijas (sin llamadas a OpenAI todavía). Aplica branding del negocio (logo/color) leído desde `negocios`.
- Sembrar en la base: el negocio piloto "Seguros de Salud y Vida" (slug `segurosdesaludyvida`) con un Precalificador de prueba "Familia" con 3-4 preguntas fijas, solo para validar que el motor funciona de punta a punta.

**Qué NO hace esta etapa:** nada de IA (ni build-time ni en vivo), nada de Google Calendar/Gmail reales, nada de dashboard de creación de Precalificadores todavía (puede haber un formulario mínimo interno solo para insertar el Precalificador de prueba vía SQL/seed, no vía UI).

**Entregable de salida (lo que la Etapa 2 debe encontrar ya hecho):**
- Repo desplegado en Vercel, accesible en una URL de prueba.
- Esquema de Supabase migrado y documentado (archivo de migración en `database/migrations/`).
- Tipos TypeScript generados desde el esquema (`lib/tipos/`).
- El flujo público funcionando end-to-end con preguntas fijas para el negocio piloto.
- RLS verificado: confirmar que un segundo negocio de prueba (aunque sea vacío) no puede ver datos del primero.

---
### ETAPA 2 — Motor de IA + dashboard de configuración

**Prerrequisito:** Etapa 1 completa y verificada (schema, RLS, motor de flujo, negocio piloto sembrado).

**Objetivo:** que Lorena (o cualquier admin de cualquier negocio) pueda crear un Precalificador ella misma desde el dashboard, con preguntas generadas por IA, sin ayuda técnica.

**Tareas:**
- Integración con OpenAI (sección 17.1 y 17.2 de `MASTER_BLUEPRINT.md`): generación de preguntas base (build-time) + razonamiento en vivo con guardrails + tope duro de 10 preguntas.
- Guardrails de contenido (`lib/openai/guardrails.ts`) — lista de patrones prohibidos, validación antes de mostrar o persistir cualquier pregunta generada.
- Dashboard — sección "Precalificadores": crear (nombre + descripción + objetivo → IA genera → revisar/editar/aprobar), listar, editar, generar el enlace público (`/p/[slug]/[codigo]`).
- Dashboard — configuración del negocio: logo, color primario, nombre (edición del propio registro `negocios`).
- Reemplazar el Precalificador de prueba sembrado en la Etapa 1 por uno real ("Familia") creado desde este flujo.

**Qué NO hace esta etapa:** cierre/agenda con Google Calendar real (puede quedar simulado/mock), notificaciones por email reales, sección de Leads del dashboard (puede no existir todavía o ser un placeholder).

**Entregable de salida (lo que la Etapa 3 debe encontrar ya hecho):**
- Un Precalificador completo, creado 100% desde el dashboard (sin tocar la base de datos a mano), funcionando de punta a punta hasta la pantalla de resumen (antes del cierre real).
- Guardrails probados con al menos un caso que debería descartarse (ej. forzar que la IA intente preguntar algo prohibido y confirmar que se descarta).

---

### ETAPA 3 — Cierre, integraciones externas y validación multi-negocio

**Prerrequisito:** Etapa 2 completa (Precalificador real funcionando hasta resumen).

**Objetivo:** cerrar el flujo completo (agenda real + notificación real) y **demostrar que el producto sirve para más de un negocio**, no solo para Seguros de Salud y Vida.

**Tareas:**
- Formulario de contacto consolidado (nombre, apellido, teléfono, correo — una pantalla).
- Integración real con Google Calendar: **flujo de conexión OAuth por negocio** desde el dashboard (botón "Conectar mi Google Calendar"), no una cuenta hardcodeada. Al agendar, usa el Calendar del negocio dueño del Precalificador.
- Notificaciones por email: al completarse un lead, enviar aviso al `correo_notificacion` configurado por el negocio (decidir en esta etapa si se envía desde una cuenta propia del SaaS con reply-to del negocio, o si cada negocio conecta su propio Gmail — para Seguros de Salud y Vida específicamente, el envío es desde `segurosdesaludyvidaahora@gmail.com`, pero el mecanismo debe soportar un correo distinto por negocio).
- Dashboard — sección Leads: tabla con filtros, detalle, notas fechadas, borrado (micro-CRM, sección 21 de `MASTER_BLUEPRINT.md`).
- **Prueba de multi-negocio real:** crear un SEGUNDO negocio de prueba (puede ser ficticio, ej. "Negocio Demo") con su propio Precalificador, su propio Calendar de prueba y su propio correo de notificación, y confirmar que funciona de punta a punta sin tocar código ni configuración compartida con Seguros de Salud y Vida. Este es el criterio de aceptación de que el pivote a SaaS realmente funciona.
- Coordinar con la sesión del HOME el reemplazo de los enlaces de WhatsApp temporales (ver `MASTER_BLUEPRINT.md` sección 22, nota "Integración pendiente en el HOME") por el enlace/iframe real del Precalificador de Seguros de Salud y Vida.
- Deploy final, prueba end-to-end en producción.

**Entregable de salida:** producto vendible — un negocio nuevo puede darse de alta (manualmente, ver sección 2 de este documento) y quedar operativo sin escribir código.

---

## 4. Qué queda fuera de las 3 etapas (backlog post-venta, no bloqueante)

- Auto-registro (signup) de negocios nuevos sin intervención manual.
- Cobro / planes / facturación.
- Dominio propio o subdominio por negocio (hoy todos comparten el dominio del SaaS, diferenciados por `slug` en la URL).
- Aprendizaje diferido (sección 17.3 de `MASTER_BLUEPRINT.md`) — sigue siendo backlog, no depende del pivote.
- Blindaje regulatorio/legal para verticales fuera de seguros (ej. si se vende a una clínica, revisar si aplican reglas de datos de salud distintas — **no asumir que las guardrails de seguros alcanzan para cualquier vertical**, evaluarlo cuando aparezca el primer cliente fuera del rubro seguros).

---

## 5. Regla de cierre para cualquier etapa

Al terminar una etapa, quien la ejecutó debe:
1. Actualizar `PHASE_0_ARCHITECTURE_REQUEST.md` / este documento si algo cambió de alcance (no dejarlo solo en el historial de chat de esa sesión).
2. Dejar un commit claro en GitHub con lo que quedó funcionando y lo que no.
3. Confirmar explícitamente los "criterios de aceptación" de esa etapa antes de considerarla cerrada — no pasar a la siguiente etapa con pendientes silenciosos.

