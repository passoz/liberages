# Plano de Negócios — Liberages

> **Confidencial.** Documento estratégico para operação e direcionamento do produto.
> Última atualização: Julho 2026

---

## Sumário Executivo

### O que é

**Liberages** é uma plataforma digital (PWA + desktop app) construída em Go single-binary com React SPA, voltada para o público liberal brasileiro. A plataforma atua fundamentalmente como uma **Rede Social e Mapa Interativo**, onde o descobrimento geográfico e a afinidade de interesses se cruzam. Ela combina gamificação estilo "Pokémon GO do prazer", um feed social algorítmico, eventos curados pela comunidade, e uma arquitetura *privacy-first*.

### Oportunidade

- Mercado dominado por **1 player** (Sexlog) com UX datada e modelo abusivo de paywall.
- Banimento de apps para adultos nas lojas (App Store/Play Store) abre espaço para **PWA** de alta performance.
- Público liberal clama por ferramentas de privacidade reais (Modo Fantasma, Blur facial, E2E futuro) que os concorrentes ignoram.

### Diferencial

| Pilar | O que resolve | Como |
|-------|---------------|------|
| **Mapa-radar gamificado** | Dificuldade de saber quem está a fim, ao seu redor | Radar de check-ins (com TTL) e eventos. Identidade protegida no radar, revelada apenas via chat pós-match. |
| **Privacidade em camadas** | Medo de exposição | Blur facial automático, Modo Fantasma (premium), botão Modo Falso (free), marca d'água dinâmica contra prints (rastreável). |
| **Comunidade Verificada** | Perfis falsos e golpistas | "Web of Trust": o selo de Verificado exige confirmação de 4 amigos reais. Criação de eventos públicos é restrita a verificados. |
| **Economia Fechada** | Atrito financeiro e engajamento | **Moeda Única** que unifica XP e microtransações. Compra-se com R$, gasta-se no app. Sem cash-out (zero atrito legal). |

### Modelo de receita

Freemium em 2 camadas: **Free** (limitado a 30 swipes/dia, 3 álbuns, com anúncios B2B) e **Premium** (R$29,90/mês ou pagamento híbrido com Moeda Única). Receita impulsionada por vendas de Moeda Única (usada para Match Boosts, presentes virtuais e descontos em parceiros B2B). O sistema de cash-out para usuários não existe, eliminando complexidade do Banco Central.

### Equipe & Investimento
Operação enxuta, solo (full-stack Go/React), bootstrapping com custo de infra quase nulo graças ao SQLite e single-binary.

## 1. Descrição do Negócio

### 1.1 Visão

Tornar-se a plataforma de referência para o público liberal brasileiro — o lugar onde	privacidade real, descoberta geográfica gamificada e comunidade moderada convergem.

### 1.2 Missão

Conectar pessoas do público liberal com segurança, discrição e privacidade real, oferecendo uma experiência moderna e superior às plataformas existentes.

### 1.3 Valores

- **Privacidade como direito, não feature** — o usuário controla o quanto mostra
- **Segurança primeiro** — ferramentas reais de proteção (botão de pânico, contato de emergência, encontro seguro)
- **Transparência** — regras claras, moderação explicável, sem shadowbanning arbitrário
- **Comunidade** — plataforma centrada em pessoas, não em conteúdo
- **Inovação constante** — mapa-radar gamificado, matching por intenção, lives com máscara, PWA/desktop app
- **Operação enxuta** — custo quase nulo como princípio, não como contingência

### 1.4 Produto

Liberages é entregue como **PWA** (installável no celular sem App Store/Play Store) e **desktop app** (system tray com atalho de ocultação discreta). Arquitetura técnica:

- **Single binary Go** (API + BFF + estáticos React) servindo tudo na porta `:3000`
- **React SPA + PWA** — Vite, React Router, Tailwind CSS v4 + Glassmorphism
- **SQLite** (escalável até ~50k usuários ativos em modo write-heavy, depois migrar para PostgreSQL)
- **Leaflet** com tiles próprios (sem CDN, sem dependência externa)

### 1.5 Módulos do Produto

#### Módulo 1 — Mapa Interativo + Radar de Intenção (MVP)

Este é o coração do produto. São dois mapas conceitualmente distintos que convivem na mesma interface:

**Mapa de Locais (descoberta fixa):**
- Mapa Leaflet com marcadores SVG customizados por tipo de local (casas de swing, motéis, bares, praias)
- Locais são pontos fixos, públicos — não envolvem privacidade de pessoas
- CRUD de locais (admin) com upload de imagens
- Busca textual com SQLite FTS5

**Radar de Atividade (gamificação, estilo Pokémon GO):**
- O usuário **não vê localização exata de ninguém** — apenas clusters fuzzy por bairro/zona
- Usuário marca "estou a fim hoje" (botão raio de curto alcance) — status anônimo
- Quando dois usuários compatíveis por intenção entram no mesmo raio, ambos recebem notificação: "alguém próximo está a fim" — match implícito sem revelar identidade
- **Checkins em locais públicos** (casa de swing, bar liberale) são visíveis — gamificação real do "tô no X, quem mais está?"
- **Notificações de checkin próximo**: você está em casa e recebe "3 casais acabaram de fazer check-in no Y, a 1.5km"
- Identidade só é revelada após consentimento mútuo (match/mensagem)
- Modo noturno padrão, modo anônimo (cluster em vez de ponto individual)
- Filtros: tipo de interação, distância, tempo (agora/hoje/semana)

#### Módulo 2 — Perfis e Privacidade

- Perfis individuais e de casal (ver Módulo 9)
- **Níveis de privacidade**:
  - Nível 0: Visitante — vê só o mapa público
  - Nível 1: Age gate verificado — vê locais, perfis com fotos borradas
  - Nível 2: Perfil completo — interação normal
  - Nível 3: Verificado (documento + selfie) — selo azul, grupos exclusivos
- **Kit de Relacionamento** — campos obrigatórios no perfil: status (solteiro, casal aberto, poliamor, etc.), o que busca (troca, ménage, amizade liberal, bate-papo), frequência (novato a hardcore)
- **Lista de Fetiches com Match %** — usuário marca até 15 fetiches de catálogo curado; sistema calcula compatibilidade percentual entre perfis; feed de swipe ordena por compatibilidade
- **Mood do Momento** — status temporário de 1h com emoji (🔥 a fim, 🙈 tímido, 😈 brutal, 👀 só de olho, etc.) que aparece no perfil, no radar e no mapa
- Blur facial automático (IA), modo invisível, selfie destrutível (some após 10s)
- Galeria com controle de quem vê cada foto

#### Módulo 3 — Matching por Intenção + Swipe

- **Radar de intenção** (Módulo 1) — match implícito por proximidade
- **Swipe (curtir/não curtir)** estilo Tinder — feed de perfis ordenado por compatibilidade de fetiches + intenção + localização
- Match: ambos curtiram → chat liberado; integrado ao radar (se ambos marcaram "a fim hoje", match tem prioridade)
- Super like (1/dia free, ilimitado Premium) — notifica o outro usuário
- Undo do último swipe (Premium)
- **Bucket List (lista de desejos)** — usuário adiciona locais do mapa à lista privada; se dois usuários têm o mesmo local, ambos recebem notificação de compatibilidade
- **Encontro Surpresa** — match pode virar encontro às cegas: sistema sorteia local seguro + horário nos próximos 7 dias; ambos confirmam para acontecer
- **Match Turbinado (Boost)** — pagamento único (R$2-5) para aparecer no topo do feed de swipe por 1h
- Limite diário para free users (5-10/dia), ilimitado para Premium

#### Módulo 4 — Eventos com Geolocalização

- Criação de eventos: festas, encontros em motel, surubas, jantar liberal
- Eventos públicos e privados (com convite)
- **Check-in com PIN** (evita penetras)
- "Eventos próximos a você agora" no mapa
- Integração B2B com casas de swing reais
- **Agenda Liberal** — calendário com feriados sazonais (Carnaval, Réveillon, Dia do Sexo 06/09) + eventos fixos de casas de swing + notificações "Faltam 3 dias para o Carnaval"
- **Carona Solidária** — usuários indo pro mesmo evento marcam carona (oferece vaga ou precisa); chat temporário expira 24h após evento; endereço exato nunca exposto (só bairro/ponto); avaliação anônima pós-carona
- **Lista de Presença Anônima** — "Confirmaram: 47" com heatmap demográfico (60% casais, 25% mulheres solteiras, faixa etária predominante, 5 verificados) sem identificar indivíduos
- Histórico privado de eventos frequentados

