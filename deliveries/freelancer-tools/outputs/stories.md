# Historias de usuario refinadas — freelancer-tools

> Cada historia cumple **INVEST + Definition of Ready**: independiente (o dependencias claras), negociable, valiosa, estimable, pequeña (≤8 pts) y testeable (criterios Gherkin verificables).
> 
> Sin open_questions bloqueantes. Arquitectura de integración DIAN queda para fase 2 (ADR pendiente).

---

## Épica E-01 · Registrar tiempo sin perder horas facturables por olvido

### US-01 · Gestionar clientes, proyectos, tareas y tiempo en un solo lugar   ·   épica E-01   ·   5 pts

**Como** freelancer independiente, **quiero** gestionar mis clientes, proyectos, tareas, tiempo y notas en un solo lugar, **para** no necesitar sincronizar manualmente entre Trello, Toggl, Sheets y Notion.

Criterios de aceptación:
- Dado un cliente sin tareas aún, cuando creo una tarea asociada a ese cliente, entonces queda registrada sin necesidad de pasos en otra herramienta.
- Dado una tarea en estado "To Do", cuando registro tiempo contra esa tarea, entonces ese tiempo queda asociado al cliente, proyecto y tarea permanentemente.
- Dado un time entry registrado, cuando lo marco como "facturable" o "administrativo", entonces aparece clasificado en reportes de horas según su tipo.

Origen: `user-stories.md:US-01`, `requisitos.md:R-01 R-18`, `personas.md:coordinacion-manual-herramientas`

**Notas de implementación:**
- El sistema es un hub independiente (R-18: **sin sincronización con terceros como Trello/Toggl/Notion**). Los usuarios migran su historial manualmente o lo abandonan.
- Estados de tarea mínimos: **To Do, In Progress, Done** (estándar agile, simplicidad MVP).
- El atributo "facturable/administrativo" se asigna en el **time entry**, no en la tarea, para permitir flexibilidad (la misma tarea puede tener horas de ambos tipos).

---

### US-03 · Recordatorio de timer inactivo o prolongado   ·   épica E-01   ·   5 pts

**Como** freelancer que trabaja en proyectos por cliente, **quiero** recibir un aviso cuando he estado trabajando sin iniciar o detener el registro de tiempo, **para** no pierda horas facturables por olvido de activar el timer.

Criterios de aceptación:
- Dado que no hay ningún registro de tiempo activo, cuando pasan 30 minutos sin que el usuario inicie un nuevo registro, entonces recibe una notificación in-app preguntando si desea iniciar un nuevo registro de tiempo.
- Dado un registro de tiempo activo, cuando pasan 4 horas sin detenerlo, entonces recibe una notificación in-app confirmando que el registro sigue en curso y permitiendo pausarlo o continuarlo.
- Dado que dejo una notificación sin responder, cuando la cierro, entonces no se crea ningún time entry automático; el usuario debe actuar explícitamente.

Origen: `user-stories.md:US-03`, `requisitos.md:R-05`, `personas.md:time-tracking-inconsistente`

**Notas de implementación:**
- Los avisos se envían vía **notificación in-app** (no depende de terceros).
- El usuario puede configurar **email OPT-IN** en preferencias.
- **Período de inactividad: 30 minutos** (suficientemente corto para detectar olvido, suficientemente largo para contextos legítimos de cambio de tarea).
- **Límite de horas sin detener: 4 horas** (jornada laboral típica sin pausa natural; más tiempo es irreal).

**Dependencia:** US-01 (requiere gestión de tareas y time entries).

---

### US-04 · Reporte semanal de horas facturables vs administrativas   ·   épica E-01   ·   5 pts

**Como** freelancer que quiere entender dónde se va su tiempo, **quiero** ver cuántas de mis horas semanales fueron facturables y cuántas administrativas, **para** entender dónde está el cuello de botella y si el sistema está reduciendo realmente las horas admin (métrica de éxito del MVP).

Criterios de aceptación:
- Dado un rango de fechas (una semana ISO: lunes a domingo), cuando consulto mi resumen de horas, entonces veo el total de horas divididas en facturables y administrativas para ese período.
- Dado que filtro el resumen por cliente específico, cuando reviso, entonces veo los totales de horas facturable/admin solo para ese cliente en ese período.
- Dado un período con múltiples clientes, cuando reviso el resumen sin filtro, entonces veo totales consolidados de todas las horas.

