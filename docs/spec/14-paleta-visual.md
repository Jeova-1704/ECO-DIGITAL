# 14 — Paleta Visual

## CSS Variables (Design Tokens)

Referência: `ARQ:13–31`

```css
:root {
  --bg:        #0b0e1a;    /* Fundo principal */
  --bg-2:      #11152a;    /* Fundo secundário */
  --ink:       #ecedf6;    /* Texto principal */
  --ink-soft:  #a7adc8;    /* Texto secundário */
  --line:      #232844;    /* Bordas, separadores */
  --warm:      #f4ead6;    /* Cor quente (plataformas) */
  --solid:     #e9e1c6;    /* Variação quente */
  --good:      #67e8c5;    /* Cor positiva (solidário, cura, acerto) */
  --good-2:    #9af2d3;    /* Cor positiva clara */
  --toxic:     #c93b8a;    /* Cor tóxica (nuvens, inimigos, erros) */
  --toxic-2:   #6c1e6a;    /* Cor tóxica escura */
  --accent:    #f0a05a;    /* Cor de destaque (CTA, power-ups, prints) */
  --accent-2:  #fbd49a;    /* Cor de destaque clara */
  --danger:    #ff5d6c;    /* Cor de perigo (dano, game over) */
  --gold:      #f6c64a;    /* Medalha de ouro */
  --silver:    #cfd5e6;    /* Medalha de prata */
  --bronze:    #c98f5a;    /* Medalha de bronze */
}
```

## Classificação Semântica

### Positivo / Bom

| Token | Hex | Uso |
|-------|-----|-----|
| `--good` | #67e8c5 | Plataformas solidárias, escudo, player (corpo), progresso |
| `--good-2` | #9af2d3 | Variação clara, glow positivo, avatar do mapa |

### Tóxico / Ruim

| Token | Hex | Uso |
|-------|-----|-----|
| `--toxic` | #c93b8a | Nuvens de toxicidade, inimigos (bot, fake hostil), erros |
| `--toxic-2` | #6c1e6a | Variação escura, hater |
| `--danger` | #ff5d6c | Dano, game over, boss border, flash vermelho |

### Destaque / Ação

| Token | Hex | Uso |
|-------|-----|-----|
| `--accent` | #f0a05a | Botões CTA, prints, power-ups (ally), boss tint |
| `--accent-2` | #fbd49a | Variação clara, gradientes |

### Neutro / Base

| Token | Hex | Uso |
|-------|-----|-----|
| `--bg` | #0b0e1a | Fundo principal |
| `--bg-2` | #11152a | Fundo secundário |
| `--ink` | #ecedf6 | Texto principal |
| `--ink-soft` | #a7adc8 | Texto secundário, labels |
| `--line` | #232844 | Bordas, separadores |

### Plataformas

| Token | Hex | Uso |
|-------|-----|-----|
| `--warm` | #f4ead6 | Plataformas (chão e posts), cabeça do player |
| `--solid` | #e9e1c6 | Variação de plataforma |

### Medalhas

| Token | Hex | Uso |
|-------|-----|-----|
| `--gold` | #f6c64a | Medalha de ouro, print dourado |
| `--silver` | #cfd5e6 | Medalha de prata |
| `--bronze` | #c98f5a | Medalha de bronze, plataformas instáveis |

---

## Cores no Canvas

Cores usadas diretamente no JS (sem CSS variables):

### Background
| Cor | Hex | Uso |
|-----|-----|-----|
| Céu topo | #0d122a | Gradiente sky |
| Céu base | #06081a | Gradiente sky |
| Glow blobs | rgba(40,60,120,.35) | Parallax |

### Player
| Cor | Hex | Uso |
|-----|-----|-----|
| Corpo normal | #9af2d3 | Corpo principal |
| Corpo dano | #ff5d6c | Flash de dano |
| Cabeça | #f4ead6 | Pele |
| Roupa | #283063 | Boné, pernas |
| Olhos | #1a1224 | Olhos, boca |

### Inimigos
| Cor | Hex | Uso |
|-----|-----|-----|
| Troll | #ff5d6c | Corpo |
| Troll escuro | #3a1a26 | Sombra |
| Bot | #c93b8a | Corpo |
| Fake amigável | #67e8c5 | Corpo |
| Fake hostil | #c93b8a / #ff5d6c | Flash |
| Hater | #6c1e6a | Corpo escuro |
| Hater claro | #c93b8a | Overlay |

### Nuvem
| Cor | Hex | Uso |
|-----|-----|-----|
| Halo tóxico | rgba(201,59,138,.35) | Gradiente radial |
| Corpo topo | #c93b8a | Gradiente |
| Corpo base | #3a0e3a | Gradiente |
| Solidária halo | rgba(103,232,197,.35) | Gradiente |
| Solidária topo | #9af2d3 | Gradiente |
| Solidária base | #67e8c5 | Gradiente |

---

## Tipografia

### Famílias

| Uso | Fonte | CSS |
|-----|-------|-----|
| Corpo | Sistema | `ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Helvetica, Arial, sans-serif` |
| Mono | JetBrains | `"JetBrains Mono", ui-monospace, "SF Mono", Menlo, Consolas, monospace` |

### Classes

| Classe | Uso |
|--------|-----|
| `.mono` | Fonte monospace genérica |
| `.eyebrow` | Labels uppercase, 11px, letter-spacing .32em |
| `.label` | Labels HUD, 10px, letter-spacing .18em |

### Tamanhos no Canvas

| Uso | Tamanho | Fonte |
|-----|---------|-------|
| Labels de nuvem | 11px | bold ui-monospace |
| Labels de plataforma | 11px | ui-monospace |
| Ícones de mapa | 14px | bold ui-monospace |
| Nome do boss | 12px | bold ui-monospace |
| Labels de inimigo | 9-11px | ui-monospace |
| Boss HUD label | 10px | ui-monospace |
| Zona de silêncio | 12px | bold ui-monospace |
| Espelho | 12px | bold ui-monospace |

---

## Efeitos Visuais

### Gradientes Comuns

| Tipo | Cores | Uso |
|------|-------|-----|
| Linear (botão) | #f7c98a → #f0a05a | Botões `.btn` |
| Linear (plataforma) | #f4ead6 → #d6caa6 | Chão |
| Linear (plataforma) | #f4ead6 → #cdc09c | Posts |
| Linear (nuvem) | #c93b8a → #3a0e3a | Toxicidade |
| Linear (solidária) | #9af2d3 → #67e8c5 | Resolvida |
| Linear (unstable) | #f4c89a → #c98f5a | Instável |

### Shadows

| Elemento | Sombra |
|----------|--------|
| Frame | `0 30px 80px -20px rgba(0,0,0,.6), 0 0 0 1px #1a1f3a inset` |
| Botão | `0 10px 30px -10px rgba(240,160,90,.5), inset 0 -2px 0 rgba(0,0,0,.08)` |
| Plataformas | offset (3,5), rgba(0,0,0,.35) |
| Nuvens | gradiente radial (halo) |

### Animações CSS

| Nome | Duração | Uso |
|------|---------|------|
| `shake` | 0.35s | Erro em puzzle / resposta errada |
| Opacidade | 0.25s | Hint, toast |
| Transform | 0.15s | Botão hover |

### Responsive Breakpoint

```css
@media (max-width: 720px) {
  /* Título: grid 1 coluna, esconde titleArt */
  /* Vitória: grid 1 coluna */
  /* Archive: grid 2 colunas */
}
```
