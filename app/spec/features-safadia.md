# Safadarias — Features Criativas e Provocantes

> Features projetadas para três eixos estratégicos do Liberages:
> 1. **Estimular circulação** — nada acontece sozinho no meio liberal; o app precisa gerar momentum, urgência e serendipidade
> 2. **Dar motivo para veteranos/influentes usarem o app** — eles já têm rede offline; a plataforma precisa oferecer ferramentas impossíveis de replicar sem app
> 3. **Quebrar o cold-start de iniciantes** — casais novatos travam; precisam de injeção social guiada

---

## Eixo 1 — Geradores de Circulação e Momentum

### 48. Festa Relâmpago (Flash Party)

 festas espontâneas que criam urgência e movimento em tempo real.

- Usuário com reputação alta (Nível 5+ OU selo Anjo OU verificado ao vivo) pode disparar uma **Festa Relâmpago**
- Notificação push para todos no raio configurável (5km, 10km, cidade): "⚡ Festa Relâmpago iniciando em 2h no [local]"
- Dashboards do local parceiro mostra "ocupação disparou" em tempo real
- Quem confirma presença na festa relâmpago aparece no contador
- Limite: 1 festa relâmpago por usuário por semana (free), 3/semana (Premium)
- Local pode ser parceiro (casa de swing) ou aberto (bar, praia)
- Após a festa: álbum coletivo temporário (todos que confirmaram podem postar fotos do evento, com blur)
- **Monetização**: parceiro B2B pode pagar para ser "local sugerido" em festas relâmpago
- **Fila de espera automática**: se a festa relâmpago encher (limite de confirmados = lotação do local), próximo usuário que tenta confirmar entra em fila de espera e recebe notificação se vaga abrir

### Schema — `flash_parties`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| organizer_id | TEXT | NOT NULL, FK → users |
| location_id | TEXT | NULL, FK → locations (NULL = local avulso) |
| custom_location_name | TEXT | NULL |
| custom_lat | REAL | NULL |
| custom_lng | REAL | NULL |
| radius_meters | INTEGER | NOT NULL DEFAULT 10000 |
| starts_at | TEXT | NOT NULL (agora + 2h padrão) |
| capacity_limit | INTEGER | NULL |
| confirmed_count | INTEGER | NOT NULL DEFAULT 0 |
| status | TEXT | NOT NULL DEFAULT 'active' (active, completed, cancelled) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `flash_party_rsvps`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| flash_party_id | TEXT | NOT NULL, FK → flash_parties |
| user_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'confirmed' (confirmed, waitlist) |
| confirmed_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/flash-party                    → criar festa relâmpago (requer reputação)
GET   /api/v1/flash-party/active              → festas relâmpago ativas perto
POST  /api/v1/flash-party/{id}/rsvp           → confirmar presença
DELETE /api/v1/flash-party/{id}/rsvp          → cancelar presença
GET   /api/v1/flash-party/{id}                → detalhes + confirmados (contagem)
POST  /api/v1/flash-party/{id}/album           → postar foto no álbum coletivo
GET   /api/v1/flash-party/{id}/album           → álbum coletivo (só confirmados)
```

---

### 49. Termômetro da Noite

Gauge em tempo real: "Como tá a night?" — votação ao vivo por quem está no local.

- Usuários com checkin ativo em um	local podem votar no "Termômetro": escala de 1 a 5 🔥
  - 🥶 Morto (1) — vazio, sem clima
  - 😐 Morno (2) — gente chegando
  - 🙂 Animado (3) — rolando
  - 😈 Quente (4) — bom clima
  - 🔥🔥🔥 Loteou (5) — lotado, rolando solto
- Média das votações aparece como badge no mapa: "🔥🔥🔥 4.2 no [local]"
- Atualiza a cada 5 minutos (média móvel dos últimos 15min)
- Votação anônima — ninguém sabe quem votou o quê
- **FOMO engine**: se um local passar de 4.0, notificação push para quem está na região: "🔥 [Local] tá bombando agora — 47 pessoas, termômetro 4.2"
- Histórico do termômetro por local: mostra picos históricos ("sábado às 23h é sempre 🔥")
- Free users: votam 1x por local por noite; Premium: votam a cada 30min (acompanha evolução)

### Schema — `night_thermometer_votes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| location_id | TEXT | NOT NULL, FK → locations |
| user_id | TEXT | NOT NULL, FK → users |
| score | INTEGER | NOT NULL (1-5) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/thermometer/{location_id}/vote    → votar (requer checkin ativo)
GET   /api/v1/thermometer/{location_id}         → média atual + tendência
GET   /api/v1/thermometer/hot                    → locais com termômetro > 4.0 agora
GET   /api/v1/thermometer/{location_id}/history  → histórico por horário/dia da semana
```

---

### 50. Mural de Desejos Anônimos

Usuários postam desejos anônimos — a comunidade pode "atender".

- Usuário posta: "Procuro casal para same room em [cidade] nesse fim de semana" (max 200 chars)
- Post é **100% anônimo** — ninguém sabe quem postou
- Comunidade pode reagir: 🔥 (topa), 👀 (curioso), 🙏 (tô dentro), ou ignorar
- Se o autor ver uma reação que lhe interessa, pode "revelar" para aquela pessoa → envia convite privado
- Até a revelação, o autor é "Desejante Anônimo #X3K"
- Desejos expiram em 72h (urgência!)
- Categorias: mesma noite, fim de semana, viagem, primeira vez, específico (BDSM, voyeur, etc.)
- Moderação: robô filtra conteúdo proibido; se passar, aparece imediatamente
- **Mural da cidade**: filtrar por cidade/raio
- Badge: "Desejo Realizado" quando autor marca que aconteceu (opcional)

### Schema — `wishes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| author_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL — max 200 chars |
| category | TEXT | NOT NULL (tonight, weekend, travel, first_time, specific) |
| city | TEXT | NOT NULL |
| anonymous_alias | TEXT | NOT NULL — "Desejante #X3K" |
| status | TEXT | NOT NULL DEFAULT 'active' (active, fulfilled, expired, removed) |
| expires_at | TEXT | NOT NULL — 72h após created_at |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `wish_reactions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| wish_id | TEXT | NOT NULL, FK → wishes |
| user_id | TEXT | NOT NULL, FK → users |
| emoji | TEXT | NOT NULL (🔥, 👀, 🙏) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `wish_reveals`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| wish_id | TEXT | NOT NULL, FK → wishes |
| reactor_user_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, accepted, rejected) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/wishes                    → criar desejo anônimo
GET    /api/v1/wishes                    → mural de desejos (filtra por cidade/categoria)
DELETE /api/v1/wishes/{id}               → remover próprio desejo
POST   /api/v1/wishes/{id}/react         → reagir (🔥/👀/🙏)
POST   /api/v1/wishes/{id}/reveal/{user_id} → revelar identidade para um reagente
GET    /api/v1/wishes/mine              → meus desejos ativos
POST   /api/v1/wishes/{id}/fulfilled     → marcar como realizado
```

