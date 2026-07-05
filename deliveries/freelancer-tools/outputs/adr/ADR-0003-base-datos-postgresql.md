# ADR-0003: Base de datos relacional — PostgreSQL para ACID

**Fecha:** 2026-07-05  
**Estado:** Aceptado  
**Contexto de decisión:** Historias US-02, US-05, US-06, US-07 (pagos e invoices), R-16 (tributario)

---

## Problema

El MVP necesita manejar **transacciones financieras críticas:**
- US-02: Precarga automática de horas en factura (actualizar estado de time entries).
- US-05: Cálculo de IVA y total de factura.
- US-06: Registro de pagos y actualización de saldo pendiente.
- US-07: Dashboard con totales consolidados (facturado, cobrado, pendiente, vencido).

Una **lectura sucia, dirty read, o actualización no serializable** causa:
- Duplicación de pago (usuario registra el mismo pago 2 veces por conexión lenta).
- Invoices con cálculo de IVA incorrecto (dinero perdido).
- Saldos pendientes incorrectos (user think they owe X pero owe Y).
- **Dinero real perdido → Pérdida de confianza → Fracaso del MVP.**

**Fuerzas:**
- **R-16 (tributario colombiano):** La DIAN no acepta "aproximaciones". Error en IVA = multa.
- **Personas (Felipe, Marcela):** Dinero real. Felipe "regaló" 2M COP por desorganización; Marcela perdió 300k COP/mes.
- **Fiabilidad:** US-06/US-07 requieren totales exactos (no "probably correct").
- **Compliance:** Auditoría de pagos e invoices requiere trazabilidad.

---

## Alternativas consideradas

1. **PostgreSQL + ACID (actual)**
   - ✅ Transacciones ACID: garantía de consistencia.
   - ✅ Indices: queries rápidas (cliente_id, período).
   - ✅ Enumerables: estados de tarea, tipo de pago.
   - ✅ Versionado de schema.
   - ❌ Overhead inicial: setup, migrations.
   - ❌ Escalabilidad: sharding si crece >100k users (aceptable para MVP).

2. **MongoDB / NoSQL**
   - ✅ Esquema flexible.
   - ✅ Escalabilidad horizontal nativa.
   - ❌ **No ACID:** eventual consistency. Multiple writes pueden dejar saldos inconsistentes.
   - ❌ Complejidad: requiere compensating transactions (lógica custom).
   - ❌ Risk: dinero real. No es aceptable.

3. **Firebase Firestore**
   - ✅ Serverless, escalable automáticamente.
   - ✅ Security rules.
   - ❌ Transacciones limitadas a 25 operaciones por transacción (MVP precisaría múltiples si invoice tiene >25 líneas).
   - ❌ Costo imprevisible (pay-per-operation).
   - ❌ Riesgo: lock-in a Google.

4. **DynamoDB**
   - ✅ Escalabilidad.
   - ❌ Transacciones débiles (condicionales, pero no ACID completo).
   - ❌ Costoso para queries complejas (dashboard con filtros).

5. **SQLite (development only)**
   - ✅ Cero setup, desarrollo rápido.
   - ❌ No escala a múltiples servidores (MVP usará 1 server en staging/prod).
   - ❌ No replicación (si servidor cae, datos perdidos).

---

## Decisión

**PostgreSQL 14+, con transacciones ACID explícitas en operaciones financieras críticas.**

**Justificación:**
- Gold standard para financiero.
- Colombia (AWS RDS PostgreSQL disponible en región sa-east-1).
- Costo: < $50 USD/mes para MVP (managed RDS).

---

## Consecuencias

### ✅ Positivas
- **Consistencia garantizada:** ACID transactions = saldos siempre correctos.
- **Auditabilidad:** PostgreSQL tiene wal (write-ahead logging); cada transacción es registrada.
- **Índices eficientes:** Queries rápidas (cliente_id, fecha, período) para US-04 (reporte) y US-07 (dashboard).
- **Enumerables:** Estados de task, tipo de pago, status de invoice como enums de DB (no strings).

### ⚠️ Negativas
- **Setup inicial:** Migrations, schema design. +0.5 sprint (aceptable).
- **Escalabilidad vertical límite:** MVP soporta ~10k daily active users en 1 instancia. Si crece >100k users, requiere read replicas o sharding (Fase 3+).
  - **Mitigación MVP:** Monitoreo de CPU/RAM. Si latencia crece, escalar verticalmente (t3.medium → t3.large).

