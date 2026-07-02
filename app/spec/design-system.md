# Design System — Liberages

> Diretrizes visuais para o frontend React SPA + PWA.
> Última atualização: Julho 2026

---

## 1. Filosofia

**"Não é Sexlog."** O design do Liberages deve comunicar: sofisticação sem ser brega, safadeza sem ser vulgar, discrição sem ser frio.

Luminosidade, elegância e um toque provocante equilibrado.

O fundo **não é escuro**. Dark mode existe mas é propositalmente decepcionante — só alguns elementos escurecem.

---

## 2. Paleta de Cores

### 2.1 Núcleo

| Cor | Hex | Uso |
|-----|-----|-----|
| **Branco** | `#FFFFFF` | Fundo principal, cards, modais, formulários |
| **Branco-quente** | `#FEFCF5` | Fundo alternativo (leve aquecimento sem perder claridade) |
| **Dourado** | `#C9A84C` | Acentos principais: badges, destaques, hover states, ícones premium |
| **Dourado-claro** | `#E8D48B` | Hover suave, backgrounds de destaque |
| **Fúcsia** | `#D2366D` | CTAs primários, links, indicadores de match, notificações, badges de atividade |
| **Fúcsia-claro** | `#F0A0C0` | Hover de CTA, badges secundários, tags |
| **Fúcsia-escuro** | `#A81A4F` | Active state de CTA, bordas de destaque |

### 2.2 Neutros

| Cor | Hex | Uso |
|-----|-----|-----|
| **Cinza-50** | `#F8F8F8` | Fundo de inputs, hover de cards |
| **Cinza-100** | `#F0F0F0` | Bordas sutis, divisores |
| **Cinza-200** | `#E0E0E0` | Bordas de inputs, separadores |
| **Cinza-400** | `#999999` | Texto secundário, labels |
| **Cinza-600** | `#666666` | Texto de corpo |
| **Cinza-900** | `#1A1A1A` | Títulos, texto principal |
| **Preto** | `#000000` | Apenas para texto em contextos muito específicos (quase nunca) |

### 2.3 Semânticas

| Cor | Hex | Uso |
|-----|-----|-----|
| **Verde** | `#2EAF6E` | Sucesso, verificado online, disponível |
| **Vermelho** | `#D94A4A` | Erro, alerta, perigo, botão de pânico |
| **Amarelo** | `#F5A623` | Aviso, atenção |
| **Azul** | `#4A90D9` | Links seguros, info |

### 2.4 Glassmorphism (Modais e Overlays)

| Propriedade | Valor |
|-------------|-------|
| background | `rgba(255, 255, 255, 0.7)` |
| backdrop-filter | `blur(16px)` |
| border | `1px solid rgba(255, 255, 255, 0.3)` |
| box-shadow | `0 8px 32px rgba(0, 0, 0, 0.08)` |

---

## 3. Tipografia

### 3.1 Famílias

| Contexto | Fonte | Fallback |
|----------|-------|----------|
| **Títulos** (h1-h3) | Serif (Playfair Display, Cormorant Garamond ou similar) | `'Playfair Display', 'Cormorant Garamond', Georgia, serif` |
| **Corpo** (parágrafos, cards, labels) | Semi-serif (IBM Plex Serif, Lora, ou PT Serif) | `'IBM Plex Serif', 'Lora', Georgia, serif` |
| **UI / Botões / Navegação** | Sans-serif limpa (Inter, Plus Jakarta Sans) | `'Inter', 'Plus Jakarta Sans', system-ui, sans-serif` |
| **Monospace** (códigos, dados) | JetBrains Mono | `'JetBrains Mono', 'Fira Code', monospace` |

### 3.2 Escala

| Nível | Tamanho | Peso | Font Family |
|-------|---------|------|-------------|
| **h1** | 2.5rem (40px) | Bold (700) | Serif |
| **h2** | 2rem (32px) | SemiBold (600) | Serif |
| **h3** | 1.5rem (24px) | SemiBold (600) | Serif |
| **h4** | 1.25rem (20px) | Medium (500) | Semi-serif |
| **Corpo** | 1rem (16px) | Regular (400) | Semi-serif |
| **Pequeno** | 0.875rem (14px) | Regular (400) | Semi-serif |
| **Micro** | 0.75rem (12px) | Regular (400) | Sans-serif (UI) |
| **Label** | 0.875rem (14px) | Medium (500) | Sans-serif (UI) |
| **Botão** | 1rem (16px) | SemiBold (600) | Sans-serif (UI) |

### 3.3 Estilo de Títulos

Todos os títulos em Serif devem ter **letter-spacing: -0.02em** e **line-height: 1.1** para um visual sofisticado e enxuto.

---

## 4. Dark Mode (A Decepção)

Dark mode **não é o padrão**. O padrão é o tema claro (branco predominante).

O dark mode é uma decepção proposital:

