# Features — Especificação

> Features do Liberages além do mapa interativo e radar de intenção. Cada feature é incremental e segue a stack padrão (Go + SQLite + sqlc + React SPA).

---

## 1. Fotolog Diário

Postagem de foto diária — estilo Fotolog antigo. Usuário posta 1 foto por dia com legenda curta.

- Limite de 1 foto/dia (free) — ilimitado para Premium
- Foto aparece no feed cronológico (fotolog) do usuário
- Blur facial automático opcional
- Comentários habilitados (com moderação)
- Fotos expiram depois de 24h por padrão (configurável pelo usuário: 24h, 7d, permanente)
- Marcadores de intenção/interação podem aparecer no fotolog

### Schema — `daily_photos`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| image_path | TEXT | NOT NULL — caminho no filesystem |
| caption | TEXT | NOT NULL DEFAULT '' |
| blur_enabled | INTEGER | NOT NULL DEFAULT 1 |
| expires_at | TEXT | NULL — NULL = permanente |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/fotolog          → postar foto do dia
GET    /api/v1/fotolog          → feed global (paginado)
GET    /api/v1/fotolog/{user_id} → fotolog de um usuário específico
DELETE /api/v1/fotolog/{id}     → deletar própria foto
```

---

## 2. Álbum

Galerias de fotos organizadas pelo usuário. Diferente do fotolog (que é cronológico e 1/dia), álbum é curadoria do usuário.

- Criar múltiplos álbuns com nome, descrição e capa
- Adicionar fotos a álbuns (upload ou do fotolog)
- Controle de privacidade por álbum: público, só amigos, só verificados, privado
- Blur facial automático opcional por foto
- Comentários e curtidas habilitados (configurável por álbum)
- Máximo de álbuns para free users (ex.: 3), ilimitado para Premium

### Schema — `albums`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| cover_image | TEXT | NOT NULL DEFAULT '' |
| privacy | TEXT | NOT NULL DEFAULT 'public' (public, friends, verified, private) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `album_photos`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| album_id | TEXT | NOT NULL, FK → albums |
| user_id | TEXT | NOT NULL, FK → users |
| image_path | TEXT | NOT NULL |
| caption | TEXT | NOT NULL DEFAULT '' |
| blur_enabled | INTEGER | NOT NULL DEFAULT 1 |
| sort_order | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/albums                 → criar álbum
GET    /api/v1/albums                 → listar próprios álbuns
GET    /api/v1/albums/{id}            → ver álbum (respeita privacidade)
PUT    /api/v1/albums/{id}            → editar álbum
DELETE /api/v1/albums/{id}            → deletar álbum
POST   /api/v1/albums/{id}/photos     → adicionar foto ao álbum
DELETE /api/v1/albums/{id}/photos/{photo_id} → remover foto do álbum
GET    /api/v1/users/{user_id}/albums  → álbuns públicos de um usuário
```

---

## 3. Fórum

Fórum de discussão em tópicos e categorias. Diferente de grupos (que são comunidade contínua), o fórum é estrutura de tópicos.

- Categorias (swing, BDSM, iniciantes, dicas de segurança, eventos, etc.)
- Tópicos criados por usuários (título + corpo + tags)
- Respostas em thread (aninhadas ou planas — definir)
- Votação em posts (upvote/downvote)
- Fixar tópicos (admin/moderador)
- Marcadores: Importante, Resolvido, Em Discussão
- Busca FTS5 no fórum
- Free users: leitura + 5 posts/dia — Premium: ilimitado

### Schema — `forum_categories`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| name | TEXT | NOT NULL |
| slug | TEXT | NOT NULL UNIQUE |
| description | TEXT | NOT NULL DEFAULT '' |
| sort_order | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `forum_topics`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| category_id | TEXT | NOT NULL, FK → forum_categories |
| user_id | TEXT | NOT NULL, FK → users |
| title | TEXT | NOT NULL |
| body | TEXT | NOT NULL |
| tags | TEXT | NOT NULL DEFAULT '' — JSON array |
| is_pinned | INTEGER | NOT NULL DEFAULT 0 |
| status | TEXT | NOT NULL DEFAULT 'open' (open, resolved, locked) |
| views | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `forum_posts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| topic_id | TEXT | NOT NULL, FK → forum_topics |
| user_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL |
| parent_post_id | TEXT | NULL, FK → forum_posts — NULL se resposta direta ao tópico |
| upvotes | INTEGER | NOT NULL DEFAULT 0 |
| downvotes | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET    /api/v1/forum/categories                 → listar categorias
GET    /api/v1/forum/categories/{slug}/topics     → tópicos de uma categoria
POST   /api/v1/forum/topics                      → criar tópico
GET    /api/v1/forum/topics/{id}                 → ver tópico + posts
POST   /api/v1/forum/topics/{id}/posts           → responder tópico
PUT    /api/v1/forum/posts/{id}                  → editar próprio post
DELETE /api/v1/forum/posts/{id}                  → deletar próprio post
POST   /api/v1/forum/posts/{id}/vote             → votar (up/down)
```

