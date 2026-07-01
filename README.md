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

### Rede Social Liberal com Mapa-Radar Gamificado

Plataforma para o público liberal brasileiro (swing, troca de casais, ménage, BDSM, exibicionismo, voyeurismo). O mapa interativo é o hub central da experiência — pensado como um **Pokémon GO do prazer**: você não sabe quem ao seu redor está a fim de uma aventura, até o radar de intenção te avisar.

- **Mapa de Locais** — Leaflet com marcadores customizados, catálogo geográfico de casas de swing, motéis, bares, praias
- **Radar de Atividade (gamificado)** — marque "a fim hoje", receba notificação quando alguém compatível estiver perto. Localização sempre fuzzy, identidade só revelada após match mútuo
- **Checkins em locais públicos** — "tô no X, quem mais está?" com notificações de checkin próximo
- **Perfis com privacidade em camadas** — blur facial automático, modo invisível, selfie destrutível, verificação opcional
- **Matching por intenção + localização** — "o que você busca hoje?" ao invés de swipe genérico
- **Eventos com geolocalização** — festas, encontros, surubas com check-in via PIN
- **Grupos por interesse** — públicos e privados, com curadoria e regras de conduta
- **Live streaming com máscara facial** — blur em tempo real, pay-per-view, controle de audiência
- **Age verification** — 18+ gate com cookie assinado HMAC
- **Auth** — local JWT + bcrypt (Keycloak adapter interface ready)
- **Admin dashboard** — full CRUD panel with filters, sorting, map preview
- **PWA + Desktop app** — service worker, system tray, atalho de ocultação discreta
- **Anúncios no Free** — display e nativos B2B locais; Premium sem anúncios
- **Segurança** — botão de pânico, contato de emergência, local público sugerido, rate limiting

> **Diferencial:** Não é "mais um Sexlog". É uma plataforma moderna com mapa-radar gamificado (Pokémon GO do prazer), privacidade real e UX superior — construída para escapar das limitações de app stores (PWA + desktop) e operar com custo quase nulo (Go + SQLite, single binary).

Documentação estratégica: `app/spec/dossie-mercado.md`
Especificação técnica do mapa: `app/spec/mapa-interativo.md`
Plano de negócios: `business/business-plan.md`

## Architecture

```
cmd/server/main.go            ← single entrypoint
internal/handler/api/         ← REST handlers (/api/v1/*)
internal/handler/web/         ← BFF — serves React SPA
internal/service/             ← business logic
internal/repository/sqlite/   ← sqlc implementations
internal/domain/              ← entities, interfaces
internal/middleware/          ← HTTP middlewares
internal/config/             ← environment config
internal/migrate/             ← golang-migrate setup
frontend/                     ← React SPA + PWA
db/migrations/                ← SQL migrations
db/queries/                   ← sqlc query files
```

## Documentation

- `app/AGENTS.md` — tech stack rules and conventions
- `app/spec/` — feature specifications
- `business/` — business plan and financial projections
- `memory/` — context and decisions log