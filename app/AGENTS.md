# Tech Stack Padrão

> **REGRA ABSOLUTA:** Use este tech stack exatamente como descrito. Só mude se o usuário pedir EXPLICITAMENTE. Instrução ambígua → use esta stack.

---

## Resumo Rápido

| O quê | Como |
|-------|------|
| Linguagem | Go 1.26+ |
| Binário | **1 único** (`cmd/server`) — API + BFF + React estáticos + tray |
| HTTP | `net/http` stdlib (sem gin/chi/fiber/echo) |
| Banco | SQLite + sqlc (SQL puro, **sem ORM**) |
| Frontend | React SPA + PWA (Vite, React Router, service worker) |
| Estilo | Tailwind CSS + Glassmorphism |
| Auth | JWT + bcrypt local, interface trocável |
| IDs | UUID v7 — gerado no server, nunca no frontend |
| Porta API | `:3000` |
| Porta Dev | React dev server `:5173`, API `:3000` (CORS em dev) |
| Porta Prod | `:3000` — servidor Go serve API + estáticos React (único processo) |
| Testes | Obrigatórios em todo código — unitário + integração + e2e |

---

## ⛔ Proibições absolutas

- **NÃO** criar `cmd/backend/` ou `cmd/bff/` — é **um único** `cmd/server/` (BFF é uma camada interna, não entrypoint separado)
- **NÃO** criar dois serviços no docker-compose — é **um único** serviço `server`
- **NÃO** usar gin, chi, fiber, echo ou qualquer outro router — usar `net/http` stdlib
- **NÃO** usar ORM (gorm, ent, xorm) — usar sqlc + SQL puro
- **NÃO** gerar IDs no frontend — gerar no server com `uuid.V7()`
- **NÃO** usar logrus, zerolog, zap — usar `slog` stdlib
- **NÃO** escrever strings de UI em código Go — colocar nos templates HTML
- **NÃO** compilar Go dentro do Docker — compilar no host, copiar o binário pronto
- **NÃO** colocar `bin/` no `.dockerignore` — o Dockerfile precisa do binário de `bin/`
- **NÃO** pular teste de navegador — `./e2e/...` (Playwright) é **obrigatório** em toda funcionalidade

---

## Stack Principal

### Server — Go 1.26+

- **Linguagem:** Go 1.26+ (sempre usar a versão estável mais recente)
- **HTTP Router:** `net/http` stdlib — Go 1.26+ tem roteamento avançado com método, path params e wildcards nativos
- **Hot reload dev:** Air (`air -c .air.server.toml`)
- **Debugging:** Delve (`dlv`)
- **Banco:** SQLite por padrão, mas usar **interfaces Go** para a camada de dados — permite trocar o banco sem mudar código de negócio
  - ⚠️ `mattn/go-sqlite3` requer CGO: `CGO_ENABLED=1` e imagem base **Debian** (glibc)
  - Alternativa pure Go: `modernc.org/sqlite` — `CGO_ENABLED=0`, funciona em Alpine
- **Queries:** sqlc — geração de código a partir de SQL puro, **sem ORM**
- **Migrations:** `github.com/golang-migrate/migrate`
- **Validação de entrada:** `github.com/go-playground/validator/v10` em todos os endpoints da API
- **Arquitetura:** Clean architecture — ports & adapters nas interfaces de banco e auth
- **Repository Pattern:** toda entidade tem interface em `internal/domain/repository/`, implementada com sqlc em `internal/repository/sqlite/`
- **Auth:** email + bcrypt + JWT por padrão, mas via **interface** — qualquer provider (OAuth2, Keycloak, SSO) é injetado sem mudar lógica de negócio

### Arquitetura — Único Binário, Responsabilidades Separadas

**Um único processo** (`cmd/server`) serve tudo. Internamente, o código é dividido em pacotes com responsabilidades distintas, preparados para virar binários separados no futuro se necessário.

```
cmd/server/main.go            ← entrypoint ÚNICO. Registra todas as rotas, inicia o servidor.

internal/handler/api/         ← handlers REST (JSON). Rotas sob /api/v1/.
                                 Não sabe que existe frontend. Não acessa banco direto.
internal/handler/web/         ← handlers BFF (papel de BFF). Serve React SPA estático,
                                 injeta configs, lida com auth redirects.
internal/service/             ← lógica de negócio. Usado pelos handlers.
internal/repository/sqlite/   ← implementações sqlc. Acessa o banco.
internal/domain/              ← entidades, interfaces de repositório, interfaces de auth.
internal/middleware/          ← middlewares HTTP (logging, cors, auth, recovery, request-id).
internal/config/              ← configuração carregada do ambiente.
internal/migrate/             ← setup do golang-migrate.

frontend/                     ← React SPA + PWA (Vite + React Router)
  src/                        ← código-fonte React
  public/                     ← assets públicos, manifest.json, service worker
  dist/                       ← build de produção (servido pelo Go em prod)
```

