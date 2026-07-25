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
- **Catálogo Híbrido** — Lançamento com locais curados (50-100/cidade), expansão via sugestões moderadas. Parceiros B2B gerenciam eventos e status ao vivo.
- **Radar de Atividade** — Visão anônima de quem está nos locais. Identidade só é revelada via chat (nunca pelo radar).
- **Check-in Dinâmico** — Check-in manual com TTL configurável pelo usuário (ex: 1h a 4h).
- **Destination Broadcast** — Convite informal no feed ("Vou no local X hoje"). Desaparece após o evento.
- **Caça ao Tesouro (B2B)** — Parceiros escondem tesouros no mapa; usuário coleta para ganhar a moeda da plataforma.

**Matching, Social & Descoberta:**
- **Feed Social Unificado** — Algoritmo contínuo que prioriza: 1º Amigos, 2º Pessoas Próximas, 3º Alta compatibilidade de interesses.
- **Busca FTS5** — O Liberages é uma rede social primeiro. Busca global refinada (tags, bio, cidade, pseudônimo).
- **Swipe com Limites** — Descoberta adicional via Swipe. Limite de likes diários no Free, ilimitado no Premium.
- **Algoritmo de Compatibilidade** — Match matemático simples (interseção) no catálogo fixo de fetiches.
- **Bucket List** — Quebra-gelo inteligente: se houve match e ambos querem ir ao mesmo local, o app sugere o encontro lá.

**Conteúdo & Expressão:**
- **Fotolog Diário** — Estilo clássico, 1 foto por dia (sem blur) no feed, expira em 24h.
- **Álbuns** — Galerias curadas em perfis com controle de privacidade. (Limite de 3 no Free).
- **Contos Eróticos** — Literatura criada por usuários. (Free lê, Premium ou Badges altos publicam).
- **Eventos UGC** — Qualquer usuário `Verificado` pode criar eventos públicos. Não verificados criam eventos privados (convite/PIN).

**Privacidade & Segurança (Core):**
- **Idade Verificada (Híbrido)** — Acesso inicial via auto-declaração. Interação e conteúdo adulto exigem validação de documento anônima (sem expor nome real).
- **Login via PIN** — Acesso rápido de 4 dígitos para uso no dia a dia.
- **Blur Facial Automático** — Por padrão, fotos recebem blur para proteção, com opt-in explícito para revelar o rosto.
- **Selfie Destrutível (Chat)** — Envio de fotos que expiram após 1 visualização no chat privado.
- **Alarme Psicológico de Print** — Toda foto renderiza o Hash ID de quem visualiza como marca d'água dinâmica, desencorajando vazamentos.
- **Modo Falso (Free)** — Botão de pânico (tela de calculadora/ferramenta).
- **Modo Invisível & Fantasma (Premium)** — Esconda-se de estranhos ou de todos, respectivamente.
- **Comunidades Anônimas** — Fóruns sob pseudônimos. Apenas admin da comunidade vê a identidade real para moderação.

**Monetização & Economia Unificada:**
- **Moeda Única** — Sistema integrado que unifica XP, recompensas e cash. Ganhe interagindo, gaste em boosts, presentes virtuais ou descontos B2B. (Circuito fechado, sem cash-out).
- **Free com Ads** — Usuários grátis veem anúncios locais (B2B). Premium remove ads.
- **Modelo Híbrido de Premium** — Assinaturas pagas em Dinheiro ou Parte em Moeda + Parte Dinheiro.
- **Diferenciais Premium** — Veja histórico completo de visitas e curtidas, Ghost Mode, ilimitados álbuns e swipes, publicação de contos.
- **Assinatura de Presente** — Compre premium para outros usuários.
- **Boost Avulso** — Destaque no swipe por 1h (pago em moeda).

**Admin, Moderação & Comunidade:**
- **Contas de Casal/Trisal** — Uma conta, um perfil compartilhado gerido por ambos, login unificado.
- **Web of Trust (Verificado)** — Selo concedido apenas se 4 "Amigos Reais" atestarem a veracidade do perfil. Libera privilégios como eventos públicos.
- **Moderação IA + Humana** — Triagem por IA para denúncias, veredito final sempre humano.
- **Júri Popular** — Anjos da Comunidade votam apenas em regras amplas da plataforma, não em punições individuais.

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