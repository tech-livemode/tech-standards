# Padrões de Desenvolvimento - LiveContent

## 📋 Stack Tecnológica

### Obrigatório
- **Framework**: Next.js (App Router)
- **Biblioteca UI**: React 18+
- **Componentes**: Radix UI / shadcn/ui
- **Banco de Dados**: PostgreSQL (via Supabase ou direto)
- **Gerenciador de Pacotes**: pnpm ou bun (preferência: pnpm)
- **Linters e Formatters**: Configurados e padronizados

### Recomendações
- **TypeScript**: Sempre habilitado com strict mode
- **Tailwind CSS**: Para estilização
- **Zod**: Para validação de schemas

---

## 🌐 APIs REST - Boas Práticas

### 1. Estrutura de URLs

```
✅ BOM:
GET    /api/matches           # Lista todas as partidas
GET    /api/matches/:id        # Busca partida específica
POST   /api/matches            # Cria nova partida
PUT    /api/matches/:id        # Atualiza partida completa
PATCH  /api/matches/:id        # Atualiza partida parcialmente
DELETE /api/matches/:id        # Remove partida

❌ EVITAR:
GET /api/getMatches
POST /api/createMatch
GET /api/match/:id/delete
```

### 2. Versionamento

```
/api/v1/matches
/api/v2/matches
```

Ou via header:
```
Accept: application/vnd.api+json;version=1
```

### 3. Paginação

```
GET /api/matches?page=1&limit=20&offset=0

Resposta:
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### 4. Filtros e Busca

```
GET /api/matches?status=live&tournament=brasileirao&sort=date:desc
GET /api/matches?search=flamengo
```

### 5. Relacionamentos

```
GET /api/matches/:id/goals        # Nested resource
GET /api/matches/:id?include=goals,teams  # Query parameter
```

---

## 🔢 Protocolo HTTP - Status Codes e Métodos

### Status Codes

#### 2xx - Sucesso
- **200 OK**: GET, PUT, PATCH bem-sucedidos
- **201 Created**: POST que cria recurso
- **204 No Content**: DELETE ou PUT sem corpo na resposta

#### 4xx - Erro do Cliente
- **400 Bad Request**: Validação falhou, formato inválido
- **401 Unauthorized**: Não autenticado
- **403 Forbidden**: Autenticado, mas sem permissão
- **404 Not Found**: Recurso não existe
- **409 Conflict**: Conflito (ex: duplicação)
- **422 Unprocessable Entity**: Semântica inválida (validação de negócio)
- **429 Too Many Requests**: Rate limit excedido

#### 5xx - Erro do Servidor
- **500 Internal Server Error**: Erro inesperado
- **502 Bad Gateway**: Problema no gateway/proxy
- **503 Service Unavailable**: Serviço temporariamente indisponível

### Métodos HTTP

| Método | Uso | Idempotente | Body |
|--------|-----|-------------|------|
| GET | Buscar recursos | ✅ Sim | ❌ Não |
| POST | Criar recursos | ❌ Não | ✅ Sim |
| PUT | Substituir recurso completo | ✅ Sim | ✅ Sim |
| PATCH | Atualizar parcialmente | ❌ Não* | ✅ Sim |
| DELETE | Remover recurso | ✅ Sim | ❌ Não |

*PATCH pode ser idempotente se bem implementado

### Headers Importantes

#### Request
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
X-Request-ID: <uuid>  # Para rastreamento
```

#### Response
```
Content-Type: application/json
X-Request-ID: <uuid>  # Echo do request ID
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
Cache-Control: public, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89b4"
```

---

## ⚠️ Tratamento de Erros - Padrão

### Estrutura de Resposta de Erro

```typescript
// Erro padrão
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Campos obrigatórios não preenchidos",
    "details": {
      "field": "email",
      "reason": "Email inválido"
    },
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### Códigos de Erro Padronizados

```typescript
// Erros de validação
VALIDATION_ERROR
MISSING_REQUIRED_FIELD
INVALID_FORMAT

