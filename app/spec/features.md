# Features — Especificação

> Features do Liberages além do mapa interativo e radar de intenção. Cada feature é incremental e segue a stack padrão (Go + SQLite + sqlc + React SPA).

---

## 1. Fotolog Diário

Postagem de foto diária — estilo Fotolog antigo. Usuário posta 1 foto por dia com legenda curta.

- Limite de 1 foto/dia (free) — ilimitado para Premium
- Foto aparece no feed cronológico (fotolog) do usuário
- Blr facial automático opcional
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
- Blr facial automático opcional por foto
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
- Marcadores:Importante, Resolvido, Em Discussão
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

## 5. Moderação Híbrida (Humana + Roboô)

Sistema de moderação que combina ação humana e automação (IA/robô) em camadas.

### 5.1 Camadas de Moderação

| Camada | O que faz | Quando |
|--------|----------|--------|
| **Roboô (automática)** | Triagem de fotos, detecção de conteúdo proibido (menores, violência), blur facial automático, rate limiting, detecção de spam | Em tempo real, no upload/post |
| **Humana (moderador)** | Decisão em casos limítrofes, apelação de banimentos, revisão de denúncias,_approval de ingresso | Assíncrono, em filas |

### 5.2 Regras do Roboô

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
- Validação de identidade (contas verificado): re-validar a cada 12 meses
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