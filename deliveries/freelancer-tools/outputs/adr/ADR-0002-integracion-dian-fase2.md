# ADR-0002: Integración DIAN postergada a Fase 2

**Fecha:** 2026-07-05  
**Estado:** Aceptado (con open question explícita)  
**Contexto de decisión:** E-02 (épica crítica), R-03, R-16, riesgo regulatorio alto

---

## Problema

La épica **E-02** (Generar facturas con cálculo local de IVA sin integración DIAN en MVP) es crítica pero concentra el riesgo regulatorio/técnico más alto del MVP. El requisito **R-03** pide: *"El sistema debe generar factura electrónica válida ante la DIAN (con IVA 19% y retención en la fuente calculados automáticamente)"*. Sin embargo:

1. **Integración DIAN requiere proveedor habilitado** (Siigo, Alegra, Facturador en la Nube, propio).
2. **Reglas de retención en la fuente son complejas y cambias** (dependen de tipo de cliente, ramo económico, acuerdos específicos).
3. **Validación contable/tributaria es crítica:** Error = dinero real perdido, exposición legal.
4. **MVP aún no ha resuelto qué proveedor usar** (ADR abierta).

**Fuerzas:**
- **E-02 es CRÍTICA:** Sin facturas válidas ante DIAN, no hay MVP (users como Felipe, Marcela siguen dependiendo de Siigo/Alegra).
- **Riesgo regulatorio alto:** La DIAN no acepta "aproximaciones"; un cálculo de retención incorrecto causa multas.
- **Incertidumbre técnica:** ¿Integración con API del proveedor? ¿Firma digital? ¿Certificados?
- **Time-boxing:** El MVP tiene capacidad limitada (36 pts en 8 historias). Agregar integración DIAN +1-2 sprints.
- **Felipe/Marcela:** Hoy usan Siigo/Alegra para la factura "oficial". El MVP puede generar facturas internas correctas y luego integrar.

---

## Alternativas consideradas

1. **Integración DIAN completa en MVP**
   - ✅ Usuario tiene factura oficial directamente
   - ❌ +2-3 sprints (incertidumbre: proveedor, validación tributaria)
   - ❌ Bloquea el resto de la épica E-01 y E-03 (priorización de riesgo incorrecta)
   - ❌ Si el proveedor falla, MVP es inútil
   - ❌ Requiere validación con contador/DIAN antes de lanzar

2. **Integración DIAN en MVP pero simplificada** (solo IVA, no retención)
   - ✅ Aún más rápido que alternativa 1
   - ❌ Sigue siendo incompleta; users no pueden usar en producción sin retención
   - ❌ Confuso: sistema calcula IVA pero no retención (¿por qué?)

3. **Cálculo local IVA 19% en MVP + Integración DIAN en Fase 2** (actual)
   - ✅ MVP entrega valor inmediato: facturas con IVA correcto (83% del valor)
   - ✅ Integración DIAN en Fase 2, cuando arquitectura sea clara
   - ✅ Users pueden usar MVP para gestión interna y Siigo/Alegra para factura oficial (workflow actual)
   - ❌ Aún necesitan Siigo/Alegra (pero eso es realidad del mercado)

4. **Sin cálculo de IVA en MVP (solo subtotal)**
   - ✅ Máxima simplicidad
   - ❌ Menos valor entregado (84% del valor E-02)
   - ❌ Confuso: sistema "genera facturas" pero usuario debe calcular IVA manualmente

---

## Decisión

**Implementar cálculo local de IVA 19% en MVP (US-05, 3 pts). Integración DIAN + retención en Fase 2.**

**Justificación:**
- MVP entrega 83% del valor de E-02 sin riesgo regulatorio.
- Workflow realista: MVP genera factura interna → usuario exporta/copia a Siigo/Alegra para factura oficial ante DIAN.
- Fase 2 resuelve la arquitectura de integración DIAN (ADR pendiente: ¿cuál proveedor?).
- Timeline: MVP en 2-3 sprints. Fase 2 integracion DIAN en 1-2 sprints adicionales.

---

## Consecuencias

### ✅ Positivas
- **MVP lanzable en 2-3 sprints** sin bloqueos regulatorios.
- **Valor entregado:** Usuarios reducen reescrituras manual (Sheets → Siigo) en ~50%.
- **Reducción de riesgo:** IVA es matemático y determinístico; no requiere validación regulatoria especial.
- **Flexibilidad Fase 2:** Architects tendrá más tiempo para investigar proveedores habilitados.

### ⚠️ Negativas
- **Incompleto:** Felipe/Marcela aún necesitan Siigo/Alegra para factura "oficial" ante DIAN.
  - **Realidad:** Eso es cierto hoy; MVP no empeora la situación.
  - **Roadmap:** Fase 2 lo resuelve.
- **User expectation:** Usuario puede pensar que la factura generada es válida ante DIAN (NO).
  - **Mitigación:** UX explícita: "Factura interna (IVA calculado). Para factura oficial ante DIAN, usa [proveedor habilitado]".

### 📊 Riesgo
**Medio.** La arquitectura de integración DIAN aún no está resuelta. Riesgo: Fase 2 descubre un constraint no anticipado (p. ej. firma digital obligatoria en MVP, no solo en Fase 2).

---

## Trazabilidad

- **E-02 (épica crítica):** Generar facturas. MVP = IVA; Fase 2 = DIAN + retención.
- **R-03:** Factura electrónica DIAN (split: MVP = IVA local; Fase 2 = integración DIAN).
- **R-16:** Contexto tributario colombiano nativo (MVP = IVA 19%; Fase 2 = retención).
- **US-05 (3 pts):** Cálculo automático de IVA 19% (historia MVP).
- **Personas (Felipe, Marcela):** Ambas usan Siigo/Alegra hoy. MVP reduce fricción pero no es substituto.

---

## ADR abierta para Fase 2

### ¿Cuál es el proveedor habilitado por DIAN a integrar en Fase 2?

| Opción | Ventajas | Desventajas |
|--------|----------|-------------|
| **Siigo** | Felipe la usa hoy. API documentada. | Costo: ~$60k/mes (Felipe la menciona como caro). |
| **Alegra** | Más orientada a freelancers. Menor costo que Siigo. | Menos market share; menos validación en el segmento. |
| **Facturador en la Nube** | DIAN propio. | Beta. Menos documentación. |
| **SDK/certificado DIAN propio** | Máximo control. Única solución de long term. | Altísimo costo de implementación + mantenimiento. |

**Decisión postergada:** Architect debe investigar (Fase 2) cuál opción es técnicamente viable y costo-efectiva.

---

## Notas de implementación

- **US-05 (MVP):** 
  - Input: subtotal de horas × tarifa
  - Cálculo: IVA = subtotal × 0.19
  - Output: Factura con subtotal, IVA, total
  - Nota: IVA se calcula a nivel de factura completa (no por línea)

- **Fase 2 ADR:**
  - Validar retención en la fuente con contador de Felipe/Marcela
  - Seleccionar proveedor habilitado
  - Prototipar integración (1-2 sprints)
  - Testing con contador (validación tributaria)

- **Data model:** Reservar campo `dian_invoice_number` en tabla Invoice (null en MVP, poblado en Fase 2).