---

### 51. Despertador Liberal

Agenda de "disponibilidade" que dispara notificações no horário combinado — cria antecipação e planejamento.

- Casal/indivíduo define: "Estamos livres sexta das 22h às 02h"
- 30 min antes: notificação push para matches compatíveis e nearby: "⏰ [Casal X] fica livre em 30min — é a hora!"
-match que também marcou disponibilidade no mesmo horário recebe match prioritário: "Vocês estão livres ao mesmo tempo!"
- Semana visível: calendar strip de "janelas livres" — usuário marca slots de disponibilidade
- Modo "disponível agora" (instantâneo): flag que dura 2h, visível no radar como 🔔
- Histórico de disponibilidade ajuda a identificar padrões de comportamento da base (Sábado à noite é pico)
- **Termômetro de disponibilidade**: barra horizontal na tela do radar: "23 casais disponíveis agora | 78 amanhã à noite | 12 hoje à tarde"

### Schema — `availability_windows`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| day_of_week | TEXT | NULL (mon-sun) — NULL para date específica |
| specific_date | TEXT | NULL (YYYY-MM-DD) |
| start_hour | TEXT | NOT NULL ("22:00") |
| end_hour | TEXT | NOT NULL ("02:00") |
| is_recurring | INTEGER | NOT NULL DEFAULT 0 |
| notes | TEXT | NOT NULL DEFAULT '' — "com bebê na sogra, podemos sair!" |
| active | INTEGER | NOT NULL DEFAULT 1 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/availability                 → criar janela de disponibilidade
GET    /api/v1/availability                  → minhas janelas
DELETE /api/v1/availability/{id}             → remover janela
POST   /api/v1/availability/instant          → "disponível agora" (2h)
DELETE /api/v1/availability/instant          → cancelar "disponível agora"
GET    /api/v1/availability/now              → quem está disponível agora
GET    /api/v1/availability/today             → disponibilidade de hoje
GET    /api/v1/availability/heatmap           → heatmap de disponibilidade da base (por horário)
```

---

### 52. Roleta Safada

Gamble/gamificação diária grátis que cria ritual diário e chance de premiação inesperada.

- Todo usuário, 1x/dia, gira a roleta (UI: animação de carrick de cassino)
- Prêmios possíveis:
  - 🎁 1 dia de Premium grátis (20% chance)
  - ⚡ 1 Match Boost grátis (5% chance)
  - 😈 Super like extra hoje (15% chance)
  - 👁️ Ver 1 foto borrada de alguém (10% chance)
  - 🎭 Surgimento: "Lucky match" — alguém com 90%+ compat aparece no seu feed (10% chance)
  - 💎 3 presentes virtuais grátis (5% chance)
  - 💰 R$2 de crédito no Boost (5% chance)
  - 🍃 Nada (30% chance)
- Premium: 2 giros/dia + chances melhores
- XX streak diária (7 dias → 1 giro bônus garantindo prêmio)
- **Gatilho psicológico**: notificação "Sua roleta está pronta!" diariamente → aumenta DAU

### Schema — `roulette_spins`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| prize | TEXT | NOT NULL (premium_day, boost, super_like, reveal_photo, lucky_match, gifts, credit, nothing) |
| streak_day | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/roulette/spin       → girar a roleta
GET   /api/v1/roulette/status     → giro(s) disponível(is) hoje?
GET   /api/v1/roulette/history    → histórico de prêmios
```

---

### 53. Confissão Liberal

Posts totalmente anônimos de confissões/desejos/fantasias — gera conteúdo + discussion.

- Usuário posta confissão: "Eu tenho fantasía de... / "Na última festa eu... / "Meu sonho é..."
- 100% anônimo — sistema usa pseudônimo inatingível: "Confidente #XYZ"
- Comunidade reage com 🔥/😏/😱/🤝 (solidário)/🤔
- Comentários respondem na confissão (também anônimos para comentadores)
- **Acura do "Isso também"**: botão reddit-style "isso também" (~) que soma e mostra "137 pessoas também"
- Moderada: robô filtro + reativa
- Categorias: primeira vez, suruba desconfortável, pós-festa arrependida, fetiche novo, mal-entendido, gesto inesperado de carinho, conta um segredo
- "Confissão em destaque" semanal no feed (curadoria algorítmica por votos)
- Funciona como seguro-termo: ajuda iniciantes verem que outros têm as mesmas fantasias/medos
- Premium: pode reagir com comentário escrito (free só pode reagir com emoji)

