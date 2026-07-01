# Mapa Interativo + Radar de Intenção — Feature Specification

> Geolocalização gamificada para o público liberal brasileiro. Implementado dentro da stack padrão do Liberages (Go + SQLite + sqlc + React SPA).

---

## 1. Visão Geral

O mapa é o coração do Liberages. São **dois mapas conceitualmente distintos** que convivem na mesma interface:

1. **Mapa de Locais** — descoberta de pontos fixos (casas de swing, motéis, bares, praias). Não envolve privacidade de pessoas. É um catálogo geográfico.

2. **Radar de Atividade** — gamificação estilo "Pokémon GO do prazer". Usuários não veem localização exata de ninguém. Apenas clusters fuzzy, notificações de proximidade e checkins em locais públicos. Identidade só é revelada após consentimento mútuo.

---

## 2. Mapa de Locais

### 2.1 Descrição

Catálogo geográfico de estabelecimentos e locais relevantes ao público liberal. Ponto fixo, público, sem questões de privacidade individual.

### 2.2 Funcionalidades

- Mapa Leaflet com marcadores SVG customizados por tipo de local
- CRUD de locais (admin-only) com upload de imagens
- Busca pública por nome/cidade/tipo com SQLite FTS5
- Locais privados (visíveis só para autenticados)
- Filtros por tipo, cidade, proximidade

### 2.3 Tipos de Local

```
nightclub   → Boate
lounge      → Lounge
beach       → Praia
resort      → Resort
spa         → Spa
sauna       → Sauna
club        → Clube de swing
motel       → Motel
bar         → Bar liberale
other       → Outro
```

Cada tipo tem um marcador SVG customizado no mapa.

### 2.4 Schema do Banco — locations

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| type | TEXT | NOT NULL (enum: nightclub, lounge, beach, resort, spa, sauna, club, motel, bar, other) |
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

### 2.5 Índices

```sql
CREATE INDEX idx_locations_type ON locations(type);
CREATE INDEX idx_locations_city ON locations(city);
CREATE INDEX idx_locations_coords ON locations(latitude, longitude);
CREATE VIRTUAL TABLE locations_fts USING fts5(name, description, city, content='locations');
```

---

## 3. Radar de Atividade

### 3.1 Conceito — "Pokémon GO do Prazer"

O usuário **não sabe quem ao seu redor está a fim de uma aventura** — até o radar avisar.

- O mapa **nunca mostra localização exata de pessoas** — apenas clusters fuzzy por bairro/zona
- Usuário marca "estou a fim hoje" (botão de raio de curto alcance) — status anônimo
- Quando dois usuários compatíveis por intenção entram no mesmo raio, ambos recebem notificação: "alguém próximo está a fim" — match implícito **sem revelar identidade**
- Identidade só é revelada após consentimento mútuo ( ambos aceitam o match)
- **Checkins em locais públicos** são visíveis no mapa — gamificação real do "tô no X, quem mais está?"
- **Notificações de checkin próximo**: "3 casais acabaram de fazer check-in no Y, a 1.5km de você"

### 3.2 Mecânicas de Gamificação

| Mecânica | Descrição | Privacidade |
|----------|-----------|-------------|
| **Marcar intenção** | Usuário ativa "a fim hoje" com tipo de interação desejada (swing, ménage, voyeur, etc.) | Anônimo — ninguém vê quem marcou |
| **Radar de proximidade** | Sistema detecta compatibilidade por intenção + distância | Notifica ambos, sem revelar identidade |
| **Match implícito** | Ambos recebem "alguém próximo está a fim" — podem aceitar ou ignorar | Identidade revelada só se ambos aceitarem |
| **Checkin em local público** | Usuário faz check-in em casa de swing, bar, etc. — aparece no mapa | Visível (o local é público, a presença é escolha do usuário) |
| **Notificação de checkin próximo** | Push/web notification quando checkins acontecem perto | Mostra local e contagem, não identifica usuários |
| **Heatmap de atividade** | Densidade de atividade no mapa ("onde o povo está agora") | Agregado, sem identificar indivíduos |
| **Raio de busca configurável** | Usuário define raio (1km, 5km, 10km, cidade) | Só afeta suas próprias notificações |

### 3.3 Regras de Privacidade

1. **Localização exata nunca é exposta** — sempre cluster fuzzy (bairro/zona, raio mínimo de ~500m)
2. **Intenção é anônima** — ninguém vê quem marcou "a fim", só recebe match se compatível
3. **Checkin é opt-in explícito** — usuário decide fazer check-in em local público; sistema nunca automaticamente
4. **Modo invisível** — usuário pode estar online sem aparecer no radar nem receber notificações
5. **Identidade pós-match** — só revelada após consentimento mútuo
6. **Dados de localização não são persistidos** — intenção "a fim hoje" expira em 24h ou quando desligada

### 3.4 Schema do Banco — activity_events

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| event_type | TEXT | NOT NULL (enum: intention, checkin, match) |
| intention_type | TEXT | NULL (swing, menage, voyeur, exhibition, chat) — só para event_type=intention |
| location_id | TEXT | NULL, FK → locations — só para event_type=checkin |
| latitude_fuzzy | REAL | NOT NULL — latitude aproximada (cluster) |
| longitude_fuzzy | REAL | NOT NULL — longitude aproximada (cluster) |
| radius_meters | INTEGER | NOT NULL DEFAULT 2000 — raio de busca do usuário |
| status | TEXT | NOT NULL DEFAULT 'active' (active, expired, matched, cancelled) |
| expires_at | TEXT | NOT NULL — quando a intenção/checkin expira |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### 3.5 Índices