#### Módulo 5 — Conteúdo e Expressão

- **Fotolog Diário** — estilo Fotolog antigo, 1 foto/dia com legenda, expira em 24h por padrão (configurável), blur automático, feed cronológico
- **Álbuns** — galerias curadas pelo usuário, controle de privacidade por álbum (público, amigos, verificados, privado), máximo 3 free / ilimitado Premium
- **Story 24h** — foto ou vídeo curto (máx. 30s) com blur opcional, reação rápida por emoji (🔥😈😍), resposta via DM, Premium vê quem viu
- **Contos Eróticos** — literatura erótica escrita por usuários (Markdown básico), categorias, tags, votação, comentários moderados, destaque semanal, badge "Escritor(a) do Mês"
- **Live Streaming com Privacidade** — máscara facial em tempo real, controle de audiência, pay-per-view (tips, ingressos), lives em grupo (até 4)
- **Marketplace de Criadores** — venda de conteúdo (fotos, vídeos, packs, assinatura mensal); criador define preço (R$5-R$200); comissão 15% da plataforma; pagamento via Pix

#### Módulo 6 — Comunidade e Discussão

- **Fórum** — categorias (swing, BDSM, iniciantes, segurança, eventos), tópicos, respostas em thread, votação up/down, fixar tópicos, marcadores (Importante, Resolvido), busca FTS5; free: 5 posts/dia, Premium ilimitado
- **Comunidades com Presença Anônima** — grupos onde o usuário pode participar sem revelar identidade (pseudônimo gerado: "Anônimo #4A7B", consistente dentro da comunidade); moderadores veem identidade real
- **Grupos com Curadoria** — públicos e privados, categorias por fetiche/cidade/interação, ranking por atividade, regras explícitas, moderação com banimento explicado
- **Mural de Elogio Anônimo** — elogios anônimos moderados antes de aparecer; receptor aceita ou rejeita; badges "Admiradx" (10), "Cobiçadx" (50), "Musa/Muse" (100); 1 elogio/pessoa/mês
- **Confiança Mútua** — dois usuários se marcam como "conheço pessoalmente"; aparece como contagem no perfil ("conhecido por N pessoas"); desvincular a qualquer momento

#### Módulo 7 — Gamificação e Engajamento

- **Badges (Conquistas)** — selos desbloqueados ao atingir marcos: Primeiro Checkin (bronze), Explorador (prata, 5 locais), Borboleta Social (prata, 10 matches), Veterano (ouro, 1 ano), Verificado (prata), Criador de Conteúdo (prata, 50 fotos), Matcher Noturno (ouro, 20 matches 23h-5h), Cidade Inteira (ouro, 20 locais) — configurável exibir/ocultar
- **XP e Níveis** — interações dão XP (login +5, fotolog +10, match +50, checkin +20, validar identidade +100); níveis desbloqueiam features (níveis 2-10-20 com bônus); imagem de progressão
- **Ranking Semanal** — opt-in, top 10 da semana com badge temporária "Top 10"; categorias: geral, checkins, fórum, comunidades; reset semanal; prêmio puramente social
- **Desafio Liberal Semanal** — toda segunda-feira novo desafio (foto criativa, checkin, conto, fórum, mood); quem completa ganha 100 XP + badge temporária; 4/4 no mês = badge "Mestre dos Desafios"
- **Caça ao Tesouro (B2B)** — parceiros escondem tesouros no mapa visíveis só a X km; usuário coleta → cupom/voucher/desconto; monetização: parceiro paga por lead
- **Jogo da Verdade Liberal** — mini-game pós-match: 5 perguntas de quebra-gelo (leve/médio/picante), respostas reveladas só quando ambos respondem, combinações destacadas com 🔥
- **Júri Popular** — denúncias complexas vão para painel de 5 usuários verificados que votam (remover/manter); maioria simples decide; XP por participar; badge "Juiz Justo"
- **Selo Anjo da Comunidade** — usuários exemplares (reportes >80% acerto, +20 respostas úteis no fórum) ganham badge dourada + benefícios (+5 matches/dia, prioridade em filas); renovação mensal

#### Módulo 8 — Segurança e Privacidade Avançada

- **Moderação Híbrida** — robô (IA) tria em tempo real (blur, spam, menores, violência); humano decide casos limítrofes; fila de moderação com log completo; apelo de banimento com revisão por moderador diferente
- **Validação Periódica** — re-verificação de idade a cada 6 meses, identidade a cada 12 meses; conta não validada em 30 dias perde privilégios
- **Ingresso Moderado** — cadastro aprovado manualmente (auto-approvar se robô não flagar + idade verificada; humano só vê flagueados); SLA 24h
- **Verificação por Vídeo-Chamada** — além de documental, chamada de vídeo de 2min com moderador; badge "Verificado ao Vivo" (dourado, distinto do azul)
- **Câmera ao Vivo de Locais** — parceiros com câmera pública do ambiente (sem foco em pessoas); usuário vê antes de ir; status "aberto agora" + ocupação (cheio/médio/vazio)
- **Alarme de Screenshots** — PWA detecta possível screenshot e notifica dono do conteúdo; não impede (impossível em PWA) mas constrange e notifica; selfie destrutível some imediatamente se alarme dispara
- **Botão de Pânico / Modo Falso** — atalho (agitar celular, botão flutuante) leva pra tela falsa (calculadora, clima, notas), invalida sessão, salva estado para retomada com PIN
- **Login Rápido com PIN** — PIN de 4 dígitos para sessões curtas (15min); útil em intervalos/no trabalho; bloqueia após 3 tentativas erradas
- **Criptografia E2E nas DMs** — todas as mensagens diretas criptografadas no dispositivo (Web Crypto + ECDH); servidor nunca tem acesso ao plaintext
- **Modo Fantasma Total** — usuário some completamente (não aparece em busca, não recebe broadcasts, só quem já tem contato te vê online); gratuito (segurança não é paga); duração máxima 24h
- **Block + Report** com justificativa; rate limiting anti-spam/anti-assédio

#### Módulo 9 — Modo Casal

- **Conta compartilhada de casal** — duas contas individuais linkadas, mesmo perfil público ([NomeA] & [NomeB])
- Chat de casal unificado; swipe com double opt-in (ambos curtam para match contar)
- **Moodboard do Casal** — painel colaborativo privado (fotos, texto, locais favoritos, datas); só os dois vêem; lembretes ("Vocês se conheceram há 3 meses 🎉"); exportável em PDF (Premium)
- Desvincular a qualquer momento; perfil de casal removido

#### Módulo 10 — Monetização de Microtransações

- **Marketplace de Criadores** — comissão 15% sobre vendas de conteúdo (ver Módulo 5)
- **Match Boost** — R$2-5 para aparecer no topo do feed de swipe por 1h; máximo 1/dia; estatísticas pós-boost
- **Presente Virtual Picante** — emojis animados exclusivos (🌹🔥😈🍑🔞) que custam R$1-3 cada; aceitação com badge temporário; pode ser anônimo; comissão 20% da plataforma
- **Assinatura de Presente** — comprar Premium/VIP de presente para outro usuário; pode ser anônimo; QR code para dar em casas de swing; não acumulável (estende a partir da data atual)
- **Cartão Fidelidade Digital (B2B)** — checkins em locais parceiros acumulam pontos; 100 pontos = R$5 desconto; voucher gerado na plataforma; parceiro confirma e pontua

#### Módulo 11 — Admin e Operação

- **Painel Admin** — full CRUD, filtros, sorting, preview de mapa, moderação de conteúdo/perfis/eventos, gestão de denúncias, métricas em tempo real
- **Sistema de Tickets de Suporte** — categorias (denúncia, técnico, conta, cobrança, sugestão); prioridade: normal (free), alta (Premium/VIP); resposta dentro da plataforma
- **Análise de Sentimento da Comunidade** — dashboard de moderação: denúncias recebidas, taxa de resolução, tempo de resposta, sentimento geral (-1 a +1); alertas de pico; exportação CSV mensal
- **Painel Público de Status** — status.liberages.com: uptime, incidentes, histórico; status por componente (API, mapa, radar, fórum, upload); feed RSS
- **Convite por QR Code** — gerar QR para conexão presencial (casas de swing); contém ID criptografado + timestamp (expira em 5min); também para checkin em locais parceiros

---

## 2. Análise de Mercado

### 2.1 Tamanho de Mercado

O Brasil é um dos maiores consumidores de conteúdo adulto do mundo. O mercado de redes sociais para o público liberal é dominado por um player principal:

