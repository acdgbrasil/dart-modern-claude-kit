---
name: api-security-guardian
description: |
  Especialista em segurança de APIs (REST, GraphQL, WebSocket, gRPC) para aplicações Web/JS/TS/React/Node.js, Dart/Flutter, Swift/Vapor e Deno/Hono. Cobre validação de input, rate limiting, CORS, headers de segurança, proteção contra abuse, e design seguro de APIs. Use esta skill SEMPRE que o usuário mencionar: API, endpoint, REST, GraphQL, WebSocket, gRPC, CORS, rate limiting, API key, throttling, API gateway, middleware de segurança, Express.js security, Fastify security, NestJS security, Vapor security, Hono middleware, BFF proxy, proteção de API, abuso de API, ou quando estiver desenhando/revisando endpoints de API. Acione também quando o usuário perguntar sobre headers HTTP de segurança, Content-Security-Policy, proteção de rotas no backend, Zitadel OIDC, X-Actor-Id, ou __Host-session cookie.
---

# API Security Guardian — Proteção de APIs Web

Você é um especialista em API Security com foco em proteger endpoints REST, GraphQL e WebSocket contra ataques comuns e abuso. Seu conhecimento abrange desde design seguro até hardening em produção.

## Context

This skill assumes a multi-tier architecture (browser/app -> BFF/API gateway -> backend). Adapt the OWASP guidance below to your project's specific framework, IdP, and trust boundaries. Common shapes:

```
Browser/App (no secrets, no tokens visible to JS)
  v
BFF / API gateway (session cookie or bearer, CSRF + Fetch Metadata, domain validation)
  v
Backend service (JWT validation, RBAC, audit headers)
```

Key dimensions to map for your stack:
- Where does authentication terminate? (BFF vs backend)
- What headers does each tier expect? (e.g. an actor/audit header on mutations, `X-Requested-With` for CSRF)
- What request-body validation runs at the BFF before proxying?
- What CSP / cookie / fetch-metadata policy protects the browser tier?

## Framework de Análise

Ao analisar ou projetar uma API, avalie estas 8 dimensões:

### 1. Transport Security (HTTPS)

Toda comunicação deve ser criptografada. Sem exceções.

```typescript
// Express.js - Forçar HTTPS
app.use((req, res, next) => {
  if (req.header('x-forwarded-proto') !== 'https') {
    return res.redirect(301, `https://${req.hostname}${req.url}`);
  }
  next();
});

// HSTS Header - diz ao browser para SEMPRE usar HTTPS
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});
```

**Example — Deno/Hono BFF:**
```typescript
// Hono middleware - Security Headers
app.use('*', async (c, next) => {
  await next();
  c.header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  c.header('X-Content-Type-Options', 'nosniff');
  c.header('X-Frame-Options', 'DENY');
});
```

**Example — Swift/Vapor Backend:**
```swift
// Vapor middleware
struct SecurityHeadersMiddleware: AsyncMiddleware {
    func respond(to request: Request, chainingTo next: any AsyncResponder) async throws -> Response {
        let response = try await next.respond(to: request)
        response.headers.add(name: .strictTransportSecurity, value: "max-age=31536000; includeSubDomains")
        response.headers.add(name: .xContentTypeOptions, value: "nosniff")
        response.headers.add(name: .xFrameOptions, value: "DENY")
        return response
    }
}
```

### 2. Input Validation & Sanitization

Toda entrada deve ser validada antes de qualquer processamento.

```typescript
// Schema validation com Zod (recomendado para TS)
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100).trim(),
  email: z.string().email().max(254).toLowerCase(),
  age: z.number().int().min(13).max(150).optional(),
});

// Middleware de validação
function validate(schema: z.ZodSchema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ 
        error: 'Validation failed',
        details: result.error.issues // em dev; remover em prod
      });
    }
    req.body = result.data; // dados validados e tipados
    next();
  };
}

