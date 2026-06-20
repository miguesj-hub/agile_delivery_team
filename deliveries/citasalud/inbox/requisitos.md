# Requisitos Candidatos — CitaSalud

> Generado el 2026-06-18 · Fuentes: `recepcionista.md`, `doctora.md`, `paciente.md`

---

## Funcionales

- **[R-01]** El sistema debe permitir que los pacientes reserven citas en línea, fuera del horario de atención telefónica de la clínica.
  - Tipo: funcional
  - Origen: `recepcionista.md` · Recepcionista / `paciente.md` · Paciente

- **[R-02]** El sistema debe validar la disponibilidad en tiempo real e impedir que se asigne una misma franja horaria a más de un paciente.
  - Tipo: funcional
  - Origen: `recepcionista.md` · Recepcionista

- **[R-03]** El sistema debe enviar recordatorios automáticos a los pacientes (por WhatsApp o canal equivalente) antes de la cita, sin intervención manual de la recepcionista.
  - Tipo: funcional
  - Origen: `recepcionista.md` · Recepcionista / `paciente.md` · Paciente

- **[R-04]** La médico debe poder consultar y gestionar su agenda desde cualquier dispositivo, sin necesidad de llamar a la clínica.
  - Tipo: funcional
  - Origen: `doctora.md` · Médico

- **[R-05]** Al momento de agendar, el paciente debe poder registrar el motivo de la consulta, visible para la médico antes de la cita.
  - Tipo: funcional
  - Origen: `doctora.md` · Médico

- **[R-06]** El sistema debe exponer la disponibilidad de turnos de forma que los pacientes puedan consultarla sin necesidad de llamar por teléfono.
  - Tipo: funcional
  - Origen: `recepcionista.md` · Recepcionista / `paciente.md` · Paciente

- **[R-07]** La médico debe poder bloquear franjas de su agenda (congresos, vacaciones, imprevistos) y dichos bloqueos deben reflejarse de inmediato para todos.
  - Tipo: funcional
  - Origen: `doctora.md` · Médico / `recepcionista.md` · Recepcionista

---

## No funcionales

- **[R-08]** El sistema debe ser accesible desde dispositivos móviles (celular), tanto para la médico como para los pacientes.
  - Tipo: no funcional
  - Origen: `doctora.md` · Médico / `paciente.md` · Paciente

- **[R-09]** El sistema debe estar disponible fuera del horario de atención de la clínica (mínimo: las 24 horas del día para la consulta de disponibilidad y el agendamiento).
  - Tipo: no funcional
  - Origen: `paciente.md` · Paciente
