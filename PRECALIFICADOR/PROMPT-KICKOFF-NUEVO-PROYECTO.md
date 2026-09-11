# PROMPT — Kickoff del proyecto independiente "Precalificador" (SaaS)

**Propósito de este archivo:** respaldo del prompt de arranque entregado al desarrollador/sesión que construye el Precalificador como SaaS genérico, independiente de este repositorio. Este archivo vive aquí solo como referencia histórica de la decisión — el proyecto real NO se construye en este repo (ver advertencia dentro del prompt).

**Carpeta local del proyecto nuevo:** `/Volumes/miltondavila/Precalificador` (en la Mac mini de Milton Dávila, accesible por red local).

---

```
Vas a arrancar la construcción de "Precalificador": un SaaS de autoservicio
inteligente que precalifica prospectos con preguntas guiadas por IA y los
agenda en una cita real con un asesor humano.

Tu carpeta de trabajo es:
/Volumes/miltondavila/Precalificador
(en la Mac Mini de Milton Dávila)

Este es un proyecto NUEVO E INDEPENDIENTE. No es una copia del proyecto de
Seguros de Salud y Vida ni vive dentro de su carpeta o su repositorio.

═══════════════════════════════════════════════════════════════════
PRINCIPIO RECTOR — esto no se pierde de vista NUNCA:

Seguros de Salud y Vida (Lorena Alvarez) es el PRIMER CLIENTE de este
producto. Toda la información de negocio, reglas, lenguaje y ejemplos que
vas a encontrar en su documentación son tu referencia constante y tu caso
de validación real.

PERO no vas a programar exclusivamente para ese cliente, ni vas a usar su
carpeta, su repositorio ni su infraestructura. Este SaaS se construye de
forma INDEPENDIENTE — genérico desde la arquitectura, para poder venderse
a cualquier negocio después.

Información y ejemplo = siempre Seguros de Salud y Vida.
Código, repo, infraestructura y documentación de este proyecto = propios,
independientes, genéricos. Nunca mezcles estas dos cosas.
═══════════════════════════════════════════════════════════════════

⚠️ INFRAESTRUCTURA — MUY IMPORTANTE, léelo antes de crear nada:

Este proyecto se construye con cuenta y repositorio propios de SolucionWeb:
- GitHub: repo nuevo bajo la cuenta/organización de SolucionWeb.
- Vercel: proyecto nuevo bajo la cuenta de SolucionWeb.
- Supabase: proyecto nuevo bajo la cuenta de SolucionWeb.

NUNCA uses el GitHub, Vercel o Supabase de Lorena/Seguros de Salud y Vida
— son de uso exclusivo de ese negocio. Si no tenés acceso a las cuentas de
SolucionWeb o no sabés cuáles son, detenete y pedile los datos a Milton
antes de crear nada.

═══════════════════════════════════════════════════════════════════
PASO 1 — Leer, solo como referencia (NO copiar literal, NO editar esos
archivos, son de otro proyecto):

https://github.com/segurosdesaludyvidaahora-prog/segurosdesaludyvida/tree/main/PRECALIFICADOR

1. README.md
2. MASTER_BLUEPRINT.md — el producto tal como se pensó originalmente para
   el cliente piloto: visión, flujo de 12 pantallas, reglas de negocio,
   modelo de datos, motor de IA (build-time + en vivo), seguridad,
   dashboard, criterios de aceptación.
3. PLAN-DE-FASES-DESARROLLO.md — el pivote a SaaS multi-negocio ya
   decidido: tabla `negocios`, RLS por negocio, Google Calendar y
   notificaciones por negocio (no hardcodeados), y las 3 etapas de
   construcción (fundación multi-negocio → motor de IA + dashboard →
   integraciones y validación con un segundo negocio de prueba).

Leelos completos. Ahí está TODO lo que ya decidimos: por qué el motor es
uno solo y genérico (no una app por segmento), por qué las preguntas las
genera la IA a partir de un objetivo y no las escribe Lorena a mano, por
qué hay un tope duro de 10 preguntas con guardrails de contenido
prohibido, por qué el dato de contacto va en una sola pantalla
consolidada, y por qué cada negocio necesita su propio Calendar y su
propio correo de notificación para que esto sea vendible.

═══════════════════════════════════════════════════════════════════
PASO 2 — Crear, en TU carpeta (/Volumes/miltondavila/Precalificador), tu
propio set de documentos. No los copies de SSV: reescribilos generalizados,
usando lo del Paso 1 como insumo de contenido, no como plantilla a calcar.

Necesitás como mínimo estos archivos:

1. README.md
   Índice del proyecto: qué es, en qué orden se lee el resto, estado
   actual.

2. MASTER_BLUEPRINT.md
   La versión GENÉRICA del blueprint: mismo producto, misma inteligencia
   (motor de flujo, generación de preguntas por IA, razonamiento en vivo
   con tope de 10 y guardrails, contacto consolidado, cierre y agenda),
   pero escrito para "un negocio" en abstracto, no para Seguros de Salud
   y Vida. El modelo de datos debe incluir la tabla `negocios` (tenant)
   desde el diseño base, no como agregado posterior.

3. REGLAS-DE-COORDINACION.md
   Cómo se coordina el trabajo en ESTE proyecto: dónde vive el código
   (SolucionWeb GitHub/Vercel/Supabase), flujo de trabajo (local →
   revisión → producción), cómo se manejan credenciales (nunca se
   escriben tokens/API keys en ningún documento del proyecto — esa regla
   se mantiene igual que en SSV), y cualquier convención de código que
   definas.

4. TODO.md o ROADMAP.md
   Lista de tareas concreta basada en las 3 etapas de
   PLAN-DE-FASES-DESARROLLO.md del Paso 1, pero adaptada como plan de
   ESTE repo (no reference al de SSV como si fuera el mismo proyecto).

5. FASE_0_ARQUITECTURA.md (o el nombre que prefieras)
   El mismo principio de Fase 0 que ya usamos: no se escribe código de la
   aplicación hasta que arquitectura, modelo de datos, flujos y roadmap
   estén documentados aquí y aprobados por Milton.

Seguros de Salud y Vida se documenta en estos archivos nuevos como **el
negocio piloto (tenant #1)** — su información real (nombre, segmentos,
reglas de negocio del seguro, lenguaje de marca) se usa como el EJEMPLO
concreto con el que vas a poblar y probar el sistema, pero el producto
que describís en tus documentos es genérico.

═══════════════════════════════════════════════════════════════════
REGLA FINAL: no escribas código de la aplicación hasta tener el Paso 2
completo y que Milton confirme que la arquitectura está aprobada. Si algo
del Paso 1 es ambiguo o te genera dudas de "esto es del cliente o esto es
del producto", detenete y preguntá antes de decidir solo.
```

