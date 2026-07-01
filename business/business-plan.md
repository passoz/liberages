# Plano de Negócios — Liberages

> **Confidencial.** Documento estratégico para investimento, operação e direcionamento do produto.
> Última atualização: Julho 2026

---

## Sumário Executivo

### O que é

**Liberages** é uma plataforma digital (PWA + desktop app) construída em Go single-binary com React SPA, voltada para o público liberal brasileiro (swing, troca de casais, ménage, BDSM, exibicionismo, voyeurismo). O produto central é o **Lustmapia** — uma rede social liberal com mapa interativo como hub da experiência, combinando descoberta geográfica, comunidade, eventos, conteúdo e privacidade em camadas.

### Oportunidade

- Mercado dominado por **1 player** (Sexlog) com ~17 milhões de usuários cadastrados, mas sem inovar desde 2007.
- Sexlog cobra R$49,90/mês por um produto com UX datada, sem mobile nativo, sem mapa e moderação frágil.
- Banimento de apps para adultos nas lojas (App Store/Play Store) abre espaço para **PWA** — alternativa que bypassa gates das lojas.
- Público liberal é altamente engajado, recorrente e disposto a pagar por privacidade e melhores ferramentas.

### Diferencial

| Pilar | O que resolve | Como |
|-------|---------------|------|
| **Privacidade em camadas** | Medo de exposição (fotos identificáveis, perfis falsos) | Blur facial automático, perfil anônimo, modo invisível, selfie destrutível, verificação opcional |
| **Descoberta geográfica** | Dificuldade de encontrar pessoas/eventos próximos | Mapa interativo Leaflet com marcadores por tipo, heatmap de atividade, "rolês rolando agora" |
| **Comunidade moderada** | Perfis falsos, golpistas, ambiente tóxico | Verificação de identidade opcional, reputação, moderação híbrida (IA + humana), selo verificado |

### Modelo de receita

Freemium em 3 camadas: **Free** (grátis), **Premium** (R$29,90/mês), **VIP** (R$49,90/mês) + comissão de 15% sobre conteúdo pago + parcerias B2B (casas de swing, motéis, sex shops).

### Equipe

Inicial: 1 fundador técnico (full-stack Go + React), com terceirização de design e moderação conforme escala.

### Investimento solicitado

Seed de **R$150.000** para 12 meses de runway: hospedagem, domínios, marketing inicial, serviços de moderação/IA, reserva legal.

### Projeção (ano 1)

| Métrica | Mês 6 | Mês 12 | Mês 18 |
|---------|-------|--------|--------|
| Usuários cadastrados | 5.000 | 25.000 | 100.000 |
| Usuários pagantes | 150 | 1.000 | 5.000 |
| MRR | R$4.500 | R$30.000 | R$150.000 |
| CAC | R$8,00 | R$6,00 | R$4,50 |
| LTV | R$180 | R$359 | R$599 |
| LTV/CAC | 22x | 60x | 133x |

---

## 1. Descrição do Negócio

### 1.1 Visão

Tornar-se a plataforma de referência para o público liberal brasileiro — o lugar onde privacidade real, descoberta geográfica e comunidade moderada convergem.

### 1.2 Missão

Conectar pessoas do público liberal com segurança, discrição e privacidade real, oferecendo uma experiência moderna e superior às plataformas existentes.

### 1.3 Valores

- **Privacidade como direito, não feature** — o usuário controla o quanto mostra
- **Segurança primeiro** — ferramentas reais de proteção (botão de pânico, contato de emergência, encontro seguro)
- **Transparência** — regras claras, moderação explicável, sem shadowbanning arbitrário
- **Comunidade** — plataforma centrada em pessoas, não em conteúdo
- **Inovação constante** — mapa, matching por intenção, lives com máscara, PWA/desktop app

### 1.4 Produto

Liberages é entregue como **PWA** (installável no celular sem App Store/Play Store) e **desktop app** (system tray com atalho de ocultação discreta). Arquitetura técnica:

- **Single binary Go** (API + BFF + estáticos React) servindo tudo na porta `:3000`
- **React SPA + PWA** — Vite, React Router, Tailwind CSS v4 + Glassmorphism
- **SQLite** (escalável até ~100k usuários ativos, depois migrar para PostgreSQL)
- **Leaflet** com tiles próprios (sem CDN, sem dependência externa)

### 1.5 Módulos do Produto

#### Módulo 1 — Mapa Interativo (MVP)

- Mapa Leaflet com marcadores SVG customizados por tipo de local
- Heatmap de atividade (densidade de usuários/eventos online)
- Modo noturno padrão, modo anônimo (cluster em vez de ponto individual)
- Filtros: tipo de interação, distância, tempo (agora/hoje/semana)
- CRUD de locais (admin) com upload de imagens
- Busca textual com SQLite FTS5

#### Módulo 2 — Perfis e Privacidade

- Perfis individuais e de casal
- **Níveis de privacidade**:
  - Nível 0: Visitante — vê só o mapa público
  - Nível 1: Age gate verificado — vê locais, perfis com fotos borradas
  - Nível 2: Perfil completo — interação normal
  - Nível 3: Verificado (documento + selfie) — selo azul, grupos exclusivos
- Blur facial automático (IA), modo invisível, selfie destrutível (some após 10s)
- Galeria com controle de quem vê cada foto

#### Módulo 3 — Matching por Intenção

- Não é Tinder swipe — é "o que você busca hoje?"
- Filtros: troca de casal, ménage, voyeur, exibição, bate-papo
- Filtros de gênero, orientação, janela temporal (agora/hoje/semana)
- Algoritmo ordena por compatibilidade + distância
- Limite diário para free users (5 matches), ilimitado para Premium

#### Módulo 4 — Eventos com Geolocalização

- Criação de eventos: festas, encontros em motel, surubas, jantar liberal
- Eventos públicos e privados (com convite)
- **Check-in com PIN** (evita penetras)
- "Eventos próximos a você agora" no mapa
- Integração B2B com casas de swing reais
- Histórico privado de eventos frequentados

#### Módulo 5 — Live Streaming com Privacidade

- Lives com máscara facial (blur/filtro em tempo real)
- Controle de audiência: só verificados, só amigos, só seguidores
- Modo pay-per-view (tips, ingressos virtuais)
- Lives em grupo (até 4 pessoas)

#### Módulo 6 — Grupos com Curadoria

- Grupos públicos e privados
- Categorias por fetiche, cidade, tipo de interação
- Ranking por atividade
- Regras explícitas de conduta em cada grupo
- Moderação com banimento explicado

#### Módulo 7 — Segurança

- **Botão de pânico** — notificação silenciosa para moderação + emergência
- **Contato de emergência** — notificado se não houver check-out seguro
- **Local público sugerido** — primeiro encontro em local público
- **Block + Report** com justificativa
- **Rate limiting** — anti spam, anti assédio

#### Módulo 8 — Admin Dashboard

- Painel admin completo com filtros, sorting, preview de mapa
- Moderação de conteúdo, perfis, eventos
- Gestão de denúncias
- Métricas em tempo real

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
3. **Sem descoberta geográfica** — não há mapa, busca por localização é primitiva
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

O Sexlog tem 17M de usuários mas **não inova há anos**. É um produto de 2007 com atualizações incrementais. Uma plataforma moderna, com foco em privacidade real, UX superior e descoberta geográfica pode capturar:

- Usuários insatisfeitos com o Sexlog (UX, preço, falta de features)
- Novos entrantes que nunca usaram Sexlog por estigma ou UX pobre
- Público que não usa plataformas para adultos por questões de privacidade

---

## 3. Posicionamento Estratégico

### 3.1 Proposição de Valor

> **"O Waze do Prazer vira uma comunidade"**
> O mapa não é um extra — é o centro da experiência. Cada local, evento, encontro e perfil tem uma âncora geográfica.

### 3.2 Estratégia de Posicionamento