app.post('/api/users', validate(CreateUserSchema), createUserHandler);
```

**Example — Domain Validation with Smart Constructors (Deno/Hono):**
```typescript
// Branded Types + Smart Constructors returning Result<T, E>
type DocumentId = Brand<string, 'DocumentId'>;
const DocumentId = (raw: string): Result<DocumentId, 'INVALID_DOCUMENT'> => {
  const cleaned = raw.replace(/\D/g, '');
  if (!isValidDocument(cleaned)) {
    return err('INVALID_DOCUMENT');
  }
  return ok(cleaned as DocumentId);
};

// Validate at the BFF BEFORE proxying to the backend
app.post('/api/resource', async (c) => {
  const body = await c.req.json();
  const docResult = DocumentId(body.documentId);
  if (!docResult.ok) {
    return c.json({ error: docResult.error }, 400);
  }
  // Only after domain validation succeeds, proxy to the backend
});
```

**Example — Swift/Vapor cross-field validator:**
```swift
// Inter-field validation rules (e.g. mutually exclusive flags)
struct CrossValidator {
    static func validate(_ dto: ResourceDTO) -> [ValidationError] {
        var errors: [ValidationError] = []
        if dto.flagA == true && dto.flagB == true {
            errors.append(.incompatibleFields("flagA", "flagB"))
        }
        return errors
    }
}
```

**Regras de ouro:**
- Request size limits: `express.json({ limit: '10kb' })` para JSON, maior para uploads
- Rejeitar Content-Types inesperados (HTTP 415)
- Validar path params, query params, headers — nao so body
- Para arrays: limitar `maxItems`; para strings: limitar `maxLength`
- Prefira validacao de dominio via smart constructors retornando `Result<T,E>` — erros sao valores, nunca exceptions soltas no dominio/aplicacao

### 3. Authentication & API Keys

```typescript
// API Key via header (nunca na URL)
// INSEGURO: GET /api/data?apiKey=secret123
// SEGURO: Authorization: Bearer <token>

// Middleware de autenticação
async function requireApiKey(req, res, next) {
  const key = req.header('Authorization')?.replace('Bearer ', '');
  if (!key) return res.status(401).json({ error: 'API key required' });
  
  // Compare com timing-safe para prevenir timing attacks
  const storedKey = await getApiKeyFromDB(key);
  if (!storedKey || !crypto.timingSafeEqual(
    Buffer.from(key), Buffer.from(storedKey.value)
  )) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  req.apiClient = storedKey.client;
  next();
}
```

### 4. Rate Limiting & Throttling

Protege contra abuso, brute force, e DoS.

```typescript
import rateLimit from 'express-rate-limit';

// Rate limit global
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 min
  max: 100,                   // 100 requests por window
  standardHeaders: true,      // RateLimit-* headers
  legacyHeaders: false,
  message: { error: 'Too many requests' }
});

// Rate limit específico para endpoints sensíveis
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,
  keyGenerator: (req) => req.body.email || req.ip,
  handler: (req, res) => {
    res.status(429).json({ error: 'Too many attempts. Try again later.' });
  }
});

app.use('/api/', globalLimiter);
app.post('/api/auth/login', authLimiter);
```

**Para GraphQL**: limite por complexidade da query, não por request count:
```typescript
// Limitar profundidade e complexidade de queries GraphQL
const depthLimit = require('graphql-depth-limit');
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  validationRules: [
    depthLimit(5),                    // máximo 5 níveis de nesting
    createComplexityLimitRule(1000),   // custo máximo da query
  ],
  introspection: false,               // DESABILITAR em produção
});
```

### 5. CORS (Cross-Origin Resource Sharing)

CORS mal configurado é uma das falhas mais comuns.

```typescript
import cors from 'cors';

// INSEGURO - Aceita qualquer origem
app.use(cors()); // NÃO FAZER

// SEGURO - Whitelist de origens
const allowedOrigins = [
  'https://myapp.com',
  'https://admin.myapp.com'
];

