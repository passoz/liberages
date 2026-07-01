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

---

## 29. Mural de Elogio Anônimo

Usuários podem enviar elogios anônimos para perfis — moderados antes de aparecerem.

- Qualquer usuário pode enviar um elogio para outro (texto curto, máx. 200 caracteres)
- Elogio é anônimo — quem recebe **nunca** sabe quem enviou
- Elogio passa por moderação (robô + humano) antes de aparecer
- Usuário pode aceitar ou rejeitar cada elogio
- Se aceito, aparece num "Mural de Elogios" no perfil público
- Após 10 elogios aceitos: badge "Admiradx" no perfil
- Após 50: badge "Cobiçadx"
- Após 100: badge "Musa/Muse da Comunidade"
- Um elogio por pessoa por mês (evita abuso)
- Usuário pode bloquear de receber elogios (configuração de privacidade)

### Schema — `compliments`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| sender_id | TEXT | NOT NULL, FK → users |
| recipient_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL — max 200 chars |
| moderation_status | TEXT | NOT NULL DEFAULT 'pending' (pending, approved, rejected) |
| is_accepted | INTEGER | NOT NULL DEFAULT 0 |
| accepted_at | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/compliments                   → enviar elogio anônimo
GET   /api/v1/compliments/inbox             → elogios recebidos (pendentes de aceitação)
POST  /api/v1/compliments/{id}/accept       → aceitar elogio no mural
POST  /api/v1/compliments/{id}/reject       → rejeitar elogio
GET   /api/v1/users/{user_id}/compliments   → mural público de elogios
```

---

## 30. Jogo da Verdade Liberal

Mini-game interativo dentro do match: perguntas de quebra-gelo com respostas reveladas só depois de ambos responderem.

- Após um match, qualquer um dos dois pode iniciar o Jogo da Verdade
- Sistema seleciona 5 perguntas de um banco curado (categorias: leve, médio, picante)
- Cada pergunta é enviada aos dois — cada um responde separadamente
- Resposta do outro só é revelada depois que ambos responderam aquela pergunta
- Se a resposta combina (ex.: ambos "sim"), destaca em verde com emoji 🔥
- Ao final, mostra resumo das respostas combinadas
- Novas perguntas adicionadas semanalmente pela moderação
- Usuários podem sugerir perguntas (passam por curadoria)

### Banco inicial de perguntas

| Leve | Médio | Picante |
|------|-------|---------|
| Qual seu lugar favorito na cidade? | Topa um encontro às cegas? | Qual seu fetiche mais secreto? |
| O que te trouxe ao Liberages? | Prefere day use ou pernoite? | Já fez menage? |
| Prefere praia ou montanha? | Curte exibicionismo? | Qual seu filme adulto favorito? |
| Qual seu hobby? | Já frequentou casa de swing? | Topa ser dominado(a)? |

### Schema — `truth_game_questions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| category | TEXT | NOT NULL (mild, medium, spicy) |
| question | TEXT | NOT NULL |
| is_active | INTEGER | NOT NULL DEFAULT 1 |
| suggested_by | TEXT | NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `truth_game_sessions`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| match_id | TEXT | NOT NULL, FK → matches |
| status | TEXT | NOT NULL DEFAULT 'in_progress' (in_progress, completed) |
| started_by | TEXT | NOT NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| completed_at | TEXT | NULL |