Não competir head-to-head com Sexlog em SEO e massa crítica. Competir em **inovação, privacidade e UX** — capturar usuários que buscam algo melhor, não usuários que já estão satisfeitos.

### 3.3 Vantagem Competitiva Sustentável

1. **PWA + Desktop app** — bypassa App Store/Play Store (banimento de apps adultos é barreira de entrada)
2. **Mapa como hub** — nenhum concorrente tem, difícil de replicar bem
3. **Privacidade em camadas** — blur facial IA, selfie destrutível, modo invisível
4. **Single binary Go** — custo de infraestrutura extremamente baixo (1 processo, SQLite)
5. **Desktop app discreto** — system tray com ocultação (ninguém vê no celular)
6. **Comunidade moderada** — ambiente seguro atrai e retém mulheres (segmento mais carente)

---

## 4. Modelo de Receita

### 4.1 Estrutura de Preços

| Tier | Preço | Funcionalidades |
|------|-------|-----------------|
| **Free** | Grátis | Perfil, mapa básico, 5 matches/dia, chat limitado, fotos com blur automático |
| **Premium** | R$29,90/mês | Match ilimitado, modo invisível, lives privadas, verificação azul, relatório de privacidade, sem anúncios |
| **VIP** | R$49,90/mês | Tudo do Premium + prioridade no matching, conteúdo exclusivo de creators, convites VIP, acesso antecipado |

### 4.2 Receitas Adicionais

| Fonte | Descrição | Margem estimada |
|-------|-----------|------------------|
| **Comissão conteúdo exclusivo** | 15% sobre transações de conteúdo pago entre usuários (tips, pay-per-view) | 15% |
| **Parceria B2B** | Casas de swing, motéis, sex shops pagam por presença destacada no mapa | 80%+ |
| **Eventos promovidos** | Destaque pago no feed de eventos | 90%+ |
| **Anúncios nativos** | Apenas free users, sem anúncios adultos invasivos | CPM variável |

### 4.3 Estratégia de Preço

- **Mais barato que Sexlog** no Premium (R$29,90 vs R$49,90) — aquisição agressiva
- **VIP igual ao Sexlog** (R$49,90) mas com mais valor
- **Mulheres solteiras** ganham Premium grátis (estratégia de equilíbrio de gênero)
- **Desconto anual** (30% off) para aumentar LTV e reduzir churn

### 4.4 Projeção de Mix

| Métrica | Ano 1 | Ano 2 | Ano 3 |
|---------|-------|-------|-------|
| % Free | 92% | 85% | 80% |
| % Premium | 6% | 11% | 15% |
| % VIP | 2% | 4% | 5% |
| ARPU (receita por usuário pago) | R$34/mês | R$38/mês | R$42/mês |

---

## 5. Estratégia de Marketing e Vendas

### 5.1 Canais de Aquisição

| Canal | Custo | Volume estimado | Qualidade |
|-------|-------|-----------------|-----------|
| **SEO orgânico** (blog "IBGE do Sexo") | Baixo | Médio | Alta |
| **Boca a boca / indicação** | Zero | Médio-alto | Altíssima |
| **Grupos de Telegram/WhatsApp** | Baixo | Alto | Média |
| **Content marketing** (blog, YouTube) | Médio | Médio | Alta |
| **Parcerias com casas de swing** | Zero | Baixo-médio | Altíssima |
| **Influencers liberais** | Médio | Médio | Alta |
| **SEM (Google Ads)** | Alto | Alto | Média-baixa |
| **Cripto comunidade** (USDT) | Baixo | Baixo | Média |

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
| **Beta fechado** | 2 meses | 500 usuários ativos | Convites em grupos Telegram/WhatsApp, parcerias com casas de swing |
| **Soft launch** | 1 mês | 5.000 usuários | Abertura pública, press release em blogs liberais, influencers |
| **Growth** | 3 meses | 25.000 usuários | SEO, content marketing, programa de indicação, SEM |
| **Scale** | 6 meses | 100.000+ usuários | Escalar canais comprovados, parcerias B2B, cripto |