| Pacote | Responsabilidade | Futuro (se separar) |
|--------|-----------------|---------------------|
| `internal/handler/api/` | Handlers REST — JSON in/out | `cmd/backend` |
| `internal/handler/web/` | BFF — serve React SPA, auth, config | `cmd/bff` |
| `internal/service/` | Lógica de negócio | compartilhado |
| `internal/repository/` | Acesso ao banco via sqlc | compartilhado |
| `frontend/` | React SPA + PWA | `cmd/frontend` (ou serviço separado) |

**Separação futura:** mover `internal/handler/api/` para `cmd/backend/`, `internal/handler/web/` para `cmd/bff/` e `frontend/` para deploy separado. Os pacotes `service` e `repository` ficam compartilhados em `internal/`.

### Frontend — React SPA + PWA

- **Framework:** React 19+ com Vite — SPAs com navegação cliente-side
- **Roteamento:** React Router v7 — navegação declarativa, lazy loading de rotas
- **Estado:** React Context + hooks — sem Redux/Zustand a menos que explicitamente necessário
- **Estilo:** Tailwind CSS v4 + Glassmorphism (`backdrop-blur`, `bg-white/10`, `border-white/20`, `shadow-lg`)
- **PWA:** Service worker, manifest.json, install prompt, cache offline
- **HTTP Client:** `fetch` nativo — sem Axios
- **Segurança:** JWT armazenado em cookie `HttpOnly`/`SameSite` ou `localStorage` com CSRF token
- **Dev server:** Vite na porta `:5173` com proxy para API `:3000`
- **Build:** `npm run build` gera `frontend/dist/` — servido pelo Go em produção
- **Hot reload:** Vite HMR nativo

### Estrutura de componentes React

```
frontend/
├── public/
│   ├── manifest.json           ← PWA manifest
│   └── sw.js                   ← service worker
├── src/
│   ├── main.tsx                ← entrypoint React
│   ├── App.tsx                 ← raiz com React Router
│   ├── routes/                 ← páginas (lazy loaded)
│   ├── components/             ← componentes reutilizáveis
│   ├── hooks/                  ← hooks customizados
│   ├── contexts/               ← React contexts (auth, etc.)
│   ├── lib/                    ← fetch wrapper, utils
│   └── styles/                 ← Tailwind + CSS global
├── index.html
├── vite.config.ts
├── tailwind.config.ts
└── package.json
```

### Padrões React (siga estes à risca)

**Componentes funcionais com TypeScript:**

```tsx
// frontend/src/components/UserList.tsx
export function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/v1/users")
      .then(res => res.json())
      .then(setUsers)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;

  return (
    <div className="grid gap-4">
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

**React Router com lazy loading:**

```tsx
// frontend/src/routes/index.tsx
import { lazy } from "react";
import { createBrowserRouter } from "react-router-dom";
import { Layout } from "../components/Layout";

const HomePage = lazy(() => import("./home"));
const LoginPage = lazy(() => import("./login"));
const DashboardPage = lazy(() => import("./dashboard"));

export const router = createBrowserRouter([
  {
    element: <Layout />,
    children: [
      { path: "/", element: <HomePage /> },
      { path: "/login", element: <LoginPage /> },
      { path: "/dashboard", element: <DashboardPage /> },
    ],
  },
]);
```

**Formulário com validação e loading:**

```tsx
export function LoginForm() {
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);
    setError("");

    const form = new FormData(e.currentTarget);
    const res = await fetch("/api/v1/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email: form.get("email"), password: form.get("password") }),
    });

    if (!res.ok) {
      setError("Credenciais inválidas");
      setLoading(false);
      return;
    }

    window.location.href = "/dashboard";
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input type="email" name="email" required className="input-glass" />
      <input type="password" name="password" required className="input-glass" />
      {error && <p className="text-red-400 text-sm">{error}</p>}
      <button type="submit" disabled={loading} className="btn-glass">
        {loading ? "Entrando..." : "Entrar"}
      </button>
    </form>
  );
}
```

**Modal com `<dialog>` nativo:**

```tsx
export function UserModal({ userId, open, onClose }: Props) {
  const ref = useRef<HTMLDialogElement>(null);

  useEffect(() => {
    open ? ref.current?.showModal() : ref.current?.close();
  }, [open]);

  return (
    <dialog ref={ref} onClose={onClose} className="backdrop-blur bg-white/10 rounded-xl p-6">
      <h2>Editar Usuário</h2>
      <UserForm userId={userId} onSuccess={onClose} />
      <button onClick={onClose}>Fechar</button>
    </dialog>
  );
}
```

---

## Estrutura de Diretórios

```text
projeto/
├── cmd/
│   └── server/                  # entrypoint ÚNICO — NÃO criar outros entrypoints
│       └── main.go
├── internal/
│   ├── domain/                  # entidades e interfaces (ports)
│   │   ├── entity/              # structs das entidades
│   │   └── repository/          # interfaces dos repositórios
│   ├── auth/                    # implementações de autenticação
│   │   └── local/               # adapter padrão (bcrypt + JWT)
│   ├── repository/              # implementações sqlc (adapters)
│   │   └── sqlite/
│   ├── handler/
│   │   ├── api/                 # handlers REST (JSON) — rotas /api/v1/*
│   │   └── web/                 # handlers BFF — serve React SPA, auth redirects
│   ├── service/                 # lógica de negócio (use cases)
│   ├── middleware/              # RequestID, SlogLogger, Recoverer, CORS, Auth
│   ├── config/                  # leitura de variáveis de ambiente
│   └── migrate/                 # setup do golang-migrate
├── frontend/                    # React SPA + PWA (Vite)
│   ├── src/
│   ├── public/
│   ├── dist/                    # build de produção (servido pelo Go em prod)
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   └── package.json
├── db/
│   ├── migrations/              # NNNNNN_descricao.up.sql / .down.sql
│   └── queries/                 # arquivos .sql para o sqlc gerar código Go
├── .air.server.toml             # configuração do Air para hot reload
├── .env.example
├── .gitignore
├── Dockerfile                   # Dockerfile ÚNICO — copia bin/server + frontend/dist/
├── docker-compose.yml           # dev (Air + Delve)
├── docker-compose.test.yml      # test (banco isolado + e2e)
├── docker-compose.prod.yml      # prod (binário compilado no host)
├── Makefile
├── sqlc.yaml
└── README.md
```

---

## Padrões de ID

- **Todo ID é UUID v7** (sortable por tempo) — sem exceção
- Lib: `github.com/google/uuid`, gerar com `uuid.New()` após `uuid.SetVersion(7)` ou `uuid.V7()` se disponível
- IDs gerados sempre **no servidor**, nunca no frontend
- No banco (SQLite): coluna `TEXT PRIMARY KEY` — SQLite não tem tipo UUID nativo

---

## Build

### Binário único

```bash
# Desenvolvimento (hot reload)
air -c .air.server.toml