- **Sexlog**: ~17 milhões de usuários cadastrados (2022), 15 mil fotos/dia, 1 mil vídeos/dia, mais de 1 milhão de lives realizadas
- **Crescimento durante a pandemia**: +37% — hábito permaneceu pós-pandemia
- **Mercado total estimado**: 20-30 milhões de brasileiros com interesse em lifestyle liberal (estimativa conservadora baseada em pesquisas de comportamento sexual)

### 2.2 Segmentação de Cliente

| Segmento | Tamanho relativo | Comportamento | Disposição a pagar |
|----------|-----------------|----------------|---------------------|
| **Casais liberais (swing)** | Maioria (40-50%) | Seletivos, discretos, frequentam casas de swing | Média-alta |
| **Homens solteiros** | Superabundância na oferta (30-40%) | Maior dificuldade de engajamento | Alta (pagam por acesso) |
| **Mulheres solteiras** | Demanda altíssima, oferta baixa (5-10%) | Tratamento VIP | Baixa (ferramentas premium gratuitas para elas) |
| **Casais bissexuais** | Nicho dentro do nicho | Buscam outro casal ou mulher | Média |
| **Público BDSM/fetichista** | Menor, altamente engajado (5-10%) | Valoriza segurança, consentimento, regras | Alta |
| **Exibicionistas/voyeurs** | Motor da plataforma (10-15%) | Consomem e produzem conteúdo | Média-alta (pay-per-view) |

### 2.3 Dores não resolvidas pelo mercado atual

1. **Privacidade frágil** — fotos podem ser identificadas, não há blur automático
2. **Interface datada** — Sexlog parece site dos anos 2000, UX pobre no mobile
3. **Sem descoberta geográfica gamificada** — não há mapa, não há radar de "quem está a fim agora"
4. **Moderação insuficiente** — perfis falsos, golpistas, falta de verificação
5. **Preço elevado** — R$49,90/mês para produto defasado
6. **Estigma social** — não pode ter presença em redes sociais tradicionais
7. **Sem app real** — Sexlog é site mobile-responsive, banido das lojas
8. **Falta de curadoria de eventos** — festas postadas em formato textual sem geolocalização
9. **Segurança em encontros** — não há ferramentas para encontros seguros
10. **Comunidade fragmentada** — grupos sem algoritmo de recomendação

### 2.4 Concorrência

#### Concorrente principal: Sexlog

| Aspecto | Detalhe |
|---------|---------|
| Fundação | 2007 |
| Usuários | ~17 milhões cadastrados |
| Modelo | Freemium (R$9,90 primeiro mês, R$49,90/mês depois) |
| Força | Massa crítica, marca consolidada, SEO dominante |
| Fraqueza | Interface datada, sem mobile nativo, sem mapa, moderação frágil |
| Funcionalidades | Perfis, grupos, lives, fotos, vídeos, chat, eventos |
| Live cams | Sim, com agendamento. 1M+ lives |

#### Outros concorrentes

| Plataforma | Foco | Diferença |
|------------|------|-----------|
| D4 Swing | Swing/troca de casais | Regional, menor alcance |
| BDSM Lovers | BDSM, fetichismo | Nicho específico, menor |
| FetLife (internacional) | Fetichismo, BDSM | Global, não foca Brasil |
| Tinder/Outros | Namoro genérico | Não atende público liberal |

### 2.5 Por que o Sexlog não é imbatível

O Sexlog tem 17M de usuários mas **não inova há anos**. É um produto de 2007 com atualizações incrementais. Uma plataforma moderna, com foco em privacidade real, UX superior e mapa-radar gamificado pode capturar:

- Usuários insatisfeitos com o Sexlog (UX, preço, falta de features)
- Novos entrantes que nunca usaram Sexlog por estigma ou UX pobre
- Público que não usa plataformas para adultos por questões de privacidade

---

## 3. Posicionamento Estratégico

### 3.1 Proposição de Valor

> **"O Pokémon GO do prazer vira uma comunidade"**
> O mapa não é um extra — é o centro da experiência. Você não sabe quem ao seu redor está a fim de uma aventura — até o radar de intenção te avisar. Cada local, evento, checkin ePerfil tem uma âncora geográfica gamificada.

### 3.2 Estratégia de Posicionamento

Não competir head-to-head com Sexlog em SEO e massa crítica. Competir em **inovação, privacidade, gamificação e UX** — capturar usuários que buscam algo melhor, não usuários que já estão satisfeitos.

### 3.3 Vantagem Competitiva Sustentável

1. **Mapa-radar gamificado (Pokémon GO liberal)** — nenhum concorrente tem. Difícil de replicar bem porque combina real-time, geolocalização fuzzy, matching por intenção e gamificação
2. **PWA + Desktop app** — bypassa App Store/Play Store (banimento de apps adultos é barreira de entrada)
3. **Privacidade em camadas** — blur facial IA, selfie destrutível, modo invisível, E2E, modo fantasma, modo falso
4. **Single binary Go + SQLite** — custo de infraestrutura quase nulo (1 processo, 1 VPS barato)
5. **Desktop app discreto** — system tray com ocultação (ninguém vê no celular)
6. **Comunidade moderada** — ambiente seguro atrai e retém mulheres (segmento mais carente)
7. **Operação solo enxuta** — custo fixo baixíssimo permite sobreviver com pouca receita inicial
8. **Sistema de Padrinho/Madrinha** — veteranos acolhem iniciantes, transferem credibilidade, quebram cold-start — impossível de replicar offline sem ferramenta
9. **Geradores de momentum (Festa Relâmpago, Termômetro da Noite, Mural de Desejos)** — o meio precisa de estímulo para circular; o app cria urgência e serendipidade que nem Sexlog nem WhatsApp conseguem gerar
10. **Ecossistema de veteranos (Anfitriões, Cartão de Visita, Ranking)** — dá motivo para influencers do meio usarem o app: organizam, monetizam influência, ampliam alcance — ferramentas impossíveis offline

### 3.4 Os Três Eixos Estratégicos das Features Safadarias

O plano de features (71 no total) ataca três problemas estruturais do mercado liberal:

| Eixo | Problema | Solução (features) |
|------|----------|-------------------|
| **Circulação** | Nada acontece sozinho no meio liberal; sem estímulo, ninguém se mexe | Festa Relâmpago (urgência), Termômetro da Noite (FOMO), Mural de Desejos (serendipidade), Despertador Liberal (antecipação), Roleta Safada (ritual diário), Confissão Liberal (conteúdo+discussão) |
| **Veteranos** | Influencers do meio já têm rede, não precisam do app | Padrinho/Madrinha (mentoria→status), Carta de Apresentação (expressão), Ranking de Anfitriões (reputação→monetização), Cartão de Visita Digital (networking), Convite Exclusivo para Iniciantes (transfer de social capital) |
| **Iniciantes** | Casais novatos travam no cold-start, não sabem como entrar | Trilha de Iniciação (jornada guiada 90 dias), FAQ Interativo (educação), Cofre de Limites (segurança), Detector de Compatibilidade (reduz medo), Modo Newbie (acolhimento), Encontro Guiado por Especialista (escolta) |

---

## 4. Modelo de Receita

### 4.1 Estrutura de Preços e Limites

A barreira entre Free e Premium foi desenhada para maximizar a retenção (Daily Active Users) e forçar a conversão nos heavy-users.

| Tier | Preço | Funcionalidades |
|------|-------|-----------------|
| **Free** | Grátis (com anúncios) | Perfil, Busca FTS5, Radar anônimo, Limite de 30 swipes/dia, Limite de 3 álbuns, Vê visitas/likes apenas por janela de 2h, Blur automático, Modo Falso (segurança). |
| **Premium** | Pagamento Híbrido | Tudo do Free **sem anúncios**. Swipes e Álbuns ilimitados. Histórico total de visitas/likes. **Modo Fantasma/Invisível**. Permissão para publicar Contos. |

### 4.2 Economia de Moeda Única (Circuito Fechado)

O Liberages opera com uma **Moeda Única** virtual que substitui transações diretas em reais para micro-interações, unificando gamificação e receita.