---

## 6. Análise Financeira

### 6.1 Custos Operacionais (Mensais)

| Item | Mês 1-6 | Mês 7-12 | Mês 13-18 |
|------|---------|----------|-----------|
| Hospedagem (VPS + CDN) | R$200 | R$500 | R$1.500 |
| Domínios | R$20 | R$20 | R$20 |
| E-mail (transacional) | R$100 | R$200 | R$400 |
| Storage (imagens) | R$0 | R$200 | R$800 |
| Moderação IA | R$500 | R$1.000 | R$2.000 |
| Moderação humana | R$0 | R$1.500 | R$3.000 |
| Marketing/SEM | R$2.000 | R$5.000 | R$10.000 |
| Serviços legais/jurídico | R$500 | R$500 | R$1.000 |
| **Total mensal** | **R$3.320** | **R$8.920** | **R$18.720** |

### 6.2 Projeção de Receita (18 meses)

| Mês | Usuários | Pagantes | MRR | Custos | Resultado |
|-----|----------|----------|-----|--------|-----------|
| 1 | 200 | 5 | R$150 | R$3.320 | -R$3.170 |
| 3 | 1.500 | 40 | R$1.360 | R$3.500 | -R$2.140 |
| 6 | 5.000 | 150 | R$4.500 | R$5.000 | -R$500 |
| 9 | 12.000 | 500 | R$15.000 | R$7.000 | +R$8.000 |
| 12 | 25.000 | 1.000 | R$30.000 | R$9.000 | +R$21.000 |
| 15 | 50.000 | 2.500 | R$75.000 | R$13.000 | +R$62.000 |
| 18 | 100.000 | 5.000 | R$150.000 | R$19.000 | +R$131.000 |

### 6.3 Break-Even

**Break-even projetado: mês 7-8** (5.000-7.000 usuários, ~200 pagantes)

### 6.4 Métricas Unitárias

| Métrica | Ano 1 | Ano 2 | Ano 3 |
|---------|-------|-------|-------|
| CAC | R$8,00 | R$6,00 | R$4,50 |
| LTV (média) | R$180 | R$359 | R$599 |
| LTV/CAC | 22x | 60x | 133x |
| Churn mensal | 8% | 6% | 5% |
| Payback period | 2 meses | 1.5 meses | 1 mês |

### 6.5 Investimento Solicitado

**Seed: R$150.000**

| Alocação | Valor | % |
|----------|-------|---|
| Desenvolvimento (12 meses) | R$60.000 | 40% |
| Marketing e aquisição | R$45.000 | 30% |
| Infraestrutura e serviços | R$25.000 | 17% |
| Reserva legal/jurídico | R$10.000 | 7% |
| Reserva de emergência | R$10.000 | 7% |

### 6.6 Projeção Trienal

| Ano | Usuários | Receita total | Custos | Lucro líquido |
|-----|----------|---------------|--------|---------------|
| Ano 1 | 25.000 | R$180.000 | R$75.000 | R$105.000 |
| Ano 2 | 100.000 | R$720.000 | R$180.000 | R$540.000 |
| Ano 3 | 250.000 | R$1.800.000 | R$360.000 | R$1.440.000 |

---

