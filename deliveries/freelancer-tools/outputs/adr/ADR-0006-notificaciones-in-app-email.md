# ADR-0006: Notificaciones — In-app + Email OPT-IN, sin dependencia de terceros

**Fecha:** 2026-07-05  
**Estado:** Aceptado  
**Contexto de decisión:** US-03 (recordatorio timer), US-08 (alerta vencida), R-18 (sin integraciones frágiles)

---

## Problema

El MVP requiere **notificaciones críticas:**
- **US-03:** Recordatorio a los 30 min sin timer activo (in-app mandatory).
- **US-08:** Alerta cuando factura vence (in-app mandatory, email optional).

El requisito **R-18** dice: *"El sistema debe mantener sus propios módulos... sincronizados de forma confiable, sin depender de integraciones externas frágiles"*.

**Fuerzas:**
- **R-18:** Integraciones frágiles (Zapier fallió). Notificaciones deben ser confiables.
- **US-03, US-08:** Críticas para el MVP (Felipe perdió $5M COP; necesita alertas vencidas).
- **Personas (Daniela):** Abandonó Zapier por fallas; desconfianza en terceros.
- **Email OPT-IN:** Usuarios quieren controlar si reciben email (GDPR, privacidad).

**Trade-offs:**
- **In-app (obligatorio):** Siempre funciona si sistema está online. Requiere que usuario abra la app.
- **Email (optional):** Llega incluso si usuario no abre la app. Pero email puede no entregarse (spam).
- **SMS/Push:** Más intrusivo. Requiere integración (Twilio). Evitar en MVP.

---

## Alternativas consideradas

1. **In-app + Email vía SendGrid/Twilio (third-party)**
   - ✅ Email entrega confiable.
   - ❌ R-18 pide no depender de integraciones frágiles. SendGrid/Twilio = terceros.
   - ❌ Si SendGrid API falla, notificaciones de email no se envían.
   - ❌ Costo: SendGrid ~$29/mes (aceptable, pero tercera dependencia).

2. **In-app only (no email)**
   - ✅ Máxima confiabilidad (sin terceros).
   - ✅ Máxima privacidad.
   - ❌ Felipe no recibirá alerta vencida si no abre la app ese día (problema real).
   - ❌ Menor valor entregado.

3. **In-app + Email vía SMTP propio**
   - ✅ No depende de SendGrid/Twilio.
   - ✅ Control total sobre contenido y timing.
   - ✅ Email OPT-IN (usuarios deciden).
   - ❌ Overhead: configurar servidor SMTP (AWS SES, Mailgun, o SMTP local).
   - ❌ Deliverabilidad: sin historial de IP, emails pueden ir a spam.
   - ✅ Pero manejable con AWS SES (confiable, low cost).

4. **Push notifications vía WebSocket**
   - ✅ In-app instant (no polling).
   - ✅ Sin terceros.
   - ❌ Requiere conexión activa (usuario debe tener app abierta).
   - ❌ Mobile: requiere app nativa (fuera del MVP).

5. **Background job + scheduled emails**
   - ✅ Determinístico (cron dispara alertas a 00:00).
   - ✅ Control total.
   - ❌ Requiere SMTP (infraestructura adicional).

---

## Decisión

**Notificaciones in-app (obligatorio) + Email vía AWS SES (OPT-IN, no Twilio/SendGrid).**

**Justificación:**
- **R-18 compliance:** No terceros frágiles. AWS SES es infraestructura confiable (mismo proveedor que DB).
- **Confiabilidad:** In-app = 100% (si sistema online). Email = 95%+ (SES tiene historial de 99.9% uptime).
- **Cost:** AWS SES ~$0.10 per 1000 emails (negligible para MVP).
- **Simplicity:** SMTP es estándar, mantenible.
- **Privacy:** Email OPT-IN respeta preferencias del usuario.

---

## Consequencias

