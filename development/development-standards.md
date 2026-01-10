# Padrões de Desenvolvimento - LiveMode

## 📋 Stack Tecnológica

### Obrigatório
- **Framework**: Next.js (App Router)
- **Biblioteca UI**: React.js
- **Componentes**: Radix UI / Shadcn/ui
- **Banco de Dados**: PostgreSQL (via Supabase ou direto)
- **Gerenciador de Pacotes**: pnpm ou bun (preferência: pnpm)
- **Linters e Formatters**: Configurados e padronizados (ainda estamos decidindo nossos padrões)

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

### 2. Versionamento (recomendado)

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
- **429 Too Many Requests**: Rate limit excedido (em raros casos, pois nossos produtos são internos)

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

*PATCH pode ser idempotente se bem implementado. Idempotência significa que fazer a mesma requisição múltiplas vezes produz o mesmo resultado. Por exemplo, `PATCH /users/1 { "status": "active" }` executado 10 vezes deve resultar no mesmo estado final (status = "active").

### Headers Importantes

#### Request
```
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
Accept-Encoding: gzip, deflate  # Indica que cliente aceita resposta comprimida
X-Request-ID: <uuid>  # Para rastreamento (gerado no cliente ou servidor)
```

**Nota sobre X-Request-ID**: No Node.js, é fácil gerar usando `crypto.randomUUID()` (nativo desde Node 14.17.0). Esta função é muito rápida e eficiente, não causa impacto de performance:
```typescript
import { randomUUID } from 'crypto'
const requestId = randomUUID()
// Ou usar biblioteca uuid: import { v4 as uuidv4 } from 'uuid'
// Performance: crypto.randomUUID() é otimizado e não causa lentidão
```

#### Response
```
Content-Type: application/json
Content-Encoding: gzip  # Se resposta foi comprimida
X-Request-ID: <uuid>  # Echo do request ID (mesmo do request ou gerado pelo servidor)
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
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
import { randomUUID } from 'crypto'

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

// lib/request-id.ts - Helper para gerenciar Request ID
export function getRequestId(request: Request): string {
  // Tenta pegar do header, senão gera um novo
  return request.headers.get('X-Request-ID') || randomUUID()
}

// Uso em API routes
export async function GET(request: Request) {
  const requestId = getRequestId(request)

  try {
    // ... lógica
    return NextResponse.json(
      { data: result },
      {
        status: 200,
        headers: {
          'X-Request-ID': requestId
        }
      }
    )
  } catch (error) {
    if (error instanceof ApiError) {
      return NextResponse.json(
        {
          error: {
            code: error.code,
            message: error.message,
            details: error.details,
            requestId,
            timestamp: new Date().toISOString()
          }
        },
        {
          status: error.statusCode,
          headers: {
            'X-Request-ID': requestId
          }
        }
      )
    }

    // Erro não esperado
    logger.error('Unexpected error', { requestId, error })
    return NextResponse.json(
      {
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Erro interno do servidor',
          requestId,
          timestamp: new Date().toISOString()
        }
      },
      {
        status: 500,
        headers: {
          'X-Request-ID': requestId
        }
      }
    )
  }
}
```

**Middleware para adicionar Request ID automaticamente (opcional):**

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { randomUUID } from 'crypto'

export function middleware(request: NextRequest) {
  const requestId = request.headers.get('X-Request-ID') || randomUUID()

  const response = NextResponse.next()
  response.headers.set('X-Request-ID', requestId)

  return response
}

export const config = {
  matcher: '/api/:path*',
}
```

### Tratamento no Cliente

#### Opção 1: Com Exceções (Padrão)

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

#### Opção 2: Sem Exceções (Result Pattern - Recomendado para casos específicos)

```typescript
// lib/api-client.ts
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E }

