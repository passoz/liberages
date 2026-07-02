# Plano de Negócios — Liberages

> **Confidencial.** Documento estratégico para operação e direcionamento do produto.
> Última atualização: Julho 2026

---

## Sumário Executivo

### O que é

**Liberages** é uma plataforma digital (PWA + desktop app) construída em Go single-binary com React SPA, voltada para o público liberal brasileiro (swing, troca de casais, ménage, BDSM, exibicionismo, voyeurismo). O produto central é o **Mapa Interativo** — uma rede social liberal onde a descoberta geográfica é o hub da experiência, combinando gamificação estilo "Pokémon GO do prazer" (você não sabe quem ao seu redor está a fim de uma aventura), comunidade, eventos, conteúdo e privacidade em camadas.

### Oportunidade

- Mercado dominado por **1 player** (Sexlog) com ~17 milhões de usuários cadastrados, mas sem inovar desde 2007.
- Sexlog cobra R$49,90/mês por um produto com UX datada, sem mobile nativo, sem mapa e moderação frágil.
- Banimento de apps para adultos nas lojas (App Store/Play Store) abre espaço para **PWA** — alternativa que bypassa gates das lojas.
- Público liberal é altamente engajado, recorrente e disposto a pagar por privacidade e melhores ferramentas.

### Diferencial

| Pilar | O que resolve | Como |
|-------|---------------|------|
| **Mapa-radar gamificado** | Dificuldade de saber quem está a fim, ao seu redor, agora | Radar de intenção em tempo real (estilo Pokémon GO): você não vê quem é — só sabe que alguém compatível está perto. Checkins em locais públicos com notificações próximas. Localização nunca é exata, sempre fuzzy. |
| **Privacidade em camadas** | Medo de exposição (fotos identificáveis, perfis falsos) | Blur facial automático, perfil anônimo, modo invisível, selfie destrutível, verificação opcional |
| **Comunidade moderada** | Perfis falsos, golpistas, ambiente tóxico | Verificação de identidade disponível para todos (não só pagantes), reputação, moderação híbrida (IA + humana), selo verificado |

### Modelo de receita

Freemium em 3 camadas: **Free** (grátis, com anúncios nativos), **Premium** (R$29,90/mês, sem anúncios), **VIP** (R$49,90/mês) + comissão de 15% sobre conteúdo pago + parcerias B2B (casas de swing, motéis, sex shops). Free users geram receita via anúncios display e nativos — não são sosti­nháveis como custo zero.

### Equipe

**Um fundador técnico (full-stack Go + React), solo de forma permanente.** Toda contratação ou parceria é opcional e contingente a receita excedente. Não há plano de headcount.

### Investimento

**Bootstrapping from zero.** Seed de R$0 a R$15.000 (recursos próprios). A operação é desenhada para custos quase nulos. Se investidor aparecer, ótimo — mas o plano não depende disso para existir.

### Projeção (ano 1)

| Métrica | Mês 6 | Mês 12 | Mês 18 |
|---------|-------|--------|--------|
| Usuários cadastrados | 3.000 | 15.000 | 60.000 |
| Usuários pagantes | 90 | 450 | 1.800 |
| MRR assinaturas | R$2.500 | R$12.600 | R$50.400 |
| Receita microtransações | R$200 | R$2.000 | R$11.000 |
| Receita de anúncios (free) | R$300 | R$2.000 | R$10.000 |
| Receita B2B | R$200 | R$1.500 | R$5.000 |
| MRR total | R$3.200 | R$18.100 | R$76.400 |
| CAC | R$2,00 | R$1,50 | R$1,00 |
| LTV | R$120 | R$240 | R$400 |
| LTV/CAC | 60x | 160x | 400x |

> ⚠️ LTV/CAC acima de 25x deve ser tratado como sinal de que o CAC está subestimado — não como previsão literal. CAC real em bootstrapping orgânico é difícil de medir porque o custo é tempo, não dinheiro.

