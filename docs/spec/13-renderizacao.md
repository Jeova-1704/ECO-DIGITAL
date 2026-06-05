# 13 — Renderização

## Pipeline de Renderização

Referência: `ARQ:2280–2314`

```javascript
function render(){
  applyTransform();
  // 1. Background (gradiente céu)
  // 2. Parallax (glow blobs + grid)
  // 3. (se MAP) renderMap() e return
  // 4. (se TITLE) return (só background)
  // 5. (se nível carregado):
  //    a. translate(-L.camX, 0)  ← câmera
  //    b. drawTheme(L)
  //    c. drawPlatforms(L)
  //    d. drawObstacles(L)
  //    e. drawPrints(L)
  //    f. drawClouds(L)
  //    g. drawPowerups(L)
  //    h. drawProjectiles(L)
  //    i. drawFallers(L)
  //    j. drawEnemies(L)
  //    k. drawAlly(L)
  //    l. drawBoss(L)
  //    m. drawGoal(L)
  //    n. drawPlayer(L)
  //    o. restore
  // 6. (se boss) drawBossHud(L)
}
```

### Ordem Z (back-to-front)

```
Background (gradiente)
Parallax (glow blobs + grid)
Tema (tint por mundo)
Chão (ground platforms)
Obstáculos (silence zone, falling marker, mirror, unstable falling)
Plataformas (posts, unstable standing)
Prints (colecionáveis)
Nuvens (toxicidade / solidárias)
Power-ups
Projéteis
Caixas caindo (fallers)
Inimigos
Aliado
Boss
Goal (saída)
Player
Boss HUD (screen-space)
```

---

## Background

Referência: `ARQ:2284–2286`

```javascript
const sky = gctx.createLinearGradient(0,0,0,VH);
sky.addColorStop(0, '#0d122a');
sky.addColorStop(1, '#06081a');
gctx.fillStyle = sky;
gctx.fillRect(0,0,VW,VH);
```

Gradiente azul-escuro vertical.

---

## Parallax

Referência: `ARQ:2316–2334`

### Glow Blobs

10 esferas de gradiente azul-esverdeado, com parallax horizontal:

```javascript
const cam = (G.L ? G.L.camX : 0) * 0.2;
for (let i=0; i<10; i++){
  const x = ((i*420) - (cam%420)) % (VW+420) - 210;
  const y = 120 + (i%3)*180;
  // gradiente radial rgba(40,60,120,.35) → transparente
}
```

- Fator parallax: 0.2× (move 20% da câmera)
- Distribuição: 3 alturas (120, 300, 480), espaçamento 420px

### Grid

Linhas verticais com parallax e horizontais fixas:

```javascript
const camG = (G.L ? G.L.camX : 0) * 0.5;
// Linhas a cada 60px, cor rgba(255,255,255,0.03)
```

- Fator parallax vertical: 0.5×
- Horizontais: fixas, a cada 60px
- Cor: branco 3% opacidade

---

## Tema Visual por Mundo

Referência: `ARQ:2336–2345`

```javascript
function drawTheme(L){
  const th = L.cfg.theme || 'feed';
  let tint = 'rgba(120,160,255,.05)';
  if (th.startsWith('chat'))        tint = 'rgba(120,220,180,.05)';
  if (th.startsWith('forum') || th.startsWith('boss-forum')) tint = 'rgba(200,120,200,.05)';
  if (th.startsWith('boss-feed'))   tint = 'rgba(240,160,90,.05)';
  if (th.startsWith('boss-chat'))   tint = 'rgba(120,160,255,.05)';
  gctx.fillStyle = tint;
  gctx.fillRect(0, 0, L.width, VH);
}
```

| Tema | Tint | Visual |
|------|------|--------|
| feed | Azul claro | Atmosfera de rede social |
| chat | Verde claro | Atmosfera de mensageiro |
| forum | Roxo | Atmosfera de fórum anônimo |
| boss-feed | Laranja | Confronto no feed |
| boss-chat | Azul claro | Confronto no chat |
| boss-forum | Roxo | Confronto no fórum |

---

## Plataformas

Referência: `ARQ:2347–2403`

### Ground (chão)