### Schema — `confessions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| author_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL — max 500 chars |
| category | TEXT | NOT NULL |
| anonymous_alias | TEXT | NOT NULL — "Confidente #XYZ" |
| upvotes | INTEGER | NOT NULL DEFAULT 0 |
| status | TEXT | NOT NULL DEFAULT 'published' (published, moderated, removed) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `confession_comments`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| confession_id | TEXT | NOT NULL, FK → confessions |
| author_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL |
| is_premium | INTEGER | NOT NULL DEFAULT 0 |
| anonymous_alias | TEXT | NOT NULL |
| moderation_status | TEXT | NOT NULL DEFAULT 'approved' |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/confessions             → confessar
GET    /api/v1/confessions             → feed (por categoria, votação/recência)
POST   /api/v1/confessions/{id}/vote   → votar (🔥/😏/😱/🤝/🤔)
POST   /api/v1/confessions/{id}/me-too → "isso também" (+1 contagem)
POST   /api/v1/confessions/{id}/comments → comentar (Premium)
GET    /api/v1/confessions/featured   → confissão em destaque da semana
```

---

## Eixo 2 — Ferramentas para Veteranços/Influentes

### 54. Padrinho/Madrinha Liberal

Sistema de mentoria: veteranos "adotam" casais iniciantes e os guiam.

- Usuário com Nível 10+ OU selo Anjo OU verificado ao vivo pode se tornar **Padrinho/Madrinha**
- Iniciante (conta < 60 dias OU sem checkins) pode solicitar um padrinho
- Sistema sugere padrinhos por: compatibilidade de fetiches, proximidade, reputação, idioma/estilo
- Padrinho pode: ver perfil completo do afilhado (incremental trust), receber convite para evento do afilhado, enviar mensagens prioritárias, aparecer como "recomendado por [Padrinho]" no perfil do afilhado
- Afilhado ganha: +5 matches/dia enquanto tem padrinho ativo, badge "Protegido por [Nome]", acesso a eventos que exigem indicação
- Padrinho ganha: XP dobrado das interações com afilhado, badge "Padrinho Liberal" (acumula: "1Casal", "5 Casais", "Padrinho de Comunidade" após 10 contínuos)
- Período mínimo: 30 dias. Após, qualquer um pode desvincular
- **Carta de Recomendação**: após 60 dias de mentoria ativa, padrinho pode escrever uma carta pública de recomendação → aparece no perfil do afilhado como "Recomendado por [Nome] (Padrinho)" — transfere credibilidade
- Limite: padrinho pode ter 3 afilhados ativos simultaneamente (Premium: 5)

### Schema — `mentorships`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| mentor_id | TEXT | NOT NULL, FK → users |
| mentee_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'active' (active, completed, cancelled) |
| started_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| ended_at | TEXT | NULL |
| recommendation_written | INTEGER | NOT NULL DEFAULT 0 |

### API

```
POST   /api/v1/mentorship/request              → iniciante solicita padrinho (sugestões automáticas)
POST   /api/v1/mentorship/accept/{request_id}  → padrinho aceita
POST   /api/v1/mentorship/{id}/end             → encerrar mentoria (qualquer um)
POST   /api/v1/mentorship/{id}/recommend       → escrever carta de recomendação (após 60 dias)
GET    /api/v1/mentorship/mine                 → minhas mentorias (como padrinho e afilhado)
GET    /api/v1/mentorship/available-mentors    → padrinhos disponíveis
```

---

### 55. Carta de Apresentação

Substitui a bio seca por uma carta-íntima: iniciativa, expectativas, limites, fotos selecionadas que você CONTROLA que o outro veja primeiro.

- Usuário escreve uma "carta de apresentação" (Markdown bem formatado, até 2000 chars)
- Carta pode ser: pública (todo match vê), privada (só matches), ou "na primeira impressão" (mostrada automaticamente na primeira vez que alguém abre o perfil)
- Anexar até 3 fotos "autorais" — fotos selecionadas especialmente para a carta (não do álbum geral)
- Versões: pode ter versão "público" e versão "matches" (mais detalhada, revela mais para quem já conectou)
- **Videocarta** (Premium/VIP): vídeo curto de 30s com blur facial opcional, mostrando personalidade
- Carta é vinculada ao perfil mas com "selo de atualização": "Atualizada em [data]" — cartas atualizadas recentemente ganham destaque no feed
- Iniciantes favorecidos: 7 primeiros dias com carta publicada ganham +10 boost no ranking

### Schema — `intro_letters`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users (UNIQUE — uma carta por usuário) |
| public_body | TEXT | NOT NULL — versão pública (max 2000 chars) |
| matches_body | TEXT | NULL — versão privada para matches |
| video_path | TEXT | NULL — videocarta (Premium) |
| is_published | INTEGER | NOT NULL DEFAULT 1 |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `letter_photos`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| letter_id | TEXT | NOT NULL, FK → intro_letters |
| image_path | TEXT | NOT NULL |
| caption | TEXT | NOT NULL DEFAULT '' |
| sort_order | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
PUT   /api/v1/intro-letter              → criar/atualizar carta
GET   /api/v1/intro-letter              → minha carta
GET   /api/v1/users/{id}/intro-letter    → carta de outro (respeita visibilidade)
POST   /api/v1/intro-letter/photos        → adicionar foto autoral
DELETE /api/v1/intro-letter/photos/{id}   → remover foto
```

---

### 56. Termômetro de Consenso (Pact Board)

Antes de qualquer encontro, todos os envolvidos marcam o que está em pauta — visual, acordado, accountability.

- Funciona para matches de 2, 3, 4+ pessoas (surubas, ménages)
- Criar um "Pacto": dono define opções multi-seleção:
  - Beijo na boca? (sim/não)
  - Troca de casal? (sim/não)
  - Same room (sexo no mesmo ambiente)? (sim/não)
  - Soft swap (sem penetração cruzada)? (sim/não)
  - Full swap? (sim/não)
  - Voyeurismo? (sim/não)
  - Exibicionismo? (sim/não)
  - BDSM? (se sim: qual Role)
  - Fotos/filmes durante? (sim, com máscara / sim, sem identificação / não)
  - Preservativo OBRIGATÓRIO? (sim — sempre)
  - Roteiro/bebidas ****? (quem leva o quê)
- Cada participante recebe o pacto e marca suas respostas
- **Só revela respostas quando TODOS responderam** (anonimato até completo)
- Exibe: área de concordância (verde) e área de divergência (amarelo) — divergências pedem conversa explícita
- 📝 Pacto assinado: todos veem e concordam → "Pacto Acordado 🔒" badge no chat
- Timestamp e log criptografado de resposta (responsabilidade)
- Pode ser revisto/alterado a qualquer momento (qualquer um pode propor revisão)
- Após encontro: todos podem marcar "aconteceu conforme pacto" (auditoria social pós-evento)