- **Compra (Inflow):** Usuários compram pacotes de moedas com R$ via Mercado Pago.
- **Ganho (Earn):** Usuários ganham pequenas frações de moeda por alto engajamento (completar desafios, check-ins B2B).
- **Gasto (Burn):** Moedas são gastas com Match Boost (topo do swipe por 1h), Presentes Virtuais, Desbloqueio de Badges funcionais, e compra de Vouchers em parceiros B2B.
- **Pagamento Híbrido:** O Premium pode ser pago em R$ integral, ou com abatimento parcial usando Moedas (ex: 1500 moedas + R$14,90).
- **Política No-Cashout:** Usuários não podem sacar a moeda para R$ nem transferir P2P, eliminando mercado paralelo e exigências do Banco Central.

### 4.3 Receita de Anúncios (Free)
Anúncios B2B locais (Casas de swing, motéis) aparecem nativamente no mapa e no feed para usuários Free. Custo por Clique/Impressão gerido pela plataforma.

### 4.4 Assinatura de Presente
Um vetor inovador de receita: usuários podem comprar o Premium para outros usuários usando a Moeda Única, funcionando como um gesto de alto interesse (icebreaker de luxo).

## 5. Modelo B2B — Locais Parceiros

> **O que é B2B no Liberages:** Locais frequentados pelo público liberal (casas de swing, motéis, bares, sex shops, resorts) pagam para ter **visibilidade, leads e ferramentas de engajamento** dentro da plataforma. O B2B não é venda de anúncio tradicional — é venda de **acesso a um público qualificado e geolocalizado** que o local não alcança sozinho.

### 5.1 Por que B2B faz sentido

O mapa de locais é o centro da experiência do Liberages. Cada local listado é um **ponto de descoberta, check-in e encontro**. O B2B capitaliza isso:

| Pra quem | O problema | O que o Liberages resolve |
|----------|-----------|---------------------------|
| **Casa de swing** | Depende de mídia paga e boca a boca para atrair frequentadores | Mapa com destaque + termômetro da noite + caça ao tesouro = visibilidade contínua para público qualificado |
| **Motel** | Concorre com dezenas de motéis na mesma região | Destaque no mapa para quem está perto e com intenção ativa ("tô a fim") |
| **Sex shop** | Cliente não sabe que a loja existe perto dele | Aparecer no mapa como local de interesse + anúncio geolocalizado para público compatível |
| **Resort liberal** | Precisa encher pacotes de temporada | Caça ao tesouro (cupom de desconto coletável no mapa) + broadcast de eventos sazonais |

### 5.2 Produtos B2B

| Produto | O que é | Modelo de precificação | Receita estimada por parceiro/mês |
|---------|---------|------------------------|-----------------------------------|
| **Listing Destacado** | Local aparece com pin dourado, foto e descrição expandida no mapa. Prioridade em buscas. | Assinatura mensal (R$49-199/mês) | R$49-199 |
| **Caça ao Tesouro** | Parceiro cria oferta/desconto que aparece como tesouro no mapa (visível a X km). Usuário coleta → voucher. | CPL (custo por lead) — R$2-5 por voucher resgatado | R$100-500 |
| **Termômetro da Noite** | Parceiro vê ocupação reportada pelos usuários no local + heatmap de horários de pico. Versão paga: dashboard com histórico. | Plano básico grátis (média ao vivo), Premium (R$29/mês, dashboard completo) | R$29-99 |
| **Evento Promovido** | Evento do local destacado no feed de eventos + notificação push para usuários no raio. | Por evento (R$19-49) | R$19-49 |
| **Câmera ao Vivo** | Feed de câmera pública do ambiente (sem foco em pessoas). Usuário vê antes de ir. | Assinatura mensal (R$99-299/mês) | R$99-299 |
| **Cartão Fidelidade** | Checkins acumulam pontos → descontos. Parceiro paga taxa por voucher resgatado + taxa fixa mensal de participação. | Fixa R$29/mês + R$1 por voucher resgatado | R$29-200 |
| **Festa Relâmpago (local sugerido)** | Quando usuário cria uma festa relâmpago, o local parceiro aparece como sugestão prioritária. | CPM ou CPL — R$5 por festa que escolher o local | R$50-200 |

### 5.3 Funnel B2B — Como o parceiro entra

```
Descoberta → Cadastro gratuito → Período trial → Upgrade para pago → Crescimento
```

| Etapa | Ação | Ferramenta |
|-------|------|------------|
| **1. Descoberta** | Parceiro encontra o Liberages por busca ou indicação | SEO + prospecção manual do fundador |
| **2. Cadastro gratuito** | Parceiro se cadastra como local (validação manual pelo fundador) | CRUD de locais + formulário de parceiro |
| **3. Período trial (30 dias)** | Local ganha listing destacado grátis + termômetro ao vivo | Dashboard do parceiro (gratuito) |
| **4. Upgrade** | Após trial, pode assinar listing destacado ou outros produtos | Portal de planos + pagamento via Pix |
| **5. Crescimento** | Parceiro ativo divulga o app para seus clientes (boca a boca orgânico) | QR code do local para check-in |

### 5.4 Dashboard do Parceiro

O parceiro tem um dashboard próprio (dentro do app, acesso web) com:

- **Visão geral:** checkins hoje/semana/mês no seu local, termômetro médio, tesouros coletados
- **Checkins:** quantos usuários fizeram check-in, heatmap por dia da semana/horário
- **Tesouros:** quantos criou, quantos coletados, valor em descontos concedidos
- **Vouchers de fidelidade:** quantos resgatados, valor total de desconto
- **Ocupação reportada:** termômetro da noite com gráfico de tendência
- **Faturamento:** histórico de pagamentos, nota fiscal

### 5.5 Implementação técnica (v1 vs futuro)

| Produto B2B | v1 (faz agora) | Futuro (faz depois) |
|------------|----------------|---------------------|
| Listing Destacado | Flag `is_featured` no local + ordenação por destaque no mapa | Sistema de assinatura com pagamento recorrente Pix |
| Caça ao Tesouro | CRUD manual no admin | Dashboard do parceiro para autogerenciamento |
| Termômetro da Noite | Votação anônima com média ao vivo (feature pública) | Dashboard premium com histórico |
| Evento Promovido | Flag no evento + destaque no feed | Automação com pagamento por evento |
| Câmera ao Vivo | Link para stream externa (YouTube privado, etc.) | Integração WebRTC/HLS própria |
| Cartão Fidelidade | Pontuação manual no checkin + voucher gerado pelo admin | Sistema automático de pontos + voucher |
| Dashboard do parceiro | Admin vê os dados do local e repassa pro parceiro | Self-service com login próprio |

> **Princípio B2B:** o modelo de receita B2B não precisa estar pronto no dia 1. Os locais podem ser cadastrados como locais comuns no mapa (sem custo). O B2B vira produto quando houver demanda de parceiros. Até lá, o mapa já entrega valor pros dois lados: usuários encontram locais, locais ganham visibilidade gratuita.

> **Nota:** a maioria dos produtos B2B depende de features que serão implementadas progressivamente (Caça ao Tesouro, Cartão Fidelidade, Termômetro da Noite, Câmera ao Vivo). Até que essas features existam na plataforma, o B2B opera apenas com Listing Destacado e Evento Promovido — que são simples flags no banco de dados.

### 5.6 Estratégia de Prospecção B2B

- **Fase 1 (0-3k usuários):** Cadastro manual dos locais conhecidos pelo fundador. Sem prospecção ativa B2B. Locais ganham visibilidade grátis.
- **Fase 2 (3k-15k):** Quando o app tiver usuários ativos em uma cidade, começar prospecção manual. Abordagem: "Seu local já tem N checkins no Liberages. Quer um dashboard com esses dados?"
- **Fase 3 (15k+):** Quando a receita B2B começar a ser relevante (R$1.000+/mês), considerar automação do onboarding de parceiros.

---

## 6. Arquitetura de Interfaces (Postergados)

Os componentes postergados (upload, backup, moderação de fotos, push notifications) seguem o mesmo padrão do resto da stack: **interfaces Go na camada de domínio, implementação concreta plugável.**

### 6.1 Storage (upload de arquivos)

```go
// internal/domain/storage.go
type Storage interface {
    Upload(ctx context.Context, path string, r io.Reader) error
    Delete(ctx context.Context, path string) error
    URL(ctx context.Context, path string) (string, error)
}
```

| Implementação | Quando usar |
|--------------|-------------|
| `storage/local` — filesystem no VPS | v1 (padrão) |
| `storage/s3` — S3/MinIO compatível | Quando filesystem atingir limite (>50GB) |

### 6.2 Moderação de Conteúdo (fotos)

```go
// internal/domain/moderation.go
type ContentModerator interface {
    ModerateImage(ctx context.Context, r io.Reader) (*ModerationResult, error)
}

type ModerationResult struct {
    Safe    bool
    Blur    bool
    Reject  bool
    Labels  []string
}
```