---

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
3. **Privacidade em camadas** — blur facial IA, selfie destrutível, modo invisível
4. **Single binary Go + SQLite** — custo de infraestrutura quase nulo (1 processo, 1 VPS barato)
5. **Desktop app discreto** — system tray com ocultação (ninguém vê no celular)
6. **Comunidade moderada** — ambiente seguro atrai e retém mulheres (segmento mais carente)
7. **Operação solo enxuta** — custo fixo baixíssimo permite sobreviver com pouca receita inicial

---

## 4. Modelo de Receita

### 4.1 Estrutura de Preços

| Tier | Preço | Funcionalidades |
|------|-------|-----------------|
| **Free** | Grátis (com anúncios) | Perfil, mapa + radar básico, 5 matches/dia, chat limitado, fotos com blur automático, anúncios display e nativos |
| **Premium** | R$29,90/mês | Tudo do Free **sem anúncios**, match ilimitado, modo invisível, lives privadas, verificação azul, relatório de privacidade |
| **VIP** | R$49,90/mês | Tudo do Premium + prioridade no matching, conteúdo exclusivo de creators, convites VIP, acesso antecipado |

### 4.2 Receita de Anúncios (Free)

Free users não são sustentáveis como custo zero. Anúncios são o motor de receita do tier free.

| Tipo de anúncio | Descrição | CPM estimado |
|-----------------|-----------|---------------|
| **Anúncios B2B locais** | Casas de swing, motéis, sex shops com geolocalização no mapa | R$8-15 (alto valor, alta relevância) |
| **Display adult-friendly** | Redes display (ExoClick, JuicyAds) — banner e inline no feed | R$2-5 |
| **Affiliate links** | Sex toys, lingerie, produtos — CPA | R$5-20 por conversão |
| **Anúncio próprio (upgrade)** | Banner "Cansado dos anúncios? Vire Premium" — conversão mais importante, custo zero | N/A (reduz churn de ads, aumenta MRR) |

**Política editorial de anúncios:**
- ❌ Nada de pop-under, redirects, malware ads, auto-play de áudio
- ❌ Nada de anúncios em telas de configuração, privacidade ou segurança
- ✅ Apenas display estático e nativo inline no feed e no mapa
- ✅ Anúncios B2B locais aparecem como marcadores diferenciados no mapa (não invasivos)

**Projeção de receita de anúncios:**

| Período | Free users | Page views/mês | CPM médio | Receita ads/mês |
|---------|------------|-----------------|-----------|-----------------|
| Mês 6 | 2.700 | 100k | R$2,50 | R$250 |
| Mês 12 | 13.500 | 600k | R$3,00 | R$1.800 |
| Mês 18 | 54.000 | 2.5M | R$4,00 | R$10.000 |

### 4.3 Receitas Adicionais

| Fonte | Descrição | Margem estimada |
|-------|-----------|------------------|
| **Comissão Marketplace de Criadores** | 15% sobre vendas de conteúdo (fotos, vídeos, packs, assinaturas) | 15% |
| **Match Boost** | R$2-5 pagamento único para topo do feed de swipe por 1h | 95%+ |
| **Presente Virtual Picante** | Emojis animados a R$1-3 cada, comissão 20% | 20% |
| **Assinatura de Presente** | Comprar Premium/VIP de presente para outro usuário | ~100% (mesma receita da assinatura) |
| **Parceria B2B (presença no mapa)** | Casas de swing, motéis, sex shops pagam por destaque no mapa | 80%+ |
| **Parceria B2B (Caça ao Tesouro)** | Parceiro paga por lead gerado via tesouro coletável no mapa | 85%+ |
| **Cartão Fidelidade (B2B)** | Parceiro paga rateio de vouchers resgatados no programa de pontos | 70%+ |
| **Eventos promovidos** | Destaque pago no feed de eventos | 90%+ |

### 4.4 Estratégia de Preço