### Schema — `pacts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| creator_id | TEXT | NOT NULL, FK → users |
| title | TEXT | NOT NULL |
| status | TEXT | NOT NULL DEFAULT 'draft' (draft, voting, agreed, revising, completed) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| agreed_at | TEXT | NULL |

### Schema — `pact_participants`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| pact_id | TEXT | NOT NULL, FK → pacts |
| user_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, answered, agreed) |
| invited_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| answered_at | TEXT | NULL |

### Schema — `pact_items`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| pact_id | TEXT | NOT NULL, FK → pacts |
| question | TEXT | NOT NULL ("Beijo na boca?") |
| options | TEXT | NOT NULL — JSON array ["sim", "não", "talvez"] |

### Schema — `pact_answers`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| pact_id | TEXT | NOT NULL, FK → pacts |
| pact_item_id | TEXT | NOT NULL, FK → pact_items |
| user_id | TEXT | NOT NULL, FK → users |
| answer | TEXT | NOT NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/pacts                             → criar pacto
POST   /api/v1/pacts/{id}/invite/{user_id}       → convidar participante
POST   /api/v1/pacts/{id}/answer                 → responder itens (batch)
GET    /api/v1/pacts/{id}                        → ver pacto (só revela respostas se todos responderam)
POST   /api/v1/pacts/{id}/agree                  → concordar com resultado final
POST   /api/v1/pacts/{id}/revise                 → propor revisão (volta para voting)
POST   /api/v1/pacts/{id}/complete               → marcar como "aconteceu conforme pacto"
```

---

### 57. Ranking de Anfitriões

Veteranos que organizam festas ganham reputação como anfitriões — review economy.

- Qualquer usuário pode se registrar como "Anfitrião" (requer verificação ao vivo ou +20 confiança mútua)
- Anfitrião cria evento (suruba, jantar liberal, festa privada)
- Após evento, participantes recebem formulario de review rápido (estrutura de estrelas em 4 categorias):
  - 🍽️ Hospitalidade
  - 😈 Clima
  - 🔒 Discrição
  - 🧹 Higiene e cuidado
- Reviews são anônimas (não revela quem deu a nota)
- Anfitrião tem média visível: "⭐ 4.8 (23 reviews)"
- Ranking de anfitriões por cidade: top 10 destacados
- Anfitriões com >4.5 e >10 reviews ganham badge "Anfitrião Confirmedo" (prata), >4.8 e >30 reviews badge "Anfitrião Premier" (ouro)
- Eventos de anfitriões premier recebem destaque no feed
- **Incentivo a veteranos**: anfitriões premier podem cobrar "ticket simbólico" via plataforma (comissão 10% em pix) — monetiza a influência sem terceiros

### Schema — `hosts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users (UNIQUE) |
| bio | TEXT | NOT NULL DEFAULT '' |
| avg_hospitality | REAL | NOT NULL DEFAULT 0 |
| avg_atmosphere | REAL | NOT NULL DEFAULT 0 |
| avg_discretion | REAL | NOT NULL DEFAULT 0 |
| avg_cleanliness | REAL | NOT NULL DEFAULT 0 |
| total_reviews | INTEGER | NOT NULL DEFAULT 0 |
| badge_tier | TEXT | NOT NULL DEFAULT 'none' (none, confirmed, premier) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `host_reviews`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| host_id | TEXT | NOT NULL, FK → hosts |
| event_id | TEXT | NOT NULL, FK → events |
| reviewer_id | TEXT | NOT NULL, FK → users |
| hospitality | INTEGER | NOT NULL (1-5) |
| atmosphere | INTEGER | NOT NULL (1-5) |
| discretion | INTEGER | NOT NULL (1-5) |
| cleanliness | INTEGER | NOT NULL (1-5) |
| comment | TEXT | NOT NULL DEFAULT '' |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/hosts/register                    → tornar-se anfitrião
GET    /api/v1/hosts                              → listar anfitriões (por cidade)
GET    /api/v1/hosts/{user_id}                     → perfil de anfitrião + reviews
POST   /api/v1/hosts/{user_id}/review              → avaliar anfitrião (após evento)
GET    /api/v1/hosts/ranking                       → top 10 por cidade
```

---

### 58. Cartão de Visita Digital Liberal

Veteranos têm "status" — cartão de visita digital честно transmite quem são e onde circulam, pode ser compartilhado fora do app.

- Gera um Cartão de Visita digital (CSS) com:
  - Nome (ou apelido)
  - Selo verificado (azul, dourado se verificação ao vivo)
  - Tipo: casal/individual, orientação
  - Cidade
  - Badge principal (Anfitrião Premier, Padrinho, Anjo, etc.)
  - Trust score (CONEXÕES de confiança mútua)
  - Match % com quem está vendo (se ambos têm Liberages)
  - Fetiches principais (4 principais do usuário)
- Pode ser exportado como: link público (com hash criptografado), QR code, ou PDF
- Configurável: escolhe o que mostrar
- **Link funciona sem login**: se alguém receber o link fora do app, vê o cartão em página pública (sem fotos)
- Clicando no cartão → "Conectar no Liberages" → leva pro app/página de cadastro
- **Use case real**: casal em casa de swing conhece outro casal, en vez de trocar Zap (que revela número real), mostra o QR do cartão
- Link expira em 24h por padrão (configurável: 1h, 24h, 7d, permanente)

### Schema — `business_cards`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| share_hash | TEXT | NOT NULL UNIQUE — token URL-safe |
| config | TEXT | NOT NULL DEFAULT '' — JSON: o que mostrar |
| expires_at | TEXT | NULL — NULL = permanente |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/business-card          → gerar cartão
GET    /api/v1/business-card/mine     → meu cartão atual
DELETE /api/v1/business-card          → revogar cartão
GET    /api/v1/public/card/{hash}      → view pública (sem login) do cartão
```

---