| Implementação | Quando usar |
|--------------|-------------|
| `moderation/none` — sempre safe (moderação manual) | v1 (até ~3k usuários) |
| `moderation/opennsfw` — IA open-source self-hosted | v2 (3k-15k usuários) |
| `moderation/rekognition` — AWS Rekognition / Google Cloud Vision | Quando volume justificar custo |

### 6.3 Push Notifications

```go
// internal/domain/notifications.go
type PushNotification interface {
    Send(ctx context.Context, userID string, title, body string, data map[string]string) error
    SendBatch(ctx context.Context, userIDs []string, title, body string, data map[string]string) error
}
```

| Implementação | Quando usar |
|--------------|-------------|
| `push/none` — sem push (só notificações in-app via WebSocket pool) | v1 (postergado) |
| `push/fcm` — Firebase Cloud Messaging | Quando push for necessário |
| `push/webpush` — Web Push API nativa (sem depender de FCM) | Alternativa se FCM for problemático para conteúdo adulto |

### 6.4 Backup

```go
// internal/domain/backup.go
type BackupManager interface {
    Backup(ctx context.Context) error
    Restore(ctx context.Context, snapshotID string) error
    ListSnapshots(ctx context.Context) ([]Snapshot, error)
}
```

| Implementação | Quando usar |
|--------------|-------------|
| `backup/sidecar` — script externo que copia SQLite + arquivos para S3 | v1 (sidecar container) |
| `backup/internal` — integrado ao binário Go (agendado via time.Ticker) | Alternativa mais simples que sidecar |

### 6.5 Princípio geral

Toda funcionalidade postergada segue este padrão:

```
internal/domain/<feature>/   ← interface
internal/<feature>/<impl>/   ← implementação concreta
```

Isso garante que o código de negócio nunca precisa mudar quando a implementação troca — exatamente como já está definido para auth e repository.

---

## 7. Estratégia de Marketing e Vendas

### 5.1 Canais de Aquisição

| Canal | Custo | Volume estimado | Qualidade |
|-------|-------|-----------------|-----------|
| **SEO orgânico** (blog "IBGE do Sexo") | Zero | Médio | Alta |
| **Boca a boca / indicação** | Zero | Médio-alto | Altíssima |
| **Grupos de Telegram/WhatsApp** | Zero | Alto | Média |
| **Content marketing** (blog, YouTube) | Tempo | Médio | Alta |
| **Parcerias com casas de swing** | Zero | Baixo-médio | Altíssima |
| **Influencers liberais** | Baixo (permuta) | Médio | Alta |
| **SEM (Google Ads)** | Alto — só se receita permitir | Alto | Média-baixa |
| **Cripto comunidade** (USDT) | Zero | Baixo | Média |

> **Princípio:** aquisição 100% orgânica até break-even. SEM só entra se houver receita excedente para financiar.

### 5.2 Conteúdo e SEO

- Blog **"IBGE do Sexo"** — dados, estatísticas, pesquisas sobre comportamento sexual brasileiro
- Top-funnel: "guia swing", "casas de swing em [cidade]", "como começar no lifestyle"
- Mid-funnel: comparações, dicas de segurança, etiqueta liberal
- Bottom-funnel: termos transacionais ("rede social liberal", "app swing")

### 5.3 Programa de Indicação

- Free user convida → ambos recebem 1 mês de Premium grátis
- Premium user convida → ganha 1 mês grátis por conversão

### 5.4 Parcerias B2B

- Casas de swing: listing destacado no mapa + verificação de estabelecimento
- Motéis: localização no mapa + promoções geolocalizadas
- Sex shops: anúncios nativos segmentados por público

### 5.5 Launch Strategy

| Fase | Duração | Objetivo | Estratégia |
|------|---------|----------|------------|
| **Validação** | Contínua (inicia junto com o código) | Coletar emails de interessados | Landing page com mockups do mapa + radar divulgada em 3-5 grupos Telegram/WhatsApp |
| **Beta fechado** | 3 meses | 300 usuários ativos | Convites em grupos Telegram/WhatsApp (primeiros da landing page), parcerias com casas de swing |
| **Soft launch** | 2 meses | 2.000 usuários | Abertura pública, press release em blogs liberais, influencers (permuta) |
| **Growth** | 6 meses | 10.000 usuários | SEO, content marketing, programa de indicação |
| **Scale** | 7+ meses | 50.000+ usuários | Escalar canais comprovados, parcerias B2B, cripto |

> Cronograma conservador para operação solo. Sem pressão de runway externo.

---

## 8. Modo Guerrilha — Estratégia de Custo Quase Nulo

> **Princípio fundamental:** o plano não depende de investimento externo. Cada real gasto precisa ser justificado contra a alternativa de não gastar.

### 6.1 Filosofia Low-Cost

A stack tecnológica (Go single binary + SQLite + React PWA + filesystem) foi escolhida **especificamente** para permitir operação com custos quase nulos. Não é um detalhe técnico — é o motor do modelo de negócio.

| Recurso | Custo "startup normal" | Custo Liberages (Modo Guerrilha) |
|---------|------------------------|-----------------------------------|
| VPS | R$500-2.000/mês (AWS, k8s) | R$25-50/mês (Hetzner/Contabo, 1 VPS, 1 binário) |
| Banco | R$200-500/mês (RDS Postgres) | R$0 (SQLite no mesmo VPS) |
| Storage de imagens | R$200-800/mês (S3) | R$0 (filesystem no VPS; migrar só no limite) |
| CDN | R$200/mês (Cloudflare Pro) | R$0 (Cloudflare free tier) |
| E-mail transacional | R$100-400/mês | R$0 (SES free tier 62k/mês) → R$10/mês depois |
| Map tiles | R$200+/mês (MapTiler/Mapbox) | R$0 (OpenStreetMap self-hosted no mesmo VPS) |
| Moderação IA | R$500-2.000/mês (AWS Rekognition) | R$0 (manual pelo fundador) → IA open-source self-hosted depois |
| Monitoramento | R$100-300/mês (Datadog) | R$0 (Prometheus + Grafana self-hosted) |
| Ferramentas SaaS | R$100-500/mês | R$0 (open source: git, Linear free, Notion free) |
| **Total mensal** | **R$1.300-5.000** | **R$25-60** |

### 6.2 Quando Sair do Modo Guerrilha

A operação permanece no Modo Guerrilha até que um destes gatilhos seja acionado:

| Gatilho | Limite | Ação |
|---------|--------|------|
| **SQLite write throughput** | >50 writes/s médio | Migrar para PostgreSQL (mesmo VPS inicial) |
| **Usuários ativos diários** | >30.000 | Considerar segundo VPS (read replica) |
| **Storage de imagens** | >50GB | Migrar para S3 ou similar |
| **Volume de moderação** | >100 denúncias/dia | Contratar moderador part-time |
| **Receita mensal** | >R$15.000 | Investir em SEM eContents de marketing pago |

### 6.3 Princípios Operacionais Pós-MVP

1. **SQLite (não Postgres)** até onde aguentar — WAL mode, pragma tuning, índices otimizados
2. **VPS único** (não k8s, não AWS) — 1 binário, 1 processo, 1 servidor
3. **Filesystem local** (não S3) para imagens — migrar só acima de 50GB
4. **Sem CDN pago** até tráfego justificar (Cloudflare free tier suffice inicialmente)
5. **Sem IA de moderação paga** — IA open-source self-hosted (ex.: OpenNSFW2) ou moderação 100% manual até 25k usuários
6. **E-mail transacional via SES** a R$0,10 por mil enviados
7. **Zero ferramentas SaaS de gestão pagas**
8. **Map tiles self-hosted** (OpenStreetMap data, renderizado no próprio VPS)
9. **Toda feature é avaliada contra custo:** "isto adiciona R$X/mês? Pode ser feito com C$0?"

---

## 9. Análise Financeira

### 7.1 Custos Operacionais — Três Cenários

| Item | Modo Guerrilha (real, bootstrapped) | Custo Otimizado (pós-break-even) | Custo Ideal (com seed/investidor) |
|------|-------------------------------------|----------------------------------|-----------------------------------|
| VPS | R$30 | R$80 | R$200 |
| Domínio | R$3 | R$3 | R$3 |
| E-mail | R$0 | R$10 | R$100 |
| Storage | R$0 | R$0 | R$200 |
| Moderação IA | R$0 | R$0 | R$500 |
| Moderação humana | R$0 | R$0 | R$1.500 |
| Marketing/SEM | R$0 | R$500 | R$5.000 |
| Jurídico | R$0 | R$0 | R$500 |
| **Total mensal** | **R$33** | **R$593** | **R$8.003** |