- **Mais barato que Sexlog** no Premium (R$29,90 vs R$49,90) — aquisição agressiva
- **VIP igual ao Sexlog** (R$49,90) mas com mais valor
- **Mulheres solteiras** ganham Premium grátis (estratégia de equilíbrio de gênero)
- **Desconto anual** (30% off) para aumentar LTV e reduzir churn
- **Anúncios só no Free** — o ads paga o free existir; o Premium paga o usuário escapar do ads

### 4.5 Projeção de Mix

| Métrica | Ano 1 | Ano 2 | Ano 3 |
|---------|-------|-------|-------|
| % Free | 94% | 88% | 82% |
| % Premium | 5% | 9% | 14% |
| % VIP | 1% | 3% | 4% |
| ARPU (receita por usuário pago) | R$34/mês | R$38/mês | R$42/mês |

### 4.6 Verificação de Identidade — Nota Importante

O selo azul (verificação documental) **não é exclusivo de pagantes**. Qualquer usuário pode verificar — a verificação custa simbólico (ou é grátis para os primeiros N usuários). Isso é uma decisão de segurança da comunidade: se só pagante é verificado, o golpista "verificado" é o que pagou, não o que é real. Verificação para todos melhora a integridade da base e atrai o segmento mais carente (mulheres solteiras). Pagantes ganham **prioridade** na fila de verificação, não exclusividade.

---

## 5. Estratégia de Marketing e Vendas

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
| **Beta fechado** | 3 meses | 300 usuários ativos | Convites em grupos Telegram/WhatsApp, parcerias com casas de swing |
| **Soft launch** | 2 meses | 2.000 usuários | Abertura pública, press release em blogs liberais, influencers (permuta) |
| **Growth** | 6 meses | 10.000 usuários | SEO, content marketing, programa de indicação |
| **Scale** | 7+ meses | 50.000+ usuários | Escalar canais comprovados, parcerias B2B, cripto |

> Cronograma conservador para operação solo. Sem pressão de runway externo.

---

## 6. Modo Guerrilha — Estratégia de Custo Quase Nulo

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

## 7. Análise Financeira

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

## 8. Análise de Riscos

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

## 9. Plano Operacional

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

## 10. Roadmap

### Fase 1 — "O Mapa + Radar" (MVP) — Meses 1-4

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

---

## 11. Equipe e Estrutura

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

## 12. Estratégia de Saída

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

## 13. Considerações Legais e Regulatórias

### 13.1 LGPD (Lei Geral de Proteção de Dados)

- Dados sensíveis (orientação sexual, preferências) exigem **consentimento explícito**
- Política de privacidade clara e acessível
- Direito de exclusão de dados (deletar conta e todos os dados)
- Dados criptografados em nível de aplicação
- DPO (Data Protection Officer) quando passar de determinado volume

### 13.2 Age Gate Obrigatório

- Verificação 18+ com cookie assinado HMAC
- Sem cache de data de nascimento — só confirmação de maioridade
- Banner de idade em todas as páginas públicas

### 13.3 Conteúdo Adulto

- Plataforma hospeda conteúdo adulto gerado por usuários (UGC)
- Compliance DMCA para remoção de conteúdo copyright
- Política de conteúdo proibido (menores, não consensual, zoofilia, etc.)
- Moderação ativa e reativa
- Advogado especializado em direito digital quando receita permitir

### 13.4 Pagamentos

- **Pix é o pagamento principal desde o dia 1** — não depende de adquirentes internacionais
- **Mercado Pago** como secundário (aceita Pix dentro dele)
- **Cripto (USDT)** como fallback para usuários que preferem discrição total
- **Não depender de Stripe** — Stripe banheira conteúdo adulto UGC no Brasil
- Compliance KYC/AML se necessário para pagamentos em cripto

---

## 14. Métricas de Sucesso (KPIs)

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

## 15. Análise SWOT

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
- Content moderation é complexo em escala (manual até volume justificar). 47 features requerem priorização rigorosa — não dá pra implementar tudo no MVP

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

## 16. Tecnologia como Vantagem Competitiva

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

## 17. Apêndice

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