### 59. Convite Exclusivo para Iniciantes ("Tô Com Vaga")

Veterano convida iniciante específico para evento privado — transfer de social capital e quebra de gelo institucional.

- Dono de evento privado pode marcar "Tô com vaga para iniciante" — oferta especial
- Dono define perfil desejado: casal novo (<90 dias), verified, primeira vez, etc.
- Sistema aparece na tela de iniciantes compatíveis: "🎁 Um evento privado está com vaga para iniciantes —而他 quer você lá"
- Iniciante que: ✅ aceita → ganha badge "primeira festa via Liberages", convite رایگان
- Iniciante pode aplicar: vê eventos em que foi convidado e pode "se candidatar" para disponibilidade de vaga a iniciantes
- Veterano organizador tem: histórico de iniciantes acolhidos → ponto em selo Anjo da Comunidade
- Após o evento: iniciante avalia como foi a experiência de acolhimento → "como newbie foi acolhido?" (estrelas 1-5)
- Avaliação VAI para perfil do veteranco como "Acolhedor" (badge acolhedor com reviews)

### Schema — `beginner_invites`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| event_id | TEXT | NOT NULL, FK → events |
| host_id | TEXT | NOT NULL, FK → users |
| target_user_id | TEXT | NULL —NULL = convite aberto, atinge todos matches |
| requirements | TEXT | NOT NULL DEFAULT '' —JSON (new_user: true, verified: true, etc.) |
| status | TEXT | NOT NULL DEFAULT 'open' (open, claimed, closed) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `beginner_reviews`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| event_id | TEXT | NOT NULL, FK → events |
| beginner_id | TEXT | NOT NULL, FK → users |
| hospitality_score | INTEGER | NOT NULL (1-5) |
| warmth_score | INTEGER | NOT NULL (1-5) |
| comment | TEXT | NOT NULL DEFAULT '' |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/events/{id}/beginner-invite          → criar vaga para iniciante
GET    /api/v1/beginner-invites/available             → convites disponíveis para mim (iniciante)
POST   /api/v1/beginner-invites/{id}/apply            → se candidatar
POST   /api/v1/beginner-invites/{id}/claim            → aceitar (se foi convidado diretamente)
POST   /api/v1/beginner-invites/{id}/review           → avaliar acolhimento após evento
```

---

## Eixo 3 — Injeção Social Guiada para Iniciantes

### 60. Trilha de Iniciação Liberal

Jornada gamificada dos  primeiros 90 dias — guia novatos passo a passo, com XP e badges de progresso.

- Iniciante (conta < 90 dias OU sem interação inicial) entra automaticamente na "Trilha"
- 10 passos, cada um ensina e quebra gelo:
  - **1. Complete seu perfil** → XP +50, badge "Recebi meu primeiro match"
  - **2. Escolha seus fetiches** → XP +30, libera matching
  - **3. Explore o mapa** → visite 3 locais diferentes na visualização, ganha "Explorador Virtual"
  - **4. Faça seu primeiro checkin** (local público) → XP +100, badge "Estou Aqui"
  - **5. Envie seu primeiro super like** → XP +30
  - **6. Receba/aceite um padrinho** → XP +100, badge "Acolhido"
  - **7. Participe de um grupo/comunidade** → XP +50, badge "Familia liberal"
  - **8. Jogue o Jogo da Verdade com um match** → XP +75, badge "Quebra-Gelo"
  - **9. Crie uma Festa Relâmpago ou participe de uma** → XP +80, badge "Iniciante Sasfada"
  - **10. Escreva sua primeira resenha do primeiro evento** → XP +150, badge "Cronista Liberal", certificado de conclusão da Trilha
- Cada passo tem dica, conteúdo educativo (video/blog/infográfico) e CTA
- **Indo devagar**: prazo flexível, não punitivo. Notificação: "Você completou 7/10 passos — só faltam 3!"
- Após completar: Trilha completa → badge "Liberal Graduado", 500 XP bônus, +3 matches/dia por 30 dias
- **Pais adultos podem ajudar**: casais de 1 a 5 anos entram na Trilha Opcional

### Schema — `onboarding_steps`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| slug | TEXT | NOT NULL UNIQUE |
| title | TEXT | NOT NULL |
| description | TEXT | NOT NULL |
| xp_reward | INTEGER | NOT NULL |
| sort_order | INTEGER | NOT NULL |
| cta_link | TEXT | NOT NULL — rota no app |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `user_onboarding`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| step_slug | TEXT | NOT NULL, FK → onboarding_steps |
| completed | INTEGER | NOT NULL DEFAULT 0 |
| completed_at | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET    /api/v1/onboarding/trail         → status da trilha (passo atual, completos)
POST   /api/v1/onboarding/{step}/complete → marcar passo como completo (interno)
```

---

### 61. FAQ Interativo Liberal Newbie

Premium dos bots educativos: casais veteranos respondem perguntas em vídeo curto que iniciantes veem na Trilha.

- Sistema crowdsource: app pede a usuários experientes (Nível 5+ ou Padrinho) " Grave um video de 30-60s respondendo: Como me comportar em primeira festa?"
- Respostas existem no Centro de Apreh, acessivel aos iniciantes
- Threading: iniciante pergunta, veteranos responde (video com blur opcional), votos ajudam as melhores
- Primavera: iniciantes podem pedir "Pergunta do mês" — veterans respondem
- Premium: video sem blur (veteranos ganham XP por participar)
- Respostas top: entram em destacado no blog "IBGE do Sexo"
- FAQ armazenado: SQLite (title, body ou video_path, voter up/down, tags)