> O plano opera no "Modo Guerrilha" (coluna 1) até break-even. A coluna 2 representa o cenário pós-break-even com receita reinvestida. A coluna 3 representa o cenário com seed/investidor — é o plano que NÃO dependemos.

### 7.2 Projeção de Receita (18 meses — Bootstrapped)

| Mês | Usuários | Pagantes | MRR assinaturas | Microtransações | Ads | B2B | MRR total | Custos | Resultado |
|-----|----------|----------|-----------------|------------------|-----|------|-----------|--------|-----------|
| 1 | 100 | 3 | R$90 | R$0 | R$0 | R$0 | R$90 | R$33 | +R$57 |
| 3 | 800 | 24 | R$720 | R$20 | R$50 | R$0 | R$790 | R$35 | +R$755 |
| 6 | 3.000 | 90 | R$2.500 | R$200 | R$300 | R$200 | R$3.200 | R$40 | +R$3.160 |
| 9 | 7.000 | 210 | R$5.900 | R$800 | R$800 | R$800 | R$8.300 | R$100 | +R$8.200 |
| 12 | 15.000 | 450 | R$12.600 | R$2.000 | R$2.000 | R$1.500 | R$18.100 | R$300 | +R$17.800 |
| 15 | 30.000 | 900 | R$25.200 | R$5.000 | R$5.000 | R$3.000 | R$38.200 | R$600 | +R$37.600 |
| 18 | 60.000 | 1.800 | R$50.400 | R$11.000 | R$10.000 | R$5.000 | R$76.400 | R$1.000 | +R$75.400 |

> Projeção conservadora, alinhada com aquisição 100% orgânica (zero SEM). Custos sobem apenas quando gatilhos do Modo Guerrilha são acionados.

### 7.3 Break-Even

**Break-even projetado: mês 1-2** (Modo Guerrilha custa R$33/mês — qualquer receita cobre).

Break-even "real" (receita superando custo de vida do fundador): depende de custo pessoal, não de custo de infra.

### 7.4 Métricas Unitárias

| Métrica | Ano 1 | Ano 2 | Ano 3 |
|---------|-------|-------|-------|
| CAC | R$2,00 | R$1,50 | R$1,00 |
| LTV (média) | R$120 | R$240 | R$400 |
| LTV/CAC | 60x | 160x | 400x |
| Churn mensal | 15% | 10% | 7% |
| Payback period | 1 mês | <1 mês | <1 mês |

> ⚠️ **Aviso sobre LTV/CAC:** valores acima de 25x indicam que o CAC está subestimado. Em bootstrapping orgânico o custo real é tempo (não dinheiro), então CAC monetário tende a zero. LTV/CAC não é a melhor métrica aqui — melhor usar **payback period** (quase instantâneo quando CAC ≈ R$0).

> ⚠️ **Aviso sobre churn:** 15% no ano 1 é realista para indústria adult (onde uso é pragmático e usuário sai assim que fecha encontro). O Sexlog tem churn elevado e mascara com volume de novos cadastros. Liberages precisa de retenção via gamificação (radar de intenção, checkins, notificações) para reduzir churn ao longo do tempo.

### 7.5 Investimento

**Bootstrapping from zero. Não há seed solicitado.**

| Cenário | Valor | Origem |
|---------|-------|--------|
| **Real (operando hoje)** | R$0 a R$15.000 | Recursos próprios do fundador |
| **Se investidor aparecer** | R$50.000-150.000 | Seed externo (opcional, não dependente) |

**Se seed externo materializar:**

| Alocação | Valor (c/ R$150k) | % |
|----------|-------------------|---|
| Reserva pessoal (12 meses) | R$72.000 | 48% |
| Marketing e aquisição | R$45.000 | 30% |
| Infraestrutura e serviços | R$18.000 | 12% |
| Reserva legal/jurídico | R$10.000 | 7% |
| Reserva de emergência | R$5.000 | 3% |

> **Nota:** "Desenvolvimento" não é linha de custo — o fundador é o desenvolvedor. Seed cobre custo de vida, não salary de terceiros.

### 7.6 Projeção Trienal

| Ano | Usuários | Receita total | Custos | Lucro líquido |
|-----|----------|---------------|--------|---------------|
| Ano 1 | 15.000 | R$130.000 | R$5.000 | R$125.000 |
| Ano 2 | 60.000 | R$550.000 | R$25.000 | R$525.000 |
| Ano 3 | 150.000 | R$1.400.000 | R$80.000 | R$1.320.000 |

> Custos baixos reflejam Modo Guerrilha com escalonamento só em gatilhos. Sem headcount, sem SaaS, sem cloud enterprise.

---

## 10. Análise de Riscos

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| **Banimento de meios de pagamento** | Alta | Alto | **Pix desde o dia 1** (principal), cripto (USDT) como fallback. Não depender de Stripe/adquirentes internacionais. Mercado Pago como secundário. |
| **Block de anúncios e redes sociais** | Alta | Médio | Marketing de conteúdo (blog), SEO, parcerias, comunidade orgânica |
| **Processos legais (conteúdo)** | Baixa | Alto | Age gate obrigatório, moderação IA + humana, DMCA compliance, advogado especializado (quando receita permitir) |
| **Vazamento de dados** | Baixa | Altíssimo | Criptografia E2E em mensagens, blur facial, dados sensíveis criptografados em nível aplicação |
| **Perfis falsos/golpistas** | Alta | Médio | Verificação documental disponível para todos (não só pagantes), reputação por denúncia, IA detecção fake |
| **Concorrência Sexlog** | Média | Médio | Foco em inovação (mapa-radar gamificado, privacidade, UX), não competir por SEO head-to-head |
| **Escalabilidade SQLite** | Média | Médio | SQLite WAL mode com pragma tuning. Migrar para PostgreSQL quando write throughput >50/s. Interfaces Go facilitam troca. |
| **Regulação conteúdo adulto** | Baixa | Alto | Compliance com LGPD, age gate, termos de uso claros, advogado especializado (quando receita permitir) |
| **Churn alto por estigma** | Alta | Médio | Desktop app discreto, PWA (não aparece na lista de apps), modo invisível. Gamificação (radar, checkins) para aumentar engajamento |
| **Dependência de fundador único** | **Alta** | **Alto** | Documentação completa (AGENTS.md, specs), automação de deploy, código testado. Sem mitigação real além de saúde e disciplina do fundador. É o risco aceito. |
| **SQLite write lock em escala** | Média | Médio | Produto é write-heavy (checkins, notificações, matches em tempo real). Pode bater limite antes de 100k UA — provavelmente em 25-50k. Monitorar QPS de write, migrar cedo. |

---

## 11. Plano Operacional

### 9.1 Tecnologia

| Componente | Tecnologia | Custo |
|------------|-----------|-------|
| Backend | Go 1.26+ single binary | R$0 (open source) |
| Frontend | React 19+ SPA + PWA | R$0 (open source) |
| Banco | SQLite → PostgreSQL (quando necessário) | R$0 → R$200/mês |
| Mapa | Leaflet + tiles próprios | R$0 |
| Map tiles | Self-hosted OpenStreetMap no VPS | R$0 |
| Storage de imagens | Filesystem local → S3 (quando >50GB) | R$0 → R$200/mês |
| Moderação IA | Manual → IA open-source self-hosted → API externa (se volume justificar) | R$0 → R$0 → R$500/mês |
| E-mail | Amazon SES free tier | R$0 → R$10/mês |
| CDN | Cloudflare free tier | R$0 |
| Monitoramento | Prometheus + Grafana self-hosted | R$0 |

### 9.2 Operação de Moderação

- **Fase 1 (0-3k usuários):** Moderação 100% manual pelo fundador
- **Fase 2 (3k-15k):** IA open-source self-hosted para triagem + fundador decide casos limítrofes
- **Fase 3 (15k+):** IA + moderador part-time (freelancer, contingente a receita)
- **Política:** Moderação híbrida — IA detecta, humano decide em casos limítrofes
- **Transparência:** todo banimento tem explicação e processo de apelo

### 9.3 Legal e Compliance

| Item | Status | Responsável |
|------|--------|-------------|
| Termos de uso | A redigir (templates open source como base) | Fundador |
| Política de privacidade (LGPD) | A redigir | Fundador |
| Age gate (18+ obrigatório) | Especificado | Fundador |
| DMCA compliance | A implementar | Fundador |
| CNPJ e regime tributário | A definir (MEI inicial) | Contador (quando receita justificar) |