## 7. Análise de Riscos

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| **Banimento de meios de pagamento** | Média | Alto | Stripe alternativo + cripto (USDT) como fallback. Arquitetura já preparada |
| **Block de anúncios e redes sociais** | Alta | Médio | Marketing de conteúdo (blog), SEO, parcerias, comunidade orgânica |
| **Processos legais (conteúdo)** | Baixa | Alto | Age gate obrigatório, moderação IA + humana, DMCA compliance, advogado especializado |
| **Vazamento de dados** | Baixa | Altíssimo | Criptografia E2E em mensagens, blur facial, dados sensíveis criptografados em nível aplicação |
| **Perfis falsos/golpistas** | Alta | Médio | Verificação documental opcional, reputação por denúncia, IA detecção fake |
| **Concorrência Sexlog** | Média | Médio | Foco em inovação (mapa, privacidade, UX), não competir por SEO head-to-head |
| **Escalabilidade SQLite** | Baixa | Médio | Migrar para PostgreSQL quando passar de ~100k usuários ativos. Interfaces Go facilitam troca |
| **Regulação conteúdo adulto** | Baixa | Alto | Compliance com LGPD, age gate, termos de uso claros, advogado especializado |
| **Churn alto por estigma** | Média | Médio | Desktop app discreto, PWA (não aparece na lista de apps), modo invisível |
| **Dependência de fundador único** | Média | Alto | Documentação completa, automação de deploy, planejamento de contratação |

---

## 8. Plano Operacional

### 8.1 Tecnologia

| Componente | Tecnologia | Custo |
|------------|-----------|-------|
| Backend | Go 1.26+ single binary | R$0 (open source) |
| Frontend | React 19+ SPA + PWA | R$0 (open source) |
| Banco | SQLite → PostgreSQL | R$0 → R$200/mês |
| Mapa | Leaflet + tiles próprios | R$0 |
| Mapa tiles | Self-hosted ou MapTiler free tier | R$0 |
| Storage de imagens | Filesystem local → S3 | R$0 → R$800/mês |
| Moderação IA | API externa (AWS Rekognition ou similar) | R$500-R$2.000/mês |
| E-mail | Amazon SES ou similar | R$100-R$400/mês |
| CDN | Cloudflare free → pro | R$0-R$200/mês |
| Monitoramento | Prometheus + Grafana self-hosted | R$0 |

### 8.2 Operação de Moderação

- **Fase 1 (0-5k usuários):** Moderação manual pelo fundador + IA básica
- **Fase 2 (5k-25k):** IA para triagem + 1 moderador part-time
- **Fase 3 (25k+):** IA + equipe de moderação 24/7 (3 turnos)
- **Política:** Moderação híbrida — IA detecta, humano decide em casos limítrofes
- **Transparência:** todo banimento tem explicação e processo de apelo

### 8.3 Legal e Compliance

| Item | Status | Responsável |
|------|--------|-------------|
| Termos de uso | A redigir | Advogado especializado em direito digital |
| Política de privacidade (LGPD) | A redigir | Advogado |
| Age gate (18+ obrigatório) | Especificado | Fundador |
| DMCA compliance | A implementar | Fundador |
| CNPJ e regime tributário | A definir | Contador |

---

## 9. Roadmap

### Fase 1 — "O Mapa" (MVP) — Meses 1-3

- [x] Repositório e especificação criados
- [ ] Autenticação local (JWT + bcrypt)
- [ ] Age gate com cookie assinado HMAC
- [ ] Mapa Leaflet com tipos de local
- [ ] CRUD de locais (admin)
- [ ] Busca textual com SQLite FTS5
- [ ] PWA com install prompt
- [ ] Deploy em produção
- [ ] Beta fechado (500 usuários)

### Fase 2 — "A Rede" (Comunidade) — Meses 4-6

- [ ] Perfis de usuário (indivíduos e casais)
- [ ] Chat privado
- [ ] Matching por interesse + localização
- [ ] Grupos públicos e privados
- [ ] Upload de fotos com blur automático
- [ ] Moderação IA básica
- [ ] Soft launch (5.000 usuários)

### Fase 3 — "O Evento" (Encontros) — Meses 7-10

- [ ] Criação e descoberta de eventos com geolocalização
- [ ] Check-in com PIN
- [ ] Integração com casas de swing (parceria)
- [ ] Modo viagem
- [ ] Botão de pânico
- [ ] Contato de emergência
- [ ] Growth (25.000 usuários)

### Fase 4 — "O Show" (Conteúdo) — Meses 11-14

- [ ] Live streaming com privacidade
- [ ] Conteúdo exclusivo (assinatura)
- [ ] Tips e pay-per-view
- [ ] Verificação azul (documental)
- [ ] Parcerias B2B (casas de swing, motéis)