# Build simples
go build -o bin/server ./cmd/server

# Build com CGO (para mattn/go-sqlite3)
CGO_ENABLED=1 go build -o bin/server ./cmd/server
```

### Cross-compilation

```bash
# Linux
GOOS=linux GOARCH=amd64 go build -o bin/server-linux ./cmd/server

# Windows
GOOS=windows GOARCH=amd64 go build -o bin/server.exe ./cmd/server
```

Makefile sempre deve ter `build-linux`, `build-windows`, `build-cross`.

### Frontend

- React SPA em `frontend/` — Vite + React Router + TypeScript
- Dev: `npm run dev` (Vite na porta `:5173` com proxy para API `:3000`)
- Build: `npm run build` gera `frontend/dist/` — servido pelo Go em produção com `http.FileServer`
- Air recarrega o servidor Go ao detectar mudanças em `.go`
- Vite HMR recarrega o React ao detectar mudanças em `.tsx`, `.ts`, `.css`
- Tailwind CSS via PostCSS (build step), não CDN
- PWA: service worker e manifest.json em `frontend/public/`

### Modo Desktop (apenas em produção)

Em desenvolvimento (`APP_ENV=dev` ou ausente): **sem** system tray, **sem** abrir browser. Apenas serve a porta.

Em produção (`APP_ENV=production`), o servidor detecta se está em modo desktop (presença de `$DISPLAY` no Linux, detectar explorer.exe no Windows):

**Se for desktop + produção:**
1. Inicia system tray (Linux: `fyne.io/systray` ou `github.com/getlantern/systray`)
   - Ícone no tray com opção "Exit" para encerramento gracioso
2. Abre browser automaticamente:
   - Tenta Chrome (`google-chrome`, `chrome`, `chromium-browser`, `chromium`)
   - Tenta Firefox (`firefox`)
   - Tenta Edge (`msedge`, `microsoft-edge`)
   - Se não encontrar nenhum: só loga aviso, **não falha**
   - URL: `http://localhost:PORT`

**Se NÃO for desktop** (container, CI, servidor headless):
- Apenas serve a porta — sem tray, sem browser, sem aviso
- Comportamento silencioso, apenas log

---

## Configuração

- Variáveis de ambiente via arquivo `.env` (carregado na inicialização)
- Struct de config centralizada em `internal/config/config.go`
- Docker Compose injeta as vars via `environment:`
- Não usar libs pesadas — `os.Getenv()` ou lib leve como `envconfig`

```
# .env.example
PORT=3000
DATABASE_PATH=./data/app.db
JWT_SECRET=change-me-in-production
LOG_LEVEL=debug
LOG_FORMAT=text
```

---

## Database

- Migrations em `db/migrations/` — formato `NNNNNN_descricao.up.sql` / `.down.sql`
- Queries em `db/queries/` — SQL puro que o sqlc usa para gerar código Go
- `sqlc.yaml` com `engine: "sqlite"`
- Rodar `sqlc generate` após alterar qualquer `.sql` em `db/queries/`

---

## API Design

### Versionamento

Todas as rotas da API sob `/api/v1/`. Rotas do frontend React são servidas como SPA (todas as rotas não-API caem no `index.html`).