### ✅ Positivas
- **R-18 compliant:** Sin Twilio/SendGrid/Zapier. Dependencias solo: Google (OAuth), AWS (SES+RDS).
- **Reliability:** In-app notificaciones son instant (WebSocket/polling). Email vía SES es confiable.
- **User control:** Email es OPT-IN (preferencias → email_notifications_opted_in).
- **Cost:** Negligible (<$10/month para MVP).
- **Extensible:** Si email no es suficiente (Fase 2), agregamos SMS (Twilio) sin romper sistema.

### ⚠️ Negativas
- **Email deliverability:** Sin historial de IP, emails pueden ir a spam (but SES mitigates this with domain verification).
  - **Mitigation:** SPF, DKIM, DMARC records. SES provides guides.
- **SMTP infrastructure:** Requiere setup (AWS SES, domain verification, DKIM).
  - **Mitigation:** ~1h setup, documented.

### 📊 Riesgo
**Low.** AWS SES es estándar de facto. Risk: minimal.

---

## Trazabilidad

- **US-03:** Recordatorio timer (in-app, email OPT-IN).
- **US-08:** Alerta factura vencida (in-app, email OPT-IN).
- **R-18:** No depender de integraciones externas frágiles.
- **Personas (Daniela):** Desconfianza en Zapier; preferencia por sistema cerrado.

---

## Implementation Details

### In-app Notifications

#### Data model
```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    type VARCHAR(50) NOT NULL, -- 'timer_inactive', 'timer_prolonged', 'invoice_due'
    reference_id UUID, -- time_entry_id or invoice_id
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    read BOOLEAN DEFAULT FALSE,
    read_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Backend (Node.js)
```typescript
// services/notificationService.ts
export class NotificationService {
  async createNotification(userId: string, type: string, message: string, referenceId?: string) {
    return db.query(
      `INSERT INTO notifications (user_id, type, message, reference_id, created_at)
       VALUES ($1, $2, $3, $4, NOW())`,
      [userId, type, message, referenceId]
    );
  }

  async getUserNotifications(userId: string) {
    return db.query(
      `SELECT * FROM notifications WHERE user_id = $1 ORDER BY created_at DESC LIMIT 50`,
      [userId]
    );
  }

