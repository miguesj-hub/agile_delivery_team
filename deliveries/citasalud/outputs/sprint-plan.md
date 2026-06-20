# Sprint 1 — Los pacientes reservan cita online 24/7 y la médico controla su agenda desde el celular, eliminando la dependencia del teléfono para ambas partes.

**Capacidad:** 20 pts · **Comprometido:** 19 pts

| Historia | Título                                                           | Pts | Épica | Prioridad |
|----------|------------------------------------------------------------------|-----|-------|-----------|
| US-01    | Reserva de cita en línea 24/7 por el paciente                   | 8   | E-01  | 1         |
| US-02    | Prevención de dobles reservas en la misma franja horaria        | 5   | E-01  | 2         |
| US-04    | Consulta de agenda de la médico desde el celular en tiempo real | 3   | E-03  | 4         |
| US-05    | Bloqueo de franjas de agenda por la médico                      | 3   | E-03  | 5         |

---

## Orden de trabajo sugerido

El sprint se divide en dos fases para respetar las dependencias técnicas.

**Fase 1 — Semana 1 (días 1–5): núcleo del agendamiento**

- US-01 (8 pts) y US-02 (5 pts) arrancan en paralelo desde el día 1.
  US-02 depende de la misma capa de reservas que US-01, por lo que comparten
  el modelo de datos y se desarrollan de forma coordinada, no secuencial.
  Al finalizar la semana 1, el sistema debe permitir reservar una cita online
  y garantizar la exclusión mutua de franjas (doble reserva = 0).

**Fase 2 — Semana 2 (días 6–10): control de agenda para la médico**

- US-04 (3 pts) arranca cuando US-01 está integrado (la vista de agenda lee
  las citas registradas por US-01). Estimado: días 6–8.
- US-05 (3 pts) depende de que US-04 esté terminado (el bloqueo de franjas
  es una extensión de la vista de agenda). Estimado: días 8–10.

Al cierre del sprint se realiza la Review con la médico y la recepcionista
para verificar el Sprint Goal de forma conjunta.

---

## Historias fuera del sprint (y por qué)

- **US-03** (Recordatorio automático por WhatsApp, 8 pts): excluida por dos
  razones que se refuerzan mutuamente. Primera: incluirla elevaría el
  comprometido a 27 pts, superando la capacidad en 7 pts. Segunda: existe un
  supuesto activo no resuelto sobre el costo de la API de WhatsApp Business;
  si el canal resulta inviable por presupuesto, la historia cambia de forma
  sustancial (canal SMS, email u otro). Comprometer al equipo a construirla
  con ese supuesto abierto introduce riesgo de retrabajo significativo. Se
  devuelve a refinamiento: el PO debe confirmar el canal y el presupuesto con
  el dueño de la clínica antes del Sprint 2.

- **US-06** (Motivo de consulta al agendar, 2 pts): fuera del MVP, segunda
  fase. Decisión explícita del Product Owner registrada en la épica E-04
  (`mvp:fuera-de-alcance`). No entra hasta que US-01 a US-05 estén en
  producción y el MVP demuestre adopción.

---

## Riesgos y supuestos activos

1. **Supuesto de canal de notificaciones (US-03 diferida):** el presupuesto
   para la API de WhatsApp Business no está confirmado por el dueño de la
   clínica. Impacto: si no se resuelve antes del Sprint 2, US-03 podría
   requerir rediseño de canal. Acción: el PO gestiona la confirmación antes
   de la Sprint Planning 2.

2. **Disponibilidad de la médico para la Review:** US-04 y US-05 deben ser
   validadas por la Dra. S. en la Review. Si no puede asistir, la aceptación
   se bloquea. Acción: confirmar participación al inicio del sprint.

3. **Integración de confirmación por WhatsApp en US-01:** US-01 incluye envío
   de confirmación por WhatsApp al reservar. Si la API no está disponible en
   Fase 1, se acuerda un fallback (confirmación por email o SMS) para no
   bloquear la entrega del núcleo del agendamiento.
