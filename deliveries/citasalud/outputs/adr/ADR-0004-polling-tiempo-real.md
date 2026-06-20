# ADR-0004 · Estrategia de actualización en tiempo real: polling periódico con ruta de escalado a SSE

**Estado:** aceptado
**Fecha:** 2026-06-20

---

## Contexto y fuerza

**US-04** y **US-05** usan el término "en tiempo real" y "de inmediato":

- US-04, criterio 2: "Dado que se cancela o agrega una cita, cuando ocurre el cambio, **entonces la agenda de la médico se actualiza sin necesidad de recargar manualmente**."
- US-05, criterio 1: "cuando confirma [el bloqueo], entonces esas franjas desaparecen de la disponibilidad pública **de inmediato**."
- `requisitos.md` req:R-04: "La médico debe poder consultar y gestionar su agenda desde cualquier dispositivo, sin necesidad de llamar a la clínica."
- req:R-07: "Los bloqueos deben reflejarse de inmediato para todos."

El Scrum Master señaló que **la estrategia de actualización (WebSockets, SSE, polling) no está resuelta y podría afectar las estimaciones de 3 pts por historia** (US-04 y US-05). Esta decisión tiene impacto directo en la complejidad de implementación y en la validez de las estimaciones del sprint.

El contexto de uso es: **una médica que consulta su propia agenda desde el celular** (`personas.md` / `doctora.md`). No hay evidencia en el inbox de múltiples usuarios concurrentes en la vista de agenda, ni de un panel de control compartido con actualizaciones de alta frecuencia.

---

## Decisión

Se implementa **polling HTTP periódico cada 15 segundos** desde el cliente de agenda (vista de la médico) al endpoint `GET /agenda?fecha=<fecha>`.

- La latencia máxima de actualización es de 15 segundos: si se cancela una cita o se crea un bloqueo, la médico lo verá en su próxima respuesta de polling.
- Para los bloqueos de US-05: el bloqueo se persiste de forma síncrona al confirmar (`POST /bloqueos`). Desde ese momento, el endpoint de disponibilidad pública del paciente (`GET /disponibilidad`) ya no devuelve esa franja. La médico ve el bloqueo confirmado también en el siguiente ciclo de polling de su agenda.
- Si el feedback de usuarios post-MVP muestra que 15 segundos de latencia es inaceptable en la práctica, se escala a **Server-Sent Events (SSE)**: el servidor mantiene una conexión unidireccional y envía eventos al cliente cuando hay cambios. Este cambio no requiere modificar la lógica de negocio ni el esquema de base de datos; solo agrega un endpoint SSE en el API y un listener en el frontend.

---

## Alternativas consideradas

- **WebSockets (conexión bidireccional persistente)** — Permite push en tiempo real con latencia mínima. Sin embargo, requiere infraestructura adicional (un servidor con soporte de conexiones persistentes, potencialmente un proxy WebSocket, gestión de reconexiones). Para una clínica con una médica que consulta su agenda desde el celular, la diferencia perceptible entre 0 ms y 15 s de latencia no está documentada como un dolor real en el inbox. Añade complejidad de infraestructura y desarrollo que no está justificada por evidencia. Además, el inbox no indica que la vista de agenda deba reaccionar a eventos de múltiples usuarios simultáneos con alta frecuencia. `Descartado` para el MVP; elegible en fase 2 si el volumen o el número de usuarios aumenta.
- **Server-Sent Events (SSE) desde el inicio** — Técnicamente más ligero que WebSockets para actualizaciones unidireccionales (servidor → cliente). Es la ruta natural de escalado desde polling. Se descarta para el MVP no por motivos técnicos sino porque añade complejidad de implementación (gestión de conexiones abiertas, reconexión automática en el cliente) que no está justificada hasta validar que 15 s de latencia no es aceptable. `Descartado` para el MVP; es el primer escalado si el polling no cumple.
- **Polling con intervalo corto (1-2 segundos)** — Simularía tiempo real, pero generaría carga innecesaria en el servidor para un beneficio no validado. `Descartado`; 15 s es el punto de partida (OQ-05 en `architecture.md` para ajustar post-despliegue).
- **Long polling** — Más complejo de implementar que polling simple y con ventajas marginales respecto a SSE; no aporta beneficio incremental claro sobre la ruta polling → SSE elegida. `Descartado`.

---

## Consecuencias

**Lo que se gana:**
- Implementación simple: un endpoint GET estándar, sin infraestructura adicional. Las estimaciones de 3 pts para US-04 y US-05 son válidas con esta estrategia.
- Sin dependencia de librerías de WebSocket ni configuración de proxy; el servidor puede ser cualquier framework HTTP estándar.
- La ruta de escalado a SSE está documentada y no requiere cambios en la lógica de negocio (solo en la capa de transporte).

**El costo que se acepta:**
- Latencia máxima de 15 segundos en la vista de agenda de la médico. En la práctica, si la médico acaba de crear un bloqueo ella misma (US-05), lo ve confirmado de inmediato porque el frontend actualiza localmente al recibir la respuesta 200 OK del `POST /bloqueos`; el polling solo aplica a cambios iniciados por otros usuarios (paciente que cancela, recepcionista que modifica).
- Carga de red proporcional al número de clientes en la vista de agenda. Para una médica y una recepcionista, es despreciable; podría revisarse si se agregan más usuarios en fase 2.
- La decisión de escalar a SSE debe estar documentada como criterio de aceptación en el feedback de la fase 1: "si la médico reporta que la latencia de actualización interfiere con su flujo de trabajo, se implementa SSE en la siguiente iteración".
