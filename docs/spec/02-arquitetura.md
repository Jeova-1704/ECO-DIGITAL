# 02 — Arquitetura

## Visão Geral

O jogo é um **arquivo HTML único** (`Eco Digital.html`) com três seções embutidas:

1. **`<style>`** (linhas 7–325) — CSS completo
2. **`<body>` HTML** (linhas 327–606) — DOM com canvas + overlays
3. **`<script>`** (linhas 607–3046) — JS completo dentro de uma IIFE

Referência: `ARQ:1–3049`

## Estrutura do Arquivo

```
Eco Digital.html (3049 linhas)
├── <head>
│   └── <style> (318 linhas)
│       ├── CSS Variables (:root)
│       ├── Reset & Base
│       ├── Componentes (.btn, kbd, .eyebrow)
│       ├── Overlays (.overlay, #titleScreen, #mapScreen, ...)
│       ├── HUD (#hud, .hud-card, .bar, .minimap, ...)
│       ├── Puzzle (#puzzleScreen, .node, .word, .option, .step, ...)
│       ├── Archive (#archiveScreen, .print-card)
│       ├── Victory / Game Over / Result
│       └── Responsivo (@media max-width: 720px)
│
├── <body>
│   ├── #stage > #frame
│   │   ├── <canvas id="game"> (1600×900)
│   │   ├── #hud (barra de resiliência, prints, timer, minimap, hint)
│   │   ├── #mirrorFx, #flash, #toast (efeitos visuais)
│   │   └── [10 overlays de tela]
│   └── </body>
│
└── <script> (IIFE, ~2430 linhas)
    ├── Utility ($, $$, clamp, overlap, lerp)
    ├── SAVE (estado persistido)
    ├── AUDIO (Web Audio API sintetizado)
    ├── ARCHIVE (20 prints educativos)
    ├── PUZZLES (6 construtores de puzzle: A–F)
    ├── Catálogo de inimigos (makeEnemy, updateEnemy, drawEnemy)
    ├── Catálogo de power-ups (POWERUPS, drawPowerup)
    ├── LEVELS (12 configurações de nível)
    ├── WORLDS (3 mundos)
    ├── ENGINE (game loop, state machine, buildLevelRuntime)
    ├── RENDER (pipeline Canvas completo)
    ├── MAP (mapa-múndi: nós, paths, avatar)
    ├── SCREEN MANAGEMENT (hideAllOverlays, showScreen)
    ├── MAP UI (renderMapInit, showNodePanel, mapMove)
    ├── ARCHIVE UI (renderArchive)
    ├── RESULTS (finishLevel, gameOver)
    ├── INPUT (keymap, keydown/keyup listeners)
    └── BOOT (requestAnimationFrame(loop))
```

## IIFE e Escopo

Todo o JS roda dentro de uma **IIFE** (`(() => { 'use strict'; ... })()`) — `ARQ:614–3045`.

Isso garante:
- Nenhum poluição do escopo global (exceto listeners nativos)
- `'use strict'` ativo
- Variáveis encapsuladas

As únicas coisas no escopo global:
- Elementos DOM (naturais do HTML)
- `window.addEventListener` para keydown/keyup/error

## Dependências

**Zero dependências externas**. Tudo é implementado nativamente:

| Recurso | Implementação |
|---------|---------------|
| Renderização | Canvas 2D API nativa |
| Áudio | Web Audio API (`AudioContext`) |
| Parser/Template | Nenhum — HTML inline no JS via `innerHTML` |
| Física | Própria (AABB, gravidade, velocidade) |
| Estado | Objeto JS em memória (`SAVE`) |
| Áudio de ruído | `AudioBufferSourceNode` com buffer de ruído branco |

## Módulos Lógicos (dentro da IIFE)

Apesar de ser um arquivo só, o código é organizado em módulos lógicos separados por comentários de seção:

```
/* ====================== SAVE ============================ */     ARQ:625
/* ====================== AUDIO =========================== */     ARQ:639
/* ====================== CATALOG: PRINTS / ARCHIVE ======== */    ARQ:683
/* ====================== CATALOG: PUZZLES ================ */     ARQ:733
/* ====================== CATALOG: ENEMIES =================== */  ARQ:1126
/* ====================== CATALOG: POWER-UPS =================== */ARQ:1260
/* ====================== CATALOG: LEVELS =================== */   ARQ:1310
/* ====================== ENGINE ============================ */    ARQ:1642
/* ====================== RENDER ============================ */    ARQ:2280
/* ====================== MAP ============================ */       ARQ:2641
/* ====================== SCREEN MANAGEMENT ============== */       ARQ:2772
/* ====================== MAP UI ========================= */       ARQ:2791
/* ====================== ARCHIVE ======================== */       ARQ:2869
/* ====================== RESULTS / END LEVEL ============ */       ARQ:2884
/* ====================== TITLE / NAV BINDINGS ============ */      ARQ:2971
/* ====================== INPUT =========================== */      ARQ:2992
/* ====================== BOOT ============================ */      ARQ:3038
```

## Canvas

O `<canvas>` tem atributos fixos `width="1600" height="900"` (`ARQ:329`), mas é redimensionado dinamicamente pela função `fitCanvas()` (`ARQ:1646`):

```javascript
function fitCanvas(){
  const rect = canvas.getBoundingClientRect();
  const dpr = Math.min(2, window.devicePixelRatio || 1);
  canvas.width  = Math.floor(rect.width * dpr);
  canvas.height = Math.floor(rect.height * dpr);
}
```

A transformação do contexto é aplicada a cada frame via `applyTransform()` (`ARQ:1654`):

```javascript
function applyTransform(){
  const sx = canvas.width / VW;   // VW = 1600
  const sy = canvas.height / VH;  // VH = 900
  gctx.setTransform(sx, 0, 0, sy, 0, 0);
}
```

O `#frame` (container) mantém aspect-ratio 16:9 via CSS (`aspect-ratio: 16/9`), e o canvas preenche 100% dele.

## Fluxo de Dados

```
Input (teclado)
  └─> G.keys (Set<string>)
       └─> updatePlay(dt) / updateMap(dt) / tryInteract()
            └─> Mutam estado em G.L (level runtime) ou G.map
                 └─> render() lê estado e desenha no Canvas
                      └─> updateHUD() atualiza DOM do HUD
```