### Schema — `truth_game_answers`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| session_id | TEXT | NOT NULL, FK → truth_game_sessions |
| question_id | TEXT | NOT NULL, FK → truth_game_questions |
| user_id | TEXT | NOT NULL, FK → users |
| answer | TEXT | NOT NULL |
| answered_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/match/{match_id}/truth/start     → iniciar jogo
GET   /api/v1/match/{match_id}/truth/next      → próxima pergunta
POST  /api/v1/match/{match_id}/truth/answer    → responder pergunta
GET   /api/v1/match/{match_id}/truth/results   → ver resultados
```

---

## 31. Lista de Fetiches com Match %

Catálogo curado de fetiches/interesses — usuário marca os seus, sistema calcula compatibilidade.

- Lista curada de 30-50 fetiches com nome e descrição curta
- Categorias: troca de casal, ménage, BDSM, voyeur/exib, fantasia, grupo, romântico
- Usuário marca seus fetiches (mínimo 1, máximo 15)
- Match % calculado: quantos em comum / total médio de ambos
- Exibido no perfil: "Compatibilidade com [nome]: 85% 🔥"
- Feed de swipe ordenado por compatibilidade decrescente
- Filtro no swipe: "Só mostrar perfis com >70% compatibilidade"
- Premium: ver os fetiches em comum antes do match
- Atualizar a lista não perde XP (fetiches podem evoluir)

### Schema — `fetish_catalog`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| slug | TEXT | NOT NULL UNIQUE |
| name | TEXT | NOT NULL |
| description | TEXT | NOT NULL DEFAULT '' |
| category | TEXT | NOT NULL |
| sort_order | INTEGER | NOT NULL DEFAULT 0 |

### Schema — `user_fetishes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| fetish_id | TEXT | NOT NULL, FK → fetish_catalog |
| intensity | INTEGER | NOT NULL DEFAULT 5 — 1-10, o quanto curte |

### API

```
GET   /api/v1/fetishes                    → catálogo completo
PUT   /api/v1/users/me/fetishes           → atualizar meus fetiches (lista de IDs)
GET   /api/v1/users/me/fetishes           → meus fetiches marcados
GET   /api/v1/users/{id}/compatibility    → % compatibilidade com outro usuário
```

---

## 32. Mood do Momento

Status temporário que aparece no perfil e no radar com emoji + texto curto.

- Mood dura 1 hora (ou até o usuário mudar/cancelar)
- Opções pré-definidas com emoji + label + cor:

| Emoji | Label | Cor |
|-------|-------|-----|
| 🙈 | Tímido | Rosa |
| 🔥 | Afim agora | Vermelho |
| 😈 | Brutal | Roxo |
| 🥰 | Romântico | Pink |
| 👀 | Só de olho | Azul |
| 🎭 | Modo mistério | Preto |
| 🍸 | Night out | Laranja |
| 🏡 | Day use | Verde |
| 💬 | Bate-papo | Cinza |

- Ao mudar mood, notificação no radar para matches ativos
- Mood aparece como badge no perfil + ícone flutuante no mapa
- Não precisa estar com intenção ativa pra ter mood (são independentes)
- Premium: mood personalizado (texto livre de até 50 chars)

### Schema — `moods`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| slug | TEXT | NOT NULL UNIQUE |
| emoji | TEXT | NOT NULL |
| label | TEXT | NOT NULL |
| color | TEXT | NOT NULL |

### Schema — `user_mood`