app.use(cors({
  origin: (origin, callback) => {
    // Permitir requests sem origin (mobile apps, curl)
    if (!origin) return callback(null, true);
    if (allowedOrigins.includes(origin)) {
      return callback(null, true);
    }
    callback(new Error('Not allowed by CORS'));
  },
  credentials: true,          // se precisa enviar cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 86400               // cache preflight por 24h
}));
```

**Regras:**
- NUNCA use `origin: '*'` com `credentials: true`
- NUNCA reflita o Origin header do request no response sem validar
- Seja específico: liste apenas os domínios que precisam de acesso

### 6. HTTP Security Headers

```typescript
// Helmet.js — aplica headers de segurança automaticamente
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'none'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", "https:"],
      fontSrc: ["'self'"],
      connectSrc: ["'self'"],
      frameAncestors: ["'none'"],
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true },
  noSniff: true,
  frameguard: { action: 'deny' },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));

// Remover X-Powered-By (helmet faz isso, mas confirme)
app.disable('x-powered-by');

// Para APIs que retornam dados sensíveis
app.use((req, res, next) => {
  res.setHeader('Cache-Control', 'no-store');
  res.setHeader('Pragma', 'no-cache');
  next();
});
```

**Example — Deno/Hono Security Headers with CSP Nonce:**
```typescript
// src/middleware/security_headers.ts
app.use('*', async (c, next) => {
  const nonce = crypto.getRandomValues(new Uint8Array(16));
  const nonceB64 = btoa(String.fromCharCode(...nonce));
  c.set('cspNonce', nonceB64);

  await next();

  c.header('Content-Security-Policy',
    `default-src 'none'; ` +
    `script-src 'self' 'nonce-${nonceB64}'; ` +
    `style-src 'self' 'nonce-${nonceB64}'; ` +
    `img-src 'self' https:; ` +
    `connect-src 'self'; ` +
    `frame-ancestors 'none'`
  );
  c.header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  c.header('X-Content-Type-Options', 'nosniff');
  c.header('X-Frame-Options', 'DENY');
  c.header('Referrer-Policy', 'strict-origin-when-cross-origin');
});
// Nonce usado em SSR: <Style nonce={c.get('cspNonce')} />
```

### 7. Error Handling & Information Disclosure

APIs não devem vazar informações internas.

```typescript
// Error handler global (Express)
app.use((err, req, res, next) => {
  // Log completo internamente
  logger.error({
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
    ip: req.ip,
    userId: req.user?.id
  });

  // Resposta genérica para o cliente
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    error: statusCode === 500 
      ? 'Internal server error'  // Nunca expor detalhes
      : err.message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});
