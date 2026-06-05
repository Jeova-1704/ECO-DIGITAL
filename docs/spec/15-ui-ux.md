# 15 — UI/UX

## Estrutura HTML

Referência: `ARQ:328–606`

### Container Principal

```html
<div id="stage">
  <div id="frame">
    <canvas id="game" width="1600" height="900"></canvas>
    <!-- HUD, efeitos, overlays -->
  </div>
</div>
```

**Layout**:
- `#stage`: `position:fixed; inset:0; display:grid; place-items:center`
- `#frame`: `width:min(96vw, calc(96vh * 16 / 9)); aspect-ratio:16/9`
- Canvas: `position:absolute; inset:0; width:100%; height:100%`

O frame mantém aspect-ratio 16:9 e se adapta ao viewport.

---

## HUD (Overlay Fixo)

Referência: `ARQ:332–361`

Sempre visível durante PLAY. `pointer-events: none` (não bloqueia canvas).

### Linha Superior (`.hud-row`)

```
[Resiliência ████████] [🖨 3 / 5 prints da fase] [Acervo: 12]
```

- Posição: `top:14px; left:18px; right:18px`
- Background: `rgba(10,13,28,.55)` com `backdrop-filter:blur(6px)`
- Border-radius: 999px (pill shape)

### Timer (`.timer`)

Centralizado, `top:54px`. Visível apenas quando nível tem `timeLimit`.

### Power Ativo (`.power-active`)

Ao lado do timer. Mostra nome + segundos restantes.

### Mini-mapa (`.minimap`)

Centralizado, `bottom:14px`, largura 44%. Contém track com dots dinâmicos.

### Hint (`.hint`)

Centralizado, `bottom:46px`. Transição suave de opacidade. Conteúdo dinâmico baseado no que está perto.

---

## Overlays por Tela

### Title Screen (`#titleScreen`)

Referência: `ARQ:365–393`

Layout: grid 2 colunas (texto + arte)

| Elemento | Conteúdo |
|----------|----------|
| Eyebrow | "Plataforma · Puzzle Educativo · 12–17 anos" |
| Título | "Eco Digital" + "O Labirinto da Reputação" (gradiente) |
| Descrição | Parágrafo sobre Nuvens de Toxicidade |
| CTAs | Jogar, Como Jogar, Sobre o Cyberbullying |
| Footer | "ECA · Lei 13.185/2015 · Lei 14.811/2024" |
| Arte | Decoração com plataformas, nuvens e personagem |

### Map Screen (`#mapScreen`)

Referência: `ARQ:396–426`

- Background transparente (canvas visível)
- Top bar com stats + botões
- Node panel (painel lateral direito)
- Hint de controles

### Intro Screen (`#introScreen`)

Referência: `ARQ:429–440`

Card centralizado com:
- Tag (MUNDO X · FASE Y)
- Nome do nível
- Descrição
- Grid 2×2 de objetivos (`.obj-card`)
- Botões: Voltar ao mapa, Iniciar fase

### Puzzle Screen (`#puzzleScreen`)

Referência: `ARQ:443–461`

Card full-screen (`inset: 5% 7%`) com grid 3 linhas:

| Seção | Conteúdo |
|-------|----------|
| Header (`.puzzle-head`) | Tag + título + subtítulo |
| Body (`.pzBody`) | Conteúdo dinâmico do puzzle |
| Footer (`.puzzle-foot`) | Status + botões Recuar/Verificar |

### Result Screen (`#resultScreen`)

Referência: `ARQ:464–481`

Card centralizado com:
- Tag ("FASE CONCLUÍDA" ou nome do boss)
- Título
- 3 medalhas circulares (64×64): Bronze, Prata, Ouro
- Checklist de objetivos
- Botões: Próxima fase, Rejogar, Mapa-múndi

### Game Over (`#overScreen`)

Referência: `ARQ:484–499`

Background blur escuro. Conteúdo centralizado:
- Dot vermelho + "Esgotamento emocional"
- Título "Você se esgotou."
- Texto sobre pedir ajuda na vida real
- Botões: Recomeçar fase, Voltar ao mapa

### Victory (`#victoryScreen`)

Referência: `ARQ:502–532`

Layout: grid 2 colunas (texto + arte)

- Background com glows verde e laranja
- Texto: "O ódio foi silenciado pela denúncia"
- Grid 2×2 com 4 canais de denúncia
- Botões: Mapa, Acervo, Tela inicial
- Arte: plataformas todas verdes (ambiente limpo)

