# Requisitos candidatos — freelancer-tools

> Basado en 3 entrevistas en primera persona: `daniela-freelancer-solo.md`,
> `felipe-freelancer-facturador.md`, `marcela-freelancer-equipo.md`. La
> confianza de cada requisito refleja cuántas entrevistadas lo sustentan de
> forma independiente: **fuerte** (2+), **moderada** (1, con detalle
> suficiente para actuar), **débil** (mención de pasada).

## Funcionales

- **[R-01]** El sistema debe centralizar en un solo lugar la gestión de
  tareas, tiempo, facturación y comunicación con clientes, eliminando la
  coordinación manual entre herramientas separadas.
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-02]** El registro de tiempo trabajado debe conectarse
  automáticamente con la generación de la factura, sin pasos manuales de
  traspaso entre sistemas.
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-03]** El sistema debe generar la factura electrónica válida ante la
  DIAN (con IVA 19% y retención en la fuente calculados automáticamente) a
  partir de un hito o entregable completado, en un solo paso.
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-04]** El sistema debe permitir registrar y conciliar pagos recibidos
  por múltiples canales (Bancolombia, Nequi, Payoneer/Wise), incluyendo
  conversión por TRM cuando aplique.
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-05]** El sistema debe recordar o alertar cuando el registro de tiempo
  de una sesión de trabajo no se ha iniciado o no se ha detenido, para
  reducir horas facturables perdidas.
  - Origen: daniela-freelancer-solo.md, marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-06]** El sistema debe distinguir y reportar, por período, cuántas
  horas fueron facturables y cuántas administrativas.
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-07]** El sistema debe permitir generar cotizaciones/propuestas para
  clientes en menos tiempo que el proceso manual actual (1.5-2 horas por
  propuesta hoy).
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-08]** El sistema debe mostrar en un solo lugar el estado de cobros:
  cuánto se ha facturado, cuánto se ha cobrado, cuánto está por cobrar y
  cuánto está vencido.
  - Origen: felipe-freelancer-facturador.md, marcela-freelancer-equipo.md,
    daniela-freelancer-solo.md
  - Confianza: fuerte

- **[R-09]** El sistema debe alertar cuando una factura vence sin pago y
  facilitar el seguimiento (follow-up) con el cliente correspondiente.
  - Origen: felipe-freelancer-facturador.md, daniela-freelancer-solo.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-10]** El sistema debe permitir registrar el alcance acordado y el
  número de revisiones incluidas por proyecto, y requerir una cotización
  adicional explícita antes de aceptar trabajo fuera de ese alcance.
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-11]** El sistema debe calcular la rentabilidad por cliente/proyecto,
  considerando horas invertidas (propias y, si aplica, del equipo), tarifas
  y costos asociados.
  - Origen: felipe-freelancer-facturador.md, marcela-freelancer-equipo.md,
    daniela-freelancer-solo.md
  - Confianza: fuerte

- **[R-12]** El sistema debe permitir registrar pagos divididos en hitos
  (p. ej. 50%/25%/25%) ligados al cronograma del proyecto.
  - Origen: felipe-freelancer-facturador.md, daniela-freelancer-solo.md
  - Confianza: fuerte

- **[R-13]** El sistema debe permitir exportar información consolidada para
  la declaración de renta / el contador.
  - Origen: daniela-freelancer-solo.md
  - Confianza: moderada

- **[R-14]** *(específico de freelancers con equipo)* El sistema debe
  permitir que varios contratistas registren su propio tiempo por
  cliente/proyecto, con visibilidad consolidada para quien coordina.
  - Origen: marcela-freelancer-equipo.md
  - Confianza: moderada

- **[R-15]** *(específico de freelancers con equipo)* El sistema debe
  permitir calcular el pago a contratistas a partir de las horas que ellos
  mismos reportan, en vez de requerir estimación manual de quien coordina.
  - Origen: marcela-freelancer-equipo.md
  - Confianza: moderada

## No funcionales

- **[R-16]** El sistema debe aplicar correctamente el contexto tributario
  colombiano (IVA 19%, retención en la fuente, régimen simple/actividad
  empresarial) de forma nativa, no como una capa añadida sobre una
  herramienta pensada para otro país.
  - Tipo: no funcional (localización / cumplimiento)
  - Origen: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md
  - Confianza: fuerte

- **[R-17]** El sistema debe permitir exportar todos los datos del usuario
  en un formato utilizable fuera del sistema, para no depender de la
  continuidad de un proveedor.
  - Tipo: no funcional (portabilidad de datos / confianza)
  - Origen: daniela-freelancer-solo.md
  - Confianza: moderada

- **[R-18]** El sistema debe mantener sus propios módulos (tareas, tiempo,
  facturación) sincronizados de forma confiable, sin depender de
  integraciones externas frágiles como las que fallaron con Zapier.
  - Tipo: no funcional (fiabilidad)
  - Origen: daniela-freelancer-solo.md
  - Confianza: débil

- **[R-19]** El modelo de precio del producto debe evitar que el costo total
  se sienta como una carga acumulada de suscripciones. **Ver evidencia
  conflictiva abajo: no hay acuerdo sobre si esto significa pago único o
  suscripción moderada.**
  - Tipo: no funcional (restricción de modelo de negocio)
  - Origen: daniela-freelancer-solo.md, marcela-freelancer-equipo.md
  - Confianza: en conflicto

---

## Evidencia conflictiva

**Modelo de precio (pago único vs. suscripción):**
- Daniela (daniela-freelancer-solo.md) rechaza explícitamente cualquier
  suscripción mensual: *"no me voy a montar en otra suscripción
  mensual [...] el modelo de suscripción mensual lo odio"*. Prefiere un
  pago único de ~USD 150-200 o hasta 800.000-1.000.000 COP una sola vez.
- Marcela (marcela-freelancer-equipo.md) sí acepta un modelo de
  suscripción mensual — *"podría pagar doscientos, doscientos cincuenta mil
  pesos al mes si realmente me resuelve la vida"* — siempre que no cobre por
  usuario/licencia como las herramientas "gringas" que ya usa (Adobe, Alegra,
  Canva ≈ 350.000 COP/mes en conjunto).
- Felipe (felipe-freelancer-facturador.md) no declara una preferencia
  explícita entre pago único y suscripción; ya paga ~60.000 COP/mes por
  Siigo y no lo cuestiona como modelo, solo como relación precio/valor
  ("para lo que yo necesito es demasiado").

No se resuelve esta tensión en este documento — el cliente/equipo del
discovery debe decidir el modelo de precio con esta evidencia a la vista, no
el agente.

---

## Advertencias

- Ningún requisito de facturación/cumplimiento (R-03, R-16) tiene evidencia
  fuera de Colombia — el segmento validado es exclusivamente
  freelancers colombianos.
- R-14 y R-15 dependen enteramente de la persona "freelancer con equipo"
  (Marcela); su generalización a otros freelancers con equipo no está
  validada (n=1).
- No hay evidencia de primera mano de contratistas (los colaboradores de
  Marcela) sobre cómo experimentarían el registro de tiempo desde su propio
  lado — es un punto ciego a considerar antes de construir R-14.
