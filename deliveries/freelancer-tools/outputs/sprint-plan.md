# Sprint 1 — Establecer foundation de time-tracking centralizado sin pérdida de horas facturables

**Capacidad:** 20 pts · **Comprometido:** 15 pts

---

## Sprint Goal

**"Validar que el freelancer puede registrar y clasificar tiempo de forma consistente en un solo lugar, visualizando exactamente dónde se va su tiempo (facturable vs administrativo), eliminando pérdida de ingresos por olvido de timer."**

---

## Historias comprometidas

| # | Historia | Épica | Pts | Dependencias | Estado |
|---|----------|-------|-----|--------------|--------|
| 1 | US-01 | E-01 | 5   | Ninguna | ✅ Ready |
| 2 | US-03 | E-01 | 5   | US-01 | ✅ Ready |
| 3 | US-04 | E-01 | 5   | US-01 | ✅ Ready |

**Total:** 15 pts (75% de 20 pts; margen de seguridad: 5 pts)

---

## Justificación

### Orden de ejecución:

1. **US-01 (5 pts)** — CRUD clientes/proyectos/tareas/time entries con clasificación facturable/administrativo
2. **US-03 (5 pts)** — Recordatorios inactividad (30 min) y timer prolongado (4 horas)
3. **US-04 (5 pts)** — Reporte semanal horas facturables vs administrativas, filtrable por cliente

### Por qué E-02/E-03 quedan para Sprint 2:

**US-02 tiene bloqueante:** ¿tarifa actual o histórica en precarga de factura? Esta pregunta debe resolverse antes de que US-02 pueda entrar.

US-05, US-06, US-07, US-08 dependen transitivamente de US-02, por lo que quedan bloqueadas también.

---

## Criterios de cierre del Sprint

1. [ ] US-01: CRUD cliente/proyecto/tarea/time entry funcional; clasificación facturable/admin
2. [ ] US-03: notificaciones in-app a 30 min y 4 horas
3. [ ] US-04: reporte semanal filtrable por cliente
4. [ ] Smoke testing: flujo end-to-end sin crashes
5. [ ] US-02 refinada para Sprint 2: open_question resuelta