### 📊 Riesgo
**Muy Bajo.** PostgreSQL es estándar de facto para transaccional. Riesgo: cero.

---

## Trazabilidad

- **US-02:** Precarga automatiza horas en factura (requires transaction para actualizar time_entry.locked_at).
- **US-05:** Cálculo IVA (aritmético, pero en transacción con total de invoice).
- **US-06:** Registro de pagos (transaction: crear payment + actualizar invoice.total_paid).
- **US-07:** Dashboard (aggregate queries con GROUP BY en facturas y pagos).
- **R-16:** IVA 19% (matemático, requiere precisión decimal, no float).

---

## Schema MVP

```sql
-- Clientes
CREATE TABLE clients (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    name VARCHAR(255) NOT NULL,
    hourly_rate NUMERIC(12, 2) NOT NULL, -- COP
    invoice_due_days INT DEFAULT 30,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Invoices
CREATE TABLE invoices (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    client_id UUID NOT NULL REFERENCES clients(id),
    invoice_number VARCHAR(50) UNIQUE,
    status VARCHAR(20) DEFAULT 'draft', -- 'draft', 'emitted', 'cancelled'
    subtotal_cop NUMERIC(12, 2) NOT NULL, -- sum(lines)
    iva_amount_cop NUMERIC(12, 2) NOT NULL, -- subtotal * 0.19
    total_cop NUMERIC(12, 2) NOT NULL, -- subtotal + iva
    issue_date DATE,
    due_date DATE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Payments
CREATE TABLE payments (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    invoice_id UUID NOT NULL REFERENCES invoices(id),
    amount_cop NUMERIC(12, 2) NOT NULL,
    payment_date DATE NOT NULL,
    channel VARCHAR(50), -- 'bancolombia', 'nequi', 'payoneer', etc.
    created_at TIMESTAMP DEFAULT NOW()
);

-- Views para dashboard (computed pero consistentes con transacciones)
CREATE VIEW invoice_summary AS
SELECT 
    user_id,
    EXTRACT(YEAR_MONTH FROM issue_date) AS period,
    SUM(total_cop) as facturado_total,
    COALESCE(SUM(p.amount_cop), 0) as cobrado_total,
    SUM(total_cop) - COALESCE(SUM(p.amount_cop), 0) as pendiente_total,
    SUM(CASE WHEN due_date < NOW() THEN total_cop ELSE 0 END) as vencido_total
FROM invoices i
LEFT JOIN payments p ON i.id = p.invoice_id
WHERE i.status = 'emitted'
GROUP BY user_id, period;
```

---

## Transacciones críticas

```javascript
// US-06: Registrar pago + actualizar saldo
async function recordPayment(invoiceId, amountCOP) {
  const client = await db.connect();
  try {
    await client.query('BEGIN ISOLATION LEVEL SERIALIZABLE');
    
    // 1. Verificar que invoice existe y saldo es válido
    const invoice = await client.query(
      'SELECT * FROM invoices WHERE id = $1 FOR UPDATE',
      [invoiceId]
    );
    const totalPaid = await client.query(
      'SELECT COALESCE(SUM(amount_cop), 0) as paid FROM payments WHERE invoice_id = $1',
      [invoiceId]
    );
    
    const amountPending = invoice.rows[0].total_cop - totalPaid.rows[0].paid;
    if (amountCOP > amountPending + 1) { // Allow 1 COP rounding error
      throw new Error('Payment exceeds outstanding balance');
    }
    
    // 2. Insertar payment
    await client.query(
      'INSERT INTO payments (id, invoice_id, amount_cop, payment_date) VALUES ($1, $2, $3, $4)',
      [uuidv4(), invoiceId, amountCOP, new Date()]
    );
    
    // 3. Commit
    await client.query('COMMIT');
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

## Notas de implementación

- **ORM:** Sequelize o TypeORM + migrations (Flyway/Sequelize).
- **Backup:** AWS RDS automated backups (daily, 7-day retention).
- **Monitoring:** CloudWatch alerts si CPU > 80% o latency > 500ms.
- **Decimal precision:** Usar NUMERIC(12, 2) para COP, nunca float (rounding errors).
- **Timezone:** Almacenar todo en UTC; convertir en app layer a UTC-5 (Colombia).