export async function apiRequest<T>(
  url: string,
  options?: RequestInit
): Promise<Result<T, ApiError>> {
  try {
    const response = await fetch(url, options)
    const data = await response.json()

    if (!response.ok) {
      const error = new ApiError(
        response.status,
        data.error?.code || 'UNKNOWN_ERROR',
        data.error?.message || 'Erro desconhecido',
        data.error?.details
      )
      logger.error('API request failed', { url, error })
      return { success: false, error }
    }

    return { success: true, data }
  } catch (error) {
    const apiError = error instanceof ApiError
      ? error
      : new ApiError(500, 'NETWORK_ERROR', 'Network request failed', error)
    logger.error('API request failed', { url, error: apiError })
    return { success: false, error: apiError }
  }
}

// Uso sem try/catch
const result = await apiRequest<User>('/api/users/1')
if (result.success) {
  console.log(result.data) // TypeScript sabe que é User
} else {
  console.error(result.error.message) // TypeScript sabe que é ApiError
}
```

**Quando usar cada abordagem:**
- **Com exceções**: Padrão mais comum, funciona bem com try/catch
- **Sem exceções (Result)**: Melhor para casos onde você quer controle explícito de erros, evita try/catch aninhados, e TypeScript oferece melhor type safety

> **⚠️ Decisão Pendente**: Ainda estamos em processo de decisão se vamos adotar um padrão único para toda a organização. Por enquanto, ambos os padrões são aceitos, mas recomendamos consistência dentro do mesmo projeto. A decisão final será documentada aqui quando definida.

---

## 📝 Logging

### Bibliotecas de Logging

#### Pino (Recomendado)

**Por que Pino?**
- ⚡ **Performance**: Extremamente rápido (até 5x mais rápido que Winston)
- 📦 **Tamanho**: Biblioteca pequena e leve
- 🔄 **Async por padrão**: Não bloqueia o event loop
- 🎯 **JSON estruturado**: Logs em formato JSON nativo
- 🛠️ **Extensível**: Fácil de integrar com ferramentas de análise

**Alternativas:**
- **Winston**: Mais popular, mais recursos, mas mais lento
- **Bunyan**: Similar ao Pino, mas menos mantido
- **console.log**: Nativo, mas não estruturado e sem níveis

### Estrutura de Logs com Pino

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
    service: 'live-mode',
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
  "service": "live-mode",
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

### 1. Testes (Recomendação)
- **Unit tests**: Jest ou Vitest (recomendado)
- **Integration tests**: Para APIs (recomendado quando aplicável)
- **E2E**: Playwright ou Cypress (opcional)
- **Coverage mínimo**: 70% (meta, não obrigatório)

### 2. Documentação de APIs

**Opções disponíveis:**

- **OpenAPI/Swagger**: Especificação padrão para documentação de APIs REST
  - Gera documentação interativa automaticamente
  - Útil para documentação formal e contratos de API
  - Pode ser integrado com ferramentas de geração de código

- **Postman Collections**: Recomendado para documentação prática e testes
  - Permite criar collections com exemplos reais de requests
  - Facilita testes manuais e automação de testes de API
  - Pode ser versionado junto com o código
  - Exporta/importa facilmente entre ambientes
  - Útil para onboarding de novos desenvolvedores
  - **Vantagem**: Mais prático que OpenAPI para uso diário, permite testar APIs diretamente

**Recomendação**: Usar ambos quando possível:
- OpenAPI/Swagger para documentação formal e contratos
- Postman Collections para testes práticos e exemplos de uso

**Boas práticas:**
- Exemplos de requests/responses reais
- Manter atualizada com mudanças na API
- Incluir casos de erro comuns
- Documentar autenticação e headers necessários

### 3. Validação de Dados
- Zod para schemas
- Validação no cliente e servidor
- Mensagens de erro claras

### 4. Segurança
- Variáveis de ambiente para secrets
- Validação de inputs (sanitização)
- CORS configurado
- HTTPS obrigatório em produção
- **Headers de segurança (helmet.js)**: Biblioteca que configura automaticamente headers HTTP de segurança importantes:
  - `X-Content-Type-Options: nosniff` - Previne MIME type sniffing
  - `X-Frame-Options: DENY` - Previne clickjacking
  - `X-XSS-Protection: 1; mode=block` - Proteção XSS
  - `Strict-Transport-Security` - Força HTTPS
  - `Content-Security-Policy` - Política de segurança de conteúdo
  - E outros headers de segurança essenciais

  **Implementação no Next.js:**
  ```typescript
  // next.config.js ou middleware
  import helmet from 'helmet'
  // Para Next.js, use next-safe ou configure headers manualmente
  // next.config.js:
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-XSS-Protection', value: '1; mode=block' },
        ],
      },
    ]
  }
  ```

### 5. Performance

**Otimização de Código:**
- **Lazy loading de componentes**: `React.lazy()` e `dynamic()` no Next.js
- **Code splitting**: Separar código por rotas e features
- **Tree shaking**: Remover código não utilizado
- **Bundle analysis**: Usar `@next/bundle-analyzer` para identificar bundles grandes
  ```bash
  # Analisar bundle
  ANALYZE=true pnpm build
  ```

**Otimização de Imagens (Recomendação):**
- Usar componente `Image` do Next.js (otimização automática)
- Formatos modernos: WebP, AVIF quando suportado
- Lazy loading de imagens abaixo do fold
- Tamanhos responsivos com `srcset`
- Compressão adequada (não perder qualidade visível)

**Métricas importantes (Metas audaciosas, não critérios obrigatórios):**
Estas são métricas que miramos alcançar, mas não são critérios obrigatórios para todos os projetos:
- **LCP (Largest Contentful Paint)**: < 2.5s (meta)
- **FID (First Input Delay)**: < 100ms (meta)
- **CLS (Cumulative Layout Shift)**: < 0.1 (meta)
- **TTFB (Time to First Byte)**: < 600ms (meta)
- **Bundle size**: Monitorar tamanho total do JavaScript (recomendação)

**Ferramentas (Recomendações):**
Ferramentas úteis para monitorar e analisar performance (não obrigatórias):
- Lighthouse (Chrome DevTools) - análise de performance
- WebPageTest - testes de performance detalhados
- Next.js Analytics - métricas do Next.js
- Bundle Analyzer - análise de tamanho de bundles

### 6. CI/CD
- Lint e testes no pipeline (se houver testes)
- Builds automáticos
- Deploy automatizado

### 7. Versionamento

**Conventional Commits:**
Formato padronizado de mensagens de commit que facilita automação e geração de changelogs:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Tipos principais:**
- `feat`: Nova funcionalidade
- `fix`: Correção de bug
- `docs`: Mudanças na documentação
- `style`: Formatação, ponto e vírgula, etc (não afeta código)
- `refactor`: Refatoração de código
- `perf`: Melhoria de performance
- `test`: Adição ou correção de testes
- `chore`: Tarefas de manutenção (deps, config, etc)
- `ci`: Mudanças em CI/CD
- `build`: Mudanças no sistema de build

**Exemplos:**
```
feat(auth): add OAuth2 login
fix(api): resolve memory leak in user endpoint
docs(readme): update installation instructions
refactor(utils): simplify date formatting function
```

**Semantic Versioning (SemVer):**
Formato: `MAJOR.MINOR.PATCH` (ex: `1.2.3`)

- **MAJOR** (1.0.0): Mudanças incompatíveis com versões anteriores
- **MINOR** (0.1.0): Novas funcionalidades compatíveis com versões anteriores
- **PATCH** (0.0.1): Correções de bugs compatíveis

**Regras:**
- Versão inicial: `0.1.0` ou `1.0.0`
- Incrementar MAJOR quando houver breaking changes
- Incrementar MINOR quando adicionar funcionalidades
- Incrementar PATCH quando corrigir bugs

**Changelog (Recomendação):**
Manter um changelog é recomendado para rastrear mudanças do projeto. Pode ser feito manualmente ou com auxílio de IA Dev:
- Manter arquivo `CHANGELOG.md` atualizado (recomendado)
- Formato recomendado: [Keep a Changelog](https://keepachangelog.com/)
- Seções: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`
- Gerar automaticamente com ferramentas como `standard-version` ou `semantic-release` (opcional)
- **Dica**: Use IA Dev para ajudar a gerar/atualizar o changelog baseado nos commits