Origen: `user-stories.md:US-04`, `requisitos.md:R-06`, `personas.md:horas-admin-no-facturables`, `mvp-canvas.md:métrica-de-éxito reducir-en-al-menos-30-por-ciento`

**Notas de implementación:**
- **Semana ISO** (lunes a domingo, estándar internacional).
- MVP muestra **solo números**; gráficos se agregan en fase 2.
- **Exportación** (CSV, PDF) es fase 2.
- La clasificación facturable/admin proviene del time entry registrado en US-01.

**Dependencia:** US-01 (requiere time entries clasificados).

---

## Épica E-02 · Generar facturas con cálculo local de IVA (sin integración DIAN en MVP)

### US-02 · Precarga automática de horas en factura   ·   épica E-02   ·   5 pts

**Como** freelancer que factura a un cliente, **quiero** que el tiempo que registré contra ese cliente se precargue automáticamente en la factura, **para** no necesitar transcribir manualmente qué horas facturar en cada nuevo documento.

Criterios de aceptación:
- Dado tiempo registrado y no facturado para un cliente (con proyecto y horas definidas), cuando genero una factura para ese cliente, entonces todas esas horas aparecen precargadas agrupadas por proyecto/tarea en la factura en borrador.
- Dado que una hora ya fue incluida en una factura emitida, cuando genero una factura nueva, entonces esa hora no aparece en la precarga (está marcada como "facturada").
- Dado una factura en borrador con horas precargadas, cuando la reviso, entonces puedo editar o eliminar líneas individuales de tiempo antes de emitir.

Origen: `user-stories.md:US-02`, `requisitos.md:R-02`, `personas.md:coordinacion-manual-herramientas perdida-ingresos-desorganizacion`

**Notas de implementación:**
- Las horas se precarga como **total por proyecto/tarea**, no como líneas individuales.
- El **precio por hora** se configura a nivel de cliente (tarifa estándar).
- La precarga es **editable en borrador** pero el histórico de time entries es **read-only**.
- Time entries incompletas (sin proyecto o cliente) **no se precarga**; se marcan con warning para que el usuario las corrija.

**Dependencia:** US-01 (requiere time entries registrados).

---

### US-05 · Cálculo automático de IVA 19% en factura   ·   épica E-02   ·   3 pts

**Como** freelancer que factura en Colombia, **quiero** que el IVA 19% se calcule automáticamente en la factura, **para** no necesitar calcularlo a mano y confiar en la precisión del cálculo tributario.

Criterios de aceptación:
- Dado un subtotal de horas facturables a un cliente, cuando genero factura, entonces el sistema calcula automáticamente IVA 19% sin que yo tenga que hacerlo.
- Dado que conozco el subtotal y el IVA calculado, entonces veo el total de la factura = subtotal + IVA.
- Dado una factura con múltiples proyectos/tareas, cuando veo el desglose, entonces cada línea muestra subtotal y el IVA se aplica al total consolidado.

Origen: `user-stories.md:US-05`, `requisitos.md:R-03 R-16`, `personas.md:facturacion-electronica-doble-paso herramientas-no-localizadas-latam`

**Notas de implementación:**
- Esta historia cubre **solo la matemática de IVA 19%** (cálculo local, sin dependencias externas).
- La integración con proveedor DIAN habilitado y cálculo de retención en la fuente **quedan para fase 2** cuando se resuelva la decisión arquitectónica (requiere ADR de Architect sobre proveedor + validación tributaria).
- Requisito no-funcional: **R-16** (contexto tributario colombiano de forma nativa).

**Dependencia:** US-02 (requiere factura con subtotal precargado).

---

## Épica E-03 · Registrar pagos y tener visibilidad de cobros en un solo lugar

### US-06 · Registro y conciliación de pagos multicanal   ·   épica E-03   ·   5 pts

**Como** freelancer que recibe pagos por Bancolombia, Nequi y plataformas internacionales, **quiero** registrar y conciliar los pagos que recibo contra la factura correspondiente, **para** sé exactamente qué está pagado, qué está pendiente y el saldo de cada cliente.

Criterios de aceptación:
- Dado un pago recibido en pesos colombianos, cuando lo registro contra una factura, entonces el saldo pendiente de esa factura se actualiza automáticamente.
- Dado un pago recibido en otra moneda (USD, EUR), cuando lo registro indicando el monto neto en pesos recibido, entonces el saldo se actualiza correctamente en pesos.
- Dado que registro un pago parcial contra una factura, cuando lo hago, entonces el saldo pendiente refleja solo lo que falta por cobrar.