| Coluna | Tipo | Notas |
|---|---|---|
| user_id | TEXT | NOT NULL (PK), FK → users |
| mood_id | TEXT | NOT NULL, FK → moods |
| custom_text | TEXT | NULL — só para Premium |
| expires_at | TEXT | NOT NULL — 1h após created |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/moods                    → lista de moods disponíveis
PUT   /api/v1/users/me/mood           → definir mood atual
DELETE /api/v1/users/me/mood           → limpar mood
GET   /api/v1/users/{id}/mood         → ver mood de outro usuário
```

---

## 33. Kit de Relacionamento

Perfil mostra o estilo de relação do usuário — evita mal-entendidos.

- Campos obrigatórios no perfil (visíveis para outros usuários):
  - **Status:** solteiro(a), em casal aberto, casal liberal, poliamor, relacionamento aberto, solteiro(a) mas buscando casal
  - **O que busca:** troca de casal, ménage (H), ménage (M), mulher solteira, homem solteiro, casal, amizade liberal, bate-papo
  - **Frequência:** novato, esporádico, frequente, hardcore
- Filtro de busca por kit de relacionamento
- Match sugerido prioriza compatibilidade de busca

### Schema — adicionar colunas à `users`

| Coluna | Tipo | Notas |
|---|---|---|
| relationship_status | TEXT | NOT NULL DEFAULT '' |
| seeking | TEXT | NOT NULL DEFAULT '' — JSON array |
| frequency | TEXT | NOT NULL DEFAULT '' |
| orientation | TEXT | NOT NULL DEFAULT '' |

### API

```
PUT  /api/v1/users/me/kit        → atualizar kit de relacionamento
GET  /api/v1/users/me/kit        → ver meu kit
GET  /api/v1/users/{id}/kit      → ver kit de outro
```

---

## 34. Contos Eróticos

Seção de literatura erótica escrita pelos próprios usuários — conteúdo de alto engajamento e baixo custo.

- Usuários escrevem e publicam contos (título + corpo + tags + categoria)
- Categorias: swing, BDSM, ménage, exibicionismo, romântico, fantasia, iniciante, experimental
- Tags por fetiche/posição/cenário
- Votação da comunidade (upvote/downvote)
- Comentários com moderação (robô filtra spam, humano revisa)
- Destaque semanal no feed: "Conto em Destaque — [título]"
- Top 1 do mês ganha badge "Escritor(a) do Mês"
- Editor de texto simples (Markdown básico: negrito, itálico, parágrafo)
- Moderação: contos passam por robô primeiro (detecção de menores, violência explícita, não-consentimento)
- Limite: 1 conto/semana free, 3/semana Premium, ilimitado VIP
- Leitura sem limites para todos os tiers

### Schema — `stories`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| title | TEXT | NOT NULL |
| body | TEXT | NOT NULL |
| category | TEXT | NOT NULL |
| tags | TEXT | NOT NULL DEFAULT '' — JSON array |
| status | TEXT | NOT NULL DEFAULT 'draft' (draft, published, moderated, rejected) |
| upvotes | INTEGER | NOT NULL DEFAULT 0 |
| downvotes | INTEGER | NOT NULL DEFAULT 0 |
| views | INTEGER | NOT NULL DEFAULT 0 |
| published_at | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `story_comments`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| story_id | TEXT | NOT NULL, FK → stories |
| user_id | TEXT | NOT NULL, FK → users |
| body | TEXT | NOT NULL |
| moderation_status | TEXT | NOT NULL DEFAULT 'approved' (pending, approved, rejected) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST   /api/v1/stories                     → publicar conto
GET    /api/v1/stories                     → feed de contos (paginado)
GET    /api/v1/stories/{id}                → ler conto
PUT    /api/v1/stories/{id}                → editar próprio conto
DELETE /api/v1/stories/{id}                → deletar próprio conto
POST   /api/v1/stories/{id}/vote           → votar (up/down)
GET    /api/v1/stories/{id}/comments       → comentários
POST   /api/v1/stories/{id}/comments       → comentar
GET    /api/v1/stories/featured            → conto em destaque da semana
```

---

## 35. Desafio Liberal Semanal

Missões semanais que mantêm engajamento alto e geram conteúdo orgânico.

- Toda segunda-feira: novo desafio é postado (via admin — pode ser automatizado)
- Tipos de desafio:
  - **📸 Foto criativa:** "Poste uma foto com algo vermelho"
  - **📍 Checkin:** "Faça checkin em um lugar que nunca foi"
  - **✍️ Conto:** "Escreva um conto de no máximo 500 caracteres sobre primeira vez"
  - **💬 Fórum:** "Responda 3 tópicos no fórum essa semana"
  - **🎭 Kit:** "Atualize seu kit de relacionamento"
  - **🔥 Mood:** "Use um mood picante por pelo menos 1h"