### Fase 5 — "A Fortaleza" (Segurança Premium) — Meses 15-18

- [ ] Relatório de privacidade
- [ ] Detecção de screenshots
- [ ] Verificação documental completa
- [ ] Pagamentos em cripto (fallback)
- [ ] Desktop app com system tray
- [ ] Scale (100.000+ usuários)

---

## 10. Equipe e Estrutura

### 10.1 Equipe Atual

| Papel | Pessoa | Dedicação |
|-------|--------|-----------|
| Fundador / Tech Lead / Full-stack | Fernando Passos | Full-time |

### 10.2 Contratações Planejadas

| Fase | Papel | Quando | Tipo |
|------|-------|--------|------|
| Fase 2 | Designer UI/UX | Mês 4 | Freelancer |
| Fase 2 | Moderador | Mês 5 | Part-time |
| Fase 3 | Dev frontend | Mês 8 | Full-time |
| Fase 4 | Dev backend | Mês 12 | Full-time |
| Fase 4 | Gerente de comunidade | Mês 12 | Full-time |
| Fase 5 | Especialista em segurança | Mês 15 | Consultor |

### 10.3 Cultura Organizacional

- **Remoto-first** — time distribuído
- **Open source friendly** — libs open source quando possível, contribuir de volta
- **Privacidade como valor interno** — o time usa o produto
- **Documentação como vantagem** — AGENTS.md e specs sempre atualizados

---

## 11. Estratégia de Saída

### 11.1 Cenários

| Cenário | Horizonte | Descrição | Valuation estimada |
|---------|-----------|-----------|---------------------|
| **Aquisição** | 3-5 anos | Sexlog ou concorrente adquire para modernizar base | 5-10x ARR |
| **Investimento série A** | 2-3 anos | VC entra para escalar | R$5-15M |
| **Bootstrap rentável** | Contínuo | Crescimento orgânico sem investimento externo | N/A |
| **White-label** | 2 anos | Licenciar tecnologia para outros mercados (clubes, eventos) | Receita adicional SaaS |

### 11.2 Métricas para Saída

- 100.000+ usuários ativos
- R$150.000+ MRR
- LTV/CAC > 20x
- Churn < 5% mensal
- NPS > 50

---

## 12. Considerações Legais e Regulatórias

### 12.1 LGPD (Lei Geral de Proteção de Dados)

- Dados sensíveis (orientação sexual, preferências) exigem **consentimento explícito**
- Política de privacidade clara e acessível
- Direito de exclusão de dados (deletar conta e todos os dados)
- Dados criptografados em nível de aplicação
- DPO (Data Protection Officer) quando passar de determinado volume

### 12.2 Age Gate Obrigatório

- Verificação 18+ com cookie assinado HMAC
- Sem cache de data de nascimento — só confirmação de maioridade
- Banner de idade em todas as páginas públicas

### 12.3 Conteúdo Adulto

- Plataforma hospeda conteúdo adulto gerado por usuários (UGC)
- Compliance DMCA para remoção de conteúdo copyright
- Política de conteúdo proibido (menores, não consensual, zoofilia, etc.)
- Moderação ativa e reativa
- Advogado especializado em direito digital na retainer

### 12.4 Pagamentos

- **Gateways tradicionais** (Stripe, Mercado Pago) podem banir — ter alternativa
- **Cripto (USDT)** como fallback — preparado na arquitetura
- Compliance KYC/AML se necessário para pagamentos em cripto

---

## 13. Métricas de Sucesso (KPIs)

### 13.1 North Star Metric

**Usuários ativos semanais (WAU)** que usam o mapa pelo menos 1x por semana.

### 13.2 KPIs por Categoria

