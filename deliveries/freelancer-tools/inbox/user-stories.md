# User stories — freelancer-tools

> Alcance: MVP centrado en el **núcleo de valor compartido** por las 3
> personas — cerrar el círculo horas trabajadas → factura → cobro — para el
> segmento de freelancer **solo** (Daniela, Felipe). Las funcionalidades
> específicas de freelancer con equipo (Marcela) quedan fuera de este MVP;
> ver justificación en `mvp-canvas.md`.

- **[US-01]** Como freelancer independiente, quiero gestionar mis tareas, mi
  tiempo, mis facturas y mis notas de cliente en un solo lugar, para no
  tener que sincronizar manualmente varias herramientas.
  - Criterios de aceptación:
    - Dado un cliente con al menos una tarea activa, cuando registro tiempo
      contra esa tarea, entonces ese registro queda asociado al cliente y al
      proyecto sin necesidad de repetirlo en otra herramienta.
    - Dado que actualizo el estado de una tarea, cuando reviso el proyecto,
      entonces el estado se refleja sin pasos adicionales.
  - Fuente: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md (R-01)

- **[US-02]** Como freelancer, quiero que el tiempo que registro contra un
  cliente se convierta directamente en las líneas de una factura, para no
  perder horas facturables por datos desincronizados entre herramientas.
  - Criterios de aceptación:
    - Dado tiempo registrado y no facturado para un cliente, cuando genero
      una factura para ese cliente, entonces todas las horas pendientes
      aparecen precargadas en la factura.
    - Dado que una hora ya fue incluida en una factura, cuando genero una
      factura nueva, entonces esa hora no se duplica.
  - Fuente: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md (R-02)

- **[US-03]** Como freelancer, quiero recibir un aviso cuando llevo un rato
  trabajando sin tener el registro de tiempo activo, para no perder horas
  facturables por olvido.
  - Criterios de aceptación:
    - Dado que no hay ningún registro de tiempo activo, cuando pasa un
      periodo prolongado de inactividad de registro, entonces recibo un
      aviso para iniciar o descartar el registro.
    - Dado un registro de tiempo activo, cuando pasan varias horas sin
      detenerlo, entonces recibo un aviso para confirmar si sigue en curso.
  - Fuente: daniela-freelancer-solo.md, marcela-freelancer-equipo.md (R-05)

- **[US-04]** Como freelancer, quiero ver cuántas de mis horas semanales
  fueron facturables y cuántas administrativas, para entender dónde se va
  mi tiempo.
  - Criterios de aceptación:
    - Dado un rango de fechas, cuando consulto mi resumen semanal, entonces
      veo el total de horas divididas en facturables y administrativas.
  - Fuente: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md (R-06)

- **[US-05]** Como freelancer, quiero generar la factura electrónica de un
  cliente (con IVA y retención en la fuente calculados automáticamente) sin
  tener que preparar la información aparte en una hoja de cálculo, para no
  duplicar el trabajo de facturación.
  - Criterios de aceptación:
    - Dado un conjunto de horas o entregables pendientes de facturar a un
      cliente, cuando genero la factura, entonces el sistema calcula
      subtotal, IVA (19%) y retención en la fuente aplicable sin que yo
      tenga que calcularlos a mano.
    - Dado un cliente marcado como persona jurídica con retención aplicable,
      cuando genero su factura, entonces la retención se aplica
      automáticamente; dado un cliente sin retención, entonces no se aplica.
  - Fuente: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md (R-03, R-16)

- **[US-06]** Como freelancer, quiero registrar los pagos que recibo por
  distintos canales (transferencia Bancolombia, Nequi, plataformas
  internacionales) asociados a la factura correspondiente, para no tener que
  conciliar manualmente cada canal.
  - Criterios de aceptación:
    - Dado un pago recibido en pesos colombianos, cuando lo registro contra
      una factura, entonces el saldo pendiente de esa factura se actualiza.
    - Dado un pago recibido en otra moneda, cuando lo registro, entonces
      puedo indicar el monto neto recibido en pesos para que se refleje
      correctamente en el saldo.
  - Fuente: daniela-freelancer-solo.md, felipe-freelancer-facturador.md,
    marcela-freelancer-equipo.md (R-04)

- **[US-07]** Como freelancer, quiero ver en un solo lugar cuánto he
  facturado, cuánto me han pagado, cuánto está pendiente y cuánto está
  vencido, para no tener que revisar varias hojas de cálculo para saberlo.
  - Criterios de aceptación:
    - Dado un conjunto de facturas emitidas, cuando entro al resumen de
      cobros, entonces veo el total facturado, cobrado, pendiente y vencido.
    - Dado que filtro por cliente, cuando reviso el resumen, entonces veo
      esos mismos totales solo para ese cliente.
  - Fuente: felipe-freelancer-facturador.md, marcela-freelancer-equipo.md,
    daniela-freelancer-solo.md (R-08)

- **[US-08]** Como freelancer, quiero que se me avise cuando una factura
  vence sin haber sido pagada, para hacer seguimiento a tiempo en vez de
  darme cuenta semanas después.
  - Criterios de aceptación:
    - Dado que una factura llega a su fecha de vencimiento sin pago
      registrado, cuando eso ocurre, entonces recibo un aviso identificando
      cliente y monto.
    - Dado que registro el pago de una factura vencida, cuando lo hago,
      entonces deja de aparecer como pendiente de seguimiento.
  - Fuente: felipe-freelancer-facturador.md, daniela-freelancer-solo.md,
    marcela-freelancer-equipo.md (R-09)

---

## Validación cruzada

- Cada historia cita al menos un dolor documentado en `personas.md` /
  `evidence-map.json` (coordinacion-manual-herramientas,
  time-tracking-inconsistente, facturacion-electronica-doble-paso,
  seguimiento-manual-pagos, pagos-multiples-canales-latam,
  horas-admin-no-facturables, perdida-ingresos-desorganizacion).
- Dolores compartidos de alta confianza que **no** tienen historia en este
  MVP (quedan fuera de alcance a propósito, ver `mvp-canvas.md`):
  `propuestas-tiempo-perdido`, `scope-creep`, `rentabilidad-cliente-invisible`,
  `carga-mental-constante` (se atiende parcialmente vía US-07/US-08, no de
  forma directa), `coordinacion-equipo-manual`, `pago-contratistas-basado-en-estimacion`,
  `comunicacion-clientes-dispersa`, `limite-escalar-sin-infraestructura`.
- Ningún requisito funcional core (R-01 a R-06, R-08, R-09, R-16) queda sin
  historia.