---

## 4. Comunidades com Presença Anônima

Grupos/comunidades onde o usuário pode participar de forma anônima — sem revelar identidade para outros membros.

- Criar comunidade com regras, descrição, categoria
- **Modo anônimo opcional** ao entrar — o usuário participa sem nome/foto visíveis
- Mesma pessoa pode ter perfil público em uma comunidade e anônimo em outra
- Posts e comentários no modo anônimo mostram pseudônimo gerado (ex.: "Anônimo #4A7B")
- Pseudônimo é consistente dentro da mesma comunidade (não muda a cada post)
- Moderadores da comunidade veem a identidade real (para moderação)
- Comunidades públicas (entrada livre) e privadas (convite/aprovação)

### Schema — `communities`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| category | TEXT | NOT NULL |
| is_private | INTEGER | NOT NULL DEFAULT 0 |
| created_by | TEXT | NOT NULL, FK → users |
| member_count | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `community_members`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| community_id | TEXT | NOT NULL, FK → communities |
| user_id | TEXT | NOT NULL, FK → users |
| is_anonymous | INTEGER | NOT NULL DEFAULT 0 |
| anonymous_alias | TEXT | NULL — gerado se is_anonymous=1 |
| role | TEXT | NOT NULL DEFAULT 'member' (member, moderator, owner) |
| joined_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `community_posts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| community_id | TEXT | NOT NULL, FK → communities |
| user_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL |
| is_anonymous | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/communities                         → criar comunidade
GET    /api/v1/communities                         → listar (busca + paginação)
GET    /api/v1/communities/{id}                    → ver comunidade
POST   /api/v1/communities/{id}/join              → entrar (com flag anonymous)
POST   /api/v1/communities/{id}/leave             → sair
POST   /api/v1/communities/{id}/posts             → postar na comunidade
GET    /api/v1/communities/{id}/posts              → feed da comunidade
DELETE /api/v1/communities/{id}/posts/{post_id}   → deletar post (próprio ou mod)
```

---

## 5. Moderação Híbrida (Humana + Robô)

Sistema de moderação que combina ação humana e automação (IA/robô) em camadas.

### 5.1 Camadas de Moderação

| Camada | O que faz | Quando |
|--------|----------|--------|
| **Robô (automática)** | Triagem de fotos, detecção de conteúdo proibido (menores, violência), blur facial automático, rate limiting, detecção de spam | Em tempo real, no upload/post |
| **Humana (moderador)** | Decisão em casos limítrofes, apelação de banimentos, revisão de denúncias, approval de ingresso | Assíncrono, em filas |

### 5.2 Regras do Robô

- Foto enviada → IA classifica (safe, blur_needed, reject) → se blur_needed, aplica blur automaticamente; se reject, bloqueia e notifica moderador
- Post/comentário → IA detecta spam, linguagem proibida, links suspeitos → auto-oculta e envia para revisão humana
- Rate limiting perfeito: X posts/min, Y fotos/dia, Z mensagens/hora — configurável por role
- Detecção de conta falsa: padrões de comportamento (registro + imediato post de link, foto genérica, etc.) → flag para revisão humana

### 5.3 Regras Humanas

- Moderador revisa fila de itens flagueados pelo robô
- Decisão final em casos de: banimento, suspensão, remoção de conteúdo
- Apelo: usuário pode contestar decisão → moderador diferente revisa
- Log de moderação: toda ação é registrada (quem, quando, o quê, motivo)

### Schema — `moderation_queue`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| item_type | TEXT | NOT NULL (photo, post, profile, message, account) |
| item_id | TEXT | NOT NULL |
| reason | TEXT | NOT NULL |
| auto_action | TEXT | NOT NULL (blur, hide, block, flag) |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, approved, rejected, escalated) |
| moderator_id | TEXT | NULL, FK → users |
| moderator_notes | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| resolved_at | TEXT | NULL |

### API

```
GET    /api/v1/admin/moderation/queue         → fila de moderação (paginada)
POST   /api/v1/admin/moderation/{id}/approve  → aprovar item
POST   /api/v1/admin/moderation/{id}/reject   → rejeitar/banir item
POST   /api/v1/admin/moderation/{id}/escalate → escalar (caso grave)
GET    /api/v1/admin/moderation/log            → log de ações de moderação
```

---

## 6. Validação Periódica

Verificação de contas em intervalos regulares — não só uma vez no cadastro.

- Validação de idade: re-verificar a cada 6 meses (re-confirmar 18+)
- Validação de identidade (conta verificada): re-validar a cada 12 meses
- Validação de fotos: IA re-checa fotos periodicamente para detectar conteúdo que mudou de classificação
- Notificação ao usuário: "Sua conta precisa de re-validação. Toque para confirmar."
- Conta não re-validada em 30 dias após notificação: perde privilégios (verificação azul, postar, etc.) até re-validar