> **Princípio:** jurídico é DIY com templates até que receita excedente permita contratar advogado especializado. Advogado é contingente, não linha fixa.

---

## 12. Roadmap

### Fase 1 — "O Mapa + Radar" — Meses 1-4

- [x] Repositório e especificação criados
- [ ] Autenticação local (JWT + bcrypt) + PIN rápido
- [ ] Age gate com cookie assinado HMAC
- [ ] Mapa de Locais — Leaflet com tipos de local, CRUD admin
- [ ] Radar de Atividade — intenção "a fim hoje", clusters fuzzy, notificação de proximidade
- [ ] Checkins em locais públicos com notificações de checkin próximo
- [ ] Busca textual com SQLite FTS5
- [ ] PWA com install prompt
- [ ] Deploy em produção (VPS R$30/mês)
- [ ] Beta fechado (300 usuários)

### Fase 2 — "A Rede" (Perfis + Matching + Conteúdo) — Meses 5-9

- [ ] Perfis de usuário (indivíduos) com Kit de Relacionamento
- [ ] Lista de Fetiches + Match %
- [ ] Mood do Momento
- [ ] Swipe (curtir/não curtir) com match
- [ ] Chat privado com E2E (Web Crypto)
- [ ] Fotolog Diário + Álbum + Story 24h
- [ ] Modo Fantasma Total
- [ ] Modo Falso (botão de emergência)
- [ ] Alarme de Screenshots
- [ ] Upload de fotos com blur automático
- [ ] Moderação Híbrida (robô + manual)
- [ ] Ingresso Moderado por Humano
- [ ] Badges + XP + Níveis (gamificação base)
- [ ] Anúncios display no free (ad network adult-friendly)
- [ ] Soft launch (2.000 usuários)

### Fase 3 — "Comunidade + Eventos" (Meses 10-15)

- [ ] Fórum (categorias, tópicos, votação, FTS5)
- [ ] Comunidades com Presença Anônima
- [ ] Grupos com Curadoria
- [ ] Mural de Elogio Anônimo
- [ ] Confiança Mútua
- [ ] Contos Eróticos
- [ ] Criação e descoberta de eventos com geolocalização
- [ ] Check-in com PIN de eventos
- [ ] Agenda Liberal (calendário + feriados sazonais)
- [ ] Carona Solidária
- [ ] Lista de Presença Anônima em eventos
- [ ] Ranking Semanal (opt-in)
- [ ] Desafio Liberal Semanal
- [ ] Jogo da Verdade Liberal (pós-match)
- [ ] Bucket List + notificação de match por local em comum
- [ ] Encontro Surpresa
- [ ] QR Code de conexão + checkin
- [ ] Botão de pânico + Contato de emergência + local público sugerido
- [ ] Anúncios B2B locais no mapa + Caça ao Tesouro
- [ ] Growth (10.000 usuários)

### Fase 4 — "Modo Casal + Conteúdo Premium" (Meses 16-22)

- [ ] Modo Casal (conta compartilhada + double opt-in + moodboard)
- [ ] Live streaming com máscara facial
- [ ] Marketplace de Criadores (venda de conteúdo, comissão 15%)
- [ ] Match Boost (R$2-5 pagamento único para topo do feed)
- [ ] Presente Virtual Picante (microtransação)
- [ ] Assinatura de Presente (comprar Premium/VIP para outro)
- [ ] Cartão Fidelidade Digital (B2B)
- [ ] Verificação azul (documental, disponível para todos)
- [ ] Verificação por Vídeo-Chamada
- [ ] Validação Periódica (idade a cada 6 meses, identidade a cada 12)
- [ ] Parcerias B2B (casas de swing com câmera ao vivo, fidelidade)
- [ ] Geral growth (25-35k usuários)

### Fase 5 — "A Fortaleza" (Segurança + Escala) (Meses 23-30+)

- [ ] Júri Popular (denúncias por votação de verificados)
- [ ] Selo Anjo da Comunidade
- [ ] Análise de Sentimento da Comunidade (dashboard)
- [ ] Painel Público de Status (status.liberages.com)
- [ ] Sistema de Tickets de Suporte
- [ ] Detecção de screenshots aprimorada
- [ ] Pagamentos em cripto (USDT) como fallback
- [ ] Desktop app com system tray
- [ ] Migração SQLite → PostgreSQL (se gatilho de write throughput disparar)
- [ ] Broadcast "Tô a fim agora" (notificação ampla no raio)
- [ ] Scale (50.000+ usuários)

> **Cronograma conservador para operação solo.** Prazos podem estender conforme realidade. Sem pressão de runway externo.
>
> ⚠️ **O roadmap acima é referencial.** A ordem de implementação será definida pelo fundador separadamente, baseada em agrupamento natural de features e dependências técnicas.

---

## 13. Equipe e Estrutura

### 11.1 Equipe Atual

| Papel | Pessoa | Dedicação |
|-------|--------|-----------|
| Fundador / Tech Lead / Full-stack / Moderação / Marketing / Tudo | Fernando Passos | Full-time (e solo de forma permanente) |

### 11.2 Parcerias e Outsourcing Opcionais

Toda parceria abaixo é **opcional e contingente a receita excedente**. Não há plano de headcount. Nada disto precisa acontecer para o produto sobreviver.

| Item | Quando considerar | Tipo |
|------|--------------------|-----|
| Designer UI/UX (freelancer) | Se UX travar aquisição | Projeto pontual |
| Moderador part-time | Se denúncias >100/dia | Freelancer |
| Dev frontend (freelancer) | Se velocidade de features for gargalo | Projeto pontual |
| Advogado direito digital | Se processo legal materializar | Retainer |
| Gerente de comunidade | Se escala B2B justificar | Part-time |

### 11.3 Cultura Organizacional

- **Solo de forma permanente** — não há meta de crescer headcount
- **Open source friendly** — libs open source quando possível, contribuir de volta
- **Privacidade como valor interno** — o fundador usa o produto
- **Documentação como vantagem** — AGENTS.md e specs sempre atualizados
- **Custo quase nulo como disciplina** — cada real gasto é justificado contra a alternativa de não gastar

---

## 14. Estratégia de Saída

### 12.1 Cenários

| Cenário | Horizonte | Descrição | Valuation estimada |
|---------|-----------|-----------|---------------------|
| **Aquisição** | 3-5 anos | Sexlog ou concorrente adquire para modernizar base | 5-10x ARR |
| **Investimento série A** | 2-3 anos | VC entra para escalar (opcional) | R$5-15M |
| **Bootstrap rentável** | Contínuo | Crescimento orgânico, sem investimento externo — **cenário preferido** | N/A |
| **White-label** | 2 anos | Licenciar tecnologia do mapa-radar para outros mercados | Receita adicional SaaS |

### 12.2 Métricas para Saída

- 50.000+ usuários ativos
- R$50.000+ MRR
- LTV/CAC > 15x (realista)
- Churn < 10% mensal
- NPS > 50

---

## 15. Considerações Legais e Regulatórias

### 15.1 Age Verification (Porta de Entrada)
A legislação para plataformas adultas exige controle de maioridade. O Liberages usa um modelo híbrido para balancear conversão e compliance:
1. **Soft Gate (Acesso Inicial):** Auto-declaração (18+) gerando um cookie HMAC. Permite navegar, ver o feed e buscar perfis.
2. **Hard Gate (Acesso Completo):** Para interagir (chat, postar, check-in), o usuário deve enviar documento. A verificação é anônima: o sistema valida a idade e descarta a imagem/nome real, guardando apenas o hash de validação associado ao `user_id`.

### 15.2 Risco Financeiro e KYC (Moeda Única)
Por operar um sistema de **Moeda Única em Circuito Fechado** (closed-loop economy), a plataforma não se qualifica como instituição de pagamento perante o Banco Central. Como não há *cash-out* (saque) para usuários, a plataforma não realiza repasses financeiros P2P, mitigando a 100% o risco de lavagem de dinheiro e reduzindo drasticamente a carga de KYC. Compensações B2B são feitas via contratos de publicidade offline.

### 15.3 Privacidade e LGPD
A arquitetura é *privacy-by-design*:
- Nomes reais não são armazenados (apenas pseudônimos/apelidos).
- O Radar nunca revela coordenadas brutas.
- O E2E (End-to-End Encryption) nas DMs está no roadmap (post-MVP) para garantir que nem o servidor possa ler mensagens privadas, com a arquitetura preparada desde o Dia 1.
- Contas de Casal são uma conta única administrada por duas pessoas (sem retenção duplicada de dados).