```

**Nunca expor:**
- Stack traces em produção
- Mensagens de erro de banco de dados
- Paths internos do servidor
- Versões de software/framework
- Queries SQL/NoSQL que falharam

### 8. GraphQL-Specific Security

```typescript
// Checklist de segurança GraphQL
const securityChecklist = {
  introspection: false,           // Desabilitar em produção
  depthLimit: 5,                  // Máximo 5 níveis
  complexityLimit: 1000,          // Custo máximo
  batchingLimit: 5,               // Máximo 5 queries por batch
  fieldLevelAuth: true,           // Autorização por campo
  persistedQueries: true,         // Apenas queries pré-aprovadas
  rateLimiting: true,             // Por IP/user/query
  inputValidation: true,          // Validar todos os arguments
  errorMasking: true,             // Não vazar erros internos
};
```

### 9. WebSocket Security

```typescript
// Checklist WebSocket
const wsChecklist = {
  // Autenticação no handshake (não depois)
  authOnConnect: true,
  // Validar Origin header
  validateOrigin: true,
  // Rate limit por conexão
  messageRateLimit: '100/min',
  // Validar e sanitizar TODA mensagem recebida
  inputValidation: true,
  // Timeout para conexões idle
  idleTimeout: '5min',
  // Limitar tamanho de mensagem
  maxMessageSize: '1mb',
  // Usar WSS (WebSocket Secure) - nunca WS
  requireTLS: true,
};
```

## Common BFF Security Patterns

### Fetch Metadata Validation (BFF)
Validate `Sec-Fetch-Site` on `/api/*` to reject unauthorized cross-origin requests:
```typescript
// src/middleware/fetch_metadata.ts
app.use('/api/*', async (c, next) => {
  const site = c.req.header('Sec-Fetch-Site');
  if (site && site !== 'same-origin' && site !== 'none') {
    return c.json({ error: 'Forbidden' }, 403);
  }
  await next();
});
```

### X-Requested-With CSRF Protection
```typescript
// POST/PUT/DELETE to the BFF must include X-Requested-With
app.use('/api/*', async (c, next) => {
  if (['POST', 'PUT', 'DELETE'].includes(c.req.method)) {
    if (c.req.header('X-Requested-With') !== 'XMLHttpRequest') {
      return c.json({ error: 'CSRF validation failed' }, 403);
    }
  }
  await next();
});
```

### Audit Header (Backend)
```swift
// Vapor - extract an actor/audit header on every mutation
extension Request {
    var actorId: ActorId {
        get throws {
            guard let raw = headers.first(name: "X-Actor-Id"),
                  let id = ActorId(raw) else {
                throw Abort(.badRequest, reason: "X-Actor-Id header required")
            }
            return id
        }
    }
}
```

### Domain Validation at the BFF (Result Pattern)
The BFF should validate domain types via smart constructors BEFORE proxying to the backend. This guarantees malformed requests never reach the backend.

### Client-Side Service Security (Flutter)
```dart
// base-client.ts equivalent in Dart
class BaseClient {
  final Dio _dio;
  BaseClient(this._dio) {
    _dio.options.headers['X-Requested-With'] = 'XMLHttpRequest';
    _dio.options.extra['withCredentials'] = true; // same-origin cookies
  }
  // Returns Result<T, E> — never throws
}
```

## When the User Is Designing a New API

Guide them through these decisions:
1. **Authentication**: JWT vs Session vs API Key — which fits the threat model and trust topology?
2. **Authorization**: RBAC vs ABAC — what granularity is required?
3. **Rate limiting**: Global vs per-endpoint vs per-user?
4. **Versioning**: URL (`/v1/`) vs Header — security implications of each?
5. **Response format**: Avoid leaking internals — design explicit DTOs and a uniform error envelope with structured codes.
6. **Logging**: What is safe to log for attack detection without violating privacy regulations (LGPD/HIPAA/etc.)?

## Referências OWASP

Todos os cheatsheets relevantes estão em `references/`. Consulte-os para embasar cada recomendação:

| Tópico | Arquivo |
|--------|---------|
| REST Security | `references/REST_Security_Cheat_Sheet.md` |
| REST Assessment | `references/REST_Assessment_Cheat_Sheet.md` |
| GraphQL | `references/GraphQL_Cheat_Sheet.md` |
| WebSocket | `references/WebSocket_Security_Cheat_Sheet.md` |
| HTTP Headers | `references/HTTP_Headers_Cheat_Sheet.md` |
| CSP | `references/Content_Security_Policy_Cheat_Sheet.md` |
| CSRF | `references/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md` |
| AJAX | `references/AJAX_Security_Cheat_Sheet.md` |
| Microservices | `references/Microservices_Security_Cheat_Sheet.md` |

## Quick API Security Checklist

When reviewing APIs in a multi-tier project, verify:
- [ ] Browser NEVER sees JWT, refresh token, client secret, or backend URL
- [ ] Session cookie uses `__Host-` prefix with HttpOnly, Secure, SameSite=Strict
- [ ] `X-Requested-With: XMLHttpRequest` validated on BFF mutations
- [ ] `Sec-Fetch-Site` validated on `/api/*`
- [ ] An audit/actor header is present on ALL mutations to the backend
- [ ] Domain validation (smart constructors) at the BFF BEFORE proxying
- [ ] CSP nonce per request (never `unsafe-inline`)
- [ ] PKCE verifiers bounded with TTL and max-entry caps
- [ ] Errors returned as `Result<T,E>` (no exceptions leaking from domain/application)
- [ ] Uniform response envelope with structured error codes
- [ ] Sensitive PII rendered server-side, NEVER serialized into browser-side JS state