### Schema — `validation_cycles`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| validation_type | TEXT | NOT NULL (age, identity, photos) |
| last_validated_at | TEXT | NOT NULL |
| next_validation_due | TEXT | NOT NULL |
| status | TEXT | NOT NULL DEFAULT 'valid' (valid, pending, expired) |
| notified_at | TEXT | NULL |

### API

```
GET   /api/v1/validation/status     → ver status de validação do usuário
POST  /api/v1/validation/age        → re-confirmar idade
POST  /api/v1/validation/identity   → re-validar identidade (upload documento + selfie)
```

---

## 7. Ingresso Moderado por Humano

Cadastro/onboarding aprovado manualmente por um humano antes de liberar acesso total à plataforma.

- Usuário se cadastra → conta criada com status `pending_approval`
- Moderador humano revisa: nome, e-mail, foto (se houver), IP, padrões de spam
- Aprovação manual libera a conta → status `active`
- Rejeição: e-mail com motivo e processo de re-tentativa
- Fila de ingresso separada da fila de moderação de conteúdo
- SLA: 24h para aprovação (subir para 48h se volume alto)
- Hack para volume: auto-approvar se robô não flagar + usuário verificado por idade — humano só vê os flagueados

### Schema — adicionar coluna à `users`

| Coluna | Tipo | Notas |
|---|---|---|
| approval_status | TEXT | NOT NULL DEFAULT 'pending_approval' (pending_approval, active, rejected) |
| approved_by | TEXT | NULL, FK → users (moderador) |
| approved_at | TEXT | NULL |

### API

```
GET   /api/v1/admin/approvals          → fila de cadastros pendentes
POST  /api/v1/admin/approvals/{user_id}/approve  → aprovar cadastro
POST  /api/v1/admin/approvals/{user_id}/reject   → rejeitar cadastro (com motivo)
```

---

## 8. Curtir / Não Curtir (Tipo Tinder)

Matching por swipe — curtir ou não curtir perfis, com match quando ambos se curtiram.

- Feed de perfis baseado em critérios (intenção, localização, gênero, etc.)
- Swipe right = curtir, swipe left = não curtir
- Match: ambos curtiram → chat liberado entre os dois
- Limite diário para free users (ex.: 10/dia) — ilimitado para Premium
- Super like (1/dia free, ilimitado Premium) — notifica o outro usuário
- Undo do último swipe (Premium)
- Não mostrar perfis já avaliados (exceto se perfil mudar significativamente)
- Integrado ao radar de intenção: se ambos marcaram "a fim hoje" e curtem, match tem prioridade no chat

### Schema — `swipes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL — quem fez o swipe |
| target_user_id | TEXT | NOT NULL — quem recebeu |
| action | TEXT | NOT NULL (like, dislike, super_like) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

UNIQUE(user_id, target_user_id) — não pode avaliar a mesma pessoa duas vezes

### Schema — `matches`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_a_id | TEXT | NOT NULL, FK → users |
| user_b_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'active' (active, unmatched, blocked) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/swipe/feed          → feed de perfis para avaliar (paginado, baseado em filtros)
POST  /api/v1/swipe/{target_user_id} → avaliar (like/dislike/super_like)
GET   /api/v1/matches              → listar matches ativos
POST  /api/v1/matches/{id}/unmatch → desfazer match
```

---

## 9. Conquistas (Badges)

Sistema de selos/badges que o usuário desbloqueia ao atingir marcos na plataforma.

- Badges visíveis no perfil do usuário (configurável: exibir ou ocultar)
- Categorias: social (matches), exploração (checkins), longevidade (tempo de conta), conteúdo (posts), verificação (segurança)
- Badges conquistados têm data e rarity (bronze, prata, ouro)
- Notificação ao desbloquear novo badge
- Badge pode ser NFT futuramente para usuários que quiserem (futuro distante)

### Lista inicial de badges

| Badge | Condição | Rarity |
|-------|----------|--------|
| Bem-vindo(a) | Criar conta | Bronze |
| Primeiro checkin | Fazer primeiro checkin | Bronze |
| Explorador | Fazer checkin em 5 locais diferentes | Prata |
| Borboleta Social | 10 matches | Prata |
| Frequentador | 30 checkins no total | Prata |
| Veterano | 1 ano de conta | Ouro |
| Fiel | 2 anos de conta | Ouro |
| Vip | 12 meses consecutivos como Premium/VIP | Ouro |
| Verificado | Ter verificação azul | Prata |
| Super Verificado | Verificação por vídeo | Ouro |
| Criador de Conteúdo | 50 fotos no total | Prata |
| Mestre do Álbum | 10 álbuns criados | Prata |
| Fórum Top | 100 posts no fórum | Ouro |
| Multi-fetiche | Participar de 5 comunidades diferentes | Prata |
| Matcher Noturno | 20 matches em modo noturno (23h-5h) | Ouro |
| Cidade Inteira | Checkin em 20 locais diferentes | Ouro |

### Schema — `badges`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| slug | TEXT | NOT NULL UNIQUE — identificador do badge |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL |
| rarity | TEXT | NOT NULL (bronze, silver, gold) |
| icon | TEXT | NOT NULL — caminho do ícone SVG |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `user_badges`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| badge_id | TEXT | NOT NULL, FK → badges |
| awarded_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/badges               → listar todos os badges disponíveis
GET   /api/v1/users/{id}/badges    → badges de um usuário
POST  /api/v1/badges/award         → (admin) conceder badge manualmente
```

