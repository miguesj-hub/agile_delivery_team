# User Stories — CitaSalud

> Generado el 2026-06-18 · Basado en `personas.md`, `requisitos.md` y `evidence-map.json`

---

## Núcleo del MVP

Las historias US-01 a US-05 cubren el valor mínimo entregable: eliminan los dolores más frecuentes y compartidos entre las tres personas primarias (dobles reservas, barrera telefónica e inasistencias sin aviso).

---

- **[US-01]** Como paciente, quiero reservar una cita en línea en cualquier momento del día para no tener que llamar durante mi horario de almuerzo ni acumular intentos fallidos.
  - Criterios de aceptación:
    - Dado que el paciente accede al sistema fuera del horario de atención, cuando elige médico, fecha y hora disponibles y confirma, entonces la cita queda registrada y recibe confirmación por WhatsApp.
    - Dado que el paciente intenta seleccionar una franja ya ocupada, cuando intenta confirmarla, entonces el sistema la muestra como no disponible y lo invita a elegir otra.
  - Fuente: `paciente.md`, `recepcionista.md`

---

- **[US-02]** Como recepcionista, quiero que el sistema impida asignar la misma franja horaria a dos pacientes para que dejen de ocurrir las dobles reservas (~2 por semana).
  - Criterios de aceptación:
    - Dado que dos solicitudes de reserva coinciden en la misma franja, cuando la primera se confirma, entonces la segunda recibe la franja como bloqueada antes de finalizar.
    - Dado que existe una cita activa en una franja, cuando se intenta agregar otra en el mismo horario, entonces el sistema rechaza la operación con un mensaje de conflicto.
  - Fuente: `recepcionista.md`

---

- **[US-03]** Como paciente, quiero recibir un recordatorio automático por WhatsApp antes de mi cita para no olvidarla ni generar una inasistencia sin previo aviso.
  - Criterios de aceptación:
    - Dado que un paciente tiene una cita agendada, cuando faltan 24 horas para el turno, entonces el sistema envía un mensaje de WhatsApp con fecha, hora y nombre de la médico.
    - Dado que el paciente recibe el recordatorio, cuando responde que cancela, entonces la cita queda anulada y la franja vuelve a estar disponible en la agenda.
  - Fuente: `paciente.md`, `recepcionista.md`, `doctora.md`

---

- **[US-04]** Como médico, quiero consultar mi agenda completa desde el celular para no tener que llamar a recepción para saber cuántos pacientes tengo al día siguiente.
  - Criterios de aceptación:
    - Dado que la médico accede desde un dispositivo móvil, cuando abre su vista de agenda, entonces ve la lista de citas del día con nombre del paciente, hora y estado (confirmada / cancelada).
    - Dado que se cancela o agrega una cita, cuando ocurre el cambio, entonces la agenda de la médico se actualiza sin necesidad de recargar manualmente.
  - Fuente: `doctora.md`

---

- **[US-05]** Como médico, quiero bloquear franjas de mi agenda (congresos, vacaciones, imprevistos) para que los pacientes no puedan agendar en esos períodos.
  - Criterios de aceptación:
    - Dado que la médico selecciona un rango de fechas u horas para bloquear, cuando confirma, entonces esas franjas desaparecen de la disponibilidad pública de inmediato.
    - Dado que hay un bloqueo activo, cuando un paciente intenta agendar en ese período, entonces el sistema no le ofrece esa franja ni ninguna subfranja dentro de ella.
  - Fuente: `doctora.md`, `recepcionista.md`

---

## Segunda fase (fuera del núcleo del MVP)

- **[US-06]** Como médico, quiero que el paciente registre el motivo de la consulta al agendar para aprovechar al máximo el tiempo de consulta sin averiguar información básica al inicio.
  - Criterios de aceptación:
    - Dado que el paciente completa el formulario de agendamiento, cuando llega al paso de confirmación, entonces puede escribir el motivo de la visita (campo opcional, máx. 200 caracteres).
    - Dado que la cita existe, cuando la médico revisa su agenda, entonces puede ver el motivo junto al nombre del paciente antes de que llegue al consultorio.
  - Fuente: `doctora.md`