**Exemplo de CHANGELOG.md:**
```markdown
## [1.2.0] - 2024-01-15

### Added
- OAuth2 authentication support
- User profile endpoint

### Changed
- Updated API error response format

### Fixed
- Memory leak in user endpoint
- Date formatting timezone issue
```

**Ferramentas úteis (Recomendações):**
Estas são ferramentas recomendadas que podem ajudar no processo de versionamento, mas não são obrigatórias:
- `commitlint`: Valida formato de commits (recomendado)
- `husky`: Git hooks para validação (recomendado)
- `standard-version`: Gera versões e changelogs automaticamente (opcional)
- `semantic-release`: Automação completa de releases (opcional)

### 8. Organização de Arquivos e Textos

**Separação de Conteúdo e Código:**

Textos, mensagens e conteúdo que podem mudar frequentemente devem ser isolados do código para facilitar manutenção, tradução e atualização.

**Estrutura recomendada:** (ou outra caso queira trazer a sugestão)

```
project/
├── src/                    # Código fonte
│   ├── components/         # Frontend
│   ├── lib/
│   └── ...
├── content/                # Conteúdo isolado (textos, mensagens)
│   ├── messages/           # Mensagens compartilhadas (Frontend + Backend)
│   │   ├── frontend/       # Mensagens específicas do frontend
│   │   │   ├── pt-BR.json
│   │   │   └── en-US.json
│   │   └── backend/       # Mensagens específicas do backend (APIs)
│   │       ├── pt-BR.json
│   │       └── en-US.json
│   ├── api-errors/         # Códigos e mensagens de erro de API (Backend)
│   │   ├── pt-BR.json
│   │   └── en-US.json
│   ├── validation/         # Mensagens de validação (Backend)
│   │   ├── pt-BR.json
│   │   └── en-US.json
│   ├── emails/             # Templates de email (Backend)
│   │   ├── welcome.html
│   │   └── reset-password.html
│   ├── notifications/      # Mensagens de notificação (Backend)
│   │   ├── push/
│   │   └── sms/
│   └── legal/              # Textos legais (Frontend + Backend)
│       ├── terms/
│       └── privacy/
├── public/                  # Assets estáticos
└── ...
```