---

## 10. Nível de Usuário por XP

Sistema de progressão: interações dão XP, XP desbloqueia níveis, níveis liberam features.

### Fontes de XP

| Ação | XP | Limite |
|------|----|--------|
| Login diário | +5 | 1x/dia |
| Postar no fotolog | +10 | 1x/dia |
| Postar no fórum | +15 | 5x/dia |
| Curtir perfil | +2 | 20x/dia |
| Match | +50 | — |
| Checkin | +20 | 5x/dia |
| Post em comunidade | +10 | — |
| Fazer upload de foto | +5 | 10x/dia |
| Validar identidade | +100 | 1x (grátis na primeira) |
| Receber like | +3 | — |

### Níveis e Bônus

| Nível | XP necessário | Bônus |
|-------|---------------|-------|
| 1 | 0 | Acesso básico |
| 2 | 100 | +5 matches/dia |
| 3 | 300 | Bordas personalizadas no perfil |
| 4 | 600 | +3 fotolog retroativos |
| 5 | 1000 | Badge especial nível 5 no perfil |
| 10 | 5000 | Badge "Veterano Exclusive" |
| 20 | 20000 | Acesso a beta de features futuras |

### Schema — `user_xp`

| Coluna | Tipo | Notas |
|---|---|---|
| user_id | TEXT | NOT NULL, FK → users (PK) |
| xp | INTEGER | NOT NULL DEFAULT 0 |
| level | INTEGER | NOT NULL DEFAULT 1 |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `xp_log`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| action | TEXT | NOT NULL |
| xp_gained | INTEGER | NOT NULL |
| source_id | TEXT | NULL — ID do item relacionado (post, match, etc.) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET  /api/v1/xp                 → meu XP e nível
GET  /api/v1/xp/history         → histórico de ganhos de XP
```

---

## 11. Ranking Semanal de Atividade

Ranking opcional (opt-in) de usuários mais ativos da semana.

- Baseado em: checkins + posts + matches + XP ganho na semana
- Usuário precisa ativar "aparecer no ranking" nas configurações de privacidade
- Top 10 semanal — destaque no feed
- Categorias de ranking: geral, checkins, fórum, comunidades
- Reset semanal (segunda-feira 00:00)
- Prêmio simbólico: badge "Top 10 da Semana" (expira em 7 dias) para quem ficou no topo
- Sem premiação material — incentivo puramente social e gamificado

### Schema — `weekly_ranking`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| category | TEXT | NOT NULL (general, checkins, forum, communities) |
| score | INTEGER | NOT NULL |
| position | INTEGER | NULL — atualizado no fechamento da semana |
| week_start | TEXT | NOT NULL — data de início da semana |
| week_end | TEXT | NOT NULL — data de fim da semana |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/ranking/current     → ranking desta semana
GET   /api/v1/ranking/{week_start} → ranking de uma semana específica
```

---

## 12. Modo Caça ao Tesouro

Gamificação B2B: locais parceiros escondem "tesouros" no mapa que só aparecem quando o usuário está perto geograficamente.

- Parceiro (casa de swing, motel) cria uma oferta/promoção/convite que aparece como tesouro no mapa
- Tesouro só é visível quando usuário está a X km do local
- Usuário "coleta" o tesouro → recebe cupom/voucher/discount
- Tesla limitados no tempo e quantidade
- Monetização B2B: parceiro paga por tesouro criado (CPL — cost per lead)
- Tesouros aparecem no radar como ícones diferenciados (baú, presente)

