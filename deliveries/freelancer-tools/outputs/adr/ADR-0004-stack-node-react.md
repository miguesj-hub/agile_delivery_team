# ADR-0004: Stack tecnológico — Node.js + React + TypeScript

**Fecha:** 2026-07-05  
**Estado:** Aceptado  
**Contexto de decisión:** US-01, US-03 (timer responsivo), performance, equipoVelocidad de desarrollo

---

## Problema

El MVP necesita:
1. **Timer responsivo (US-03):** 30 min inactividad, 4h sin pausa. Notificaciones deben ser "inmediatas" (<500ms latency).
2. **Dashboard en tiempo real (US-07):** Totales actualizados cuando usuario registra pago o factura.
3. **Desarrollo rápido:** 8 historias, 36 pts, presión de timeline.
4. **Tipo-seguridad:** Precisión en cálculos (IVA 19%, saldos). Errores en producción = dinero real perdido.
5. **Full-stack JavaScript:** Team preferiblemente usa un solo lenguaje (hipótesis: para MVP, monolith es aceptable).

**Fuerzas:**
- **Performance (US-03):** Timer UI no debe tener lag. Notificaciones in-app deben ser sub-500ms.
- **Developer velocity:** Node.js + npm ecosystem. Prototipar rápido.
- **Productividad:** TypeScript elimina clases de bugs en cálculos (typos, type mismatches).
- **Full-stack simplicity:** 1 lenguaje = menos context-switch.
- **Personas (Felipe, Marcela):** No importa el stack; importa que funcione. MVP es MVP.

---

## Alternativas consideradas

1. **Node.js + Express (backend) + React (frontend) + TypeScript** (actual)
   - ✅ Performance: Node.js no-blocking I/O, ideal para notificaciones.
   - ✅ TypeScript: type-safety en cálculos (IVA, saldos).
   - ✅ React: componentes reutilizables, desarrolladores abundantes.
   - ✅ Dev velocity: npm ecosystem es enorme (BullMQ, Zod, etc.).
   - ✅ Full-stack: 1 lenguaje.
   - ❌ Escalabilidad horizontal: requiere orchestration (Docker, Kubernetes). MVP no lo necesita.

2. **Python + FastAPI (backend) + React (frontend)**
   - ✅ Python es robusto para cálculos (Decimal precision).
   - ✅ FastAPI es muy rápido (ASGI).
   - ✅ Tipo-seguridad (type hints).
   - ❌ Más lento que Node.js en cold starts (si serverless).
   - ❌ 2 lenguajes (Python + JS) = context-switch.
   - ❌ Deployment: requiere Python runtime.

3. **Go (backend) + React (frontend)**
   - ✅ Muy rápido (compiled).
   - ✅ Escalabilidad horizontal nativa.
   - ❌ Desarrollo más lento (compiled, typesystem más rígido).
   - ❌ 2 lenguajes.
   - ❌ Go tiene menos librerías para dominio business (auditoría, tributario).

4. **Monolith Fullstack Ruby on Rails**
   - ✅ 1 lenguaje.
   - ✅ Convención > Configuración.
   - ✅ Scaffold rápido.
   - ❌ Menos tipo-seguridad que TypeScript.
   - ❌ Performance: lentitud relativa en timers en tiempo real.

5. **AWS Lambda (serverless) + API Gateway + React**
   - ✅ Escalabilidad automática.
   - ✅ Pago solo por uso (MVP: bajo costo).
   - ❌ Cold starts (>1s latency si lambda no está warm).
   - ❌ US-03 (timer con notificaciones) sufre con cold starts.
   - ❌ Complejidad: múltiples lambdas, step functions para workflows.

6. **Firebase / Vercel + Next.js fullstack**
   - ✅ Fullstack, full-stack JS.
   - ✅ Serverless, escalable.
   - ❌ Lock-in a Google/Vercel.
   - ❌ Menos control sobre transacciones (R-03, pagos).
   - ❌ Firestore transacciones débiles (25 operaciones max).

---

## Decisión

**Node.js + Express (backend) + React (frontend) + TypeScript + PostgreSQL.**

**Justificación:**
- **Performance:** Node.js no-blocking I/O → timer responsivo (US-03 <500ms latency).
- **Type-safety:** TypeScript → cálculos correctos (IVA, saldos).
- **Developer velocity:** npm ecosystem + scaffolding templates (Create React App / Vite).
- **Full-stack:** 1 lenguaje (JavaScript/TypeScript).
- **Operational simplicity:** 1 Docker image para backend + frontend. Deployment simple (Vercel o Docker + AWS).

---

## Consecuencias

