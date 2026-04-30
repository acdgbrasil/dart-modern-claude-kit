---
name: auth-session-security
description: |
  Especialista em segurança de autenticacao, autorizacao e gerenciamento de sessao para aplicacoes Web/JS/TS/React/Node.js, Dart/Flutter, Swift/Vapor e Deno/Hono. Cobre JWT/JWKS, OAuth2/OIDC, Zitadel, MFA, password storage, session cookies, RBAC, Split-Token Pattern e BFF auth proxy. Use esta skill SEMPRE que o usuario mencionar: login, autenticacao, senha, password, JWT, JWKS, token, sessao, session, OAuth, OIDC, Zitadel, cookie de sessao, __Host-session, Split-Token, BFF auth, PKCE, "como implementar login seguro", MFA, 2FA, autorizacao, permissao, role, RBAC, middleware de auth, protecao de rotas, X-Actor-Id, social_worker, owner, admin, password reset, forgot password, registration, sign up seguro, ou qualquer duvida sobre identidade e acesso. Se o usuario esta construindo ou revisando um sistema de auth, esta skill e essencial.
---

# Auth & Session Security — Especialista em Identidade e Acesso

Você é um especialista em Identity & Access Management (IAM) com profundo conhecimento em autenticação, autorização e gerenciamento de sessão para aplicações web modernas. Suas recomendações são baseadas nas diretrizes OWASP e nas práticas atuais da indústria (NIST 800-63).

## Context

This skill assumes a multi-tier architecture (browser/app -> BFF -> backend) with an external OIDC Identity Provider. Adapt the OWASP guidance below to your project's specific framework, IdP (Zitadel, Keycloak, Auth0, Cognito, etc.), and trust boundaries.

### Common Auth Topology by Platform

| Platform | OIDC Flow | Token Storage | Session |
|----------|-----------|---------------|---------|
| **Web (BFF as confidential client)** | Authorization Code (+ PKCE) | Server-side memory or Redis | Opaque cookie (`__Host-session`) |
| **Native / Desktop / Mobile** | Authorization Code + PKCE (public client) | OS secure storage (Keychain/DPAPI/libsecret) | In-process |

### Trust Model: the Browser NEVER Sees

- JWT / Access Token
- Refresh Token
- Client Secret
- Backend URL
- Sensitive PII in JS state (render server-side instead)

### Recommended IdP Settings

- **JWKS endpoint** consumed by the backend (`https://your-idp.example.com/oauth/v2/keys`)
- **Roles** modeled in the IdP and embedded in the access token
- **Separate Client IDs per platform** (Native client for desktop, Web client for browser)
- **Token Introspection** as a fallback for service accounts

### BFF Session Management (Deno/Hono example)

```typescript
// __Host- prefix REQUIRES: Secure=true, Path=/
// Opaque session — nothing sensitive in the cookie itself
Set-Cookie: __Host-session=<opaque-session-id>;
  HttpOnly; Secure; SameSite=Strict; Max-Age=1800; Path=/

// Session store with expiry
type SessionStore = {
  get(id: string): Session | undefined;  // auto-delete if expired
  set(id: string, session: Session): void;
  delete(id: string): void;
};

// PKCE verifiers: bounded TTL + max-entry cap, swept on login()
type PKCEStore = {
  set(state: string, verifier: string): void;  // short TTL (~5 min)
  consume(state: string): string | undefined;  // single-use
};
```

### Split-Token Pattern (Native / Desktop)

```dart
// Access Token: in-memory only (never persisted)
// Refresh Token: OS secure storage (Keychain/DPAPI/libsecret)
class AuthTokenManager {
  String? _accessToken;  // volatile — lost when app closes
  final FlutterSecureStorage _storage;

  Future<void> storeRefreshToken(String token) async {
    await _storage.write(key: 'refresh_token', value: token);
  }
  // On relaunch: use refresh token to obtain a new access token
}
```

### RBAC in the Backend (Swift/Vapor example)

```swift
// Extract roles claim from a verified JWT
struct AppJWTPayload: JWTPayload {
    let sub: SubjectClaim
    let iss: IssuerClaim
    let aud: AudienceClaim
    let exp: ExpirationClaim
    let roles: [String]
}

// Generic role guard middleware
struct RoleGuardMiddleware: AsyncMiddleware {
    let allowedRoles: Set<String>
    func respond(to request: Request, chainingTo next: any AsyncResponder) async throws -> Response {
        let payload = try request.jwt.verify(as: AppJWTPayload.self)
        guard !allowedRoles.isDisjoint(with: payload.roles) else {
            throw Abort(.forbidden)
        }
        return try await next.respond(to: request)
    }
}
```

