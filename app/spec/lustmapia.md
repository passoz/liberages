# Lustmapia — Feature Specification

> Geolocalização para locais de entretenimento adulto/liberal no Brasil. Implementado dentro da stack padrão do Liberages.

## Origem

Repo referência: `passoz/lustmapia` clonado em `/workspace/lustmapia`. Contém a lógica de negócio, UI e fluxos que devem ser re-implementados na stack do Liberages (Go + SQLite + sqlc + React SPA).

## O que o lustmapia original faz

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

## Adaptação para a stack Liberages

| Lustmapia original | Liberages (como implementar) |
|---|---|
| Next.js 16 App Router | React SPA + Vite + React Router (frontend/) |
| PostgreSQL 17 + Drizzle ORM | SQLite + sqlc (SQL puro, sem ORM) |
| Keycloak OIDC | Auth local bcrypt+JWT + interface para adapter Keycloak futuro |
| AWS S3 SDK v3 | Interface de storage — filesystem local primeiro, S3 depois |
| Leaflet via CDN | Leaflet servido como asset estático do Go (sem CDN) |
| CSS Modules | Tailwind CSS v4 + Glassmorphism |
| Zod validation | `go-playground/validator/v10` no Go |
| Vitest + Playwright | `testing` stdlib + httptest + playwright-go |
| Docker Compose 5 serviços | Docker Compose 1 serviço (único binário Go) |
| Sem paginação | Paginação obrigatória (cursor-based ou offset) |
| Sem índices | SQLite FTS5 para busca textual |
| Sem rate limiting | Rate limiting no login (5 tentativas/min/IP) |
| Sem logging | `slog` stdlib em todos os níveis |
| Sem CI/CD | GitHub Actions completo |

## Schema do Banco (SQLite)

### locations

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| type | TEXT | NOT NULL (enum: nightclub, lounge, beach, resort, spa, sauna, club, other) |
| is_private | INTEGER | NOT NULL DEFAULT 0 |
| latitude | REAL | NOT NULL |
| longitude | REAL | NOT NULL |
| city | TEXT | NOT NULL |
| country | TEXT | NOT NULL DEFAULT 'BR' |
| website | TEXT | NOT NULL DEFAULT '' |
| instagram | TEXT | NOT NULL DEFAULT '' |
| twitter | TEXT | NOT NULL DEFAULT '' |
| map_icon | TEXT | NOT NULL DEFAULT '' |
| logo | TEXT | NOT NULL DEFAULT '' |
| main_image | TEXT | NOT NULL DEFAULT '' |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### users

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| email | TEXT | NOT NULL UNIQUE |
| password_hash | TEXT | NOT NULL |
| name | TEXT | NOT NULL |
| role | TEXT | NOT NULL DEFAULT 'user' (user, admin) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### age_verifications

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| ip_hash | TEXT | NOT NULL |
| birthdate | TEXT | NOT NULL |
| verified_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Índices

```sql
CREATE INDEX idx_locations_type ON locations(type);
CREATE INDEX idx_locations_city ON locations(city);
CREATE INDEX idx_locations_coords ON locations(latitude, longitude);
CREATE VIRTUAL TABLE locations_fts USING fts5(name, description, city, content='locations');
```

## Endpoints da API

### Auth

```
POST /api/v1/auth/register    → criar conta
POST /api/v1/auth/login       → login → TokenPair
POST /api/v1/auth/refresh     → novo access token via refresh token
POST /api/v1/auth/logout      → revogar token
GET  /api/v1/auth/me          → dados do usuário atual
```

### Age Gate

```
POST /api/v1/age-gate/verify  → verificar idade, setar cookie assinado
GET  /api/v1/age-gate/status  → checar se idade já foi verificada
```

### Locations (público)

```
GET  /api/v1/locations              → lista com paginação + busca + filtro por tipo
GET  /api/v1/locations/{id}         → detalhe de um local
GET  /api/v1/location-types         → tipos estáticos
```

### Locations (admin)

```
GET    /api/v1/admin/locations       → lista admin (inclui privados)
POST   /api/v1/admin/locations       → criar local
PUT    /api/v1/admin/locations/{id}  → atualizar local
DELETE /api/v1/admin/locations/{id}  → deletar local
```

### Files

```
POST /api/v1/admin/files/upload     → upload de imagem (multipart)
```

## Frontend (React SPA)

### Rotas

| Rota | Componente | Acesso |
|---|---|---|
| `/` | Map | Público (age gate requerido) |
| `/login` | AuthLogin | Público |
| `/registro` | AuthRegister | Público |
| `/locations/{id}` | LocationModal (modal) | Público/Privado |
| `/admin` | AdminDashboard | Admin only |

### Componentes principais

- `Map` — Leaflet com marcadores SVG por tipo, geolocation, popups
- `LocationCard` — card do local no popup/modal
- `LocationModal` — formulário de criação/edição (admin)
- `AgeGate` — verificação 18+ com calendário
- `AuthLogin` / `AuthRegister` — formulários de auth
- `AdminDashboard` — painel admin com tabela, filtros, ações
- `PWAInstallPrompt` — prompt de instalação PWA

## Tipos de Local

```
nightclub   → Boate
lounge      → Lounge
beach       → Praia
resort      → Resort
spa         → Spa
sauna       → Sauna
club        → Clube
other       → Outro
```

Cada tipo tem um marcador SVG customizado no mapa.

## Diferenças críticas vs lustmapia original

1. **Paginação obrigatória** — queries retornam no máximo 50 itens por página
2. **Índices no banco** — tipo, cidade, coordenadas e FTS5 para busca
3. **Rate limiting** — login limitado a 5 tentativas/minuto por IP
4. **Logging completo** — slog em todos os níveis
5. **Testes obrigatórios** — unitário + integração + API + e2e-browser em toda feature
6. **CI/CD** — pipeline completo com quality gate
7. **Sem dependências CDN** — tudo servido pelo binário Go
8. **Auth local primeiro** — Keycloak é adapter futuro, não padrão
9. **Storage local primeiro** — S3 é adapter futuro
10. **1 serviço Docker** — não 5 como no original

## Arquivos do repo referência

O código fonte do lustmapia original está em `/workspace/lustmapia` para consulta. Não copiar arquivos diretamente — re-implementar na stack do Liberages.