  async markAsRead(notificationId: string) {
    return db.query(
      `UPDATE notifications SET read = TRUE, read_at = NOW() WHERE id = $1`,
      [notificationId]
    );
  }
}
```

#### Frontend (React)
```typescript
// components/NotificationCenter.tsx
export const NotificationCenter = () => {
  const [notifications, setNotifications] = useState([]);
  
  // Poll backend for new notifications every 10s (or WebSocket for real-time)
  useEffect(() => {
    const interval = setInterval(async () => {
      const res = await fetch('/api/notifications', { headers: { Authorization: `Bearer ${token}` } });
      const data = await res.json();
      setNotifications(data);
    }, 10000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div className="notification-center">
      {notifications.filter(n => !n.read).map(n => (
        <div key={n.id} className="notification">
          <p>{n.message}</p>
          <button onClick={() => markAsRead(n.id)}>Dismiss</button>
        </div>
      ))}
    </div>
  );
};
```

### Email Notifications

#### AWS SES Setup
```bash
# 1. Verify domain in AWS SES console
# 2. Add SPF, DKIM, DMARC records to DNS
# 3. Request production access (move out of sandbox)
```

#### Backend (Node.js + nodemailer)
```typescript
import nodemailer from 'nodemailer';

const transporter = nodemailer.createTransport({
  host: 'email-smtp.us-east-1.amazonaws.com',
  port: 587,
  auth: {
    user: process.env.AWS_SES_USER,
    pass: process.env.AWS_SES_PASS,
  },
});

export async function sendEmailNotification(user: User, notification: Notification) {
  if (!user.preferences.email_notifications_opted_in) {
    return; // OPT-IN: respeta preferencia
  }

  const subject = `Alerta: ${notification.title}`;
  const html = `
    <h2>${notification.title}</h2>
    <p>${notification.message}</p>
    <a href="${FRONTEND_URL}/dashboard">Ver en freelancer-tools</a>
  `;

  await transporter.sendMail({
    from: 'notificaciones@freelancer-tools.com',
    to: user.email,
    subject,
    html,
  });
}
```

### Job Scheduler (Cron)

#### Background job para alertas vencidas (US-08)
```typescript
// jobs/invoiceDueAlerts.ts
import BullMQ from 'bullmq';

const alertQueue = new BullMQ.Queue('invoice-due-alerts', { connection: redis });

// Trigger diariamente a las 00:00 UTC-5
export async function scheduleInvoiceDueAlerts() {
  await alertQueue.add(
    'check-due-invoices',
    {},
    {
      repeat: {
        pattern: '0 0 * * *', // Cron: 00:00 diariamente
        tz: 'America/Bogota', // Colombia timezone
      },
    }
  );
}

alertQueue.process(async (job) => {
  // 1. Find all invoices with due_date <= today + grace_days
  const invoices = await db.query(`
    SELECT * FROM invoices
    WHERE status = 'emitted'
    AND due_date <= CURRENT_DATE + INTERVAL '${gracedays} days'
    AND total_cop - COALESCE((SELECT SUM(amount_cop) FROM payments WHERE invoice_id = id), 0) > 0
  `);

  // 2. For each invoice, create notification + send email
  for (const invoice of invoices) {
    const user = await User.findById(invoice.user_id);
    const client = await Client.findById(invoice.client_id);

    // 3. In-app notification
    await notificationService.createNotification(
      user.id,
      'invoice_due',
      `Factura #${invoice.invoice_number} vencida (${client.name}, $${invoice.total_cop} COP)`,
      invoice.id
    );

    // 4. Email (OPT-IN)
    if (user.preferences.email_notifications_opted_in) {
      await sendEmailNotification(user, {
        title: 'Factura vencida',
        message: `Tu factura #${invoice.invoice_number} para ${client.name} vencio hoy. Monto: $${invoice.total_cop} COP`,
      });
    }
  }
});
```

### User Preferences

```sql
CREATE TABLE user_preferences (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    email_notifications_opted_in BOOLEAN DEFAULT FALSE,
    grace_days_before_alert INT DEFAULT 0, -- days before due date to send alert
    notification_frequency VARCHAR(50) DEFAULT 'daily', -- or 'once', 'never'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Testing

```typescript
describe('Notifications', () => {
  it('should create in-app notification immediately', async () => {
    const notification = await notificationService.createNotification(
      userId,
      'timer_inactive',
      'No timer active for 30 min'
    );
    expect(notification.created_at).toBeDefined();
  });

  it('should send email only if OPT-IN', async () => {
    const userOptedOut = { ...user, email_notifications_opted_in: false };
    const spy = jest.spyOn(transporter, 'sendMail');
    
    await sendEmailNotification(userOptedOut, notification);
    expect(spy).not.toHaveBeenCalled();
  });

  it('should schedule invoice-due checks daily at 00:00', async () => {
    // Mock time to 23:59
    // Verify job queued at 00:00
  });
});
```

---

## Monitoring

- **In-app notification latency:** <1s (from creation to appearing in UI).
- **Email delivery:** SES delivery status. Expect >95% inbox delivery.
- **Job scheduler:** Daily invoice-due job should complete in <5s (for MVP scale).
- **Unsubscribe rate:** Track % of users who opt-out of email (expect <10% for critical alerts).

---

## Future Enhancements (Fase 2)

1. **SMS notifications:** Twilio integration for critical alerts (high-value invoices).
2. **Push notifications:** Browser push (Web Push API) or mobile app push.
3. **Notification preferences:** Per-type (timer alerts vs. payment alerts).
4. **Email digest:** Weekly summary instead of per-alert (reduce email volume).
5. **Slack integration:** For team members (R-14 specific).