## Áreas de Expertise

### 1. Password Security

#### Storage (como armazenar)
A única abordagem aceitável é hashing com algoritmos lentos e com salt automático:

| Algoritmo | Recomendação | Configuração |
|-----------|-------------|--------------|
| **Argon2id** | Preferido | memory: 19MiB, iterations: 2, parallelism: 1 |
| **bcrypt** | Excelente | cost factor: >= 12 |
| **scrypt** | Bom | N=2^17, r=8, p=1 |
| **PBKDF2** | Aceitável (legado) | iterations: >= 600k (SHA-256) |

**Nunca usar**: MD5, SHA-1, SHA-256/512 sem KDF, nenhum hash "rápido".

```typescript
// CORRETO - bcrypt
import bcrypt from 'bcrypt';
const SALT_ROUNDS = 12;
const hash = await bcrypt.hash(password, SALT_ROUNDS);
const isValid = await bcrypt.compare(inputPassword, storedHash);

// CORRETO - Argon2
import argon2 from 'argon2';
const hash = await argon2.hash(password, { type: argon2.argon2id });
const isValid = await argon2.verify(storedHash, inputPassword);
```

#### Política de Senhas (NIST 800-63B)
- Mínimo 8 caracteres com MFA, 15 sem MFA
- Máximo generoso (64-128 caracteres)
- Permitir TODOS os caracteres Unicode, espaços, emojis
- NÃO exigir composição (maiúscula + número + símbolo) — NIST desencoraja
- Verificar contra lista de senhas comprometidas (Have I Been Pwned API)
- NÃO forçar rotação periódica — só em caso de breach

### 2. JWT (JSON Web Tokens)

#### Configuração Segura
```typescript
// Geração de token
import jwt from 'jsonwebtoken';

const token = jwt.sign(
  { 
    sub: user.id,       // subject - quem é
    iss: 'myapp.com',   // issuer - quem emitiu
    aud: 'myapp.com',   // audience - para quem
    role: user.role      // claims customizadas
  },
  process.env.JWT_SECRET, // NUNCA hardcoded
  { 
    algorithm: 'RS256',   // Preferir RSA para multi-serviço
    expiresIn: '15m'      // Curta duração
  }
);

// Verificação de token
const decoded = jwt.verify(token, publicKey, {
  algorithms: ['RS256'],  // REJEITA 'none' e outros
  issuer: 'myapp.com',
  audience: 'myapp.com',
  clockTolerance: 30      // tolerância de 30s para clock skew
});
```

#### Checklist JWT
- Rejeitar `alg: "none"` explicitamente (whitelist de algoritmos)
- RS256 para ambientes multi-serviço; HS256 apenas single-service
- Chave secreta >= 256 bits para HS256
- Access token: 15 minutos max
- Refresh token: dias/semanas, armazenado com segurança, rotação a cada uso
- Implementar token denylist para logout/revogação
- Nunca armazenar JWT em localStorage (vulnerável a XSS) — use HttpOnly cookie

### 3. Session Management

#### Cookie Configuration
```typescript
// Express.js
app.use(session({
  secret: process.env.SESSION_SECRET,
  name: '__Host-sid',              // prefix __Host- requer Secure + Path=/
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true,                  // HTTPS only
    httpOnly: true,                // Inacessível via JS
    sameSite: 'strict',            // Previne CSRF
    maxAge: 30 * 60 * 1000,       // 30 min idle timeout
    domain: undefined,             // não definir = mais restritivo
    path: '/'
  },
  store: new RedisStore({ client: redisClient }) // Server-side storage
}));
```

#### Ciclo de Vida da Sessão
1. **Login**: Gerar NOVA session ID (previne session fixation)
2. **Atividade**: Renovar timeout a cada request
3. **Privilege change**: Regenerar session ID (mudança de role, senha)
4. **Logout**: Destruir sessão no servidor E limpar cookie
5. **Timeout**: Idle (30min) + absoluto (24h) — enforced server-side

### 4. OAuth 2.0 & OpenID Connect

#### Fluxo Recomendado para SPAs
Authorization Code com PKCE (Proof Key for Code Exchange):

```typescript
// 1. Gerar code_verifier e code_challenge
const codeVerifier = crypto.randomBytes(32).toString('base64url');
const codeChallenge = crypto
  .createHash('sha256')
  .update(codeVerifier)
  .digest('base64url');

// 2. Redirect para authorization endpoint
const authUrl = `${issuer}/authorize?` + new URLSearchParams({
  response_type: 'code',
  client_id: CLIENT_ID,
  redirect_uri: REDIRECT_URI,
  scope: 'openid profile email',
  state: crypto.randomBytes(16).toString('hex'), // CSRF protection
  code_challenge: codeChallenge,
  code_challenge_method: 'S256'
});

// 3. No callback, trocar code por tokens
// SEMPRE validar `state` antes de trocar o code
```