// Erros de autenticação/autorização
UNAUTHORIZED
FORBIDDEN
TOKEN_EXPIRED
INVALID_CREDENTIALS

// Erros de recursos
NOT_FOUND
ALREADY_EXISTS
CONFLICT

// Erros de servidor
INTERNAL_ERROR
SERVICE_UNAVAILABLE
DATABASE_ERROR
EXTERNAL_API_ERROR
```

### Implementação em Next.js

```typescript
// lib/api-error.ts
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string,
    public details?: any
  ) {
    super(message)
    this.name = 'ApiError'
  }
}

// Uso em API routes
export async function GET(request: Request) {
  try {
    // ... lógica
    return NextResponse.json({ data: result }, { status: 200 })
  } catch (error) {
    if (error instanceof ApiError) {
      return NextResponse.json(
        {
          error: {
            code: error.code,
            message: error.message,
            details: error.details,
            requestId: generateRequestId(),
            timestamp: new Date().toISOString()
          }
        },
        { status: error.statusCode }
      )
    }

    // Erro não esperado
    logger.error('Unexpected error', { error })
    return NextResponse.json(
      {
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Erro interno do servidor',
          requestId: generateRequestId(),
          timestamp: new Date().toISOString()
        }
      },
      { status: 500 }
    )
  }
}
```

### Tratamento no Cliente

```typescript
// lib/api-client.ts
export async function apiRequest<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  try {
    const response = await fetch(url, options)
    const data = await response.json()

    if (!response.ok) {
      throw new ApiError(
        response.status,
        data.error?.code || 'UNKNOWN_ERROR',
        data.error?.message || 'Erro desconhecido',
        data.error?.details
      )
    }

    return data
  } catch (error) {
    // Log e re-throw
    logger.error('API request failed', { url, error })
    throw error
  }
}
```

---

## 📝 Logging

### Estrutura de Logs

```typescript
// lib/logger.ts
import pino from 'pino'

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  base: {
    env: process.env.NODE_ENV,
    service: 'live-content',
  },
})

// Uso
logger.info('User logged in', { userId: '123', email: 'user@example.com' })
logger.error('Failed to process payment', { error, orderId: '456' })
logger.warn('Rate limit approaching', { userId: '123', requests: 95 })
```

### Níveis de Log

- **ERROR**: Erros que precisam atenção imediata
- **WARN**: Situações anômalas que não quebram o fluxo
- **INFO**: Eventos importantes do sistema
- **DEBUG**: Informações detalhadas para debugging
- **TRACE**: Logs muito verbosos

### Contexto Obrigatório

- Request ID (para correlação)
- User ID (quando aplicável)
- Timestamp
- Nível de severidade
- Mensagem clara

### Exemplo de Log Estruturado

```json
{
  "level": "error",
  "time": "2024-01-15T10:30:00.000Z",
  "service": "live-content",
  "env": "production",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "user-123",
  "message": "Failed to upload image",
  "error": {
    "code": "CLOUDINARY_ERROR",
    "message": "Upload failed",
    "stack": "..."
  },
  "context": {
    "fileSize": 5242880,
    "fileType": "image/jpeg",
    "endpoint": "/api/upload"
  }
}
```

---

## 🚨 Crash Reports e Monitoramento

### Requisitos

- Captura automática de erros não tratados
- Stack traces completos
- Contexto (user, request, environment)
- Agregação e deduplicação
- Alertas para erros críticos

### Soluções Recomendadas

1. **Sentry** (recomendado)
2. **LogRocket**
3. **Rollbar**
4. **Bugsnag**

### Implementação Básica (Sentry)

```typescript
// lib/monitoring.ts
import * as Sentry from '@sentry/nextjs'

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  beforeSend(event, hint) {
    // Filtrar informações sensíveis
    if (event.request) {
      delete event.request.cookies
      delete event.request.headers?.authorization
    }
    return event
  },
})

