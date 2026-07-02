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

**Mapa & Radar:**
- **Mapa de Locais** — catálogo geográfico de casas de swing, motéis, bares, praias
- **Radar de Atividade (gamificado)** — marque "a fim hoje", receba notificação quando alguém compatível estiver perto. Localização sempre fuzzy, identidade só revelada após match mútuo
- **Checkins em locais públicos** — "tô no X, quem mais está?" com notificações de checkin próximo
- **Caça ao Tesouro (B2B)** — parceiros escondem tesouros no mapa visíveis só a X km; usuário coleta → cupom/voucher

**Matching & Descoberta:**
- **Swipe** — curtir/não curtir perfis, ordenado por compatibilidade de fetiches + intenção + localização
- **Lista de Fetiches com Match %** — catálogo curado, compatibilidade percentual entre perfis
- **Bucket List** — lista de desejos de locais; match quando dois têm o mesmo local
- **Encontro Surpresa** — match vira encontro às cegas em local sorteado
- **Match Boost** — R$2-5 para topo do feed de swipe por 1h

**Conteúdo & Expressão:**
- **Fotolog Diário** — estilo Fotolog antigo, 1 foto/dia, expira em 24h
- **Álbuns** — galerias curadas com controle de privacidade
- **Story 24h** — foto/vídeo com reação por emoji
- **Contos Eróticos** — literatura escrita por usuários, votação, destaque semanal
- **Live Streaming** — máscara facial em tempo real, pay-per-view, lives em grupo
- **Marketplace de Criadores** — venda de conteúdo (fotos, vídeos, packs), comissão 15%

**Comunidade:**
- **Fórum** — categorias, tópicos, votação, busca FTS5
- **Comunidades Anônimas** — participação sem revelar identidade (pseudônimo gerado)
- **Mural de Elogio Anônimo** — elogios moderados, badges de admiração
- **Confiança Mútua** — "conheço pessoalmente" como reputação social

**Gamificação:**
- **Badges** — selos por marcos (checkins, matches, tempo de conta, conteúdo)
- **XP e Níveis** — interações dão XP, níveis desbloqueiam features
- **Ranking Semanal** — opt-in, top 10 com badge temporária
- **Desafio Liberal Semanal** — missões semanais com XP bônus
- **Jogo da Verdade** — mini-game pós-match de quebra-gelo
- **Júri Popular** — denúncias votadas por usuários verificados
- **Selo Anjo da Comunidade** — usuários exemplares com benefícios

**Privacidade & Segurança:**
- **Perfis com privacidade em camadas** — blur facial automático, modo invisível, selfie destrutível
- **Modo Fantasma Total** — some completamente (busca, radar, broadcasts); gratuito
- **Modo Falso** — botão de emergência leva pra tela falsa (calculadora, clima)
- **E2E nas DMs** — criptografia ponta a ponta no dispositivo (Web Crypto + ECDH)
- **Alarme de Screenshots** — detecção e notificação ao dono do conteúdo
- **Login com PIN** — acesso rápido de 4 dígitos para sessões curtas
- **Moderação Híbrida** — robô (IA) + humana, com apelo de banimento
- **Validação Periódica** — re-verificação de idade e identidade
- **Verificação por Vídeo** — selo "Verificado ao Vivo" (dourado)

**Eventos:**
- **Eventos com geolocalização** — festas, encontros, surubas com check-in via PIN
- **Agenda Liberal** — calendário com feriados sazonais (Carnaval, Dia do Sexo)
- **Carona Solidária** — usuários indo pro mesmo evento marcam carona
- **Lista de Presença Anônima** — heatmap demográfico sem identificar indivíduos

**Modo Casal:**
- **Conta compartilhada** — duas contas linkadas, double opt-in no swipe
- **Moodboard do Casal** — painel colaborativo privado (fotos, datas, lembretes)

**Monetização de Microtransações:**
- **Presente Virtual Picante** — emojis animados a R$1-3, comissão 20%
- **Assinatura de Presente** — comprar Premium/VIP para outro usuário
- **Cartão Fidelidade (B2B)** — checkins acumulam pontos viram descontos

**Admin & Operação:**
- **Admin dashboard** — full CRUD, moderação, métricas em tempo real
- **Tickets de Suporte** — sistema integrado com prioridade por tier
- **Análise de Sentimento** — dashboard de moderação com métricas
- **Painel de Status** — status.liberages.com com uptime e incidentes
- **QR Code** — conexão presencial e checkin em locais parceiros

**Outros:**
- **Age verification** — 18+ gate com cookie assinado HMAC
- **Auth** — local JWT + bcrypt (Keycloak adapter interface ready)
- **PWA + Desktop app** — service worker, system tray, atalho de ocultação discreta
- **Anúncios no Free** — display e nativos B2B locais; Premium sem anúncios

> **Diferencial:** Não é "mais um Sexlog". É uma plataforma moderna com 47+ features gamificadas, mapa-radar estilo Pokémon GO, privacidade real (E2E, modo fantasma, modo falso) e UX superior — construída para escapar das limitações de app stores (PWA + desktop) e operar com custo quase nulo (Go + SQLite, single binary).

Documentação estratégica: `app/spec/dossie-mercado.md`
Especificação técnica do mapa: `app/spec/mapa-interativo.md`
Especificação de 47 features: `app/spec/features.md`
- 24 features safadarias (circulação/veteranos/iniciantes): `app/spec/features-safadia.md`
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