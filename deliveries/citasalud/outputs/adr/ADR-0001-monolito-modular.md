# ADR-0001 · Arquitectura monolítica modular

**Estado:** aceptado
**Fecha:** 2026-06-20

---

## Contexto y fuerza

CitaSalud es una clínica pequeña con **una médica, una recepcionista y un conjunto de pacientes recurrentes** (`personas.md`). El MVP cubre exactamente cinco historias de usuario (US-01 a US-05) con un total de 27 puntos de historia. No hay evidencia en el inbox de requisitos de escalabilidad horizontal, equipos de desarrollo múltiples que necesiten desplegar en paralelo, ni dominios técnicos con ciclos de vida distintos.

La fuerza que motiva esta decisión es la combinación de **escala mínima del negocio** (`mvp-canvas.md` — segmento: clínica pequeña) y el **principio rector de la arquitectura** (lo más simple que funcione hoy sin hipotecar el mañana). Añadir microservicios introduce complejidad operativa (orquestación, red entre servicios, trazabilidad distribuida) que el inbox no justifica.

---

## Decisión

Se adopta un **monolito modular**: una sola unidad desplegable que contiene tres módulos internos con fronteras bien definidas:

- **Módulo Reservas** — lógica de disponibilidad y reserva de citas (US-01, US-02)
- **Módulo Agenda** — consulta y bloqueos de franjas por la médico (US-04, US-05)
- **Módulo Notificaciones** — envío de mensajes WhatsApp y gestión del scheduler de recordatorios (US-03)

Los módulos se comunican mediante llamadas directas en memoria (no HTTP interno), comparten la misma base de datos y se despliegan como una sola unidad.

---

## Alternativas consideradas

- **Microservicios (un servicio por módulo)** — Añade latencia de red, complejidad de despliegue y necesidad de infraestructura adicional (service mesh, gateway, trazabilidad distribuida). No hay equipo de varias decenas de personas ni dominio suficientemente independiente que lo justifique. `Descartado`.
- **Monolito sin separación de módulos (big ball of mud)** — Más rápido inicialmente, pero dificulta la futura extracción si el negocio crece (por ejemplo, cuando se agreguen múltiples médicos en fase 2). `Descartado` para preservar la opción de evolución.
- **Arquitectura serverless (funciones)** — Apropiada para cargas muy variables e impredecibles. El volumen de una clínica pequeña es predecible y bajo; el cold start y la complejidad de orquestación no se justifican. `Descartado`.

---

## Consecuencias

**Lo que se gana:**
- Despliegue simple: un proceso, una imagen, un punto de monitoreo.
- Menor tiempo de llegada a producción para el MVP.
- Las fronteras de módulo permiten extraer un servicio en el futuro sin reescribir la lógica de negocio (split justificado si se incorporan múltiples médicos, HIS, o pagos — todos fuera de alcance según `mvp-canvas.md`).

**El costo que se acepta:**
- Escalar horizontalmente implica replicar el monolito completo (aceptable mientras el volumen sea de una clínica).
- Un bug grave en un módulo puede afectar a los demás (mitigado con pruebas por módulo y revisión de código).
- Si el negocio crece significativamente antes de la segunda fase, la deuda de separar módulos aumenta. Se revisará esta decisión cuando se incorpore un segundo médico o cuando el tráfico supere la capacidad de una instancia.