```
Sombra: #0a0d1c retângulo completo
Topo: gradiente #f4ead6 → #d6caa6, 14px, bordas 6px
Corpo: #1c213f, bordas 4px
Decoração: 3 linhas brancas 8% opacidade (simulam postagem)
```

### Post (plataforma flutuante)

```
Sombra: offset (3,5), rgba(0,0,0,.35)
Corpo: gradiente #f4ead6 → #cdc09c, bordas 6px
Ponta: triângulo abaixo (apontador de chat bubble)
```

### Glitch (barreira)

```
Sombra: rgba(0,0,0,.3)
Corpo: gradiente #3a0e3a → #c93b8a
Barras glitch: 8 faixas horizontais que se movem (animadas com glitchT*8)
Borda: tracejada branca 25% opacidade
Texto: "GLITCH" rotacionado -90°, magenta claro
```

### Unstable (instável)

```
Sem shake: normal (gradiente laranja)
Com shake: offset senoidal sin(40*t)*2
Timer visual: borda tracejada preta
Caindo: continua renderizando na posição de queda
```

---

## Prints

Referência: `ARQ:2438–2453`

```
Halo: gradiente radial laranja (rgba(240,160,90,.45))
Bob: oscilação vertical sin(t/300 + x)*4
Rotação: oscilação sin(t/600 + x)*.15
Corpo: quadrado arredondado gradiente #fbd49a → #f0a05a
Detalhe: 3 linhas horizontais pretas (texto genérico)
```

---

## Nuvens de Toxicidade

Referência: `ARQ:2455–2504`

### Nuvem Ativa (não resolvida)

```
Halo: gradiente radial magenta rgba(201,59,138,.35)
Corpo: gradiente #c93b8a → #3a0e3a, bordas 18px
Wobble: sinérgico com glitchT (oscilação horizontal)
Scan lines: 6 faixas horizontais animadas (glitch visual)
Borda: tracejada branca 18% opacidade
Label: "!! NUVEM NN" branco 85% opacidade
Face: olhos retangulares pretos + boca jagged (denteada)
Indicador de proximidade: badge laranja "!" quando nearCloud
```

### Nuvem Resolvida (Plataforma Solidária)

```
Halo: gradiente radial verde rgba(103,232,197,.35)
Plataforma: gradiente #9af2d3 → #67e8c5, bordas 8px
Label: "SOLIDÁRIA · NN" verde escuro
```

---

## Player

Referência: `ARQ:2614–2639`

Renderizado com primitivas Canvas (sem sprites):

```
Sombra: elipse preta 35% opacidade (chão)
Pernas: #283063, retângulos 8px largura, animação sin(runT)*6
Corpo: #9af2d3 (verde), retângulo arredondado
  - Quando hitFlash: #ff5d6c (vermelho)
  - Quando shielded: #9af2d3 com anel dourado
Detalhe corpo: barra horizontal preta 25% opacidade (cinto)
Cabeça: #f4ead6 (bege), retângulo arredondado
Olhos: 2 quadrados pretos 3×3, offset por facing
Boné: #283063, retângulo arredondado no topo
```

---

## Goal (Saída)

Referência: `ARQ:2601–2613`

```
Halo: gradiente radial verde rgba(103,232,197,.35)
Corpo: gradiente #9af2d3 → #67e8c5, bordas 30px
Interior: retângulo escuro rgba(10,30,25,.5), bordas 22px
Texto: "SAÍDA · AMBIENTE LIMPO" rotacionado -90°, verde claro
```

---

## Boss

Referência: `ARQ:2553–2600`

```
Sombra: offset (4,8)
Corpo: retângulo grande (180×200), bordas 24px
  - Fake: verde #67e8c5
  - Admin: escuro #1d2350
  - Hater: roxo #3a0e3a
Glitch: 10 scan lines animadas
Face: olhos (2 retângulos 30×16) + boca jagged (denteada, 10 pontos)
Label: nome do boss em magenta claro, acima
```

### Boss HUD (screen-space)

Barra de progresso no topo central (360×8px):
- Fundo: rgba(10,13,28,.7) com padding
- Label: "CHEFÃO · NOME · FASE X/3"
- Barra magenta (#c93b8a) preenchida por fase:
  - Fase 1: 0–33% (tempo/8s)
  - Fase 2: 33–66% (prints coletados)
  - Fase 3: 66–100% (derrotado)