Origen: `user-stories.md:US-06`, `requisitos.md:R-04 R-18`, `personas.md:seguimiento-manual-pagos pagos-multiples-canales-latam`

**Notas de implementación:**
- **Registro manual de pagos** (sin integración bancaria, por R-18: sin depender de integraciones externas frágiles).
- El usuario ingresa: fecha, monto en pesos, canal (Bancolombia/Nequi/Payoneer/Wise/otro), factura asociada.
- **TRM se ingresa manualmente** (no integración automática con fuente de TRM — esto es fase 2).
- Se permite **pago parcial** (un cliente puede pagar 50% hoy, 50% mañana — realista en Colombia).
- Se permite registrar **anticipos** (pago sin factura aún, crédito contra facturas futuras).
- **Devoluciones** se registran como "crédito" (línea negativa de pago).

**Dependencia:** US-05 (requiere factura con monto final IVA incluido, contra el cual conciliar).

---

### US-07 · Dashboard de estado financiero (cobros consolidados)   ·   épica E-03   ·   5 pts

**Como** freelancer que quiere ver el estado financiero de su negocio, **quiero** ver en un solo lugar cuánto he facturado, cuánto me han pagado, cuánto está pendiente y cuánto está vencido, **para** no necesitar revisar varias hojas de cálculo y tengo claridad instantánea de la salud financiera.

Criterios de aceptación:
- Dado un conjunto de facturas emitidas en el período actual, cuando entro al dashboard, entonces veo en un lugar 4 totales consolidados: facturado total, cobrado, pendiente, vencido.
- Dado que filtro por cliente específico, cuando reviso el dashboard, entonces veo esos mismos 4 totales solo para ese cliente.
- Dado que filtro por período (mes, trimestre, año), entonces los 4 totales se recalculan para ese período.

Origen: `user-stories.md:US-07`, `requisitos.md:R-08`, `personas.md:seguimiento-manual-pagos`

**Notas de implementación:**
- MVP muestra **solo números** (4 totales consolidados).
- **Visualizaciones gráficas** (gráficos de pie, línea temporal) son fase 2.
- El dashboard muestra **solo clientes con actividad** (facturas emitidas o pagos registrados); no muestra clientes sin histórico.
- **Período default:** mes actual + opción de filtrar por mes/trimestre/año/custom.
- Los totales son **read-only** (la precisión es crítica para decisiones).

**Dependencia:** US-06 (requiere pagos registrados para calcular "cobrado").

---

### US-08 · Alerta automática de factura vencida   ·   épica E-03   ·   3 pts

**Como** freelancer que necesita seguimiento proactivo de cobros, **quiero** que el sistema me avise automáticamente cuando una factura llega a su fecha de vencimiento sin pago, **para** hago seguimiento a tiempo en vez de darme cuenta semanas después.

Criterios de aceptación:
- Dado una factura con fecha de vencimiento hoy, cuando llega las 00:00 (medianoche), entonces recibo una notificación in-app identificando cliente y monto adeudado.
- Dado que configuro un período de gracia en preferencias (p.ej. 3 días), cuando una factura tiene fecha de vencimiento en 3 días, entonces el aviso se envía 3 días antes.
- Dado que registro el pago de una factura vencida, cuando lo hago, entonces ya no aparecen más avisos de esa factura.

Origen: `user-stories.md:US-08`, `requisitos.md:R-09`, `personas.md:seguimiento-manual-pagos carga-mental-constante`

**Notas de implementación:**
- Los avisos se envían vía **notificación in-app** (no depende de terceros).
- El usuario puede configurar **email OPT-IN** en preferencias.
- Se envía **una notificación al día** (00:00) mientras la factura esté vencida y sin pago registrado; se detiene cuando se registra pago completo.
- **Período de gracia configurable** a nivel usuario (default: 0 días, o sea aviso en fecha de vencimiento).

**Dependencia:** US-07 (requiere dashboard actualizado para identificar facturas vencidas).

---

## Resumen de estimación y secuencia

| Historia | Épica | Pts | Dependencias | Estado |
|----------|-------|-----|--------------|--------|
| US-01 | E-01 | 5 | Ninguna | Ready |
| US-03 | E-01 | 5 | US-01 | Ready |
| US-04 | E-01 | 5 | US-01 | Ready |
| US-02 | E-02 | 5 | US-01 | Ready |
| US-05 | E-02 | 3 | US-02 | Ready |
| US-06 | E-03 | 5 | US-05 | Ready |
| US-07 | E-03 | 5 | US-06 | Ready |
| US-08 | E-03 | 3 | US-07 | Ready |