- Quem completa: XP bônus (100 XP) + badge temporário "Desafiador(a) da Semana"
- No final do mês: quem completou todos os 4 desafios do mês ganha badge "Mestre dos Desafios" (permanente)
- Desafio é opcional — não punitivo

### Schema — `weekly_challenges`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| title | TEXT | NOT NULL |
| description | TEXT | NOT NULL |
| type | TEXT | NOT NULL (photo, checkin, story, forum, kit, mood) |
| xp_reward | INTEGER | NOT NULL DEFAULT 100 |
| week_start | TEXT | NOT NULL |
| week_end | TEXT | NOT NULL |
| is_active | INTEGER | NOT NULL DEFAULT 1 |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `user_challenges`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| challenge_id | TEXT | NOT NULL, FK → weekly_challenges |
| completed | INTEGER | NOT NULL DEFAULT 0 |
| completed_at | TEXT | NULL |

### API

```
GET   /api/v1/challenges/current   → desafio da semana
GET   /api/v1/challenges/history    → histórico de desafios completados
```

---

## 36. Moodboard do Casal

Painel colaborativo privado para o casal — como um scrapbook da relação.

- Disponível apenas para contas de casal
- Adicionar: fotos, texto, links, locais favoritos, datas importantes
- Ordem cronológica reversa (ou drag-to-reorder)
- Privado entre os dois — ninguém mais vê
- Exportável em PDF (Premium)
- "Melhores momentos": modo carrossel que mostra só as fotos
- Lembretes: "Vocês se conheceram há 3 meses 🎉" (baseado na data de match do casal)