### Schema — `treasure_hunts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| location_id | TEXT | NOT NULL, FK → locations |
| title | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| reward_type | TEXT | NOT NULL (discount, free_entry, drink, coupon, other) |
| reward_value | TEXT | NOT NULL — "20% off", "entrada grátis", etc. |
| visibility_radius_meters | INTEGER | NOT NULL — raio para aparecer no mapa |
| max_collects | INTEGER | NOT NULL DEFAULT 100 |
| collects_count | INTEGER | NOT NULL DEFAULT 0 |
| expires_at | TEXT | NOT NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `treasure_collects`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| treasure_hunt_id | TEXT | NOT NULL, FK → treasure_hunts |
| user_id | TEXT | NOT NULL, FK → users |
| collected_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/treasures/nearby     → tesouros próximos ao usuário
POST  /api/v1/treasures/{id}/collect  → coletar tesouro
GET   /api/v1/treasures/my         → tesouros que já coletei
POST  /api/v1/admin/treasures      → (admin) criar tesouro
```

---

## 13. Conta Verificada por Vídeo-Chamada

Além da verificação documental, opção de chamada de vídeo curta com moderador para selo "verificado ao vivo".

- Usuário solicita verificação por vídeo
- Moderador recebe notificação e inicia chamada (1:1, dentro da plataforma — WebRTC)
- Verificação: mostrar documento + rosto + confirmar dados da conta
- Duração máxima: 2 minutos
- Após verificação: badge "Verificado ao Vivo" no perfil (dourado, distinto do azul documental)
- Horários de verificação: agendamento via calendário interno
- Verificação é opcional e complementar à verificação documental

### Schema — `video_verifications`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| moderator_id | TEXT | NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'scheduled' (scheduled, in_progress, verified, rejected) |
| scheduled_at | TEXT | NULL |
| verified_at | TEXT | NULL |
| notes | TEXT | NULL |

### API

```
POST  /api/v1/verification/video/request  → solicitar verificação por vídeo
GET   /api/v1/verification/video/status    → status da solicitação
POST  /api/v1/admin/verifications/{id}/start  → (mod) iniciar chamada
POST  /api/v1/admin/verifications/{id}/result  → (mod) resultado (verified/rejected)
```

---

## 14. Zona de Encontro Sugerida com Câmera ao Vivo

Parceria com bares, motéis, casas de swing que têm câmera ao vivo (pública) do ambiente, visível no perfil do local.

- Local parceiro: feed de câmera ao vivo (público, ambiente geral — sem foco em pessoas)
- Usuário pode ver o ambiente ao vivo antes de ir
- "Quem está aqui agora?" — mostra contagem de checkins ativos no local (não identifica pessoas)
- Quando match sugere encontro, plataforma sugere locais próximos com câmera ao vovo
- Parceiro pago tem destaque na sugestão
- Câmera não grava, só transmite ao vivo (sem armazenamento)
- Link direto para o local no mapa com status "aberto agora" + ocupação (cheio/médio/vazio)

### Schema — `live_cameras`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| location_id | TEXT | NOT NULL, FK → locations |
| stream_url | TEXT | NOT NULL — URL da stream (HLS/WebRTC) |
| is_active | INTEGER | NOT NULL DEFAULT 0 |
| occupancy | TEXT | NULL (empty, moderate, full) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/locations/{id}/camera  → stream ao vivo do local (se ativa)
GET   /api/v1/locations/{id}/camera/status → status da câmera
```

---

## 15. Alarme de Screenshots

Detecta screenshot/tela no PWA e notifica o dono do conteúdo (foto do fotolog, foto de álbum, selfie destrutível, mensagem de foto).

- PWA: detecta evento `visibilitychange` + `blur` + timer de inatividade
- Ao detectar possível screenshot, envia notificação ao proprietário do conteúdo: "Alguém pode ter capturado sua foto"
- Conteúdo com blurred ativo recebe notificação automática
- **Não impede** screenshot (técnicamente impossível em PWA) — mas **notifica e constrange**
- Modo Premium: notificação mais rápida + log de tentativas de screenshot
- Selfie destrutível com detecção: se alarme dispara durante exibição, foto some imediatamente