| Categoria | KPI | Meta ano 1 |
|-----------|-----|------------|
| **Crescimento** | Novos cadastros/dia | 100+ |
| **Engajamento** | DAU/MAU | 25%+ |
| **Engajamento** | Sessões/usuário/dia | 3+ |
| **Monetização** | % conversão free→pago | 5%+ |
| **Monetização** | MRR | R$30.000 |
| **Retenção** | Churn mensal | <8% |
| **Retenção** | Retenção D30 | 40%+ |
| **Qualidade** | Denúncias/1k usuários | <5 |
| **Qualidade** | Tempo de moderação | <24h |
| **Privacidade** | % usuários com blur ativo | 60%+ |

---

## 14. Análise SWOT

### Forças (Strengths)

- Stack tecnológica moderna e barata (Go single binary, SQLite, React PWA)
- Especificação completa e documentada (AGENTS.md, specs)
- PWA bypassa banimento de App Store/Play Store
- Custo operacional extremamente baixo
- Fundador técnico (não depende de terceiros para desenvolver)
- Diferencial real vs concorrente (mapa, privacidade, UX)

### Fraquezas (Weaknesses)

- Fundador único (riscos de continuidade)
- Sem massa crítica inicial (network effect)
- Sem orçamento de marketing vs Sexlog
- Marca desconhecida
- Content moderation é caro e complexo em escala

### Oportunidades (Opportunities)

- Sexlog não inova há 17 anos
- Público liberal cresceu pós-pandemia
- Banimento de apps nas lojas é barreira para concorrentes
- Cripto abre novo canal de pagamento
- Desktop app discreto é diferencial sem concorrente
- Parcerias B2B com casas de swing são canal de aquisição sem custo

### Ameaças (Threats)

- Sexlog pode copiar features (mapa, blur)
- Regulação de conteúdo adulto pode apertar
- Gateways de pagamento podem banir plataforma
- Vazamento de dados seria fatal para confiança
- Plataformas maiores (Tinder, FetLife) podem entrar no nicho brasileiro

---

## 15. Tecnologia como Vantagem Competitiva

### 15.1 Por que Go + SQLite é uma vantagem

| Aspecto | Stack típica (Sexlog-like) | Liberages |
|---------|---------------------------|-----------|
| Backend | Node.js + Express ou PHP | Go single binary |
| Frontend | SSR (Next.js) ou monolito | React SPA + PWA |
| Banco | PostgreSQL + Redis | SQLite (escalável até 100k UA) |
| Deploy | 5+ containers | 1 binário + 1 DB file |
| Custo infra | R$2.000+/mês | R$200/mês |
| Cold start | Segundos | Milissegundos |
| PWA | Não | Sim (bypass App Store) |
| Desktop app | Não | Sim (system tray discreto) |

### 15.2 Arquitetura preparada para escala

- Interfaces Go na camada de dados → troca SQLite por PostgreSQL sem mudar lógica de negócio
- Interfaces Go na camada de auth → troca JWT local por Keycloak/OAuth2 sem mudar código
- Single binary → fácil deploy em qualquer VPS, sem orquestração complexa
- Frontend React SPA → independente do backend, pode ser servido por CDN

---

## 16. Apêndice

### 16.1 Glossário

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

### 16.2 Referências

- Sexlog dados públicos: relatórios de mídia 2022
- Pesquisas de comportamento sexual brasileiro (dados secundários)
- Documentação técnica: `app/AGENTS.md`, `app/spec/lustmapia.md`
- Dossiê de mercado: `app/spec/dossie-mercado.md`

### 16.3 Suposições do Plano

1. Conversão free→pago de 5% (indústria SaaS: 2-10%)
2. Churn mensal de 8% no ano 1 (indústria adult: 5-15%)
3. ARPU médio de R$34/mês (misto Premium R$29,90 + VIP R$49,90)
4. Custo de aquisição de R$8 (canais predominantemente orgânicos)
5. SQLite escala até ~100k usuários ativos (~1M acessos/dia)
6. Sem investimento adicional necessário após seed (bootstrap a partir do break-even)

---

*Documento confidencial. Não distribuir sem autorização.*
*© 2026 Liberages. Todos os direitos reservados.*