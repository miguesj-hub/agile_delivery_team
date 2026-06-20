# Historias de usuario refinadas — CitaSalud
> Refinadas el 2026-06-20 · Developer del equipo agil · Todas las historias cumplen INVEST + Definition of Ready

---

## E-01 · Agendamiento en linea sin barreras telefonicas

### US-01 · Reserva de cita en linea 24/7   ·   epica E-01   ·   8 pts
**Como** paciente, **quiero** reservar una cita en linea en cualquier momento del dia, **para** no tener que llamar durante mi horario de almuerzo ni acumular intentos fallidos.

Criterios de aceptacion (Gherkin):
- Dado que el paciente accede al sistema fuera del horario de atencion telefonica, cuando elige medico, fecha y hora disponibles y confirma, entonces la cita queda registrada y el paciente recibe confirmacion por WhatsApp.
- Dado que el paciente intenta seleccionar una franja ya ocupada, cuando intenta confirmarla, entonces el sistema la muestra como no disponible y lo invita a elegir otra franja.

Origen: us:US-01 · req:R-01 · req:R-06 · req:R-09 · evidence:paciente.md · evidence:recepcionista.md

---

### US-02 · Prevencion de dobles reservas   ·   epica E-01   ·   5 pts
**Como** recepcionista, **quiero** que el sistema impida asignar la misma franja horaria a dos pacientes de forma simultanea, **para** que dejen de ocurrir las dobles reservas (~2 por semana) que generan conflictos y dano a la reputacion de la clinica.

Criterios de aceptacion (Gherkin):
- Dado que dos solicitudes de reserva coinciden en la misma franja, cuando la primera se confirma, entonces la segunda recibe la franja como bloqueada antes de finalizar su proceso de reserva.
- Dado que existe una cita activa en una franja, cuando se intenta agregar otra cita en el mismo horario, entonces el sistema rechaza la operacion con un mensaje de conflicto claro.

Origen: us:US-02 · req:R-02 · evidence:recepcionista.md

---

## E-02 · Recordatorios automaticos para reducir no-shows

### US-03 · Recordatorio automatico por WhatsApp con opcion de cancelar   ·   epica E-02   ·   8 pts
**Como** paciente, **quiero** recibir un recordatorio automatico por WhatsApp 24 horas antes de mi cita con opcion de cancelar desde el mismo mensaje, **para** no olvidar la cita ni generar una inasistencia sin aviso que desperdicie el tiempo de la medico y el turno de otro paciente.

Criterios de aceptacion (Gherkin):
- Dado que un paciente tiene una cita agendada, cuando faltan 24 horas para el turno, entonces el sistema envia automaticamente un mensaje de WhatsApp con fecha, hora y nombre de la medico, sin intervencion manual de la recepcionista.
- Dado que el paciente recibe el recordatorio, cuando responde que desea cancelar, entonces la cita queda anulada y la franja vuelve a estar disponible en la agenda en tiempo real.

Origen: us:US-03 · req:R-03 · evidence:recepcionista.md · evidence:paciente.md · evidence:doctora.md

> **Supuestos asumidos:** (1) El costo de la API de WhatsApp Business esta dentro del presupuesto de la clinica (supuesto mvp-canvas.md — confirmacion pendiente con el dueno). (2) Tasa de adopcion suficiente para la base de pacientes; a validar post-lanzamiento con metricas de apertura.

---

## E-03 · Control de agenda para la medico

### US-04 · Consulta de agenda desde celular en tiempo real   ·   epica E-03   ·   3 pts
**Como** medico, **quiero** consultar mi agenda completa desde el celular con actualizacion en tiempo real, **para** no tener que llamar a recepcion para saber cuantos pacientes tengo al dia siguiente ni cuando estoy fuera de la clinica.

Criterios de aceptacion (Gherkin):
- Dado que la medico accede desde un dispositivo movil, cuando abre su vista de agenda, entonces ve la lista de citas del dia con nombre del paciente, hora y estado (confirmada / cancelada).
- Dado que se cancela o agrega una cita, cuando ocurre el cambio, entonces la agenda de la medico se actualiza sin necesidad de recargar manualmente la pagina.

Origen: us:US-04 · req:R-04 · req:R-08 · evidence:doctora.md

---

### US-05 · Bloqueo de franjas de agenda por la medico   ·   epica E-03   ·   3 pts
**Como** medico, **quiero** bloquear franjas de mi agenda (congresos, vacaciones, imprevistos) para que esos horarios no esten disponibles para los pacientes, **para** que los pacientes no puedan agendar en periodos en que no voy a estar disponible, evitando conflictos y reasignaciones de ultimo momento.

Criterios de aceptacion (Gherkin):
- Dado que la medico selecciona un rango de fechas u horas para bloquear, cuando confirma el bloqueo, entonces esas franjas desaparecen de la disponibilidad publica de inmediato y sin necesidad de accion adicional de la recepcionista.
- Dado que hay un bloqueo activo en un periodo, cuando un paciente intenta agendar en ese periodo, entonces el sistema no le ofrece esa franja ni ninguna subfranja dentro de ella.

Origen: us:US-05 · req:R-07 · evidence:doctora.md · evidence:recepcionista.md

---

## E-04 · Contexto clinico previo a la consulta (segunda fase)

### US-06 · Registro de motivo de consulta al agendar   ·   epica E-04   ·   2 pts

> ADVERTENCIA Segunda fase — fuera del alcance del Sprint 1 (mvp-canvas.md: "no es el cuello de botella principal"). Esta historia no impacta la metrica de inasistencia <= 8 % al mes 3. Se documenta para planificacion futura.

**Como** medico, **quiero** que el paciente registre el motivo de la consulta al agendar, visible para mi antes de que llegue al consultorio, **para** aprovechar al maximo el tiempo de consulta sin averiguar informacion basica al inicio de la cita.

Criterios de aceptacion (Gherkin):
- Dado que el paciente completa el formulario de agendamiento, cuando llega al paso de confirmacion, entonces puede escribir el motivo de la visita (campo opcional, maximo 200 caracteres).
- Dado que la cita existe con motivo registrado, cuando la medico revisa su agenda, entonces puede ver el motivo junto al nombre del paciente antes de que llegue al consultorio.

Origen: us:US-06 · req:R-05 · evidence:doctora.md

---

## Resumen del backlog refinado

| ID    | Titulo corto                              | Epica | Pts | Prioridad | Estado DoR |
|-------|-------------------------------------------|-------|-----|-----------|------------|
| US-01 | Reserva de cita en linea 24/7             | E-01  | 8   | 1         | Lista      |
| US-02 | Prevencion de dobles reservas             | E-01  | 5   | 2         | Lista      |
| US-03 | Recordatorio automatico por WhatsApp      | E-02  | 8   | 3         | Lista      |
| US-04 | Consulta de agenda desde celular          | E-03  | 3   | 4         | Lista      |
| US-05 | Bloqueo de franjas de agenda              | E-03  | 3   | 5         | Lista      |
| US-06 | Registro de motivo de consulta (fase 2)   | E-04  | 2   | 6         | Lista (fuera Sprint 1) |

**Total Sprint 1 candidato (US-01 a US-05):** 27 pts