### Schema — `screenshot_alerts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| content_owner_id | TEXT | NOT NULL, FK → users (dono do conteúdo) |
| viewer_id | TEXT | NULL, FK → users (quem viu — pode ser null se não logado) |
| content_type | TEXT | NOT NULL (fotolog, album, selfie, message) |
| content_id | TEXT | NOT NULL |
| detected_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/privacy/screenshot-alerts  → histórico de alertas de screenshot
```

---

## 16. Conta de Confiança Mútua

Dois usuários se marcam como "conheço pessoalmente" e isso aparece no perfil de ambos como reputação social.

- Usuário A solicita confiança mútua do usuário B
- Usuário B aceita → ambos recebem badge "Conhecido(a)" no perfil
- Aparece no perfil: "[Nome] é conhecido(a) por [N] pessoa(s)"
- Desfazer confiança: qualquer um pode remover, notifica o outro
- Abuso: se um usuário for denunciado por X pessoas que removeram confiança, perde a habilidade de receber novas solicitações por 30 dias
- Diferente de "amizade" — é especificamente "conheço essa pessoa fora da plataforma"
- Visível no perfil público apenas como contagem, não lista quem

### Schema — `trust_connections`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_a_id | TEXT | NOT NULL, FK → users |
| user_b_id | TEXT | NOT NULL, FK → users |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, active, removed) |
| requested_by | TEXT | NOT NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NULL |

### API

```
POST  /api/v1/trust/request/{user_id}  → solicitar confiança
POST  /api/v1/trust/{id}/accept        → aceitar
POST  /api/v1/trust/{id}/reject        → rejeitar
POST  /api/v1/trust/{id}/remove        → remover confiança
GET   /api/v1/trust/connections        → listar conexões ativas
GET   /api/v1/users/{id}/trust-count   → quantas pessoas conhecem esse usuário
```

---

## 17. Story (24h)

Foto/vídeo que desaparece em 24 horas — estilo Instagram Stories.

- Foto ou vídeo curto (máx. 30s) com blur facial opcional
- Story some automaticamente após 24h
- Reação rápida: emojis de reação (🔥, 😈, 😍, etc.) — sem comentário escrito (mais leve)
- Visualização: quem viu aparece como "Visto por [N] pessoas" (sem lista detalhada para free; Premium vê lista)
- Opção de responder story com mensagem direta
- Story pode ser vinculado ao radar: "Acabei de postar story, quem quiser ver?" aparece como notificação no radar

### Schema — `stories`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| media_path | TEXT | NOT NULL |
| media_type | TEXT | NOT NULL (photo, video) |
| blur_enabled | INTEGER | NOT NULL DEFAULT 1 |
| duration_seconds | INTEGER | NULL — só para vídeos |
| expires_at | TEXT | NOT NULL — 24h após created_at |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `story_reactions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| story_id | TEXT | NOT NULL, FK → stories |
| user_id | TEXT | NOT NULL, FK → users |
| emoji | TEXT | NOT NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/stories               → postar story
GET   /api/v1/stories/feed           → stories de quem sigo (paginado)
GET   /api/v1/stories/{id}          → ver story específico
GET   /api/v1/stories/{id}/reactions → reações (Premium: quem reagiu)
POST  /api/v1/stories/{id}/react    → reagir com emoji
POST  /api/v1/stories/{id}/reply    → responder story via DM
```

---

## 18. Modo "Tô a fim agora" Broadcast

Versão mais urgente do radar de intenção: notificação push para todos os usuários no raio.

- Disponível para usuários com intenção ativa no radar
- Broadcast: notificação push para todos no raio configurado (não só compatíveis, como no radar normal)
- Conteúdo: "Alguém no seu raio está a fim AGORA — toque para ver"
- Após tocar, abre o radar com destaque (mas não revela identidade)
- Limite: 1x/dia free, 5x/dia Premium
- Cooldown: mínimo 30 minutos entre broadcasts
- Não notifica o mesmo usuário duas vezes no mesmo período de broadcast
- Usuários podem desativar "receber broadcasts" nas configurações

### Schema — `intention_broadcasts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| radius_meters | INTEGER | NOT NULL |
| notified_count | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/radar/broadcast       → enviar broadcast (respeita limite + cooldown)
GET   /api/v1/radar/broadcasts/today → quantos broadcasts hoje
```

---

## 19. Agenda de Eventos Liberais

Calendário com feriados, datas sazonais relevantes ao público liberal e eventos fixos de casas de swing.

- Datas nacionais: Carnaval, Réveillon, Dia dos Namorados, Dia do Sexo (06/09), feriados municipais
- Eventos sazonais: "Semana do Swing", "Réveillon Liberal", "Carnaval Alternativo"
- Eventos fixos de casas de swing (ex.: "Toda quinta é noite de casal no X")
- Cada data no calendário pode ser vinculada a eventos no sistema de eventos
- Notificação opcional: "Faltam 3 dias para o Carnaval — veja eventos perto de você"
- Visualização: calendário mensal com heatmap de atividade esperada

### Schema — `event_calendar`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| date | TEXT | NOT NOT NULL (YYYY-MM-DD) |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| is_recurring | INTEGER | NOT NULL DEFAULT 0 |
| event_id | TEXT | NULL, FK → events — evento vinculado |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/calendar               → calendário completo (mês vigente)
GET   /api/v1/calendar?month=2026-12 → mês específico
```

---

## 20. Lista de Desejos (Bucket List)

Lista privada de "lugares que quero conhecer" com sugestão de match quando outro usuário também tem o mesmo local na lista.

- Usuário adiciona locais do mapa à sua bucket list
- Lista é privada por padrão (ninguém vê) — opcionalmente compartilhável com match
- **Match por bucket list**: se dois usuários têm o mesmo local na lista, ambos recebem notificação: "Você e [nome] querem conhecer o mesmo lugar!"
- Sugestão de encontro automática: "Por que não marcar de ir juntos?"
- Bucket list pode ser ordenada por prioridade
- Notificação quando um local da lista receber novo checkin