### Archive (`#archiveScreen`)

Referência: `ARQ:535–551`

- Título "Cada Print conta uma história."
- Grid 4 colunas (`repeat(4, 1fr)`) com scroll
- 20 cards (`.print-card`) com tag, título e descrição
- Cards bloqueados: 35% opacidade, "🔒" prefixo

### How to Play (`#howScreen`)

Referência: `ARQ:554–581`

Modal centralizado (max 820px):
- Título "Atravesse o Espaço Digital"
- Grid 2×3 com cards:
  1. Controles (teclas)
  2. HUD (elementos)
  3. Painel de Reparação (puzzles)
  4. Power-ups (4 tipos)
  5. Inimigos (4 tipos)
  6. Medalhas (bronze/prata/ouro)

### About (`#aboutScreen`)

Referência: `ARQ:584–605`

Modal centralizado:
- Texto sobre cyberbullying com estatísticas
- Grid 2×2: ECA, Lei 13.185, Lei 14.811, Onde denunciar

---

## Botões

### Estilos

| Classe | Visual |
|--------|--------|
| `.btn` | Gradiente laranja, pill, sombra laranja, bold, cursor pointer |
| `.btn.ghost` | Transparente, texto branco, borda `--line` |
| `.btn.small` | Padding menor, font-size 13px |
| `.btn:disabled` | 40% opacidade, cursor not-allowed |

### Hover

```css
.btn:hover { transform: translateY(-1px) }
```

### Botões por Tela

| Tela | Botões |
|------|--------|
| Title | Jogar (primary), Como Jogar (ghost), Sobre (ghost) |
| Map | Acervo (ghost small), Tela inicial (ghost small) |
| Node Panel | Começar (primary), Fechar (ghost) |
| Intro | Voltar ao mapa (ghost), Iniciar fase (primary) |
| Puzzle | Recuar (ghost), Verificar (primary) |
| Result | Próxima fase (primary), Rejogar (ghost), Mapa (ghost) |
| Over | Recomeçar (primary), Voltar ao mapa (ghost) |
| Victory | Mapa (primary), Acervo (ghost), Tela inicial (ghost) |

---

## Responsividade

### Breakpoint: 720px

```css
@media (max-width: 720px) {
  #titleScreen .inner { grid-template-columns: 1fr; }
  .titleArt { display: none; }
  .v-inner { grid-template-columns: 1fr; }
  .archive-grid { grid-template-columns: 1fr 1fr; }
}
```

- Title: layout 1 coluna, esconde arte decorativa
- Victory: layout 1 coluna
- Archive: grid 2 colunas em vez de 4

### Escala Automática

O `#frame` usa `width: min(96vw, calc(96vh * 16/9))` com `aspect-ratio: 16/9`, garantindo que o jogo se encaixe em qualquer tela mantendo proporção.

O canvas é redimensionado via `fitCanvas()` com devicePixelRatio (máximo 2x).

---

## Acessibilidade

### O que existe

- `aria-hidden="true"` na arte decorativa do título
- Textos semânticos (headings, paragraphs)
- Teclado funciona para todas as interações (mapa, jogo, menus)
- `e.preventDefault()` previne scroll indesejado

### Limitações

- **Sem suporte a touch/mobile** (sem eventos touch)
- **Sem suporte a gamepad**
- **Sem ARIA labels** nos overlays dinâmicos
- **Sem navegação por tab** nos botões inline (gerados via JS)
- **Sem indicação de foco** visível para navegação por teclado
- **Contraste**: alguns textos `--ink-soft` sobre fundo escuro têm contraste baixo
- **Sem localStorage**: progresso se perde ao recarregar

---

## Formulários e Inputs

O jogo **não usa formulários**. Toda interação é via:
- Cliques em botões (DOM)
- Cliques em elementos de puzzle (DOM)
- Teclado (Canvas + DOM)

---

## Toast (Notificação)

Referência: `ARQ:185–188`

```
Posição: top:64px, center
Background: rgba(10,13,28,.85)
Border: 1px solid #2a3057
Border-radius: 999px
Font: monospace, 12px
Duração: 1.3s
Transição: opacity + translateY
Classes: .good (borda #2c5b4b, texto verde), .bad (borda #5c2230, texto rosa)
```