### Schema — `couple_moodboards`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| couple_profile_id | TEXT | NOT NULL, FK → couple_profiles |
| item_type | TEXT | NOT NULL (photo, text, location, date) |
| content | TEXT | NOT NULL — texto ou caminho ou JSON |
| sort_order | INTEGER | NOT NULL DEFAULT 0 |
| created_by | TEXT | NOT NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/couple/moodboard            → ver moodboard
POST  /api/v1/couple/moodboard            → adicionar item
PUT   /api/v1/couple/moodboard/{id}       → editar item
DELETE /api/v1/couple/moodboard/{id}      → remover item
POST  /api/v1/couple/moodboard/export     → exportar PDF
```

---

## 37. Modo "Falso" (Botão de Emergência)

Se alguém pegar seu celular aberto no Liberages, um botão/bip leva instantaneamente pra uma tela genérica.

- Atalho configurável: agitar o celular, botão flutuante, ou um padrão de toque
- Ao ativar, imediatamente:
  1. Fecha sessão atual (invalida o cookie/token)
  2. Redireciona para tela falsa configurada (calculadora, clima, tela de login genérica)
  3. Salva estado anterior para retomada segura depois
- Telas falsas disponíveis:
  - Calculadora funcional
  - Previsão do tempo (genérica)
  - Tela de login de outro serviço
  - App de notas
- Configurador de tela falsa: usuário escolhe qual e personaliza
- Timer de retomada: após 30s na tela falsa, pode desbloquear com PIN/biometria e voltar exatamente onde estava

### Schema — `panic_config`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL (PK), FK → users |
| is_enabled | INTEGER | NOT NULL DEFAULT 1 |
| trigger_type | TEXT | NOT NULL DEFAULT 'shake' (shake, floating_button, tap_pattern) |
| fake_screen | TEXT | NOT NULL DEFAULT 'calculator' (calculator, weather, login, notes) |
| resume_pin_hash | TEXT | NULL |

### API

```
GET   /api/v1/privacy/panic/config       → ver config atual
PUT   /api/v1/privacy/panic/config       → atualizar config
POST  /api/v1/privacy/panic/trigger      → simular ativação (teste)
```

---

## 38. Login Rápido com PIN

Além do login normal, PIN de 4 dígitos para sessões curtas.

- Usuário configura PIN de 4 dígitos nas configurações de segurança
- Login rápido: na página de login, além de email+senha, campo de PIN
- PIN gera token de curta duração (15 minutos)
- Útil para abrir rápido num intervalo, no trabalho, etc.
- PIN não substitui senha — é complementar
- Após 3 tentativas erradas de PIN, bloqueia por 30 minutos
- Pode resetar o PIN via login completo (email+senha)
- PIN armazenado como hash (bcrypt), igual à senha

### Schema — adicionar coluna à `users`

| Coluna | Tipo | Notas |
|---|---|---|
| pin_hash | TEXT | NULL — hash bcrypt do PIN |
| pin_failed_attempts | INTEGER | NOT NULL DEFAULT 0 |
| pin_blocked_until | TEXT | NULL |

### API

```
POST  /api/v1/auth/pin/set        → definir/alterar PIN
POST  /api/v1/auth/pin/login      → login com PIN (retorna token 15min)
POST  /api/v1/auth/pin/reset      → resetar PIN (requer login completo)
```

---

## 39. Criptografia Ponta a Ponta nas DMs

Todas as mensagens diretas criptografadas no dispositivo — nem a plataforma consegue ler.

- Implementação: Web Crypto API + curva elíptica (ECDH) no client
- Chaves geradas no dispositivo no momento do cadastro
- Chave pública armazenada no servidor (não criptografada — é pública)
- Chave privada armazenada no IndexedDB do navegador (nunca enviada ao servidor)
- Ao enviar DM: client pega chave pública do destinatário → criptografa mensagem → envia ciphertext
- Ao receber: client descriptografa com chave privada local
- Fallback: se chave privada for perdida (limpeza de cache), servidor armazena últimas N mensagens não lidas de forma inacessível até nova geração de chaves
- Chat em grupo (comunidade): chave compartilhada entre os membros, rotacionada a cada novo membro
- Mensagens criptografadas também no backup (o backup contém ciphertext)
- Servidor nunca tem acesso ao plaintext das DMs

### Schema — `encryption_keys`

| Coluna | Tipo | Notas |
|---|---|---|
| user_id | TEXT | NOT NULL (PK), FK → users |
| public_key | TEXT | NOT NULL — chave pública P-256 (SPKI base64) |
| key_updated_at | TEXT | NOT NULL |

### Schema — `encrypted_messages`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| sender_id | TEXT | NOT NULL, FK → users |
| recipient_id | TEXT | NOT NULL, FK → users |
| ciphertext | TEXT | NOT NULL |
| iv | TEXT | NOT NULL — vetor de inicialização |
| ephemeral_key | TEXT | NOT NULL — chave efêmera do remetente |
| status | TEXT | NOT NULL DEFAULT 'sent' (sent, delivered, read) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/keys/generate        → gerar/regenerar par de chaves
GET   /api/v1/users/{id}/public-key → obter chave pública
POST  /api/v1/messages             → enviar mensagem criptografada
GET   /api/v1/messages/inbox       → receber mensagens (ciphertext)
```

---

## 40. Modo "Fantasma Total"

Privacidade máxima — usuário some completamente da plataforma para quem não tem contato direto.

| Funcionalidade | Invisível | Fantasma |
|----------------|-----------|----------|
| Não aparece no radar | ✓ | ✓ |
| Não aparece na busca | ✗ | ✓ |
| Não recebe broadcasts | ✗ | ✓ |
| Não aparece no ranking | ✗ | ✓ |
| Não recebe notificações de checkin próximo | ✗ | ✓ |
| Só quem já te adicionou te vê online | ✗ | ✓ |
| Perfil invisível para não-logados | ✗ | ✓ |
| Ícone no perfil | 👻 | 👻 |