### Schema — `newbie_questions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| author_id | TEXT | NOT NULL, FK → users |
| question | TEXT | NOT NULL |
| category | TEXT | NOT NULL (first_party, etiquette, safety, communication, etc.) |
| status | TEXT | NOT NULL DEFAULT 'open' (open, answered) |
| upvotes | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `newbie_answers`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| question_id | TEXT | NOT NULL, FK → newbie_questions |
| author_id | TEXT | NOT NULL, FK → users (veteran) |
| body | TEXT | NULL |
| video_path | TEXT | NULL — resposta em video, max 60s |
| blur_video | INTEGER | NOT NULL DEFAULT 0 |
| upvotes | INTEGER | NOT NULL DEFAULT 0 |
| is_featured | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/newbie/questions              → iniciante faz pergunta
GET    /api/v1/newbie/questions              → listar perguntas abertas/respondidas
POST   /api/v1/newbie/questions/{id}/answers  → veteran responde (texto ou video)
POST   /api/v1/newbie/answers/{id}/vote       → votar
GET    /api/v1/newbie/featured               → melhores respostas em destaque
```

---

### 62. Cofre de Memórias (Pacto de Limites Pré-Festa)

Iniciantes e veterans compartilham SEUS limites PRÉVIO a encontro — evita mal-entendidos que traumatizam novatos.

- Antes de um evento/encontro, cada participante registra seus limites (privado, criptografado, revelado durante/até após):
  - O quê é proibido
  - O que eu permito
  - Quem performa quem (papéis de trio/suruba)
  - Quem na minha vida NÃO PODE SABER (divulgação proibida)
- Antes do encontro, todos veem os limites de todos
- Após o evento, usuário marca "limites respeitados?" (estrelas 1-5) para cada participante
- Isso incentiva cultura de respeito e previne burnout de novatos traumatizados por eventos mal-acordados
- Reputação: usuários com média de respeito aos limites >90% ganham badge "Cavalheiro/Dama Liberal"

### Schema — `limit_pacts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| event_id | TEXT | NOT NULL, FK → events |
| hard_limits | TEXT | NOT NULL DEFAULT '' |
| soft_limits | TEXT | NOT NULL DEFAULT '' |
| comfort_level | TEXT | NOT NULL ('relaxado', 'médio', 'tenso') |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `respect_reviews`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| event_id | TEXT | NOT NULL, FK → events |
| reviewer_id | TEXT | NOT NULL, FK → users |
| reviewee_id | TEXT | NOT NULL, FK → users |
| respect_score | INTEGER | NOT NULL (1-5) |
| comment | TEXT | NOT NULL DEFAULT '' |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/limits/pact             → registrar meus limites para um evento
GET    /api/v1/limits/event/{id}       → ver limites de todos do evento (só participantes)
POST   /api/v1/limits/review            → avaliar respeito aos limites pós-evento
GET    /api/v1/users/{id}/respect-score → média de respeito aos limites de um usuário
```

---

### 63. Detector de Compatibilidade Noturna

Efeito "looky-loo" (só de curiosidade): tool para iniciantes medirem compatibilidade REAL com alguém antes de tentarem match.

- Iniciante ve um perfil e clica "Verificar Compatibilidade" (botão comoodd Burns)
- Sistema mostra:
  - Match % (fetiches em comum)
  - Disponibilidade compatível (janelas que coincidem)
  - Distância
  - Compatible awards/badges em comum (Anfitrião, Padrinho, Verificado)
  - **Dica algorítmica**: baseado nos históricos de matches do outro, "esta pessoa costuma combinar com casais nível iniciante" / "esta pessoa prefere veterans"
- Sem notificação ao outro (iniciante: sente se tem chance antes de arriscar)
- Premium: mostra estatísticas "Média de matches/dia dessa pessoa é N, X% são iniciantes"
- Iniciante vê tutorial "Como propor match" sobre o perfil: dicas customizadas
- Recurso: "Treinar match" — free user tem 3 clínica-rays por dia onde vê um perfil aleatório + feedback +XP +20 por CLÍNICA compaldo

### Schema — `compatibility_checks`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| checker_id | TEXT | NOT NULL, FK → users |
| target_id | TEXT | NOT NULL, FK → users |
| reason | TEXT | NOT NULL DEFAULT 'curiosity' |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/compatibility/{user_id}      → meu % de compatibilidade com alvo
GET   /api/v1/compatibility/{user_id}/full  → análise Premium (detalhes)
POST  /api/v1/compatibility/practice        → treinar com perfil aleatório
```

---

### 64. Modo "Tô Novo Por Aqui" / Newbie Flag

Flag voluntária que destaca novatos para acolhimento ativo da comunidade.

- Iniciante ativa "Tô novo por aqui" flag no perfil (opt-in, pode desligar quando quiser)
- Efeito no app:
  - Badge 👋 visível no perfil e no mapa
  - Aparecem prioritariamente no feed de "veterans confirmados" (com nova seção: "Quem chegou agora")
  - Recebem bem-vindo de veterans com propostas de "Tô com vaga para iniciante" prioritárias
  - Listados em "Parede novatos" na home de veterans
- Veterans vêem_WALL de novatos e podem "Dar boas-vindas" (envia elogio anônimo + match boost)
- Newbie flag não expira automaticamente — usuário decide quando tirar (sugestão auto alerta aos 90 dias)
- Pode ser ativada por alguém que troca de cidade também: "Tô nova por [SP]"
- Receber 3 boas-vindas: concede badge "Recebi Injeção Social"

### Schema — add column to `users`

| Coluna | Tipo | Notas |
|---|---|---|
| is_newbie_flag | INTEGER | NOT NULL DEFAULT 0 |
| newbie_context | TEXT | NOT NULL DEFAULT '' |