// Uso
try {
  // código
} catch (error) {
  Sentry.captureException(error, {
    tags: { section: 'upload' },
    extra: { fileSize, fileType },
  })
  throw error
}
```

---

## 💡 Sugestões Adicionais

### 1. Testes
- **Unit tests**: Jest ou Vitest
- **Integration tests**: Para APIs
- **E2E**: Playwright ou Cypress
- **Coverage mínimo**: 70%

### 2. Documentação de APIs
- OpenAPI/Swagger
- Exemplos de requests/responses
- Manter atualizada

### 3. Rate Limiting
- Por usuário/IP
- Diferentes limites por endpoint
- Headers informativos

### 4. Validação de Dados
- Zod para schemas
- Validação no cliente e servidor
- Mensagens de erro claras

### 5. Segurança
- Variáveis de ambiente para secrets
- Validação de inputs (sanitização)
- CORS configurado
- HTTPS obrigatório em produção
- Headers de segurança (helmet.js)

### 6. Performance
- Cache quando apropriado
- Lazy loading de componentes
- Otimização de imagens
- Bundle analysis

### 7. CI/CD
- Lint e testes no pipeline
- Builds automáticos
- Deploy automatizado
- Rollback rápido

### 8. Versionamento
- Conventional Commits
- Semantic Versioning
- Changelog mantido

### 9. Code Review
- PRs obrigatórios
- Checklist de revisão
- Aprovação antes de merge

### 10. Ambiente de Desenvolvimento
- Docker Compose para serviços locais
- Scripts padronizados (dev, build, test)
- README com setup

---

## ✅ Checklist de Implementação

- [ ] Configurar linters (ESLint + Prettier)
- [ ] Configurar formatters (Prettier)
- [ ] Implementar padrão de tratamento de erros
- [ ] Configurar logging (Pino ou Winston)
- [ ] Integrar crash reporting (Sentry)
- [ ] Criar utilitários de API (api-client, api-error)
- [ ] Documentar APIs (OpenAPI)
- [ ] Configurar rate limiting
- [ ] Adicionar testes básicos
- [ ] Configurar CI/CD
- [ ] Criar templates de PR/commits

---

## 📚 Exemplo de API Route Completa

```typescript
// app/api/matches/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { ApiError } from '@/lib/api-error'
import { logger } from '@/lib/logger'
import { z } from 'zod'
import * as Sentry from '@sentry/nextjs'

const matchSchema = z.object({
  home_team: z.string().min(1),
  away_team: z.string().min(1),
  start_date: z.string().datetime(),
})

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const requestId = crypto.randomUUID()

  try {
    logger.info('Fetching match', { requestId, matchId: params.id })

    const match = await getMatchById(params.id)

    if (!match) {
      throw new ApiError(404, 'NOT_FOUND', 'Match not found')
    }

    return NextResponse.json(
      { data: match },
      {
        status: 200,
        headers: {
          'X-Request-ID': requestId,
          'Cache-Control': 'public, max-age=300',
        },
      }
    )
  } catch (error) {
    if (error instanceof ApiError) {
      logger.warn('API error', { requestId, error })
      return NextResponse.json(
        {
          error: {
            code: error.code,
            message: error.message,
            details: error.details,
            requestId,
            timestamp: new Date().toISOString(),
          },
        },
        { status: error.statusCode }
      )
    }

    logger.error('Unexpected error', { requestId, error })
    Sentry.captureException(error, { tags: { endpoint: '/api/matches' } })

    return NextResponse.json(
      {
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Internal server error',
          requestId,
          timestamp: new Date().toISOString(),
        },
      },
      { status: 500 }
    )
  }
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const requestId = crypto.randomUUID()

  try {
    const body = await request.json()
    const validated = matchSchema.partial().parse(body)

    const match = await updateMatch(params.id, validated)

    return NextResponse.json(
      { data: match },
      {
        status: 200,
        headers: { 'X-Request-ID': requestId },
      }
    )
  } catch (error) {
    if (error instanceof z.ZodError) {
      throw new ApiError(400, 'VALIDATION_ERROR', 'Invalid input', error.errors)
    }
    // ... tratamento de erro
  }
}
```

---

**Este documento deve ser revisado e atualizado periodicamente conforme o projeto evolui.**