- Fantasma pode navegar, postar, interagir normalmente com quem já conhece
- Fantasma não é notificado de nada de desconhecidos
- Gratuito (segurança não deveria ser paga)
- Modo Fantasma tem duração máxima de 24h contínuas (evita contas abandonadas fantasma)
- Pode renovar após expirar
- Ao receber uma solicitação de amizade/mensagem de alguém não conhecido, notificação discreta: "Alguém tentou contato" — sem detalhes

### Schema — `ghost_mode`

| Coluna | Tipo | Notas |
|---|---|---|
| user_id | TEXT | NOT NULL (PK), FK → users |
| is_active | INTEGER | NOT NULL DEFAULT 0 |
| activated_at | TEXT | NULL |
| expires_at | TEXT | NULL — 24h após ativação |

### API

```
POST  /api/v1/privacy/ghost/activate     → ativar modo fantasma (24h)
POST  /api/v1/privacy/ghost/deactivate   → desativar
GET   /api/v1/privacy/ghost/status       → status atual
```

---

## 41. Carona Solidária Liberal

Usuários indo pro mesmo evento podem marcar carona — conexão segura e prática.

- Funciona apenas para eventos com presença confirmada
- Duas opções ao confirmar presença:
  - **"Vou de carro, tenho vaga"** — informa quantas vagas
  - **"Preciso de carona"** — informa de onde sai
- Quem oferece vaga vê lista de quem precisa, pode aceitar/rejeitar
- Quem precisa vê ofertas de carona no mesmo evento
- Chat de carona é temporário (expira 24h após o evento) e separado do chat normal
- Avaliação pós-carona: "Foi tranquilo?" (anônimo) — influencia reputação de carona
- Badge "Motorista Parceiro" após 5 caronas dadas
- Privacidade: endereço exato nunca é compartilhado — só bairro/ponto de encontro

### Schema — `event_rides`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| event_id | TEXT | NOT NULL, FK → events |
| user_id | TEXT | NOT NULL, FK → users |
| type | TEXT | NOT NULL (offer, request) |
| seats | INTEGER | NULL — só para offer |
| departure_area | TEXT | NOT NULL — bairro/zona |
| status | TEXT | NOT NULL DEFAULT 'open' (open, matched, cancelled) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/events/{id}/ride/offer     → oferecer carona
POST  /api/v1/events/{id}/ride/request   → solicitar carona
GET   /api/v1/events/{id}/rides          → caronas disponíveis do evento
POST  /api/v1/rides/{id}/match           → match de carona (motorista aceita)
```

---

## 42. Lista de Presença Anônima em Eventos

"Confirmaram presença: 47 pessoas" — sem mostrar quem. Mas com heatmap de perfil.

- Número total visível para todos
- Gráfico/heatmap demográfico anônimo:
  - "60% casais, 25% mulheres solteiras, 15% homens solteiros"
  - Faixa etária predominante
  - "5 usuários verificados confirmaram"
- Dados agregados apenas — sem identificar indivíduos
- Usuário pode optar por "não aparecer nas estatísticas"
- Ajuda indecisos a decidirem se vão baseado no perfil do público do evento
- Premium: heatmap mais detalhado (ex.: % por bairro de origem)

### Schema — `event_presence_stats`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| event_id | TEXT | NOT NULL (PK), FK → events |
| total_count | INTEGER | NOT NULL DEFAULT 0 |
| couple_pct | REAL | NOT NULL DEFAULT 0 |
| single_female_pct | REAL | NOT NULL DEFAULT 0 |
| single_male_pct | REAL | NOT NULL DEFAULT 0 |
| primary_age_range | TEXT | NOT NULL DEFAULT '' |
| verified_count | INTEGER | NOT NULL DEFAULT 0 |
| updated_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/events/{id}/presence       → lista de presença anônima + heatmap
```

---

## 43. Encontro Surpresa

Match que gera um convite automático: encontro às cegas em local sorteado.

