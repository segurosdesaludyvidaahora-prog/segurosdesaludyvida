# FASE 0 — Checklist de aprobación de arquitectura

**No deberá escribirse una sola línea de código de la aplicación hasta que este checklist esté marcado como aprobado.**

Referencia completa: `MASTER_BLUEPRINT.md`.

## Arquitectura
- [x] Arquitectura funcional definida (motor único, flujos como configuración por Precalificador, no apps separadas)
- [x] Stack tecnológico definido (Next.js, Vercel, Supabase, OpenAI, Google Calendar, Gmail)
- [x] Modelo de datos definido (precalificadores, sesiones, leads, notas_lead)
- [x] Flujos de usuario definidos (público + creación de Precalificador por Lorena)
- [x] Estructura de carpetas propuesta
- [x] Roadmap por fases (ver `ROADMAP.md`)
- [x] Riesgos técnicos identificados y con mitigación propuesta
- [x] Criterios de aceptación definidos

## Pendiente de aprobación explícita antes de programar
- [ ] Milton confirma inicio de Fase 1
- [ ] Lorena confirma que el flujo de creación de Precalificadores (nombre + descripción + objetivo → IA genera → ella aprueba) es el que espera usar
- [ ] Confirmar cuenta/proyecto de OpenAI a usar (API key) y presupuesto esperado por sesión
- [ ] Confirmar acceso a Google Calendar de Lorena (credenciales OAuth) para la integración
- [ ] Confirmar acceso a la cuenta `segurosdesaludyvidaahora@gmail.com` para configurar el envío de notificaciones

Una vez marcados estos puntos, se procede a Fase 1 sin necesidad de repetir la entrevista de descubrimiento — todo el contexto de negocio ya vive en `MASTER_BLUEPRINT.md`.