### Schema — `newbie_welcomes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| newcomer_id | TEXT | NOT NULL, FK → users |
| welcomer_id | TEXT | NOT NULL, FK → users |
| message | TEXT | NOT NULL DEFAULT '' |
| reacts_boost | INTEGER | NOT NULL DEFAULT 1 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/newbie/flag                    → ativar/desativar flag
GET    /api/v1/newbie/wall                     → parede de novatos (veterans vê)
POST   /api/v1/newbie/{user_id}/welcome        → dar boas-vindas
GET    /api/v1/newbie/welcomes                 → minhas boas-vindas recebidas
```

---

### 65. Encontro guiado por Especialista

Veteranos podem se voluntariar como "Escaorts do Meio" — presentes em encontros iniciantes como facilitadores neutros.

- Veteranos disponibilizam-se como "Anfitrião de Encontro Iniciante" — presente fisicamente em local público com casal novato para quebra-gelo inicial
- Usuário marca: "Topo escoltar/facilitar encontros iniciantes em [cidade]"
- Iniciante solicita pedido de escolta: simula encontro em café/restaurante com 3 pessoas presentes (dois casais iniciantes + veteran +Algo)
- Veterans respondem e escolhem convocar; posteriormente avaliam
- Apenas ele descobre quem; veteranco possui apenas encaminhar conversa; não interage menos que solicitado
- Tudo é avaliado por todos participantes; fortalece safety
- Eh uma atividade voluntária que ganha XP extra: indicação de "Anfitrião cortisol" (badge)

### Schema — `encounter_escorts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| veteran_id | TEXT | NOT NULL, FK → users |
| city | TEXT | NOT NULL |
| availability_hours | TEXT | NOT NULL — "weeklyevenings, weekends" |
| status | TEXT | NOT NULL DEFAULT 'available' (available, requested, current, inactive) |
| total_escorts | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `encounter_requests`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| requester_id | TEXT | NOT NULL, FK → users |
| veteran_id | TEXT | NOT NULL, FK → users |
| proposed_location | TEXT | NOT NULL |
| proposed_date | TEXT | NOT NULL |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, accepted, completed, cancelled) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/escort/register                → veteranco se registra como escolta
GET    /api/v1/escort/available                → lista de escolas disponíveis por cidade
POST   /api/v1/escort/request                  → iniciante pede escolta
POST   /api/v1/escort/request/{id}/accept      → veteranco aceita
POST   /api/v1/escort/request/{id}/complete    → marcar como concluído
POST   /api/v1/escort/request/{id}/review      → avaliar experiência
```

---

## Eixo 4 — Safadarias (Features Provocantes e Únicas)

### 66. Cofre de Corpo (Body Vault)

Foto de corpo inteiro verificada, anônima, armazenada com *blur completo* — para matches que pedem "verificação de proporcionalidade" sem identificar.

- Usuário envia foto de corpo (sem rosto) → moderação robô valida que é um corpo humano  (safe + adult)
- Foto é armazenada com *blur completo das feições*  (rostão  apenas contorno)
- Foto aparece como "Cofre: Verificado 1" — sem identificar — mostra forma física apenas
- Matches podem solicitar "Desbloquear cofre" → usuário libera ou não (consenso bilateral)
- Foto NUNCA é exposta sem consentimento — apenas silhueta/corpo sem rosto
- Usuário pode ter até 3 cofres com silhuetas diferentes (frente, costas, perfil)
- Remoção a qualquer momento; foto é criptografada no servidor
- Premium: cofre com dimmed (névoa branca) — maiis explícito mas ainda indireto

### Schema — `body_vaults`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| image_path_blur | TEXT | NOT NULL |
| image_path_original | TEXT | NOT NULL — criptografada no servidor |
| vault_type | TEXT | NOT NULL (front, side, back) |
| unlock_permission_level | TEXT | NOT NULL DEFAULT 'match' (nobody, match, premium_match) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `vault_unlock_requests`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| vault_id | TEXT | NOT NULL, FK → body_vaults |
| requester_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, granted, denied) |
| responded_at | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/body-vault                  → subir foto (corpo)
DELETE /api/v1/body-vault/{id}              → remover
GET    /api/v1/body-vault/mine              → meus cofres
POST   /api/v1/body-vault/{id}/request      → pedir desbloqueio
POST   /api/v1/body-vault/request/{id}/grantiction award | → conceder ou negar
```

---

### 67. Roda de Conexão (Event Mode)

Dentro de eventos/locais: funcão que exibe " Compatible Agora" - não precisa combinar prévio,inação serendipity.