- Qualquer match pode virar "Encontro Surpresa" — um dos dois inicia
- Sistema sorteia um local pré-cadastrado como seguro dentro do raio dos dois
- Sorteia dia/horário (próximos 7 dias, horário comercial + noturno)
- Ambos precisam confirmar o convite para o encontro acontecer
- Locais seguros: bares, cafeterias, restaurantes — todos com ambiente tranquilo
- Antes do encontro: cada um recebe dica sobre o outro (cor preferida, uma curiosidade)
- Após encontro: avaliação anônima (opcional)
- Badge "Aventureiro" após 3 encontros surpresa

### Schema — `surprise_dates`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| match_id | TEXT | NOT NULL, FK → matches |
| location_id | TEXT | NOT NULL, FK → locations |
| suggested_date | TEXT | NOT NULL |
| suggested_time | TEXT | NOT NULL |
| status | TEXT | NOT NULL DEFAULT 'pending' (pending, confirmed_a, confirmed_b, confirmed_both, cancelled, completed) |
| initiated_by | TEXT | NOT NULL, FK → users |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/match/{id}/surprise-date           → iniciar encontro surpresa
POST  /api/v1/surprise-dates/{id}/confirm        → confirmar
POST  /api/v1/surprise-dates/{id}/cancel         → cancelar
GET   /api/v1/surprise-dates/upcoming            → próximos encontros
```

---

## 44. Júri Popular

Denúncias complexas vão para um painel de 5 usuários verificados aleatórios que votam o veredito.

- Quando: denúncia depois de X reportes contra o mesmo usuário, ou conteúdo limítrofe que o robô não conseguiu classificar
- Seleção: 5 usuários verificados (selo azul) aleatórios, online no momento
- Júri vê o caso de forma anônima (não sabe quem é o denunciado)
- Cada júri vota: "Remover conteúdo" / "Manter conteúdo"
- Maioria simples decide (3-2)
- Votação anônima entre os jurados
- Jurado ganha XP por participar (20 XP por caso)
- Jurado com alta taxa de acerto (>80%) ganha badge "Juiz Justo"
- Jurado que vota contra a maioria em casos que depois se provam corretos ganha badge "Pensador Independente"

### Schema — `jury_cases`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| moderation_queue_id | TEXT | NOT NULL, FK → moderation_queue |
| status | TEXT | NOT NULL DEFAULT 'open' (open, voting, decided) |
| votes_remove | INTEGER | NOT NULL DEFAULT 0 |
| votes_keep | INTEGER | NOT NULL DEFAULT 0 |
| verdict | TEXT | NULL (remove, keep) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| decided_at | TEXT | NULL |

### Schema — `jury_votes`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| case_id | TEXT | NOT NULL, FK → jury_cases |
| juror_id | TEXT | NOT NULL, FK → users |
| vote | TEXT | NOT NULL (remove, keep) |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
GET   /api/v1/jury/cases/pending      → casos abertos para votação (para jurados)
POST  /api/v1/jury/cases/{id}/vote    → votar
GET   /api/v1/jury/history            → histórico de casos que participei
```

---

## 45. Selo "Anjo da Comunidade"

Usuários exemplares ganham badge visível e benefícios simbólicos.

- Quem ganha: reportes com alta taxa de acerto (+80%), ajuda em tópicos do fórum (+20 respostas com upvotes), boa reputação (baixas denúncias recebidas)
- Badge: "Anjo da Comunidade" — dourado, visível no perfil
- Benefícios: +5 matches/dia, prioridade em filas de verificação, mood extra exclusivo ("😇 Anjo")
- Renovação mensal: badge expira se não mantiver os critérios
- Dashboard do usuário: progresso para se tornar Anjo ("Faltam 3 reportes com acerto")

### Schema — `community_angels`

| Coluna | Tipo | Notas |
|---|---|---|
| user_id | TEXT | NOT NULL (PK), FK → users |
| status | TEXT | NOT NULL DEFAULT 'active' (active, expired) |
| report_accuracy | REAL | NOT NULL DEFAULT 0 |
| helpful_posts | INTEGER | NOT NULL DEFAULT 0 |
| awarded_at | TEXT | NOT NULL DEFAULT (datetime('now')) |
| expires_at | TEXT | NOT NULL |

