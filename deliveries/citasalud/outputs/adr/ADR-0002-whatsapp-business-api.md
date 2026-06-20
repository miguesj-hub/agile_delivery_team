# ADR-0002 · WhatsApp Business API como canal de notificaciones

**Estado:** aceptado
**Fecha:** 2026-06-20

---

## Contexto y fuerza

El inbox provee evidencia directa y de primera mano de que **WhatsApp es el canal preferido del paciente**:

- `personas.md` / `paciente.md`: J. (paciente, 58 años) "prefiere WhatsApp porque lo revisa todo el día". Los recordatorios por teléfono son "inconsistentes"; J. los prefiere por WhatsApp.
- `mvp-canvas.md` — Funcionalidades mínimas F1 y F3: "confirmación por WhatsApp" y "recordatorio automático por WhatsApp".
- `requisitos.md` req:R-03: "El sistema debe enviar recordatorios automáticos a los pacientes (por WhatsApp o canal equivalente)".
- `evidence-map.json` pains: `recordatorio-inconsistente` (paciente.md), `recordatorios-manuales` (recepcionista.md).

La fuerza directa es **req:R-03 + personas.md (J.)**: sin un canal confiable de notificación automática, la épica E-02 (reducir no-shows) no puede cumplir su outcome.

**Supuesto de riesgo explícito registrado en el inbox:** `mvp-canvas.md` riesgo 3: "El costo de la API de WhatsApp Business es asumible por una clínica pequeña" — marcado como supuesto, no como hecho confirmado.

---

## Decisión

Se utiliza **WhatsApp Business API a través de un proveedor habilitado por Meta** (por ejemplo: Twilio, MessageBird, u otro proveedor equivalente aprobado por Meta) para el envío de:

1. Mensaje de confirmación de reserva (US-01).
2. Recordatorio automático 24 h antes de la cita (US-03).
3. Confirmación de cancelación vía respuesta al recordatorio (US-03).

La integración se encapsula dentro del **Módulo Notificaciones** (ver ADR-0001), de modo que si en el futuro se cambia de proveedor, el cambio queda contenido en ese módulo.

El proveedor concreto (Twilio vs Meta directa vs otro) queda como **OQ-02** en `architecture.md` porque el costo no está confirmado y la aprobación de cuenta puede tener plazos variables. Esta decisión no bloquea el desarrollo del resto del MVP.

---

## Alternativas consideradas

- **SMS (Twilio SMS u operador local)** — Mayor penetración en perfiles menos digitales, pero menor costo relativo por mensaje. Sin embargo, la evidencia del inbox señala explícitamente WhatsApp como canal preferido (J., `paciente.md`); usar SMS sin evidencia que lo respalde violaría la regla de cero invención. `Descartado` como canal primario; podría ser canal de fallback en fase 2.
- **Correo electrónico** — No hay ninguna mención en el inbox de que los pacientes usen email para interactuar con la clínica. Introducirlo sería invención sin respaldo. `Descartado`.
- **Notificación push (app nativa)** — Requeriría que los pacientes instalaran una aplicación, lo que agrega fricción y un vector de adopción no respaldado por evidencia. `mvp-canvas.md` señala como riesgo la adopción digital; una app aumentaría ese riesgo. `Descartado`.
- **Llamada telefónica automática (IVR)** — Es el canal actual manual que genera el dolor `recordatorios-manuales`. Automatizarlo no resuelve la inconsistencia percibida por J. ni es el canal preferido declarado. `Descartado`.

---

## Consecuencias

**Lo que se gana:**
- Se atiende el dolor documentado de J. y el de M. con el canal que ambos ya usan.
- La API permite mensajes bidireccionales: el paciente puede cancelar respondiendo al recordatorio sin llamar (US-03, criterio de aceptación 2).
- El Módulo Notificaciones es intercambiable si se cambia de proveedor.

**El costo que se acepta:**
- **Riesgo de costo no confirmado**: la API de WhatsApp Business tiene costo por conversación (tarifa Meta). Si el dueño de la clínica rechaza el costo al revisar el presupuesto, se debe renegociar el canal antes de iniciar el desarrollo de US-03. Este riesgo está explícitamente registrado en `mvp-canvas.md` riesgo 3 y en OQ-02 de `architecture.md`.
- Dependencia de un servicio externo de terceros: si el proveedor tiene una interrupción, los recordatorios no se envían. Mitigación mínima: reintentos automáticos en el Módulo Notificaciones; canal de fallback (SMS) queda fuera del MVP.
- El proceso de aprobación de cuenta en Meta puede tomar días o semanas; debe iniciarse en paralelo con el desarrollo, no al final.
