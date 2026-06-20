# ADR-0003 · Base de datos relacional (PostgreSQL) y bloqueo pesimista para concurrencia

**Estado:** aceptado
**Fecha:** 2026-06-20

---

## Contexto y fuerza

**US-02** establece el criterio de aceptación más estricto del MVP desde el punto de vista de integridad de datos:

> "Dado que dos solicitudes de reserva coinciden en la misma franja, cuando la primera se confirma, **entonces la segunda recibe la franja como bloqueada antes de finalizar**."

`requisitos.md` req:R-02: "El sistema debe validar la disponibilidad en tiempo real e impedir que se asigne una misma franja horaria a más de un paciente."

`personas.md` / `recepcionista.md` pain `dobles-reservas`: ocurren ~2 veces por semana con el sistema manual actual. Es el dolor operativo de mayor frecuencia de M. (recepcionista) y el que más daña la reputación de la clínica (`cita-perdida`, paciente J.).

La fuerza es que **la consistencia ante escrituras concurrentes sobre la misma franja es no negociable**. No es un requisito de rendimiento (la clínica no tiene miles de reservas simultáneas), sino un requisito de correctitud.

---

## Decisión

Se adopta **PostgreSQL** como única base de datos del sistema, con **bloqueo pesimista a nivel de fila** (`SELECT FOR UPDATE`) dentro de una transacción para la operación de reserva:

```sql
-- Dentro de una transacción
SELECT id, estado FROM franjas
WHERE id = :franja_id AND estado = 'disponible'
FOR UPDATE;

-- Si devuelve fila: actualizar estado a 'reservada' y hacer COMMIT
-- Si no devuelve fila (ya estaba ocupada): ROLLBACK y devolver error 409
```

Este mecanismo garantiza que dos peticiones concurrentes sobre la misma franja no pueden ambas leer `disponible` y luego ambas escribir `reservada`; la segunda espera a que la primera libere el bloqueo y entonces lee el estado ya actualizado.

Todas las entidades del dominio (citas, franjas, bloqueos, pacientes) se modelan en el mismo esquema relacional de PostgreSQL.

---

## Alternativas consideradas

- **Base de datos NoSQL (MongoDB, DynamoDB u equivalente)** — Los modelos de documento sin transacciones ACID nativas no garantizan la consistencia requerida por US-02 ante escrituras concurrentes. Aunque algunos NoSQL ofrecen transacciones desde versiones recientes, añaden complejidad de configuración sin ventaja real para un dominio tan estructurado como citas médicas (entidades con relaciones claras: paciente, franja, médico, bloqueo). `Descartado`.
- **Bloqueo optimista (versión de fila / ETag)** — Más eficiente en escenarios de baja contención, pero requiere lógica de reintento en la capa de aplicación: si dos peticiones leen la misma versión y la segunda falla, debe reintentar y potencialmente devolver un error al usuario. Para el volumen de una clínica pequeña, la lógica de reintento no aporta beneficio de rendimiento perceptible y añade complejidad. El bloqueo pesimista es más simple y correcto para este caso. `Descartado` para el MVP; puede reevaluarse si el volumen de reservas simultáneas aumenta significativamente.
- **Bloqueo en memoria (Redis, lock de aplicación)** — Introduce un punto de fallo adicional (el servidor de caché) y complejidad de sincronización si la aplicación se escala horizontalmente. No hay evidencia en el inbox de que el volumen requiera ese nivel de rendimiento. `Descartado`.
- **SQLite** — Adecuado para desarrollo local, pero no es apropiado para producción con acceso concurrente desde múltiples conexiones. `Descartado` para producción.

---

## Consecuencias

**Lo que se gana:**
- Correctitud garantizada por el motor de base de datos: US-02 se cumple por diseño, no por convención.
- Modelo relacional alineado con el dominio: franjas, citas y bloqueos tienen relaciones claras que el esquema relacional expresa de forma natural.
- PostgreSQL es maduro, ampliamente soportado y disponible en la mayoría de los proveedores de hosting considerados (OQ-04 en `architecture.md`).

**El costo que se acepta:**
- El bloqueo pesimista introduce una espera breve si dos usuarios reservan exactamente en el mismo instante (del orden de milisegundos para esta escala). Aceptable para una clínica pequeña.
- Requiere que el equipo comprenda las garantías transaccionales de PostgreSQL; no es un obstáculo real para un equipo competente.
- Si en el futuro el esquema necesita cambios (migración), se requiere gestión de migraciones (por ejemplo: Flyway, Alembic, o equivalente según el lenguaje elegido en OQ-01).
