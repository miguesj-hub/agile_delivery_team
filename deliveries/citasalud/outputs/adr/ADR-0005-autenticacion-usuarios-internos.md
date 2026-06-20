# ADR-0005 · Autenticación por usuario y contraseña para acceso interno (médico y recepcionista)

**Estado:** aceptado
**Fecha:** 2026-06-20

---

## Contexto y fuerza

El MVP incluye dos vistas diferenciadas según el rol del usuario:

- **Vista pública (paciente):** acceso sin autenticación; el paciente solo proporciona nombre y número de WhatsApp al reservar (US-01). No hay cuenta de usuario para el paciente en el MVP.
- **Vista interna (médico / recepcionista):** acceso a datos sensibles de la agenda, bloqueos y estado de citas (US-04, US-05). Esta vista debe estar protegida.

`requisitos.md` req:R-04: "La médico debe poder consultar y gestionar su agenda **desde cualquier dispositivo**." Esto implica que la autenticación debe funcionar desde el celular sin fricción adicional.

`personas.md` / `doctora.md`: Dra. S. accede desde fuera de la clínica y necesita un mecanismo de acceso que no dependa de estar en la red local.

No existe un requisito de autenticación explícito en el inbox (no hay `R-auth` ni mención directa de mecanismo de login). Sin embargo, proteger la vista interna es una consecuencia lógica directa de req:R-04 y req:R-07: si cualquiera puede acceder a la agenda o crear bloqueos, la integridad del sistema queda comprometida.

La decisión concreta del proveedor de identidad o el flujo exacto de autenticación se registró como `OQ-03` en `architecture.md`. Este ADR resuelve el mecanismo básico para el MVP sin sobrediseñar.

---

## Decisión

Se implementa **autenticación con usuario y contraseña** para la vista interna, con las siguientes características mínimas:

- Dos usuarios iniciales configurados en el sistema: uno para la Dra. S. y uno para M. (recepcionista).
- El backend valida las credenciales y emite un **token JWT** con expiración (por ejemplo: 8 horas), que el frontend almacena en memoria o en una cookie HTTP-only segura.
- Las rutas del API correspondientes a la vista interna (`/agenda`, `/bloqueos`, `/citas`) requieren el token válido.
- La gestión de usuarios (crear, cambiar contraseña) queda fuera del MVP; se configura una vez al desplegar.

---

## Alternativas consideradas

- **Magic Link (enlace por correo o WhatsApp)** — Elimina contraseñas, pero introduce dependencia del canal para cada sesión. Útil para usuarios no técnicos, pero agrega complejidad en el flujo de autenticación y depende de la disponibilidad del canal WhatsApp (ya es una dependencia externa, no conviene que sea también el mecanismo de acceso de emergencia). `Descartado` para el MVP; elegible como mejora de UX en fase 2.
- **OAuth / SSO (Google, Microsoft)** — Apropiado si la clínica ya usa una suite de productividad con identidades gestionadas. No hay evidencia en el inbox de que exista esa infraestructura. Introduce dependencia de un proveedor externo de identidad sin justificación documentada. `Descartado`.
- **Sin autenticación (acceso por URL secreta)** — Opción mínima que algunos MVPs usan para un número muy reducido de usuarios internos. Inaceptable aquí porque la agenda contiene datos de pacientes (nombre, hora de cita) que tienen implicaciones de privacidad. `Descartado`.
- **Sesión basada en cookie de servidor (sin JWT)** — Válido técnicamente; el JWT se elige porque facilita la autenticación stateless desde el celular sin configuración especial del servidor. La diferencia en complejidad es marginal para este caso.

---

## Consecuencias

**Lo que se gana:**
- Las rutas internas del API quedan protegidas desde el día 1 del MVP.
- No se introduce ningún proveedor externo de identidad; el sistema es autosuficiente para los dos usuarios internos del MVP.
- El token JWT puede extenderse en fase 2 para incluir roles adicionales si se agregan más médicos o administradores.

**El costo que se acepta:**
- La gestión de contraseñas (recuperación, cambio) queda fuera del MVP; debe acordarse un procedimiento manual para el lanzamiento inicial (por ejemplo: el equipo técnico resetea la contraseña si se pierde).
- La Dra. S. y M. deben recordar sus credenciales; si olvidan la contraseña no hay flujo de recuperación automático en el MVP. Este riesgo es aceptable para dos usuarios en una clínica pequeña con soporte técnico cercano.
- Si en el futuro se agrega un panel para el dueño de la clínica (fuera del MVP según `mvp-canvas.md`), el modelo de roles del JWT deberá extenderse.
