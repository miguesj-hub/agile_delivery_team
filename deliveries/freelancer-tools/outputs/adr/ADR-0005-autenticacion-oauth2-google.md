# ADR-0005: Autenticación — OAuth2 + Google Sign-In

**Fecha:** 2026-07-05  
**Estado:** Aceptado  
**Contexto de decisión:** Onboarding friction, user research (Daniela, Felipe, Marcela), security

---

## Problema

El MVP necesita autenticación. Opciones:
1. **Email/password custom:** Usuarios registran y login con contraseña.
2. **OAuth2 (Google, GitHub):** Click en "Sign in with Google", autenticación instantánea.
3. **Magic links:** Email → click → autenticado (passwordless).

**Fuerzas:**
- **Daniela:** "El modelo de suscripción mensual lo odio." Implícita: fricción = rechazo. Si el registro toma >2 min, abandona.
- **Felipe/Marcela:** Ya usan Google (Gmail, Google Drive). OAuth = familiarity + zero friction.
- **Security:** OAuth delega autenticación a Google (trusted party), eliminamos el riesgo de password breach.
- **Personas en Colombia:** Google es ubiquos (99% de users tienen Gmail). GitHub es menos común.

---

## Alternativas consideradas

1. **Email/password + password recovery + email verification**
   - ✅ Full control sobre auth.
   - ❌ User experience pain: "forgot password?" → email → click → nueva contraseña (4 pasos).
   - ❌ Security overhead: hash/salt passwords, rate limiting, OWASP compliance.
   - ❌ Friction: +30-60s para registro.

2. **OAuth2 + Google** (actual)
   - ✅ 1-click signup/login.
   - ✅ Zero password management (Google handles it).
   - ✅ Instant email verification (from Google).
   - ✅ Familiar UX.
   - ❌ Dependency on Google (but acceptable; Google is reliable).
   - ❌ Risk: if Google API goes down, no logins (mitigation: rare + we handle gracefully).

3. **OAuth2 + Google + GitHub**
   - ✅ Flexibility.
   - ❌ Developers prefer GitHub, but Daniela/Felipe/Marcela (not developers) don't have GitHub.
   - ❌ Extra maintenance.

4. **Magic links (email-only, passwordless)**
   - ✅ No passwords to manage.
   - ✅ No reliance on Google OAuth.
   - ❌ Friction: user opens email, clicks link, must happen in ~10 min before expiry.
   - ❌ Email delivery not guaranteed (spam filters).
   - ❌ Testing pain (magic link flow is manual to test).

5. **SAML (enterprise SSO)**
   - ✅ Enterprise-ready.
   - ❌ Overkill for MVP (Daniela, Felipe, Marcela are individuals, not enterprises).
   - ❌ Setup complexity.

---

## Decisión

**Implementar OAuth2 + Google Sign-In como autenticación principal.**

**Optional (Fase 2):** Agregar GitHub si necesario (for team collaboration, R-14).

---

## Consequencias

### ✅ Positivas
- **Friction mínima:** Users see "Sign in with Google" → click → redirected to Google → authorized → back to app. ~15 seconds.
- **Security:** Google manages password security. We don't store passwords.
- **UX consistency:** Daniela/Felipe/Marcela already understand OAuth (Gmail, Google Drive, etc.).
- **No recovery friction:** "Forgot password?" doesn't exist.

### ⚠️ Negativas
- **Google dependency:** If Google OAuth is down, no logins (rarely happens, but possible).
  - **Mitigation:** Graceful error page, retry mechanism, notify users.
