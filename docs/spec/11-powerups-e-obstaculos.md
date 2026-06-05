# 11 — Power-ups e Obstáculos

## Power-ups

Referência: `ARQ:1260–1301`

### Definições

```javascript
const POWERUPS = {
  shield: {label:'Escudo de Empatia',     dur: 0, color:'#9af2d3', desc:'Absorve 1 ataque sem perder Resiliência.'},
  freeze: {label:'Bloqueio Temporário',   dur: 5, color:'#7cc7ff', desc:'Congela inimigos por 5s.'},
  ally:   {label:'Aliado Temporário',      dur:10, color:'#f0a05a', desc:'NPC segue você e atrai inimigos por 10s.'},
  gold:   {label:'Print Dourado',          dur: 0, color:'#f6c64a', desc:'Vale 5 prints normais.'},
};
```

| Tipo | Duração | Efeito |
|------|---------|--------|
| `shield` | Permanente (1 uso) | `L.shielded = true`. Absorve próximo dano > 1. |
| `freeze` | 5s | `e.frozen = 5` em todos os inimigos. Sem movimento/ataque. |
| `ally` | 10s | NPC (aliado) segue o player. Haters são atraídos para ele. |
| `gold` | Instantâneo | `L.printsGot += 5`. Não conta para archive. |

### Coleta

Player sobreposta ao power-up (hitbox expandida em 4px cada lado):
```javascript
if (overlap(p, {x:pu.x-2, y:pu.y-2, w:pu.w+4, h:pu.h+4}))
```

### Renderização

`drawPowerup(g, p, t)` — `ARQ:1268–1295`

Cada power-up tem:
- **Halo**: gradiente radial com cor do tipo (35% → 0% opacidade)
- **Bob**: oscilação vertical `sin(t/300 + p.x) * 4`
- **Rotação**: oscilação `sin(t/400 + p.x) * 0.2`

#### Visual por Tipo

| Tipo | Forma | Detalhe |
|------|-------|---------|
| Shield | Escudo hexagonal verde | Coração "♡" preto central |
| Freeze | Estrela de 6 pontas azul | Ponto preto central |
| Ally | Retângulo arredondado laranja | Olhos + boca (carinha) |
| Gold | Quadrado arredondado dourado | Estrela "★" preta |

### Dimensões

Todos: `w:22, h:22`

---

## Obstáculos

Referência: `ARQ:1813–1866` (update), `ARQ:2405–2435` (render)

### 1. Unstable (Plataforma Instável)

| Propriedade | Valor |
|-------------|-------|
| Tipo | `unstable` |
| Altura | 18px |
| Trigger | Player em cima (`onTop`) |

**Comportamento**:
1. `0–0.4s`: plataforma normal
2. `0.4–1.8s`: shake horizontal (`sin(40*t)*2`), timer visual tracejado
3. `>1.8s`: `broken = true`, remove da lista de colisões, começa a cair
4. Cai a 600 px/s até sair da tela (`y > VH+80`)

**Visual**: Retângulo laranja gradiente com bordas tracejadas. Quando caindo, continua renderizando até sumir.

**Níveis**: 1-3 (×6), 2-2 (×6)

---

### 2. Glitch (Barreira de Glitch)

| Propriedade | Valor |
|-------------|-------|
| Tipo | `glitch` |
| Largura | 60px |
| Altura | 220px |
| Destravável | Sim (ao resolver puzzle associado) |

**Comportamento**:
- Bloqueia passagem (colisão sólida)
- Removido quando `broken = true` (via puzzle resolvido)
- Pode ter `key` que referencia qual cloud desbloqueia

**Visual**:
- Gradiente magenta (#3a0e3a → #c93b8a)
- Barras horizontais glitch que se movem
- Borda tracejada branca
- Texto "GLITCH" rotacionado 90°
- Sombra abaixo

**Níveis**: 1-2 (×1), 2-3 (×1), 3-3 (×1)

---

### 3. Falling (Caixas Caindo)

| Propriedade | Valor |
|-------------|-------|
| Tipo | `falling` |
| Periodicidade | `period` (padrão 1.6s) |
| Largura da zona | `width` (padrão 250–300px) |

**Comportamento**:
- A cada `period` segundos, gera uma caixa em posição X aleatória dentro da zona
- Caixa: `{x, y:-30, vy:0, w:30, h:30, life:6}`
- Gravidade: 1500 px/s²
- Dano ao player: 25 por hit
- Removida após 6s ou hit

**Visual**:
- Indicador no topo: barra vermelha semi-transparente (`rgba(255,93,108,.25)`)
- Caixas: retângulo escuro com linhas brancas (texto genérico)

**Níveis**: 2-1 (×1, period 1.6s, width 250), 3-2 (×1, period 1.6s, width 300), 3-3 (×1, period 1.4s, width 300)

---

### 4. Silence (Zona de Silêncio)

| Propriedade | Valor |
|-------------|-------|
| Tipo | `silence` |
| Dimensões | variável (w, h) |

**Comportamento**:
- Dano contínuo ao player dentro da zona: `15 * dt` por segundo
- Suprime ruído estático de áudio (`setStatic(0)` dentro da zona)
- Suprime notificações sonoras

**Visual**:
- Retângulo semi-transparente (`rgba(60,60,90,.18)`)
- Borda tracejada branca
- Texto "· ZONA DE SILÊNCIO ·"

**Simbolismo**: Representa quando o ambiente digital silencia a vítima em vez de protegê-la.

**Níveis**: 2-2 (×1, 600×300), 2-B (×1, 600×300), 3-2 (×1, 500×300), 3-3 (×1, 400×400)

---

### 5. Mirror (Espelho de Cyberbullying)

| Propriedade | Valor |
|-------------|-------|
| Tipo | `mirror` |
| Largura | `w` (variável) |

**Comportamento**:
- Inverte os controles horizontais do player enquanto dentro da zona
- Ativa overlay visual `#mirrorFx` (gradiente magenta→verde)

```javascript
const inZone = p.x+p.w/2 > o.x && p.x+p.w/2 < o.x + o.w;
if (inZone && !p.mirrored) { p.mirrored = true; ... }
if (!inZone && p.mirrored) { p.mirrored = false; ... }
```

**Visual**:
- Zona semi-transparente azul (`rgba(120,220,255,.06)`)
- Borda tracejada azul
- Texto "· ESPELHO DE CYBERBULLYING — VEJA DO OUTRO LADO ·"

**Simbolismo**: Representa ver a situação da perspectiva do outro (vítima/agressor).

**Níveis**: 3-1 (×1, width 600)

---

## Resumo de Obstáculos

| Tipo | Dano | Mecânica | Níveis |
|------|------|----------|--------|
| Unstable | Indireto (queda) | Desaparece após 1.8s em pé | 1-3, 2-2 |
| Glitch | Nenhum | Bloqueia passagem, abre com puzzle | 1-2, 2-3, 3-3 |
| Falling | 25/hit | Caixas caem periodicamente | 2-1, 3-2, 3-3 |
| Silence | 15/s contínuo | Dano + silencia áudio | 2-2, 2-B, 3-2, 3-3 |
| Mirror | Nenhum | Inverte controles | 3-1 |