### 15.4 Isenção de Responsabilidade Física
Features que induziam risco físico (como "Carona Solidária" e encontros às cegas forçados) foram removidas do escopo. A criação de eventos públicos é restrita a usuários validados pelo "Web of Trust" (mínimo de 4 amigos reais), atuando como salvaguarda da comunidade.

## 16. Métricas de Sucesso (KPIs)

### 14.1 North Star Metric

**Usuários ativos semanais (WAU)** que usam o mapa-radar pelo menos 1x por semana.

### 14.2 KPIs por Categoria

| Categoria | KPI | Meta ano 1 |
|-----------|-----|------------|
| **Crescimento** | Novos cadastros/dia | 50+ |
| **Engajamento** | DAU/MAU | 25%+ |
| **Engajamento** | Sessões/usuário/dia | 3+ |
| **Engajamento** | Checkins/dia | 100+ |
| **Engajamento** | Notificações de radar enviadas/dia | 200+ |
| **Engajamento** | Fotos postadas/dia (fotolog + álbum + story) | 150+ |
| **Engajamento** | Desafios semanais completados | 30%+ dos ativos |
| **Engajamento** | Contos publicados/semana | 10+ |
| **Engajamento** | Usuários com mood ativo | 20%+ dos DAU |
| **Monetização** | % conversão free→pago | 4%+ |
| **Monetização** | MRR assinaturas | R$15.000 |
| **Monetização** | Receita microtransações/mês | R$2.000 |
| **Monetização** | Receita ads/mês | R$1.500 |
| **Monetização** | Receita B2B/mês | R$1.000 |
| **Retenção** | Churn mensal | <15% |
| **Retenção** | Retenção D30 | 30%+ |
| **Qualidade** | Denúncias/1k usuários | <5 |
| **Qualidade** | Tempo de moderação | <48h |
| **Qualidade** | Tickets de suporte resolvidos <24h | 80%+ |
| **Privacidade** | % usuários com blur ativo | 60%+ |
| **Privacidade** | % usuários com E2E habilitado nas DMs | 90%+ |
| **Gamificação** | Badges desbloqueadas/dia | 50+ |
| **Gamificação** | XP distribuído/dia | 5.000+ |

---

## 17. Análise SWOT

### Forças (Strengths)

- Stack tecnológica moderna e barata (Go single binary, SQLite, React PWA)
- Especificação completa e documentada (AGENTS.md, specs)
- PWA bypassa banimento de App Store/Play Store
- **Custo operacional quase nulo (Modo Guerrilha: R$33/mês)**
- Fundador técnico (não depende de terceiros para desenvolver)
- Diferencial real vs concorrente (mapa-radar gamificado, privacidade, UX)
- **Operação solo permanente — custo fixo mínimo**

### Fraquezas (Weaknesses)

- **Fundador único, solo de forma permanente (risco máximo aceito)**
- Sem massa crítica inicial (network effect)
- Sem orçamento de marketing vs Sexlog
- Marca desconhecida
- Content moderation é complexo em escala (manual até volume justificar). 71 features requerem priorização rigorosa — não dá pra implementar tudo no MVP

### Oportunidades (Opportunities)

- Sexlog não inova há 17 anos
- Público liberal cresceu pós-pandemia
- Banimento de apps nas lojas é barreira para concorrentes
- Pix como pagamento universal no Brasil (sem dependência de adquirentes internacionais)
- Cripto abre novo canal de pagamento
- Desktop app discreto é diferencial sem concorrente
- Parcerias B2B com casas de swing são canal de aquisição sem custo

### Ameaças (Threats)

- Sexlog pode copiar features (mapa, blur)
- Welcoming/Recombinação de features pelo Sexlog pode neutralizar diferencial
- Regulação de conteúdo adulto pode apertar
- Gateways de pagamento internacionais podem banir plataforma (mitigado com Pix)
- Vazamento de dados seria fatal para confiança
- Plataformas maiores (Tinder, FetLife) podem entrar no nicho brasileiro
- **SQLite pode bater limite de write antes do esperado** (produto é write-heavy)

---

## 18. Tecnologia como Vantagem Competitiva

### 16.1 Por que Go + SQLite é uma vantagem

| Aspecto | Stack típica (Sexlog-like) | Liberages (Modo Guerrilha) |
|---------|---------------------------|---------------------------|
| Backend | Node.js + Express ou PHP | Go single binary |
| Frontend | SSR (Next.js) ou monolito | React SPA + PWA |
| Banco | PostgreSQL + Redis | SQLite (escalável até 30-50k UA write-heavy) |
| Deploy | 5+ containers | 1 binário + 1 DB file |
| Custo infra | R$1.300-5.000/mês | **R$25-60/mês** |
| Cold start | Segundos | Milissegundos |
| PWA | Não | Sim (bypass App Store) |
| Desktop app | Não | Sim (system tray discreto) |

### 16.2 Arquitetura preparada para escala

- Interfaces Go na camada de dados → troca SQLite por PostgreSQL sem mudar lógica de negócio
- Interfaces Go na camada de auth → troca JWT local por Keycloak/OAuth2 sem mudar código
- Single binary → fácil deploy em qualquer VPS, sem orquestração complexa
- Frontend React SPA → independente do backend, pode ser servido por CDN

---

## 19. Apêndice

### 17.1 Glossário

| Termo | Definição |
|-------|-----------|
| **Lifestyle liberal** | Prática de sexualidade não monogâmica consensual (swing, troca de casais, etc.) |
| **Swing** | Troca de casais para atividades sexuais |
| **Ménage** | Relação sexual entre três pessoas |
| **BDSM** | Bondage, Dominação, Sadismo e Masoquismo |
| **Age gate** | Verificação de idade para acesso a conteúdo adulto |
| **PWA** | Progressive Web App — aplicação web installável, funciona offline |
| **BFF** | Backend for Frontend — camada intermediária entre frontend e API |
| **FTS5** | Full Text Search 5 — extensão do SQLite para busca textual |
| **UUID v7** | Identificador único sortable por tempo |
| **Glassmorphism** | Estilo de UI com efeito de vidro fosco (backdrop-blur) |
| **CAC** | Customer Acquisition Cost — custo de aquisição de cliente |
| **LTV** | Lifetime Value — receita total esperada de um cliente |
| **MRR** | Monthly Recurring Revenue — receita recorrente mensal |
| **ARPU** | Average Revenue Per User — receita média por usuário |
| **WAU** | Weekly Active Users — usuários ativos semanais |
| **DAU/MAU** | Daily/Monthly Active Users — ratio de engajamento |
| **Radar de intenção** | Funcionalidade gamificada onde usuário marca "a fim hoje" e recebe notificação quando compatível está próximo — estilo Pokémon GO |
| **Checkin** | Registro de presença em local público (casa de swing, bar) que aparece no mapa para outros usuários |
| **Localização fuzzy** | Localização aproximada (bairro/zona) que nunca revela ponto exato do usuário |
| **Modo Guerrilha** | Estratégia operacional de custo quase nulo — 1 VPS, SQLite, filesystem, sem SaaS pago |

### 17.2 Referências

- Sexlog dados públicos: relatórios de mídia 2022
- Pesquisas de comportamento sexual brasileiro (dados secundários)
- Documentação técnica: `app/AGENTS.md`
- Dossiê de mercado: `app/spec/dossie-mercado.md`
- Especificação técnica do mapa: `app/spec/mapa-interativo.md`
- Especificação de 47 features: `app/spec/features.md`
- 24 features safadarias: `app/spec/features-safadia.md`

### 17.3 Suposições do Plano

1. Conversão free→pago de 4% (conservador; indústria SaaS: 2-10%)
2. Churn mensal de 15% no ano 1 (realista para indústria adult: 12-20%)
3. ARPU médio de R$34/mês (misto Premium R$29,90 + VIP R$49,90)
4. Custo de aquisição de R$1-2 (canais predominantemente orgânicos — custo real é tempo, não dinheiro)
5. SQLite escala até ~30-50k UA em modo write-heavy (produto tem checkins, matches e notificações em tempo real)
6. Sem investimento externo necessário (bootstrapping from zero)
7. Operação solo permanente — sem headcount planejado

---

*Documento confidencial. Não distribuir sem autorização.*
*© 2026 Liberages. Todos os direitos reservados.*