**Alternativa simplificada (se preferir estrutura mais simples):**

```
project/
├── content/
│   └── messages/
│       ├── pt-BR.json      # Todas as mensagens (frontend + backend)
│       └── en-US.json
│       # Estrutura interna organiza por contexto:
│       # - ui.* (frontend)
│       # - api.* (backend)
│       # - errors.* (backend)
│       # - validation.* (backend)
```

**O que deve ser isolado:**

**Frontend:**
- **Mensagens de UI**: Botões, labels, placeholders, tooltips
- **Mensagens de erro do frontend**: Erros de validação de formulários, feedback de ações
- **Textos legais**: Termos de uso, política de privacidade (exibidos no frontend)
- **Conteúdo dinâmico**: Descrições, FAQs, help text

**Backend:**
- **Mensagens de erro de API**: Códigos de erro padronizados, mensagens de erro HTTP
- **Mensagens de validação**: Erros de validação de dados (Zod, etc)
- **Conteúdo de emails**: Templates HTML/texto de emails
- **Notificações**: Mensagens push, SMS, etc

**Compartilhado:**
- Mensagens que aparecem tanto no frontend quanto no backend (ex: "Email inválido")

**Vantagens:**
- ✅ Fácil manutenção sem tocar no código
- ✅ Suporte a múltiplos idiomas (i18n)
- ✅ Não-desenvolvedores podem atualizar conteúdo
- ✅ Versionamento separado de conteúdo
- ✅ Reduz risco de quebrar código ao alterar textos
- ✅ Facilidade para dar manutenção com AI Dev

**Implementação:**

#### Estrutura de Mensagens (Exemplo)