### Schema — `bucket_lists`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| location_id | TEXT | NOT NULL, FK → locations |
| priority | INTEGER | NOT NULL DEFAULT 0 |
| notes | TEXT | NOT NULL DEFAULT '' |
| is_shared | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET    /api/v1/bucket-list            → minha bucket list
POST   /api/v1/bucket-list            → adicionar local
DELETE /api/v1/bucket-list/{id}       → remover local
PUT    /api/v1/bucket-list/{id}       → editar prioridade/notas
GET    /api/v1/bucket-list/matches    → usuários com locais em comum
```

---

## 21. Assinatura de Presente

Dar Premium/VIP de presente para outro usuário.

- Qualquer usuário pode comprar 1 mês de Premium ou VIP para outro usuário
- Destinatário recebe notificação: "[Nome] te deu 1 mês de Premium!"
- Presente pode ser anônimo (quem dá escolhe se revela)
- Máximo de presentes recebidos por ano (evita abuso): 12
- Presente não é acumulativo (não empilha meses além da data de expiração atual — estende a partir dela)
- Pagamento via Pix (integrado ao sistema de pagamento)
- Presente pode ser enviado para: match, seguidor, ou por código de presente (QR code para dar em casas de swing)

### Schema — `gift_subscriptions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| gifter_id | TEXT | NOT NULL, FK → users |
| recipient_id | TEXT | NOT NULL, FK → users |
| tier | TEXT | NOT NULL (premium, vip) |
| months | INTEGER | NOT NULL DEFAULT 1 |
| is_anonymous | INTEGER | NOT NULL DEFAULT 0 |
| payment_id | TEXT | NULL — ID do pagamento |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/gift          → dar assinatura de presente
GET   /api/v1/gift/history   → histórico de presentes (dados e recebidos)
```

---

## 22. Modo Casal com Conta Compartilhada

Duas pessoas administram o mesmo perfil de casal, com chat e match compartilhados.

- Conta de casal: duas contas linkadas, mesmo perfil público
- Cada pessoa mantém sua conta individual (para comunidades, fórum, etc.)
- Perfil de casal aparece como [NomeA] & [NomeB]
- Chat de casal unificado: mensagens entram em um inbox compartilhado
- Swipe: ambos precisam dar like para o match contar (double opt-in)
- Desvincular: qualquer um pode desvincular a qualquer momento, perfil de casal é removido
- Verificação de casal: opcional, requer validação de ambos

### Schema — `couple_profiles`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_a_id | TEXT | NOT NULL, FK → users |
| user_b_id | TEXT | NOT NULL, FK → users |
| profile_name | TEXT | NOT NULL — ex.: "Ana & Pedro" |
| display_photo | TEXT | NOT NULL DEFAULT '' |
| is_verified | INTEGER | NOT NULL DEFAULT 0 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/couple/link           → vincular com outro usuário (convite)
POST  /api/v1/couple/accept         → aceitar convite de casal
POST  /api/v1/couple/unlink         → desvincular
GET   /api/v1/couple/profile        → ver perfil de casal
PUT   /api/v1/couple/profile        → editar perfil de casal
```

---

## 23. Marketplace de Criadores

Venda de conteúdo exclusivo dentro da plataforma (fotos, vídeos, packs).

- Criadores (usuários) podem vender conteúdo
- Tipos: foto avulsa, pack de fotos, vídeo, pack de vídeos, assinatura mensal do criador
- Precificação: criador define preço (R$5 a R$200)
- Comissão da plataforma: 15% (igual ao modelo de receita)
- Pagamento via Pix para o criador (plataforma repassa — recebe, deduz comissão, repassa)
- Conteúdo vendido fica acessível na galeria de compras do comprador (não expira)
- Controle de DM por criador: pode exigir "assinante" para mandar mensagem

### Schema — `creator_products`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| creator_id | TEXT | NOT NULL, FK → users |
| product_type | TEXT | NOT NULL (photo, pack, video, video_pack, subscription) |
| title | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| price_cents | INTEGER | NOT NULL — preço em centavos |
| media_paths | TEXT | NOT NULL DEFAULT '' — JSON array para packs |
| is_active | INTEGER | NOT NULL DEFAULT 1 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `creator_purchases`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| product_id | TEXT | NOT NULL, FK → creator_products |
| buyer_id | TEXT | NOT NULL, FK → users |
| price_cents | INTEGER | NOT NULL |
| commission_cents | INTEGER | NOT NULL — 15% |
| payment_id | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/creator/products              → criar produto
GET    /api/v1/creator/products               → listar meus produtos
GET    /api/v1/users/{id}/store               → loja do criador
POST   /api/v1/creator/products/{id}/purchase  → comprar
GET    /api/v1/purchases                      → minhas compras
```

---

## 24. Cartão Fidelidade Digital

Programa de fidelidade: checkins acumulam pontos que viram descontos em locais parceiros.

- A cada checkin em local parceiro: +10 pontos
- 100 pontos = R$5 de desconto em qualquer local parceiro
- Pontos acumulam e não expiram
- Local parceiro define quantos pontos vale um checkin nele (mínimo 5, máximo 20)
- Desconto é aplicado via voucher gerado na plataforma
- Voucher: código alfanumérico que o usuário mostra no local
- Voucher expira em 30 dias após gerado
- Parceiro confirma voucher → pontos são deduzidos
- Dashboard do parceiro: "Usuários com mais pontos no seu local", "Vouchers resgatados"

### Schema — `loyalty_points`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| location_id | TEXT | NOT NULL, FK → locations |
| points | INTEGER | NOT NULL DEFAULT 0 |
| earned_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| expires_at | TEXT | NULL |

### Schema — `loyalty_vouchers`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| location_id | TEXT | NOT NULL, FK → locations |
| code | TEXT | NOT NULL UNIQUE |
| discount_cents | INTEGER | NOT NULL |
| status | TEXT | NOT NULL DEFAULT 'active' (active, redeemed, expired) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| redeemed_at | TEXT | NULL |

### API

```
GET   /api/v1/loyalty/points       → meus pontos
POST  /api/v1/loyalty/redeem       → resgatar voucher (especifica local e valor)
GET   /api/v1/loyalty/vouchers     → meus vouchers
```

---

## 25. Sistema de Tickets de Suporte

Sistema de suporte integrado à plataforma para que usuários reportem problemas.

- Categorias: denúncia, problema técnico, dúvida sobre conta, cobrança, sugestão
- Prioridade: normal (free) → alta (Premium/VIP)
- Ticket inclui: categoria + assunto + descrição + prints (upload)
- Moderador/suporte responde via sistema (não por e-mail externo)
- Notificação push ao usuário quando ticket for respondido
- Fechar ticket: usuário confirma resolução, ou suporte fecha após X dias sem resposta

### Schema — `support_tickets`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| category | TEXT | NOT NULL |
| subject | TEXT | NOT NULL |
| description | TEXT | NOT NULL |
| status | TEXT | NOT NULL DEFAULT 'open' (open, in_progress, waiting_user, resolved, closed) |
| priority | TEXT | NOT NULL DEFAULT 'normal' (normal, high) |
| assigned_to | TEXT | NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `support_messages`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| ticket_id | TEXT | NOT NULL, FK → support_tickets |
| user_id | TEXT | NOT NULL, FK → users |
| message | TEXT | NOT NULL |
| attachment_path | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/support/tickets          → abrir ticket
GET   /api/v1/support/tickets          → meus tickets
GET   /api/v1/support/tickets/{id}     → ver ticket + mensagens
POST  /api/v1/support/tickets/{id}/messages → responder ticket
POST  /api/v1/support/tickets/{id}/close    → fechar ticket
```