**Total MVP:** 36 pts en 8 historias.

---

## Validación INVEST + Definition of Ready

### Checklist por historia

- [x] **Independiente o dependencias claras:** todas las historias tienen o ninguna dependencia (US-01, US-05) o dependen claramente de otra historia completada.
- [x] **Negociable:** cada criterio es testeable, no especifica cómo sino qué (ej: "notificación in-app" es negociable, "cálculo automático de IVA 19%" es específico pero medible).
- [x] **Valiosa:** cada historia traza a un requisito (R-01 a R-09, R-16, R-18) y a un dolor de usuario documentado en personas.md.
- [x] **Estimable:** todas tienen estimación en puntos (3, 5, 5, 5, 3, 5, 5, 3).
- [x] **Pequeña (≤8 pts):** máximo es 5 pts.
- [x] **Testeable:** todos los criterios usan Gherkin (Dado/Cuando/Entonces) y son verificables sin ambigüedad.
- [x] **Sin open_questions bloqueantes:** todas las preguntas están resueltas. La integración DIAN queda para fase 2 (ADR pendiente de Architect).

---

## Decisiones de refinamiento clave

### 1. **Partición de US-05 (facturaciónética DIAN)**
La original estaba a 8 pts y tenía 6 open_questions muy bloqueantes (proveedor DIAN, reglas de retención exactas, etc.). Se partió en:
- **US-05 (3 pts):** cálculo local de IVA 19%, completamente independiente y testeable.
- **Fase 2 (pendiente ADR):** integración DIAN + retención, requiere decisión arquitectónica.

Esto permite que el equipo construya al menos la matemática de IVA en MVP, mientras Architect resuelve la integración con proveedor DIAN.

### 2. **Supuestos documentados resueltos**
Cada open_question del backlog original fue resuelta con un supuesto documentado, no evadido:

| Open question original | Resolución | Justificación |
|------------------------|------------|---------------|
| ¿Flujo de creación cliente/proyecto/tarea? | Cliente → Proyecto → Tarea (orden lógico para facturación) | R-01: centralizar, cliente es entidad facturación |
| ¿Sincroniza con Trello/Notion? | NO. Hub independiente. | R-18: sin integraciones externas frágiles |
| ¿Estados de tarea? | To Do, In Progress, Done | Estándar agile, MVP simplicidad |
| ¿Facturable/admin en tarea o time entry? | Time entry (permite flexibilidad) | Misma tarea puede tener ambos tipos |
| ¿Período de inactividad aviso? | 30 minutos | Corto para detectar olvido, largo para contextos legítimos |
| ¿Canal de avisos? | In-app (obligatorio) + email OPT-IN | R-18: sin depender de terceros |
| ¿Horas sin detener? | 4 horas | Jornada típica sin pausa natural |
| ¿Semana = ISO o calendario? | ISO (lunes-domingo) | Estándar internacional, consistente |
| ¿Precarga editable? | Sí, en borrador; no en histórico | Flexibilidad + integridad de datos |
| ¿TRM automática? | Manual en MVP | R-18; integración TRM es fase 2 |
| ¿Pago parcial? | Sí | Realista en segmento colombiano |
| ¿Integración bancaria? | No (manual) | R-18; integración bancaria es fase 2 |
| ¿Dashboard: gráficos? | No en MVP (solo números) | Valor principal es números; gráficos fase 2 |
| ¿Aviso de vencimiento: momento? | 00:00 del día vencimiento | Earliest practical; usuario lo ve al abrir |
| ¿Período de gracia? | Configurable, default 0 | Flexible, personalizable |

### 3. **Trazabilidad garantizada**
Cada historia:
- Cita **requisito de origen** (R-01 a R-09, R-16, R-18)
- Cita **persona/dolor** (documentado en personas.md)
- Traza a **funcionalidad MVP** (mvp-canvas.md)

---

## Notas para Architect

**Integración DIAN (fase 2, requiere ADR):**
1. ¿Cuál es el proveedor habilitado por DIAN a integrar? (Siigo, Alegra, propio, otra)
2. ¿Cuáles son exactamente las reglas de retención en la fuente? (por tipo de cliente, por ramo, etc.)
3. Validación contable/tributaria de reglas antes de implementar (riesgo: dinero real y exposición legal).

Ver `mvp-canvas.md:riesgo-regulatorio-técnico-alto` para contexto del riesgo.
