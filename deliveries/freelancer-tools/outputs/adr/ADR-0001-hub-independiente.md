# ADR-0001: Hub independiente — no sincronización con terceros

**Fecha:** 2026-07-05  
**Estado:** Aceptado  
**Contexto de decisión:** R-18, personas (Daniela, Felipe, Marcela)

---

## Problema

El usuario maneja múltiples herramientas desconectadas (Trello, Toggl, Sheets, Notion, Siigo/Alegra) y debe sincronizar manualmente entre ellas. Daniela abandonó herramientas por fallas en integraciones (Zapier dejó de sincronizar tras 2 semanas). El requisito **R-18** pide: *"El sistema debe mantener sus propios módulos (tareas, tiempo, facturación) sincronizados de forma confiable, sin depender de integraciones externas frágiles"*.

**Fuerzas:**
- **R-18 (confiabilidad):** Integraciones externas son frágiles y requieren mantenimiento continuo.
- **Personas (Daniela):** Experiencia negativa explícita con Zapier; desconfianza en single-vendor.
- **Simplicity (YAGNI):** Cada integración (Trello API, Toggl API, Sheets API) agrega complejidad y puntos de falla.
- **Fase 2:** Importación asistida es más manejable que sincronización bidireccional en MVP.

---

## Alternativas consideradas

1. **Hub con sincronización bidireccional** (Trello ↔ Sistema)
   - ✅ Usuarios mantienen workflow en herramientas favoritas
   - ❌ Archi compleja (webhooks, polling, manejo de conflictos)
   - ❌ Riesgo: cuando Trello API cambia, sistema rompe
   - ❌ Testing multiplica por N (una hipótesis para cada tercero)

2. **Hub independiente + importación manual** (actual)
   - ✅ Confiabilidad MVP máxima
   - ✅ Migración bajo control del usuario
   - ✅ Código simple, puntos de falla mínimos
   - ❌ Fricción inicial: usuario debe migrar historial o abandonarlo

3. **Hub + integraciones solo lectura** (read-only de Toggl/Sheets)
   - ✅ Menos riesgo que bidireccional
   - ❌ Aún requiere mantener 3-5 APIs de terceros
   - ❌ Sigue siendo frágil (Toggl API puede deprecarse)

4. **Multi-tenancy con Zapier/Make.com**
   - ✅ Usuarios configuran sus propias integraciones
   - ❌ Delegamos el riesgo pero no lo eliminamos
   - ❌ UX confusa (usuario debe entender Zapier)

---

## Decisión

**Implementar hub independiente, sin sincronización automática con terceros en MVP.**

El usuario migra su historial manualmente o lo abandona (realistic expectation: nuevos usuarios no tienen histórico largo). Fase 2 puede ofrecer:
- Importación asistida (CSV/JSON)
- Scripts de migración (Toggl → TimeEntry, Trello → Task)
- Pero NO sincronización bidireccional.

---

## Consecuencias

### ✅ Positivas
- **Confiabilidad:** Sin dependencias de terceros frágiles.
- **Performance:** Queries rápidas (DB local, sin latencia de APIs externas).
- **Seguridad:** No exponemos credenciales del usuario a terceros (excepción: OAuth Google).
- **Simplicity:** API simple, testing determinístico.
- **Ownership:** Somos responsables de nuestro dato, no un tercero.

### ⚠️ Negativas
- **Fricción onboarding:** Usuario debe ingresar tareas/tiempo manualmente o abandonar histórico.
  - **Mitigación:** Fase 2 — importación asistida + scripts.
- **Percepción:** Puede parecer que el sistema no es "flexible" (realidad: es más confiable).
  - **Mitigación:** Comunicar explícitamente durante onboarding.

### 📊 Riesgo
**Bajo.** La evidencia (Daniela, Zapier failure) valida la decisión. Usuarios buscan reliability > integración.

---

## Trazabilidad

- **R-18:** Módulos sincronizados sin depender de integraciones frágiles.
- **Personas:** Daniela (desconfianza en Zapier, preferencia por pago único sin "suscripciones" a integraciones).
- **Historias:** US-01 (hub único para clientes, proyectos, tareas, tiempo).

---

## Notas de implementación

- **Onboarding:** Documentar explícitamente que el sistema es "su único lugar de verdad".
- **Export:** Facilitar export de datos (CSV, JSON) para evitar lock-in (R-17, fase 2).
- **API pública:** Exponer API REST para que usuarios avanzados construyan sus propias integraciones (si lo desean).