### API

```
GET   /api/v1/angels              → lista de anjos ativos
GET   /api/v1/users/me/angel-progress  → progresso para se tornar anjo
```

---

## 46. Match Turbinado (Boost)

Pagamento único para aparecer no topo do feed de swipe por 1 hora.

- Inspirado no Tinder Boost
- Durante 1 hora, o perfil aparece nos primeiros lugares do feed de swipe de todos os usuários na região
- Custo: R$2-5 por boost (definir preço exato)
- Máximo 1 boost por dia (evita saturação)
- Notificação push durante o boost: "Seu boost está ativo — 30 matches em potencial viram seu perfil agora"
- Estatísticas pós-boost: "Seu boost gerou X visualizações e Y matches"
- Ideal para: usuários em cidades turísticas, antes de eventos, fim de semana

### Schema — `boosts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| user_id | TEXT | NOT NULL, FK → users |
| price_cents | INTEGER | NOT NULL |
| duration_minutes | INTEGER | NOT NULL DEFAULT 60 |
| started_at | TEXT | NULL |
| expires_at | TEXT | NULL |
| views_generated | INTEGER | NOT NULL DEFAULT 0 |
| matches_generated | INTEGER | NOT NULL DEFAULT 0 |
| payment_id | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### API

```
POST  /api/v1/boost/buy           → comprar boost
GET   /api/v1/boost/active        → boost ativo? status
GET   /api/v1/boost/history       → histórico de boosts
```

---

## 47. Presente Virtual Picante

Emojis animados especiais que podem ser enviados com uma mensagem — cada um custa R$1-3.

- Emojis animados exclusivos: 🌹🔥😈🍑🍆💦🎭👁️‍🗨️🔞🖤
- Enviar presente: escolhe emoji + mensagem curta + destinatário
- Destinatário recebe notificação: "[Nome] te enviou um presente 🌹"
- Presente aceito aparece no perfil como badge temporário "Presenteado por [Nome]" (expira em 7 dias)
- Presente pode ser anônimo (quem envia escolhe)
- Ao ser presenteado, usuário pode retribuir com outro presente
- Presentes pagos via Pix (microtransação)
- Comissão da plataforma: 20% do valor (margem maior que assinatura)

### Schema — `virtual_gifts`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| gift_id | TEXT | NOT NULL, FK → gift_catalog |
| sender_id | TEXT | NOT NULL, FK → users |
| recipient_id | TEXT | NOT NULL, FK → users |
| message | TEXT | NOT NULL DEFAULT '' |
| is_anonymous | INTEGER | NOT NULL DEFAULT 0 |
| price_cents | INTEGER | NOT NULL |
| payment_id | TEXT | NULL |
| created_at | TEXT | NOT NULL DEFAULT (datetime('now')) |

### Schema — `gift_catalog`

| Coluna | Tipo | Notas |
|---|---|---|
| id | TEXT PK | UUID v7 |
| emoji | TEXT | NOT NULL |
| name | TEXT | NOT NULL — "Rosa", "Fogo", "Diabo", "Pêssego", "Berinjela", "Gozo", "Máscara" |
| price_cents | INTEGER | NOT NULL |
| animation_url | TEXT | NULL — GIF/webp animado |
| is_active | INTEGER | NOT NULL DEFAULT 1 |

### API

```
GET   /api/v1/gifts/catalog         → catálogo de presentes virtuais
POST  /api/v1/gifts/send            → enviar presente
GET   /api/v1/gifts/inbox           → presentes recebidos
GET   /api/v1/gifts/outbox          → presentes enviados
POST  /api/v1/gifts/{id}/accept     → aceitar/exibir presente no perfil
```
```