- **User privacy:** Google sees that user is logging into our app (but we don't share their data with Google beyond the OAuth handshake).
  - **Mitigation:** Privacy policy + GDPR compliance (for EU users, if applicable).

### 📊 Riesgo
**Very Low.** OAuth2 + Google is industry standard for consumer apps. Risk: minimal.

---

## Trazabilidad

- **Personas (Daniela, Felipe, Marcela):** All use Google/Gmail. OAuth aligns with existing behavior.
- **US-01:** Hub requires user login. OAuth eliminates friction.
- **R-18:** No external integrations (except Google OAuth, which is trusted third-party).

---

## Implementation Details

### OAuth2 Flow
```
1. User clicks "Sign in with Google"
2. Browser redirects to: https://accounts.google.com/o/oauth2/v2/auth?client_id=...
3. User logs in with Google (or already authenticated)
4. Google prompts: "freelancer-tools wants to access your email"
5. User clicks "Allow"
6. Google redirects back to: /callback?code=AUTH_CODE
7. Backend exchanges AUTH_CODE for access_token (server-to-server)
8. Backend extracts user email + name from Google's API
9. Backend creates/updates User record in DB
10. Backend creates JWT or session
11. User redirected to /dashboard, authenticated
```

### Backend (Express + Passport)
```typescript
import passport from 'passport';
import { Strategy as GoogleStrategy } from 'passport-google-oauth20';

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: '/auth/google/callback',
    },
    async (accessToken, refreshToken, profile, done) => {
      // 1. Find or create user
      let user = await User.findOne({ email: profile.emails[0].value });
      if (!user) {
        user = await User.create({
          email: profile.emails[0].value,
          name: profile.displayName,
          googleId: profile.id,
        });
      }
      // 2. Create JWT or session
      return done(null, user);
    }
  )
);

// Routes
app.get('/auth/google', passport.authenticate('google', { scope: ['email', 'profile'] }));
app.get(
  '/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    const token = jwt.sign({ userId: req.user.id }, JWT_SECRET);
    res.redirect(`${FRONTEND_URL}/dashboard?token=${token}`);
  }
);
```

### Frontend (React)
```typescript
// components/LoginPage.tsx
export const LoginPage = () => {
  const handleGoogleSignIn = () => {
    // Redirect to backend OAuth URL
    window.location.href = `${API_URL}/auth/google`;
  };

  return (
    <div>
      <h1>freelancer-tools</h1>
      <button onClick={handleGoogleSignIn}>
        Sign in with Google
      </button>
    </div>
  );
};
```

### Session Management
Option 1 (JWT):
```
- Backend generates JWT on successful OAuth
- Frontend stores JWT in localStorage (or cookie)
- Every API request includes JWT in Authorization header
- Backend validates JWT signature (no DB call needed)
```

Option 2 (Sessions):
```
- Backend creates session in Redis/DB
- Frontend stores session ID in cookie
- Every request sends cookie
- Backend validates session ID in store
```

**Decision:** JWT for MVP (stateless, simpler scaling). Sessions in Fase 2 if needed.

---

## Data Model

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255),
    google_id VARCHAR(255) UNIQUE,
    preferences JSONB DEFAULT '{}', -- {email_notifications_opted_in, grace_days_alert}
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Security Considerations

1. **HTTPS only:** All OAuth traffic must be over HTTPS. Enforce in production.
2. **Scope minimization:** Only request `email` and `profile` from Google (not calendar, drive, etc.).
3. **CSRF protection:** Validate `state` parameter in OAuth flow (Passport handles this).
4. **Token expiry:** JWT expires in 7 days. Refresh token (optional) in Fase 2.
5. **Secure cookies:** If using session cookies, set `HttpOnly`, `Secure`, `SameSite=Strict`.

---

## Future Enhancements (Fase 2)

1. **GitHub OAuth:** For team members (R-14).
2. **Apple Sign-In:** For iOS app (if mobile is built, R-15).
3. **Multi-factor auth:** Biometric or 2FA (if security risk increases).
4. **Token refresh:** Automatic JWT refresh without logout (better UX).

---

## Testing

```typescript
// tests/auth.test.ts
describe('OAuth2 Google', () => {
  it('should redirect to Google OAuth endpoint', async () => {
    const res = await request(app).get('/auth/google');
    expect(res.status).toBe(302);
    expect(res.headers.location).toContain('accounts.google.com');
  });

  it('should create user on first OAuth callback', async () => {
    const mockProfile = {
      id: 'google-123',
      displayName: 'Felipe Dev',
      emails: [{ value: 'felipe@example.com' }],
    };
    // Mock OAuth response
    // Assert user created in DB
  });

  it('should return JWT after successful auth', async () => {
    // Test full flow: Google → callback → JWT
  });
});
```

---

## Monitoring

- **Google OAuth outage:** Alert if `/auth/google/callback` returns >5% 5xx errors in 5 min.
- **Login success rate:** Track % of users who complete OAuth flow (expect >95%).
- **Token expiry:** Monitor JWT expiry-related errors (expect <1%).