- funcional dentro de evento check-in EXECUTÁVEL
- mostra: outros usuários no mesmo local compatíveis (match% > definido)
- "Compatíveis agora": filtra se INTENÇÕES são compatíveis e status "a fim hoje"
- "Tocar para ver": libera perfil; pode tomar iniciativa de mensagem
- Butterly approach: mostra usuário discreto (nome anônimo, perfil só com perguntas curtas sobre leve"
- Milhão de matching: exibe "ão há ninguém compatível neste evento agora" (não frustração)
- Iniciantes: veem top 5 compatíveis no evento com destaque; veterans despontuação por FUNIL diferente, saturation system
- Sistemas premium: advanced filter, busca avançada com vários filtros + verificação status

### Schema — `event_compatible_views`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| viewer_id | TEXT | NOT NULL, FK → users |
| event_id | TEXT | NOT NULL, FK → events |
| target_user_id | TEXT | NOT NULL, FK → users |
| compatibility_pct | REAL | NOT NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/events/{id}/compatible        → fits compatíveis presentes no evento agora
POST  /api/v1/events/{id}/reveal-to/{user_id} → revelar identidade para um compatível
```

---

### 68. Murmurinho Safado

Chat anônimo temporal em eventos: tipo Yik Yak / Secret — sem saber quem é.

- Dentro de um evento (requer chekin), usuário entra no "Murmurinho Safado"
- Pode mandar mensagem 100% anônima para o canal do evento (todos recebem)
- Mensagens expiram em 10 minutos (Ephemero)
- Usuário não vê nomes de quem postou — só pseudónimo rotacionado ("@safadx")
- Tipos de POST:
  - 👀 Declarar interesse: "Procuro casal para same room, tô na mesa 4"
  - 🎲 Puxar assunto: "Quem tá curtindo o set? Quem topa jogar pôquer de strip?"
  - 💭 Confissão de momento: "Tô nervosa, primeira vez aqui, alguém mais?"
  - ⚡ Alerta flash: alguém de SP entrou na festa (geoloc anonimizada por dentro do event)
- Não permite mensagens privadas diretamente — todos veem/seguem (radicalmente iniciantes, depois alguém pode pedir "Quem postou? Identification via moderação")
  - Moderador vê: se necessário, alguém reports ⇒ bans evaluatesHibrido
- Após evento termina: canal arquivado (não volátil), log encriptado no servidor

### Schema — `whisper_messages`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| event_id | TEXT | NOT NULL, FK → events |
| author_id | TEXT | NOT NULL, FK → users |
| anonymous_alias | TEXT | NOT NULL — "@safadx123", rotaciona |
| message_type | TEXT | NOT NULL (interest, topic, confession, alert) |
| body | TEXT | NOT NULL — max 280 chars |
| expires_at | TEXT | NOT NULL — created + 10 min |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/whisper/{event_id}             → postar mensagem (requer checkin)
GET   /api/v1/whisper/{event_id}              → feed do murmurinho (requer checkin)
```

---

### 69. Carta ao Futuro Match

Modelo correspondência — missiva à mesa para a envelope initiative um compat ível perfect-match, 50% romantico + 50% safado.

- Usuário escreve uma carta a ummatch ideal ("Carta ao Futuro Match") 2500 chars max, opc: foto de si (corpo, sem face)
- Carta fica **adormecida** até que apareça alguém cujo match% é >= 90% em todos os fetiches
- Alguém aparece: carta "acorda" e é entregue → recebe notificação " Uma carta抵达 ao seu match perfeito: [Name] escreveu para você"
- Carta pode ser respondida em modo cronológica (escreve de volta, única feature que permite long-form escrito)
- Experiência: notificação rara: carta demora ou eu ene recea = raridade faz valer
- Cartas têm templates:敬佩 instructive "Primeiro beijo", "Ideal safe + mild", "Poema Safado", etc.
- Limite: 1 carta ativa por usuário (Premium: 3)
- Ephemera: após 30 dias sem resposta, carta é devolvida ao remetente ("Carta devolvida") com xp+25

### Schema — `future_letters`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| author_id | TEXT | NOT NULL, FK → users |
| title | TEXT | NOT NULL |
| body | TEXT | NOT NULL — max 2500 chars |
| image_path | TEXT | NULL |
| compatibility_threshold | INTEGER | NOT NULL DEFAULT 90 |
| status | TEXT | NOT NULL DEFAULT 'dormant' (dormant, delivered, responded, returned) |
| delivered_to | TEXT | NULL, FK → users |
| delivered_at | TEXT | NULL |
| expires_at | TEXT | NOT NULL — 30d após entrega sem resposta |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/future-letter              → criar carta
GET    /api/v1/future-letter/mine         → minhas cartas (status)
POST   /api/v1/future-letter/{id}/reply   → responder carta recebida
DELETE /api/v1/future-letter/{id}         → cancelar (se dormant)
```

---

### 70. Pacto de Discrição

Acordo digital de discrição firme entre dois matches — reforça a antímora de tuite/grampar fotos.

- Qualquer match pode propor "Pacto de Discrição" para outro
- Terms referenciados normativamente:
  - "Não vou compartilhar fora desta conversa"
  - "Vou apagar conversas após período X se solicitado"
  - "Não farei screenshots sem consentimento"
  - "Não vou revelar identidade em outros ambientes"
  - "Consentimento verbal/e-tual  é necessário para fotos externas"
- Ambos assinam digitalmente (ack e timestamp no server)
- Pacto aparece como badge no chat entre os dois: "🔒 Pacto de Discrição Ativo"
- Após pacto assinado: tentativa de screenshot dentro do conversation triggered alert severo: "Pacto de Discrição VIOLADO" (registrado no servidor, visível para ambos)
- Violação grave (demonstrada): denúncia → análise automática; repetência CONFIGs objeto Mods: può 3 ban temporário 30d, segundo ban permanent
- Pacto pode dissolvido bilateralmente ( ambos concordam)

### Schema — `discretion_pacts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| requester_id | TEXT | NOT NULL, FK → users |
| recipient_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, active, dissolved, violated) |
| terms_accepted_by_requester | INTEGER | NOT NULL DEFAULT 1 |
| terms_accepted_by_recipient | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| activated_at | TEXT | NULL |

### Schema — `pact_violations`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| pact_id | TEXT | NOT NULL, FK → discretion_pacts |
| violator_id | TEXT | NOT NULL, FK → users |
| violation_type | TEXT | NOT NULL (screenshot, share, expose) |
| detected_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/discretion-pact/propose/{user_id}  → propor pacto
POST  /api/v1/discretion-pact/{id}/accept         → aceitar
POST  /api/v1/discretion-pact/{id}/dissolve       → dissolver (require ambos)
POST  /api/v1/discretion-pact/{id}/report         → denunciar violação
GET   /api/v1/discretion-pact/active               → meus pactos ativos
```

---

### 71. Videobox Blindado

Vídeo de 10s no perfil com proteção máxima — mostra vibe/personalidade sem identificar.

- Usuário grava vídeo curto (máx. 10s): "Amostra da minha vibe"
- Máscara facial automática em tempo real (estilo Snapchat com filtro de silhueta)
- Vídeo mostra movimento, voz, energia — mas nunad identifica rosto
- Aparece no perfil como card "Vídeo Blindado" — só visitantes podem tocar para ver
- Após 3 visualizações, lapse: necessário esperar 24h para ver novamente (evita replay obsessivo)
- Premium: vídeo sem máscara facial (escolha do usuário, com confirmacao dupla)
- Screenshots detectados: notifica o criador + patrulha
- Premium: até 3 videoboxs diferentes (saudação, dancinha, fala)
- Limite: 1 videobox ativo para free users
- Vídeo criptografado no servidor, deletado quando removido ou trocado

### Schema — `blind_videos`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| video_path | TEXT | NOT NULL |
| blur_face | INTEGER | NOT NULL DEFAULT 1 |
| duration_seconds | INTEGER | NOT NULL DEFAULT 10 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `blind_video_views`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| video_id | TEXT | NOT NULL, FK → blind_videos |
| viewer_id | TEXT | NOT NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/videobox                → subir vídeo blindado
DELETE /api/v1/videobox/{id}           → remover
GET    /api/v1/videobox/mine           → meus videoboxs
POST   /api/v1/videobox/{id}/view      → registrar visualização (respeita limite)
```