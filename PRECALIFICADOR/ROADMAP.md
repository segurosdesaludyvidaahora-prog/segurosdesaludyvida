# ROADMAP — Precalificador

Detalle completo en `MASTER_BLUEPRINT.md`, sección 22. Resumen:

## Fase 1 — MVP funcional completo (piloto "Familia")
- [ ] Setup Next.js + Supabase + Vercel
- [ ] Modelo de datos (precalificadores, sesiones, leads, notas_lead)
- [ ] Motor de flujo público genérico (`PantallaDePregunta`)
- [ ] Generación de preguntas base vía OpenAI (build-time) + revisión/aprobación de Lorena
- [ ] Razonamiento en vivo vía OpenAI con guardrails
- [ ] Formulario de contacto consolidado (1 pantalla)
- [ ] Cierre + integración real con Google Calendar
- [ ] Notificación por email vía Gmail (`segurosdesaludyvidaahora@gmail.com`)
- [ ] Dashboard: login, leads (notas + borrado), creación de Precalificadores, enlace embebible
- [ ] Deploy en Vercel, prueba end-to-end con "Familia"

**Criterio de éxito:** Lorena crea "Familia" sola desde el dashboard, un prospecto de prueba lo completa (incluyendo al menos 1 pregunta generada en vivo), Lorena recibe el email, ve el lead completo, y la cita queda en su Google Calendar real.

## Fase 2 — Réplica a las 6 puertas
- [ ] Lorena crea los 5 Precalificadores restantes desde el dashboard (Económico, Embarazo, Independiente, Mudanza, Dudas migratorias)
- [ ] Sin código nuevo — valida que el motor genérico funciona para cualquier segmento

## Fase 3 — Aprendizaje diferido y refinamiento
- [ ] Análisis periódico de respuestas guardadas para sugerir ajustes a Lorena
- [ ] Ajustes de copy/tono según datos reales de uso

## Fase 4 (futuro, sin fecha) — Multitenant real
- [ ] Solo si aparece un segundo negocio/cliente real. El modelo de datos ya lo permite (`tenant_id`); no se construye UI para esto ahora.