```
GET  /api/v1/users       ← API REST
POST /api/v1/users       ← API REST
GET  /                    ← React SPA (servido como estático)
```

### Health Check

Os endpoints de health ficam **fora** do prefixo `/api/v1/`:

```
GET /healthz    → liveness  — sempre 200 se o processo está vivo
GET /readyz     → readiness — 200 se banco está acessível, 503 se não
```

### Middleware Chain

No `net/http` stdlib, middlewares são funções que envolvem `http.Handler`. O padrão é sempre este — **nunca** chamar `mux.Use()` porque isso não existe.

```go
// cmd/server/main.go

mux := http.NewServeMux()

// Registrar todas as rotas no mux primeiro
mux.Handle("GET /healthz", health.Liveness(db))
mux.Handle("GET /readyz", health.Readiness(db))
// ... demais rotas ...

// Aplicar middlewares envolvendo o mux inteiro
// Ordem de execução na requisição: RequestID → SlogLogger → Recoverer → CORS → Auth → handler
var h http.Handler = mux
h = middleware.Auth(authSvc)(h)
h = middleware.CORS(cfg)(h)
h = middleware.Recoverer(logger)(h)
h = middleware.SlogLogger(logger)(h)
h = middleware.RequestID(h)

srv := &http.Server{
    Addr:    ":" + cfg.Port,
    Handler: h,
}
```

Cada middleware tem a assinatura padrão:

```go
func MiddlewareName(/* dependências */) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // lógica antes
            next.ServeHTTP(w, r)
            // lógica depois
        })
    }
}
```

### Graceful Shutdown

Todo `cmd/` deve tratar SIGINT/SIGTERM:

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()

srv := &http.Server{
    Addr:    ":" + cfg.Port,
    Handler: h,
    BaseContext: func(net.Listener) context.Context { return ctx },
}

go func() {
    if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
        logger.Error("server error", slog.String("error", err.Error()))
    }
}()

<-ctx.Done()
logger.Info("shutting down...")

shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
if err := srv.Shutdown(shutdownCtx); err != nil {
    logger.Error("shutdown error", slog.String("error", err.Error()))
}
```

---

## Autenticação

### Arquitetura (ports & adapters)

A interface define o contrato. A implementação padrão é local (bcrypt + JWT). Para trocar para OAuth2/Keycloak/LDAP: só criar novo adapter que implemente a interface — nenhum código de negócio muda.

```go
// internal/domain/auth.go

type AuthService interface {
    Authenticate(ctx context.Context, email, password string) (*AuthUser, error)
    ValidateToken(ctx context.Context, token string) (*AuthUser, error)
    RefreshToken(ctx context.Context, refreshToken string) (*TokenPair, error)
    RevokeToken(ctx context.Context, token string) error
}

type AuthUser struct {
    ID          string   // UUID v7
    Email       string
    Name        string
    Roles       []string
    Permissions []string
}

type TokenPair struct {
    AccessToken  string
    RefreshToken string
    ExpiresAt    time.Time
}
```

### Implementação local (padrão)

Arquivo: `internal/auth/local/service.go`

- Lib bcrypt: `golang.org/x/crypto/bcrypt`, custo 12
- Lib JWT: `github.com/golang-jwt/jwt/v5`
- Access token: 15 minutos. Refresh token: 7 dias
- Claims JWT: `sub` (user ID UUID v7), `exp`, `iat`, `jti` (UUID v4)
- Hash da senha gerado no register, comparado no login — **nunca armazenar senha em texto**
- Rate limiting no login: máximo 5 tentativas por minuto por IP

### Implementações futuras (preparadas pela interface)

- OAuth2 (Google, GitHub, etc.)
- Keycloak / OpenID Connect
- SSO corporativo (SAML, LDAP)
- API Key para machine-to-machine

### Rotas de auth (em `internal/handler/api/auth_handler.go`)

```
POST /api/v1/auth/register    → criar conta
POST /api/v1/auth/login       → login → TokenPair
POST /api/v1/auth/refresh     → novo access token via refresh token
POST /api/v1/auth/logout      → revogar token
GET  /api/v1/auth/me          → dados do usuário atual (requer token válido)
```

---

## Logging

### Configuração

- Lib: `slog` stdlib — **nenhuma lib externa**
- Formato: texto em dev (`LOG_FORMAT=text`), JSON em prod (`LOG_FORMAT=json`)
- Nível: `debug` em dev, `info` em prod (via `LOG_LEVEL=debug|info|warn|error`)

### O que logar em cada nível

| Nível | O que loga |
|-------|------------|
| `DEBUG` | Cada request (headers, body), cada query SQL com args e duração, cada auth check, cada middleware, variáveis de ambiente carregadas, tempo de cada operação (ms), cache hit/miss, goroutines iniciadas, eventos do tray |
| `INFO` | Request method/path/status/duration, login/logout de usuário, entidade criada/atualizada/deletada, server start/stop, migration executada, browser aberto, tray inicializado |
| `WARN` | Requests lentos (>1s), banco lento, tentativa de login inválida, token expirado, CORS inválido, rate limit excedido, recurso não encontrado (sem ser erro do servidor) |
| `ERROR` | Panics, banco indisponível, erro de autenticação, request malformado, falha ao abrir browser, falha ao iniciar tray |

### Setup do logger

```go
// internal/middleware/slogger.go
var logLevel = new(slog.LevelVar)