```sql
CREATE INDEX idx_activity_user ON activity_events(user_id);
CREATE INDEX idx_activity_type_status ON activity_events(event_type, status);
CREATE INDEX idx_activity_coords ON activity_events(latitude_fuzzy, longitude_fuzzy);
CREATE INDEX idx_activity_expires ON activity_events(expires_at);
```

### 3.6 Notificações

| Evento | Gatilho | Conteúdo da notificação |
|--------|---------|------------------------|
| **Match de intenção** | Dois usuários compatíveis dentro do raio | "Alguém próximo está a fim de [tipo]. Distância: ~Xkm. Tocar para ver." |
| **Checkin próximo** | Checkin em local dentro do raio configurado | "[N] casal(is) acabou(aram) de fazer check-in em [Local], a ~Xkm de você" |
| **Heatmap ativo** | Densidade de atividade aumenta em zona | "Movimento aumentando em [Zona]. [N] pessoas ativas agora." |
| **Match aceito** | Ambos aceitaram match implícito | "Match! Você e [nome] estão conectados. Abrir chat?" |

### 3.7 API — Radar de Atividade

```
POST   /api/v1/radar/intention        → marcar intenção ("a fim hoje" com tipo)
DELETE /api/v1/radar/intention         → desmarcar intenção
GET    /api/v1/radar/intention         → ver minha intenção ativa
POST   /api/v1/radar/checkin            → fazer check-in em local público
DELETE /api/v1/radar/checkin/{id}       → cancelar check-in
GET    /api/v1/radar/nearby             → ver atividade próxima (clusters + checkins + heatmap)
GET    /api/v1/radar/notifications      → listar notificações de radar
POST   /api/v1/radar/match/{id}/accept → aceitar match implícito
POST   /api/v1/radar/match/{id}/reject  → rejeitar match implícito
```

---

## 4. Schema do Banco — users e age_verifications

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

---

## 5. Endpoints da API

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

---

## 6. Frontend (React SPA)

### 6.1 Rotas

| Rota | Componente | Acesso |
|---|---|---|
| `/` | MapPage (Mapa de Locais + Radar) | Público (age gate requerido) |
| `/login` | AuthLogin | Público |
| `/registro` | AuthRegister | Público |
| `/locations/{id}` | LocationModal (modal) | Público/Privado |
| `/admin` | AdminDashboard | Admin only |

### 6.2 Componentes Principais

- `MapPage` — container principal que alterna entre Mapa de Locais e Radar de Atividade
- **Mapa de Locais:**
  - `LocationsMap` — Leaflet com marcadores SVG por tipo, geolocation, popups
  - `LocationCard` — card do local no popup/modal
  - `LocationModal` — formulário de criação/edição (admin)
- **Radar de Atividade:**
  - `RadarMap` — mapa com clusters fuzzy, heatmap, checkins visíveis
  - `IntentionButton` — botão "a fim hoje" com seletor de tipo de interação
  - `RadarNotifications` — lista de notificações de match/checkin/heatmap
  - `MatchModal` — modal de match implícito (aceitar/rejeitar)
  - `CheckinButton` — botão de check-in em local próximo
- `AgeGate` — verificação 18+ com calendário
- `AuthLogin` / `AuthRegister` — formulários de auth
- `AdminDashboard` — painel admin com tabela, filtros, ações
- `PWAInstallPrompt` — prompt de instalação PWA

### 6.3 UI — Glassmorphism

Todos os overlays no mapa (popups, modais, painéis de filtro, painel de radar) usam Glassmorphism:
- `backdrop-blur`
- `bg-white/10`
- `border-white/20`
- `shadow-lg`

---

## 7. Diferenças Críticas vs Arquitetura Antiga

1. **Mapa de Locais e Radar de Atividade são conceitualmente distintos** — não misturar
2. **Localização fuzzy obrigatória** — nunca expor coordenadas exatas de pessoas
3. **Paginação obrigatória** — queries retornam no máximo 50 itens por página
4. **Índices no banco** — tipo, cidade, coordenadas, FTS5 para busca, activity events por coords e expiry
5. **Rate limiting** — login limitado a 5 tentativas/minuto por IP
6. **Logging completo** — slog em todos os níveis
7. **Testes obrigatórios** — unitário + integração + API + e2e-browser em toda feature
8. **CI/CD** — pipeline completo com quality gate
9. **Sem dependências CDN** — tudo servido pelo binário Go
10. **Auth local primeiro** — Keycloak é adapter futuro, não padrão
11. **Storage local primeiro** — S3 é adapter futuro
12. **1 serviço Docker** — não 5
13. **Notificações em tempo real** — WebSocket ou SSE para match/checkin/heatmap (avaliar implementação)

---

## 8. Considerações Técnicas

### 8.1 Real-time

O radar de intenção e notificações de checkin requerem comunicação em tempo real. Opções:

- **SSE (Server-Sent Events)** — mais simples, unidirecional (suficiente para notificações), compatível com `net/http` stdlib
- **WebSocket** — bidirecional, necessário se houver chat em tempo real no mapa

Recomendação: **SSE para MVP** (notificações unidirecionais), WebSocket quando chat ao vivo for implementado.

### 8.2 Fuzzy Location

Algoritmo de fuzzy location:
- Pegar coordenadas exatas do usuário
- Aplicar offset aleatório de 500m-2km (configurável)
- Arredondar para grid de bairro/zona
- Persistir apenas coordenadas fuzzy, nunca as exatas

### 8.3 Batch de Notificações

Notificações de radar não devem ser instantâneas demais (spam). Agrupar:
- Match de intenção: notificar no máximo 1x a cada 5 minutos por usuário
- Checkin próximo: notificar no máximo 1x a cada 10 minutos por local
- Heatmap: notificar no máximo 1x por hora por zona