### ✅ Positivas
- **Responsive UI:** React + optimizations (memo, useMemo) → timer no tiene lag.
- **Real-time capability:** Node.js + WebSocket (Socket.io) → notificaciones in-app instant.
- **Type-safety:** TypeScript → Errors at compile time, not at 3am in production.
- **Rapid iteration:** npm install + dev server (Vite) → instant feedback loop.
- **Testability:** Jest + React Testing Library → tests rápidos y mantenibles.

### ⚠️ Negativas
- **Escalabilidad horizontal:** Node.js requiere stateless design + session store (Redis). MVP no lo necesita; Fase 2 si crece >10k DAU.
  - **Mitigación:** Arquitectura sin estado desde el inicio (no storage local en processo).
- **Type-system:** TypeScript a veces es verbose. Trade-off: verbosidad > errores.
- **JavaScript ecosystem:** Fragmentación (5 formas de hacer cada cosa). Mitigation: style guide, ESLint, prettier.

### 📊 Riesgo
**Muy Bajo.** Node.js + React son estándares de facto. Risk: cero para MVP.

---

## Trazabilidad

- **US-01:** Hub de tareas/tiempo/clientes → React SPA + Node.js API.
- **US-03:** Timer responsivo, recordatorio 30 min → WebSocket + BullMQ scheduler.
- **US-05:** Cálculo IVA 19% → TypeScript Decimal library (decimal.js).
- **US-06, US-07:** Pagos, dashboard → Node.js/Express API + React queries.

---

## Tech Stack Detallado

### Backend (Node.js)
```
├── Runtime: Node.js 20 LTS
├── Framework: Express.js 4.18+
├── Language: TypeScript 5+
├── Database
│   ├── ORM: Sequelize 6 (or TypeORM)
│   └── Migrations: Sequelize CLI (or Flyway)
├── Auth: Passport.js + @passport-js/passport-google-oauth2
├── Job Queue: BullMQ (Redis-backed)
├── HTTP Client: axios (external APIs)
├── Validation: Zod (schema validation)
├── Logging: Winston + Morgan
├── Testing: Jest + supertest (API)
├── API Docs: Swagger/OpenAPI (@nestjs/swagger or swagger-ui-express)
└── Linting: ESLint + Prettier
```

### Frontend (React)
```
├── Framework: React 18+ with Hooks
├── Language: TypeScript
├── Build: Vite (or Create React App)
├── State: Zustand (global) + TanStack Query (async)
├── Forms: React Hook Form + Zod
├── UI Components: Headless (Radix UI / Shadcn) + Tailwind CSS
├── Charts: Recharts (for future Fase 2 graphs)
├── Real-time: Socket.io-client (for notifications)
├── Testing: Vitest + React Testing Library
└── Linting: ESLint + Prettier
```

### Deployment
```
├── Frontend: Vercel (or AWS Amplify)
├── Backend: Docker + AWS ECS/Fargate (or DigitalOcean App Platform)
├── Database: AWS RDS (PostgreSQL managed)
├── Cache: Redis (if needed Fase 2)
└── CI/CD: GitHub Actions
```

---

## Performance targets (MVP)

| Métrica | Target | Rationale |
|---------|--------|-----------|
| API latency (p50) | <100ms | Django dashboard refresh instant |
| API latency (p99) | <500ms | Tolerable para timer |
| Frontend bundle | <200KB gzipped | Initial load <2s (3G speed) |
| Time to Interactive | <3s | Not a mobile app, pero responsive |
| Dashboard query | <200ms | Consolidate 4 totals from invoices/payments |

---

## Notas de implementación

### Backend folder structure
```
src/
├── config/          # Database, env, OAuth config
├── middleware/      # Auth, error handling, logging
├── services/        # Business logic (TaskService, InvoiceService, etc.)
├── controllers/     # HTTP handlers
├── models/          # Database models (Sequelize)
├── routes/          # API routes
├── jobs/            # Background jobs (alert scheduler)
├── utils/           # Helpers (IVA calc, date utils)
└── tests/           # Jest tests
```

### Environment variables
```
DATABASE_URL=postgresql://user:pass@localhost:5432/freelancer_tools
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
JWT_SECRET=... (for session management)
REDIS_URL=redis://localhost:6379 (for BullMQ, optional MVP)
SENDGRID_API_KEY=... (for email notifications)
NODE_ENV=development|production
```

---

## Future migrations (Fase 2+)

- **Horizontal scaling:** Add Node.js replicas + load balancer (AWS ALB) if DAU > 10k.
- **Microservices:** Split invoices, payments, notifications into separate services (only if team grows >5 people).
- **GraphQL:** Add GraphQL layer if frontend complexity grows.
- **Mobile:** React Native from React web codebase (significant refactor, probably Fase 3).