---

## 26. Análise de Sentimento da Comunidade

Dashboard de métricas de moderação com análise de sentimento.

- Painel admin com métricas ao vivo:
  - Denúncias recebidas (últimas 24h, 7d, 30d)
  - Taxa de resolução
  - Tempo médio de resposta
  - Sentimento geral (positivo/negativo/neutro — baseado em posts, comentários, denúncias)
- Gráficos de tendência (semanal, mensal)
- Alertas: pico de denúncias, aumento de conteúdo reportado de um tipo específico
- Exportação de relatório mensal em CSV

### Schema — `moderation_analytics`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| period_start | TEXT | NOT NULL |
| period_end | TEXT | NOT NULL |
| total_reports | INTEGER | NOT NULL DEFAULT 0 |
| resolved | INTEGER | NOT NULL DEFAULT 0 |
| avg_response_time_hours | REAL | NOT NULL DEFAULT 0 |
| sentiment_score | REAL | NULL — -1 a 1 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/admin/analytics/moderation         → dashboard de moderação
GET   /api/v1/admin/analytics/moderation/export   → exportar CSV
```

---

## 27. Painel Público de Status

Página pública de status da plataforma: `status.liberages.com`.

- Mostra uptime, incidentes atuais e históricos
- Componentes monitorados: API, mapa, radar, fórum, upload de fotos
- Status por componente: operacional / degradado / indisponível
- Incidentes: título, descrição, status (investigando, identificado, resolvido), timeline
- Histórico de uptime: 30 dias, 90 dias, 1 ano
- Feed RSS de incidentes

### Schema — `status_components`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| name | TEXT | NOT NULL |
| slug | TEXT | NOT NULL UNIQUE |
| status | TEXT | NOT NULL DEFAULT 'operational' (operational, degraded, down) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `status_incidents`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| title | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| severity | TEXT | NOT NULL (minor, major, critical) |
| status | TEXT | NOT NULL DEFAULT 'investigating' (investigating, identified, monitoring, resolved) |
| component_ids | TEXT | NOT NULL DEFAULT '' — JSON array |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| resolved_at | TEXT | NULL |

### API

```
GET   /api/v1/status         → status atual de todos os componentes
GET   /api/v1/status/history → histórico de incidentes
```

---

## 28. Convite por QR Code

Gerar QR code para convidar outro usuário presencialmente (ex.: em casa de swing, scaneia e se conectam).

- Na página do perfil do usuário: "Gerar QR Code de Conexão"
- QR code contém ID criptografado do usuário + timestamp (+ expira em 5 minutos)
- Outro usuário escaneia → pedido de amizade/conexão enviado
- Opções ao escanear: "Adicionar como amigo", "Enviar mensagem", "Dar match"
- QR code pode ser gerado para perfil individual ou perfil de casal
- Também pode ser gerado pelo local (casa de swing) → "Escaneie para fazer check-in aqui"

### Schema — `qr_codes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| location_id | TEXT | NULL, FK → locations — se QR for de checkin |
| action | TEXT | NOT NULL (connect, checkin, friend) |
| encrypted_data | TEXT | NOT NULL |
| expires_at | TEXT | NOT NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/qr/generate         → gerar QR code
POST  /api/v1/qr/scan             → escanear QR code (recebe encrypted_data devolvendo ação)
```