func Setup(level, format string) *slog.Logger {
    switch strings.ToLower(level) {
    case "debug": logLevel.Set(slog.LevelDebug)
    case "info":  logLevel.Set(slog.LevelInfo)
    case "warn":  logLevel.Set(slog.LevelWarn)
    case "error": logLevel.Set(slog.LevelError)
    default:      logLevel.Set(slog.LevelDebug)
    }

    opts := &slog.HandlerOptions{Level: logLevel, AddSource: true}
    var handler slog.Handler
    if format == "json" {
        handler = slog.NewJSONHandler(os.Stdout, opts)
    } else {
        handler = slog.NewTextHandler(os.Stdout, opts)
    }

    logger := slog.New(handler)
    slog.SetDefault(logger)
    return logger
}
```

### Middleware de request

```go
func SlogLogger(logger *slog.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            sw := &statusWriter{ResponseWriter: w}
            next.ServeHTTP(sw, r)

            attrs := []slog.Attr{
                slog.String("method", r.Method),
                slog.String("path", r.URL.Path),
                slog.Int("status", sw.status),
                slog.Duration("duration", time.Since(start)),
                slog.String("request_id", GetRequestID(r.Context())),
                slog.String("remote", r.RemoteAddr),
                slog.String("user_agent", r.UserAgent()),
            }

            switch {
            case sw.status >= 500:
                logger.LogAttrs(r.Context(), slog.LevelError, "request failed", attrs...)
            case sw.status >= 400:
                logger.LogAttrs(r.Context(), slog.LevelWarn, "request warning", attrs...)
            default:
                logger.LogAttrs(r.Context(), slog.LevelInfo, "request ok", attrs...)
            }

            if logLevel.Level() <= slog.LevelDebug {
                logger.Debug("request headers",
                    slog.Any("headers", r.Header),
                    slog.String("query", r.URL.RawQuery),
                )
            }
        })
    }
}
```

### Queries SQL

```go
logger.Debug("sql query",
    slog.String("query", sql),
    slog.Any("args", args),
    slog.Duration("duration", duration),
    slog.Int("rows", rowCount),
)
```

### Auth events

```go
logger.Info("user login",
    slog.String("user_id", user.ID),
    slog.String("email", user.Email),
    slog.String("ip", r.RemoteAddr),
)
logger.Warn("failed login attempt",
    slog.String("email", email),
    slog.String("ip", r.RemoteAddr),
    slog.Int("attempts", attempts),
)
```

### Desktop events

```go
logger.Debug("desktop detection",
    slog.Bool("has_display", hasDisplay),
    slog.String("os", runtime.GOOS),
)
logger.Info("browser opened", slog.String("browser", browser), slog.String("url", url))
logger.Warn("no browser found", slog.String("tried", "chrome, firefox, edge"))
```

---

## Makefile

```makefile
dev:              # air -c .air.server.toml & cd frontend && npm run dev (sobe API + React em paralelo)
dev-server:       # air -c .air.server.toml (hot reload)
dev-frontend:     # cd frontend && npm run dev (Vite HMR)
build-frontend:   # cd frontend && npm run build
build-server:     # CGO_ENABLED=1 go build -o bin/server ./cmd/server
build-all:        # build-frontend + build-server
build-linux:      # GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go build -o bin/server-linux ./cmd/server
build-windows:    # GOOS=windows GOARCH=amd64 go build -o bin/server.exe ./cmd/server
build-cross:      # build-linux + build-windows
db-migrate-up:    # migrate -path db/migrations -database "sqlite3://./data/app.db" up
db-migrate-down:  # migrate -path db/migrations -database "sqlite3://./data/app.db" down
db-gen:           # sqlc generate
docker-up:        # compila binário no host → empacota em imagem leve → sobe container
test:             # go test ./...
test-unit:        # go test -short ./...
test-integration: # go test -run Integration ./...
test-e2e:         # go test -run E2E ./internal/handler/api/... -count=1
test-e2e-browser: # go test -count=1 ./e2e/...   (Playwright com navegador real)
test-coverage:    # go test -coverprofile=coverage.out ./... && go tool cover -html=coverage.out
test-regression:  # go test ./... -count=1
lint:             # golangci-lint run
validate:         # lint + test + build + e2e-browser (tudo precisa passar)
clean:            # rm -rf bin/ tmp/ frontend/dist/
```

---

## Ferramentas de Qualidade

### Linting
- `golangci-lint` com configuração padrão
- `go fmt` obrigatório — CI rejeita código não formatado
- `go vet` no CI

### Git Hooks
- **pre-commit:** `go fmt ./...`, `go vet ./...`, `sqlc generate`
- **pre-push:** `go test ./...`, `golangci-lint run`

### CI/CD (GitHub Actions)

```yaml
jobs:
  lint:
    steps: [golangci-lint, go fmt, go vet]

  test-unit:
    steps: [go test -short ./...]

  test-integration:
    steps: [go test -run Integration ./...]

  test-e2e:
    steps: [go test -run E2E ./internal/handler/api/... -count=1]

  test-e2e-browser:
    steps: [go test -count=1 ./e2e/...]   # Playwright com navegador headless

  test-regression:
    steps: [go test ./... -count=1]   # sem cache

  quality-gate:
    needs: [lint, test-unit, test-integration, test-e2e, test-e2e-browser, test-regression]
    if: success()    # ⚠️ SÓ AVANÇA SE TODOS PASSAREM

  build-cross:
    needs: quality-gate
    steps: [build-linux, build-windows]

  docker-build:
    needs: build-cross
    steps: [docker build com binário do host]