| Elemento | Tema Claro (padrão) | Dark Mode |
|----------|-------------------|-----------|
| Fundo principal | `#FFFFFF` | `#F5F0EB` (quase branco, levemente quente) |
| Cards | `#FFFFFF` com border `#E0E0E0` | `#FFFFFF` (ainda branco! cards não escurecem) |
| Modais | Glassmorphism com `rgba(255,255,255,0.7)` | Glassmorphism com `rgba(255,255,255,0.7)` (mesmo) |
| Barras de navegação | `#FFFFFF` | `#FFFFFF` (não escurece) |
| Inputs | `#F8F8F8` | `#F8F8F8` (não escurece) |
| Texto principal | `#1A1A1A` | `#1A1A1A` (não muda) |
| Texto secundário | `#666666` | `#666666` (não muda) |
| **Única coisa que escurece:** | — | **Mapa** entra em modo noturno (dark tiles) |
| **Única coisa que escurece:** | — | **Fotos** com blur ganham overlay escuro sutil |
| **Única coisa que escurece:** | — | **Header/Footer do chat** ganham `#2A2528` muito sutil |

> **Regra de ouro do dark mode:** se o usuário ativar dark mode e reclamar "isso não é dark", o design está correto. O Liberages é um app de descoberta e conexão — deve parecer um café parisiense iluminado, não uma balada subterrânea.

---

## 5. Componentes

### 5.1 Botões

| Tipo | Background | Texto | Borda | Hover |
|------|-----------|-------|-------|------|
| **Primário** | Fúcsia `#D2366D` | Branco | Nenhuma | Fúcsia escuro `#A81A4F` |
| **Secundário** | Transparente | Fúcsia `#D2366D` | Fúcsia `#D2366D` 1px | Fúcsia-claro `#F0A0C0` bg |
| **Terciário** | Transparente | Cinza-900 `#1A1A1A` | Nenhuma | Dourado-claro bg |
| **Premium** | Dourado `#C9A84C` | Branco | Nenhuma | Dourado mais escuro |
| **Perigo** | Vermelho `#D94A4A` | Branco | Nenhuma | Vermelho escuro |

### 5.2 Cards

- Background: `#FFFFFF`
- Border: `1px solid #F0F0F0`
- Border-radius: `12px`
- Shadow: `0 2px 8px rgba(0, 0, 0, 0.04)`
- Hover: shadow sobe para `0 4px 16px rgba(0, 0, 0, 0.08)`

### 5.3 Modais

- Glassmorphism (ver 2.4)
- Border-radius: `16px`
- Overlay de fundo: `rgba(0, 0, 0, 0.2)`

### 5.4 Inputs

- Background: `#F8F8F8`
- Border: `1px solid #E0E0E0`
- Border-radius: `8px`
- Focus: border fúcsia `#D2366D` + box-shadow fúcsia sutil
- Label: sans-serif, 0.875rem, Medium, acima do input

### 5.5 Badges e Selos

| Tipo | Background | Texto | Borda |
|------|-----------|-------|-------|
| Badge comum | Fúcsia-claro `#F0A0C0` | Fúcsia-escuro `#A81A4F` | Nenhuma |
| Selo Premium | Dourado `#C9A84C` | Branco | Nenhuma |
| Selo Verificado | Azul `#4A90D9` | Branco | Nenhuma |
| Selo Verificado (vídeo) | Dourado `#C9A84C` | Branco | Nenhuma |
| Badge "a fim" | Fúcsia `#D2366D` | Branco | Nenhuma |
| Tag local | Cinza-100 `#F0F0F0` | Cinza-600 `#666666` | Nenhuma |

---

## 6. Ícones

- Estilo **outline** (não filled) — mais clean, menos pesado visual
- Linhas de 1.5px a 2px
- Tamanho padrão: 20x20px (ui), 24x24px (navegação), 32x32px (destaques)
- Cor: herda do texto ao lado ou fúcsia para elementos interativos
- **Premium/VIP** podem ter detalhes em dourado

---

## 7. Espaçamento

| Token | Valor | Uso |
|-------|-------|-----|
| `space-1` | 4px | Micro espaços |
| `space-2` | 8px | Padding interno de badges, tags |
| `space-3` | 12px | Padding de inputs, gaps pequenos |
| `space-4` | 16px | Padding de cards, gaps médios |
| `space-5` | 24px | Padding de modais, seções |
| `space-6` | 32px | Margem entre seções |
| `space-7` | 48px | Margem grande |
| `space-8` | 64px | Quebra de página |

---

## 8. Animações

- **Duração:** 200ms (micro-interações), 300ms (transições de página), 500ms (modais)
- **Easing:** `cubic-bezier(0.16, 1, 0.3, 1)` — saída lenta, elegante
- **Hover em cards:** shadow suave + translateY(-2px)
- **Modal:** fade in + scale(0.95 → 1)
- **Notificação:** slide in da direita
- **Like/Super like:** animação de coração/emoji explodindo (sutil)
- **Roleta:** animação de giro com easing natural (física)

---

## 9. Anexos

- `frontend/src/styles/` — Tailwind config com essas cores e tokens
- `frontend/src/components/ui/` — componentes base (Button, Card, Modal, Input, Badge)
- O tema claro é o padrão. Dark mode é implementado como `class="theme-light"` por padrão, com toggle para `theme-dim` (não `theme-dark` — de propósito)

---

> **Princípio absoluto:** O Liberages deve parecer um bistrô iluminado, não uma balada escura. Branco, dourado e fúcsia. Sofisticado, safado, claro.