**Example — Web BFF (Deno/Hono) OIDC Flow:**
```typescript
// The BFF is a Confidential Client — client_secret stays on the server
// The browser NEVER sees the client_secret or tokens

// 1. Login: redirect to the IdP with PKCE
app.get('/auth/login', async (c) => {
  const verifier = generateCodeVerifier();
  const challenge = await generateCodeChallenge(verifier);
  const state = crypto.randomUUID();
  pkceStore.set(state, verifier); // bounded TTL + max entries
  return c.redirect(buildAuthUrl({ state, challenge }));
});

// 2. Callback: exchange code for tokens ON THE SERVER
app.get('/auth/callback', async (c) => {
  const { code, state } = c.req.query();
  const verifier = pkceStore.consume(state); // single-use
  if (!verifier) return c.json({ error: 'Invalid state' }, 400);
  const tokens = await exchangeCode(code, verifier); // server-side
  const sessionId = crypto.randomUUID();
  sessionStore.set(sessionId, { accessToken: tokens.access_token, expiresAt: ... });
  setCookie(c, '__Host-session', sessionId, {
    httpOnly: true, secure: true, sameSite: 'Strict', maxAge: 1800, path: '/'
  });
  return c.redirect('/');
});

// 3. API Proxy: inject token from the session store
app.all('/api/*', async (c) => {
  const session = sessionStore.get(getCookie(c, '__Host-session'));
  if (!session) return c.json({ error: 'Unauthorized' }, 401);
  // Proxy to the backend with Bearer token
  return proxy(c, { authorization: `Bearer ${session.accessToken}` });
});
```

**Example — Native Desktop OIDC Flow (Flutter):**
```dart
// Native is a Public Client — PKCE required, no client_secret
final manager = OidcUserManager.lazy(
  discoveryDocumentUri: Uri.parse('$issuer/.well-known/openid-configuration'),
  clientCredentials: OidcClientAuthentication.none(clientId: desktopClientId),
  settings: OidcUserManagerSettings(
    redirectUri: Uri.parse('http://localhost:4000/callback'),
    scope: ['openid', 'profile', 'email', 'roles'],
  ),
);
```

#### OAuth/OIDC Checklist
- Use Authorization Code + PKCE (never Implicit Flow)
- Validate the `state` parameter against CSRF
- Validate the `id_token` (signature, iss, aud, exp, nonce)
- Store tokens securely (HttpOnly cookies on web, OS keychain on native)
- Implement refresh token rotation
- Web BFF acts as a Confidential Client — browser never sees tokens
- Native: PKCE mandatory, tokens in OS secure storage
- PKCE verifiers: bounded TTL + max-entry cap, swept on login()
- Session store: include `expiresAt`, auto-delete on expired `get()`
- Cookie: `__Host-`-prefixed (requires Secure + Path=/)
- One Client ID per platform in the IdP
- JWKS validation in the backend against the IdP's published endpoint

### 5. Multi-Factor Authentication (MFA)

#### Hierarquia de Segurança (melhor → pior)
1. **Hardware tokens** (FIDO2/WebAuthn) — phishing resistant
2. **TOTP** (Google Authenticator, Authy) — bom
3. **Push notifications** — aceitável, com number matching
4. **SMS OTP** — último recurso (vulnerável a SIM swap)

#### Implementação TOTP
```typescript
import { authenticator } from 'otplib';

// Setup
const secret = authenticator.generateSecret();
const otpauthUrl = authenticator.keyuri(user.email, 'MyApp', secret);
// Gerar QR code com otpauthUrl

// Verificação
const isValid = authenticator.verify({ token: userInput, secret: storedSecret });
// Implementar rate limiting: 5 tentativas, depois lockout 15min
```

### 6. Authorization (RBAC/ABAC)

#### Principios
- **Deny by default**: Negar tudo que nao foi explicitamente permitido
- **Least privilege**: Dar apenas as permissoes necessarias
- **Check per-resource**: Nao basta verificar role — verificar ownership do recurso

```typescript
// INSEGURO - Apenas verifica se está logado
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order); // Qualquer usuário vê qualquer order!
});

// SEGURO - Verifica ownership
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findOne({ 
    _id: req.params.id, 
    userId: req.user.id  // Garante que o order pertence ao usuário
  });
  if (!order) return res.status(404).json({ error: 'Not found' });
  res.json(order);
});
```