```

---

## Infraestrutura

### Docker Compose

Três arquivos separados, sempre consistentes:

```yaml
# docker-compose.yml (dev) — 1 serviço
services:
  server:         # Go + Air + Delve, volume do código, porta 3000

# docker-compose.test.yml — 2 serviços
services:
  server-test:    # Go + banco isolado em /tmp
  e2e:            # httptest + validação API REST

# docker-compose.prod.yml — 1 serviço
services:
  server:         # binário compilado no host, imagem Debian slim
```

### Dockerfile

```dockerfile
# Compilar o binário NO HOST antes de rodar docker build
# CGO_ENABLED=1 go build -o bin/server ./cmd/server

FROM debian:bookworm-slim
RUN apt-get update \
    && apt-get install -y ca-certificates libsqlite3-0 \
    && rm -rf /var/lib/apt/lists/*
COPY bin/server /bin/server
COPY frontend/dist /frontend/dist
EXPOSE 3000
CMD ["/bin/server"]
```

> ⚠️ `bin/` **não pode** estar no `.dockerignore` — o Dockerfile copia o binário de lá.

---

## Testes — O Software Já Nasce Testado

**Princípio:** todo código novo vem com testes no mesmo commit. Código sem teste é código incompleto.

### Pirâmide de testes

| Tipo | O que cobre | Ferramenta |
|------|-------------|------------|
| **Unitário** | Lógica de negócio, validações, services isolados (mocks) | `testing` stdlib + mocks manuais ou `gomock` |
| **Integração** | Repositórios com banco real, queries SQL, migrations | `testing` + `testcontainers-go` (SQLite real) |
| **API HTTP** | Handlers api/, middlewares, status codes, body JSON, auth | `httptest` stdlib |
| **E2E Navegador** ⚠️ | **Teste obrigatório com navegador real** — clique, formulário, navegação, modal, submit, validação visual. Toda funcionalidade do usuário passa por aqui | `playwright-go` (headless Chromium) |
| **Regressão** | Nada quebrou após mudança | `go test -count=1` (sem cache) |

### Estrutura de diretórios de teste

```text
internal/
├── handler/
│   ├── api/
│   │   ├── user_handler.go
│   │   └── user_handler_test.go       # httptest: endpoints REST
├── service/
│   ├── user_service.go
│   │   └── user_service_test.go           # unitário: lógica de negócio
├── repository/
│   └── sqlite/
│       ├── user_repository.go         # gerado por sqlc
│       └── user_repository_test.go    # integração: banco real
└── middleware/
    ├── auth.go
    └── auth_test.go                   # httptest: middleware chain

├── e2e/                                # testes com navegador REAL
│   ├── user_crud_test.go              # Playwright: criar, listar, editar, deletar usuário
│   ├── auth_test.go                   # Playwright: login, logout, token expirado
│   └── navigation_test.go             # Playwright: navegação entre páginas, modais
```

### Regra de ouro

```
╔══════════════════════════════════════════════════════════════╗
║  CÓDIGO NOVO → TESTES NOVOS. SEM EXCEÇÃO.                  ║
║  CÓDIGO ALTERADO → TESTES ATUALIZADOS. SEM EXCEÇÃO.        ║
║  BUG CORRIGIDO → TESTE DE REGRESSÃO. SEM EXCEÇÃO.          ║
║  FUNCIONALIDADE NOVA → TESTE NO NAVEGADOR. SEM EXCEÇÃO.    ║
║  PIPELINE VERMELHO → NADA SOBE. SEM EXCEÇÃO.               ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Regras

1. **Não mudar a stack sem autorização explícita** do usuário
2. Instrução ambígua → assumir esta stack
3. `sqlc` + SQL puro sempre — sem ORM
4. Interfaces de repositório sempre — sem acoplamento direto ao banco
5. Pedido de "faz um CRUD de X" → usar esta stack automaticamente
6. `validator/v10` em todas as entradas da API REST
7. **Cross-compile sempre** — gerar binário para Linux e Windows
8. **Único binário** `cmd/server` — `internal/handler/api/` + frontend React servido como estáticos no mesmo processo. Docker compose tem **1 serviço**. Separação futura em `cmd/backend` + frontend separado só se o usuário pedir
9. Modo desktop: system tray + abrir browser **apenas em produção** (Chrome → Firefox → Edge). Em dev: sem tray, sem browser, apenas serve a porta.
10. `net/http` stdlib para rotas — **proibido** gin/chi/fiber/echo
11. Go 1.26+ — usar `maps`, `slices`, `unique`, `iter` e features novas da stdlib
12. `slog` stdlib para logging — **proibido** logrus/zerolog/zap
13. `golang-migrate` para migrations
14. **Todo software já nasce testado** — código sem teste não é aceito
15. Testes unitários em todo código — nenhuma função pública sem teste (`testing` stdlib + mocks)
16. Testes de integração em todo repository — banco real com `testcontainers-go`, zero mocks no banco
17. Testes de API em todo endpoint — `httptest` stdlib para handlers, middlewares, status, body, auth
18. Testes de template — `httptest` + `golang.org/x/net/html` para verificar renderização com dados variados (nil, vazio, erro, lista cheia)
19. **Teste com navegador real em TODA funcionalidade** — Playwright (`playwright-go`) simulando clique, formulário, navegação, modal, submit. Sem exceção. `go test -count=1 ./e2e/...`
20. Todo bug corrigido gera teste de regressão — `go test -count=1` garante sem cache
21. **Código só é validado após todos os testes passarem** — unitário + integração + API + e2e + e2e-browser + regressão
22. Pipeline CI bloqueia deploy se qualquer teste falhar — quality gate sem bypass
23. **Código Go 100% em inglês** — variáveis, funções, tipos, pacotes, comentários, commits, logs
24. **UI em português** — strings de UI no frontend React, sem texto de UI hardcoded em código Go
25. Docker Compose para dev, test e prod — três arquivos separados
26. **Verificar issues abertas no remoto** — se o projeto tiver remote git, ler issues abertas e criar/atualizar `.todo/issues.md` localmente
27. **Todo ID é UUID v7** — gerado no server, `TEXT` no SQLite
28. API sob `/api/v1/` — health check em `/healthz` e `/readyz` (fora do `/api/v1/`)
29. Middleware chain fixa: `RequestID` → `SlogLogger` → `Recoverer` → `CORS` → `Auth`
30. Graceful shutdown em todo `cmd/` (SIGINT/SIGTERM, timeout 10s)
31. `golangci-lint` + `go fmt` + `go vet` no CI
32. Git hooks: pre-commit (`fmt`, `vet`, `sqlc generate`), pre-push (`test`, `lint`)
33. CI/CD GitHub Actions: `lint` → `test-unit` → `test-integration` → `test-e2e` → `test-e2e-browser` → `test-regression` → `quality-gate` → `build-cross` → `docker-build`
34. Auth local (bcrypt + JWT) por padrão, interface para trocar para OAuth2/Keycloak/SSO
35. Glassmorphism no frontend: `backdrop-blur`, `bg-white/10`, `border-white/20`, `shadow-lg`
36. Preferir modais/dialogs para formulários e detalhes em vez de navegar para nova página
37. **Compilar binário no host** — nunca dentro do container Docker (lento)
38. CGO: `mattn/go-sqlite3` → imagem base **Debian** (glibc). `modernc.org/sqlite` → Alpine funciona
39. `.dockerignore` **NÃO deve conter** `bin/` — o Dockerfile copia o binário de `bin/`
40. Logging DEBUG loga tudo: headers, body, SQL, auth events, desktop events. `LOG_LEVEL=debug` em dev, `info` em prod. Texto em dev, JSON em prod

---

## Feature: Mapa Interativo + Radar de Intenção

> Geolocalização gamificada para o público liberal brasileiro. Implementado dentro da stack padrão do Liberages (Go + SQLite + sqlc + React SPA).
>
> Especificação completa: `app/spec/mapa-interativo.md`

### O que o produto faz

- Mapa interativo com Leaflet + marcadores SVG por tipo de local
- CRUD de locais (admin-only) com upload de imagens (S3)
- Busca pública por nome/cidade/tipo
- Login/registro/logout via Keycloak (OIDC)
- Verificação de idade 18+ com calendário customizado + cookie assinado HMAC
- Dashboard admin com filtros, sorting, preview de mapa, modais
- PWA com service worker e install prompt
- Locais privados (visíveis só para autenticados)
- Health check (DB + contagens)
- OpenAPI spec completa

### Adaptação para a stack Liberages

| Arquitetura anterior (referência) | Liberages (como implementar) |
|---|---|
| Next.js 16 App Router | React SPA + Vite + React Router (frontend/) |
| PostgreSQL 17 + Drizzle ORM | SQLite + sqlc (SQL puro, sem ORM) |
| Keycloak OIDC | Auth local bcrypt+JWT + interface para adapter Keycloak futuro |
| AWS S3 SDK v3 | Interface de storage — implementar local filesystem primeiro, S3 depois |
| Leaflet via CDN | Leaflet servido como asset estático do Go (sem CDN) |
| CSS Modules | Tailwind CSS v4 + Glassmorphism |
| Zod validation | `go-playground/validator/v10` no Go |
| Vitest + Playwright | `testing` stdlib + httptest + playwright-go |
| Docker Compose 5 serviços | Docker Compose 1 serviço (único binário Go) |
| Sem paginação | Paginação obrigatória no Go (cursor-based ou offset) |
| Sem índices | SQLite FTS5 para busca textual |
| Sem rate limiting | Rate limiting no login (5 tentativas/min/IP) |
| Sem logging | `slog` stdlib em todos os níveis |
| Sem CI/CD | GitHub Actions completo (lint → test → build → docker) |

### Schema do Banco (SQLite)

```sql
-- locations
CREATE TABLE locations (
    id          TEXT PRIMARY KEY,  -- UUID v7
    name        TEXT NOT NULL,
    description TEXT NOT NULL DEFAULT '',
    type        TEXT NOT NULL,     -- enum: nightclub, lounge, beach, resort, spa, sauna, club, other
    is_private  INTEGER NOT NULL DEFAULT 0,
    latitude    REAL NOT NULL,
    longitude   REAL NOT NULL,
    city        TEXT NOT NULL,
    country     TEXT NOT NULL DEFAULT 'BR',
    website     TEXT NOT NULL DEFAULT '',
    instagram   TEXT NOT NULL DEFAULT '',
    twitter     TEXT NOT NULL DEFAULT '',
    map_icon    TEXT NOT NULL DEFAULT '',
    logo        TEXT NOT NULL DEFAULT '',
    main_image  TEXT NOT NULL DEFAULT '',
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_locations_type ON locations(type);
CREATE INDEX idx_locations_city ON locations(city);
CREATE INDEX idx_locations_coords ON locations(latitude, longitude);

-- FTS5 para busca textual
CREATE VIRTUAL TABLE locations_fts USING fts5(name, description, city, content='locations', content_rowid='rowid');

-- users (para auth local)
CREATE TABLE users (
    id            TEXT PRIMARY KEY,  -- UUID v7
    email         TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    name          TEXT NOT NULL,
    role          TEXT NOT NULL DEFAULT 'user',  -- user, admin
    created_at    TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at    TEXT NOT NULL DEFAULT (datetime('now'))
);

-- age_gate (registro de verificação)
CREATE TABLE age_verifications (
    id          TEXT PRIMARY KEY,  -- UUID v7
    ip_hash     TEXT NOT NULL,
    birthdate   TEXT NOT NULL,
    verified_at TEXT NOT NULL DEFAULT (datetime('now'))
);
```

### Endpoints da API

```
# Auth
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET  /api/v1/auth/me

# Age gate
POST /api/v1/age-gate/verify
GET  /api/v1/age-gate/status

# Locations (público)
GET  /api/v1/locations              → lista com paginação + busca + filtro por tipo
GET  /api/v1/locations/{id}         → detalhe de um local
GET  /api/v1/location-types         → tipos estáticos

# Locations (admin)
GET    /api/v1/admin/locations       → lista admin (inclui privados)
POST   /api/v1/admin/locations       → criar local
PUT    /api/v1/admin/locations/{id}  → atualizar local
DELETE /api/v1/admin/locations/{id}  → deletar local

# Files
POST /api/v1/admin/files/upload     → upload de imagem (multipart)
```

### Frontend (React SPA)

```
/                    → mapa interativo (Leaflet)
/login               → login
/registro            → registro
/locations/{id}      → detalhe do local (modal, não navega)
/admin               → dashboard admin (protegido)
```

Componentes principais:
- `Map` — Leaflet com marcadores SVG por tipo, geolocation, popups
- `LocationCard` — card do local no popup/modal
- `LocationModal` — formulário de criação/edição (admin)
- `AgeGate` — verificação 18+ com calendário
- `AuthLogin` / `AuthRegister` — formulários de auth
- `AdminDashboard` — painel admin com tabela, filtros, ações
- `PWAInstallPrompt` — prompt de instalação PWA

### Diferenças críticas vs arquitetura anterior

1. **Paginação obrigatória** — queries retornam no máximo 50 itens por página
2. **Índices no banco** — tipo, cidade, coordenadas e FTS5 para busca
3. **Rate limiting** — login limitado a 5 tentativas/minuto por IP
4. **Logging completo** — slog em todos os níveis (debug/info/warn/error)
5. **Testes obrigatórios** — unitário + integração + API + e2e-browser em toda feature
6. **CI/CD** — pipeline completo com quality gate
7. **Sem dependências CDN** — tudo servido pelo binário Go
8. **Auth local primeiro** — Keycloak é adapter futuro, não padrão
9. **Storage local primeiro** — S3 é adapter futuro
10. **1 serviço Docker** — não múltiplos serviços
11. **Mapa de Locais e Radar de Atividade são conceitualmente distintos** — não misturar na implementação
12. **Localização fuzzy obrigatória** — nunca expor coordenadas exatas de pessoas no Radar de Atividade
