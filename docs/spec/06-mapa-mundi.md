# 06 — Mapa-Múndi

## Visão Geral

O mapa-múndi é renderizado **no Canvas** (não em HTML). Mostra 3 mundos com 4 nós cada (12 no total), conectados por paths curvos.

Referência: `ARQ:2641–2770`

## Geração dos Nós

Referência: `ARQ:2643–2662`

```javascript
const MAP_NODES = [];
(function buildMapNodes(){
  const rows = [
    { worldIdx:0, y:300 },  // Mundo 1 — topo
    { worldIdx:1, y:520 },  // Mundo 2 — meio-baixo
    { worldIdx:2, y:300 },  // Mundo 3 — topo
  ];
  WORLDS.forEach((w, wi) => {
    const baseY = rows[wi].y;
    w.levels.forEach((id, li) => {
      const stepX = 90;
      const offsetX = wi * 430;
      const xx = 200 + offsetX + li * stepX;
      const yy = baseY + Math.sin(li*1.2 + wi)*30 + (li===3 ? -40 : 0);
      MAP_NODES.push({ id, x: xx, y: yy, world: w.id, isBoss: id.endsWith('-B'), label: LEVEL_BY_ID[id].name });
    });
  });
})();
```

### Layout Resultante

```
Y=300:    Mundo 1         Mundo 3
          (1-1)(1-2)(1-3)(1-B)    (3-1)(3-2)(3-3)(3-B)
          X: 200 290 380 470      X: 1060 1150 1240 1330

Y=520:         Mundo 2
               (2-1)(2-2)(2-3)(2-B)
               X: 630 720 810 900
```

### Coordenadas de Cada Nó

| ID | X | Y | Boss? |
|----|---|---|-------|
| 1-1 | 200 | 300 | Não |
| 1-2 | 290 | 322 | Não |
| 1-3 | 380 | 296 | Não |
| 1-B | 470 | 260 | Sim |
| 2-1 | 630 | 520 | Não |
| 2-2 | 720 | 542 | Não |
| 2-3 | 810 | 516 | Não |
| 2-B | 900 | 480 | Sim |
| 3-1 | 1060 | 300 | Não |
| 3-2 | 1150 | 322 | Não |
| 3-3 | 1240 | 296 | Não |
| 3-B | 1330 | 260 | Sim |

## Avatar

Referência: `ARQ:2664–2686`

O avatar é desenhado no canvas como um personagem simplificado:

```javascript
// Estado
G.map = {
  avatarX: 140,
  avatarY: VH - 220,
  walking: false,
  walkTo: null,
  currentNodeId: '1-1'
};
```

### Movimentação

- **Teclado**: `←` / `→` dispara `mapMove(dir)` (`ARQ:2855`)
- Movimenta na sequência linear de `LEVELS` (índice + dir)
- Só move para níveis desbloqueados
- Avatar caminha em linha reta até o destino (260 px/s)

### Renderização

`drawMapAvatar(x, y)` (`ARQ:2764–2770`):
- Sombra elíptica
- Corpo verde arredondado (#9af2d3)
- Cabeça bege (#f4ead6)
- Olhos escuros
- Sem animação de caminhada

## Paths entre Nós

Referência: `ARQ:2688–2706`

Renderizados como **curvas Bézier quadráticas**:

```javascript
gctx.beginPath();
gctx.moveTo(a.x, a.y);
const mx = (a.x+b.x)/2, my = (a.y+b.y)/2 + 60;
gctx.quadraticCurveTo(mx, my, b.x, b.y);
gctx.stroke();
```

- **Desbloqueados**: linha contínua verde clara `rgba(154,242,211,.6)`
- **Bloqueados**: linha tracejada fraca `rgba(255,255,255,.12)`

## Painel de Fase (NodePanel)

Referência: `ARQ:131–144` (HTML), `ARQ:2812–2846` (JS)

HTML overlay posicionado em `right:22px; bottom:22px`, 360px de largura. Mostra:

| Campo | Conteúdo |
|-------|----------|
| `#pnLbl` | `MUNDO X · FASE Y` ou `MUNDO X · CHEFÃO` |
| `#pnTitle` | Nome da fase |
| `#pnDesc` | Descrição da fase |
| `#pnObj` | Lista de objetivos (1. Chegar ao fim, 2. Puzzles, 3. Prints, etc.) |
| `#pnMedal` | Chip com medalha atual ou status |
| `#pnStart` | Botão "Começar" / "Rejogar" / "Bloqueada" (disabled) |

### Medal Chip

| Situação | Exibição |
|----------|----------|
| Medalha 3 | 🟡 OURO |
| Medalha 2 | ⚪ PRATA |
| Medalha 1 | 🟤 BRONZE |
| Desbloqueada | Não jogada |
| Bloqueada | Bloqueada |

## Renderização dos Nós

Referência: `ARQ:2730–2763`

`drawMapNode(n)` desenha para cada nó:

1. **Halo**: gradiente radial — verde (completo), laranja (desbloqueado), invisível (bloqueado)
2. **Círculo base**: preenchido — verde / bege / escuro
3. **Borda**: vermelha (boss), preta (normal)
4. **Ícone**: `#` (mundo 1), `@` (mundo 2), `§` (mundo 3), `☣` (boss)
5. **Label**: nome abaixo do nó
6. **Medalha**: ponto colorido no canto superior direito

### Indicador de Nó Atual

O nó selecionado recebe um anel:
```javascript
gctx.strokeStyle='rgba(154,242,211,.4)';
gctx.beginPath(); gctx.arc(cur.x, cur.y, 40, 0, Math.PI*2); gctx.stroke();
```

## Top Bar do Mapa

Referência: `ARQ:396–426` (HTML), `ARQ:2792–2811` (JS)

Contém:
- Título "Mapa do Espaço Digital"
- Dica de controles
- Total de prints no acervo
- Contagem de medalhas (ouro/prata/bronze)
- Botões: Acervo de Provas, Tela inicial

### Estatísticas

```javascript
let g=0, s=0, b=0;
Object.values(SAVE.completed).forEach(c => {
  if (c.medals >= 3) g++;
  else if (c.medals === 2) s++;
  else if (c.medals >= 1) b++;
});
```