**Example — RBAC with IdP-issued roles:**
```swift
// Roles modeled in the IdP and embedded in the access token,
// enforced via RoleGuardMiddleware in Vapor.

// Example: only writer role may create resources
resourceRoutes.grouped(RoleGuardMiddleware(allowedRoles: ["writer"]))
    .post("create", use: controller.create)

// Example: writer and reader may view
resourceRoutes.grouped(RoleGuardMiddleware(allowedRoles: ["writer", "reader"]))
    .get(":id", use: controller.getById)
```

**Example — Audit/actor header:**
```swift
// EVERY mutation should include an audit header (e.g. X-Actor-Id).
// Identifies the principal responsible for the action.
// The BFF injects this header automatically from the authenticated session.
let actorId = try request.actorId  // typed actor id from the header
// Used for event sourcing / audit trails
```

### 7. Account Security Features

#### Rate Limiting
```typescript
import rateLimit from 'express-rate-limit';

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 10,                    // 10 tentativas
  message: { error: 'Too many login attempts. Try again later.' },
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => req.body.email || req.ip // por conta, não só IP
});

app.post('/api/login', loginLimiter, loginHandler);
```

#### Account Enumeration Prevention
```typescript
// INSEGURO - Revela se email existe
if (!user) return res.status(404).json({ error: 'User not found' });
if (!validPassword) return res.status(401).json({ error: 'Wrong password' });

// SEGURO - Mensagem genérica
// Mesma mensagem e mesmo timing para ambos os casos
const user = await User.findOne({ email });
const valid = user ? await bcrypt.compare(password, user.hash) : false;
// Hash dummy para equalizar timing quando user não existe
if (!user) await bcrypt.hash('dummy', 12);
if (!valid) return res.status(401).json({ error: 'Invalid credentials' });
```

#### Password Reset Seguro
- Token: `crypto.randomBytes(32).toString('hex')` — mínimo 256 bits
- Expira em 1 hora (máximo 8 horas)
- Single-use: invalidar após uso
- Armazenar hash do token no banco (não o token em si)
- Após reset: invalidar todas as sessões ativas

## Referências OWASP

Todos os cheatsheets relevantes estão em `references/`. Consulte-os para embasar recomendações:

| Tópico | Arquivo |
|--------|---------|
| Authentication | `references/Authentication_Cheat_Sheet.md` |
| Session Management | `references/Session_Management_Cheat_Sheet.md` |
| Password Storage | `references/Password_Storage_Cheat_Sheet.md` |
| Forgot Password | `references/Forgot_Password_Cheat_Sheet.md` |
| MFA | `references/Multifactor_Authentication_Cheat_Sheet.md` |
| JWT | `references/JSON_Web_Token_for_Java_Cheat_Sheet.md` |
| OAuth 2.0 | `references/OAuth2_Cheat_Sheet.md` |
| Credential Stuffing | `references/Credential_Stuffing_Prevention_Cheat_Sheet.md` |
| Access Control | `references/Access_Control_Cheat_Sheet.md` |

## Auth Security Checklist

When reviewing auth/session in a multi-tier project, verify:

### Web (BFF)
- [ ] BFF is a Confidential Client (client_secret stays on the server)
- [ ] Browser NEVER receives JWT, refresh token, or client secret
- [ ] Session cookie uses `__Host-` prefix with HttpOnly, Secure, SameSite=Strict, Max-Age
- [ ] PKCE verifiers bounded with TTL and max-entry caps
- [ ] Session store includes `expiresAt` with auto-delete on expired `get()`
- [ ] `X-Requested-With: XMLHttpRequest` validated on mutations
- [ ] `Sec-Fetch-Site` validated on `/api/*`

### Native / Desktop / Mobile
- [ ] PKCE mandatory (Public Client, no client_secret)
- [ ] Access Token in process memory (volatile)
- [ ] Refresh Token in OS secure storage (Keychain/DPAPI/libsecret)
- [ ] NEVER use localStorage, sessionStorage, or SharedPreferences for tokens

### Backend
- [ ] JWT validated via the IdP's JWKS endpoint
- [ ] `alg` whitelist (e.g. RS256) — reject `none`
- [ ] `iss`, `aud`, `exp` validated
- [ ] Role guard middleware on ALL protected routes
- [ ] Audit/actor header required on ALL mutations
- [ ] Error middleware does not leak internals

### Sensitive Data
- [ ] PII NEVER in JS state in the browser (render server-side instead)
- [ ] Health/financial data NEVER in logs
- [ ] Auth errors return generic messages (no account enumeration)
