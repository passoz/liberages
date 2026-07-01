# Liberages

Single-binary Go application (API + BFF + React SPA + PWA) with SQLite, built as a unified process serving everything on port `:3000`.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Go 1.26+ |
| HTTP | `net/http` stdlib |
| Database | SQLite + sqlc |
| Frontend | React 19+ SPA + PWA (Vite, React Router) |
| Styling | Tailwind CSS v4 + Glassmorphism |
| Auth | JWT + bcrypt (interface-swappable) |
| IDs | UUID v7 (server-generated) |
| Tests | Unit + Integration + API (httptest) + E2E (Playwright) |

## Quick Start

```bash
make dev              # hot reload (API + React)
make build-all        # build frontend + server
make test             # run all tests
make validate         # lint + test + build + e2e-browser
```

## Features

### Lustmapia — Rede Social Liberal

Rede social completa para o público liberal brasileiro (swing, troca de casais, ménage, BDSM, exibicionismo, voyeurismo). O mapa interativo é o hub central da experiência, combinando descoberta geográfica com comunidade, eventos e conteúdo.

- **Mapa interativo** — Leaflet com marcadores customizados, heatmaps de atividade, modo noturno
- **Perfis com privacidade em camadas** — blur facial automático, modo invisível, selfie destrutível, verificação opcional
- **Matching por intenção + localização** — "o que você busca hoje?" ao invés de swipe genérico
- **Eventos com geolocalização** — festas, encontros em motel, surubas com check-in via PIN
- **Grupos por interesse** — públicos e privados, com curadoria e regras de conduta
- **Live streaming com máscara facial** — blur em tempo real, pay-per-view, controle de audiência
- **Age verification** — 18+ gate com cookie assinado HMAC
- **Auth** — local JWT + bcrypt (Keycloak adapter interface ready)
- **Admin dashboard** — full CRUD panel with filters, sorting, map preview
- **PWA + Desktop app** — service worker, system tray, atalho de ocultação discreta
- **Conteúdo exclusivo** — feed de assinantes, tips, pay-per-view
- **Segurança** — botão de pânico, contato de emergência, local público sugerido, rate limiting

> **Diferencial:** Não é "mais um Sexlog". É uma plataforma moderna com privacidade real, descoberta geográfica e UX superior — construída para escapar das limitações de app stores (PWA + desktop).

Documentação estratégica: `app/spec/dossie-mercado.md`
Especificação técnica: `app/spec/lustmapia.md`

## Architecture

```
cmd/server/main.go            ← single entrypoint
internal/handler/api/         ← REST handlers (/api/v1/*)
internal/handler/web/         ← BFF — serves React SPA
internal/service/             ← business logic
internal/repository/sqlite/   ← sqlc implementations
internal/domain/              ← entities, interfaces
internal/middleware/          ← HTTP middlewares
internal/config/              ← environment config
internal/migrate/             ← golang-migrate setup
frontend/                     ← React SPA + PWA
db/migrations/                ← SQL migrations
db/queries/                   ← sqlc query files
```

## Documentation

- `app/AGENTS.md` — tech stack rules and conventions
- `app/spec/` — feature specifications
- `memory/` — context and decisions log