```json
// content/messages/frontend/pt-BR.json
{
  "ui": {
    "auth": {
      "login": {
        "title": "Entrar",
        "email": "Email",
        "password": "Senha",
        "forgotPassword": "Esqueci minha senha"
      }
    }
  },
  "errors": {
    "form": {
      "required": "Este campo é obrigatório",
      "invalidEmail": "Email inválido"
    }
  }
}

// content/messages/backend/pt-BR.json
{
  "api": {
    "errors": {
      "VALIDATION_ERROR": "Campos obrigatórios não preenchidos",
      "UNAUTHORIZED": "Não autenticado",
      "FORBIDDEN": "Sem permissão para esta ação",
      "NOT_FOUND": "Recurso não encontrado",
      "INTERNAL_ERROR": "Erro interno do servidor"
    }
  },
  "validation": {
    "email": {
      "invalid": "Email inválido",
      "required": "Email é obrigatório",
      "alreadyExists": "Este email já está em uso"
    },
    "password": {
      "minLength": "Senha deve ter no mínimo {min} caracteres",
      "required": "Senha é obrigatória"
    }
  }
}
```

#### Uso no Frontend

```typescript
// src/lib/messages.ts (Frontend)
import frontendMessages from '../../content/messages/frontend/pt-BR.json'

export const t = (path: string, params?: Record<string, string>) => {
  const keys = path.split('.')
  let value: any = frontendMessages
  for (const key of keys) {
    value = value?.[key]
  }
  if (typeof value !== 'string') return path
  return params ? value.replace(/\{(\w+)\}/g, (_, key) => params[key] || '') : value
}

// Uso no componente React
const title = t('ui.auth.login.title')
const errorMessage = t('errors.form.required')
```

#### Uso no Backend (API Routes)

```typescript
// src/lib/messages.ts (Backend)
import backendMessages from '../../content/messages/backend/pt-BR.json'

export const getApiErrorMessage = (code: string, params?: Record<string, string>): string => {
  const message = backendMessages.api?.errors?.[code] || 'Erro desconhecido'
  return params ? message.replace(/\{(\w+)\}/g, (_, key) => params[key] || '') : message
}

export const getValidationMessage = (field: string, rule: string, params?: Record<string, string>): string => {
  const message = backendMessages.validation?.[field]?.[rule] || `${field} inválido`
  return params ? message.replace(/\{(\w+)\}/g, (_, key) => params[key] || '') : message
}

// Uso em API route
export async function POST(request: Request) {
  try {
    // validação...
    if (!email) {
      return NextResponse.json(
        {
          error: {
            code: 'VALIDATION_ERROR',
            message: getValidationMessage('email', 'required')
          }
        },
        { status: 400 }
      )
    }
  } catch (error) {
    return NextResponse.json(
      {
        error: {
          code: 'INTERNAL_ERROR',
          message: getApiErrorMessage('INTERNAL_ERROR')
        }
      },
      { status: 500 }
    )
  }
}
```

#### Estrutura Simplificada (Alternativa)

Se preferir uma estrutura mais simples com tudo em um arquivo:

```json
// content/messages/pt-BR.json
{
  "ui": { /* mensagens frontend */ },
  "api": { /* mensagens backend */ },
  "errors": { /* erros compartilhados */ },
  "validation": { /* validações backend */ }
}
```

**Ferramentas recomendadas (opcionais):**
- **next-i18next** ou **next-intl**: Para internacionalização completa (recomendado se precisar de i18n)
- **i18next**: Biblioteca popular para i18n (alternativa)
- **react-i18next**: Integração React (se usar i18next)

### 9. Ambiente de Desenvolvimento
- Docker Compose para serviços locais (sempre que possível)
- Scripts padronizados (dev, build, test)
- README com setup

---

## ✅ Checklist de Implementação

- [ ] Configurar linters (ESLint + Prettier)
- [ ] Configurar formatters (Prettier)
- [ ] Implementar padrão de tratamento de erros
- [ ] Configurar logging (Pino)
- [ ] Integrar crash reporting (Sentry ou Bugsnag)
- [ ] Criar utilitários de API (api-client, api-error)
- [ ] Documentar APIs (OpenAPI/Swagger e/ou Postman Collections)
- [ ] Adicionar testes básicos (opcional mas recomendado)
- [ ] Configurar CI/CD

---

## 📚 Exemplo de API Route Completa

```typescript
// app/api/matches/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { ApiError } from '@/lib/api-error'
import { getRequestId } from '@/lib/request-id'
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
  const requestId = getRequestId(request)

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
  const requestId = getRequestId(request)

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
