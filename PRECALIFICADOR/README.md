# Precalificador — Seguros de Salud y Vida

Motor de autoservicio inteligente que precalifica prospectos y los conduce a una cita de ~15 minutos con Lorena Alvarez.

## ⚠️ Este proyecto es un SaaS multi-negocio (no exclusivo de Seguros de Salud y Vida)

Desde 2026-09-11, el Precalificador se construye como producto genérico vendible a cualquier negocio — Seguros de Salud y Vida es el **negocio piloto (tenant #1)**, no el único cliente para el que se diseña. Ver `PLAN-DE-FASES-DESARROLLO.md` para el detalle completo de este cambio.

## Para retomar el trabajo

**Lee, en este orden, y completos:**
1. [`MASTER_BLUEPRINT.md`](./MASTER_BLUEPRINT.md) — fuente única de verdad del producto: visión, flujo, reglas de negocio, seguridad, modelo de datos base, criterios de aceptación.
2. [`PLAN-DE-FASES-DESARROLLO.md`](./PLAN-DE-FASES-DESARROLLO.md) — el pivote a SaaS multi-negocio, los ajustes de arquitectura que introduce, y las 3 etapas de ejecución (pensadas para que cada una la tome un desarrollador distinto sin necesitar contexto de conversaciones previas). **Este es el documento que dice qué construir primero.**

No hace falta contexto adicional de conversaciones previas — ambos documentos son autocontenidos.

**Antes de escribir código de la aplicación**, revisar también:
- `PHASE_0_ARCHITECTURE_REQUEST.md` — checklist de aprobación de Fase 0.
- `ROADMAP.md` — fases de construcción.
- `../SEGUROS-SALUD-VIDA-CODEX/01-strategy/03-WIDGET-AUTOSERVICIO-MASTER.md` — reglas de negocio del autoservicio (documento fuente, no negociable).
- `../SEGUROS-SALUD-VIDA-CODEX/01-strategy/08-AUTOSERVICIO-ARQUITECTURA-ACORDADA.md` — arquitectura original acordada (algunos puntos fueron refinados en el Master Blueprint v3.0 — ese archivo manda en caso de conflicto).

## Estado actual

Fase 0 completa (documentación y arquitectura). Pendiente: setup del proyecto Next.js/Supabase y construcción del piloto ("Familia").

## Carpeta `archivo-v1/`

Contiene las dos versiones preliminares del documento de arquitectura (previas a la entrevista completa con MAGO). Quedaron desactualizadas — el Master Blueprint las reemplaza. Se conservan solo como referencia histórica.
