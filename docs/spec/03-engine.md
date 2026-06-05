# 03 — Engine

## Game Loop

Referência: `ARQ:1762–1769`

```javascript
function loop(t){
  const dt = Math.min(0.033, (t - G.last)/1000 || 0);
  G.last = t;
  if (G.screen === SCREEN.PLAY) updatePlay(dt);
  else if (G.screen === SCREEN.MAP) updateMap(dt);
  render();
  requestAnimationFrame(loop);
}
```

- **Frequência**: `requestAnimationFrame` (60fps alvo)
- **dt**: delta time em segundos, **clampado a 0.033** (~30fps mínimo) para evitar saltos
- **Primeiro frame**: `G.last = 0`, então `dt` é 0 no primeiro ciclo (seguro)

## Constantes

| Nome | Valor | Linha | Descrição |
|------|-------|-------|-----------|
| `VW` | 1600 | 1645 | Viewport virtual — largura |
| `VH` | 900 | 1645 | Viewport virtual — altura |
| `FLOOR` | 820 | 1326 | Y do chão padrão |

## State Machine

O estado global é gerenciado por `G.screen` (`ARQ:1661`):

```javascript
const SCREEN = {
  TITLE:   'title',
  MAP:     'map',
  INTRO:   'intro',
  PLAY:    'play',
  PUZZLE:  'puzzle',
  RESULT:  'result',
  OVER:    'over',
  WIN:     'win',
  ARCHIVE: 'archive',
  HOW:     'how',
  ABOUT:   'about'
};
```

Objeto global `G` (`ARQ:1662`):

```javascript
const G = {
  screen: SCREEN.TITLE,
  keys: new Set(),
  last: 0,
  map: {
    avatarX: 140,
    avatarY: VH - 220,
    walking: false,
    walkTo: null,
    currentNodeId: '1-1'
  },
  L: null   // level runtime (construído ao iniciar nível)
};
```

## Física do Player

Referência: `ARQ:1771–1808`

### Parâmetros

| Propriedade | Valor | Descrição |
|-------------|-------|-----------|
| Velocidade horizontal | 360 px/s | `p.vx = ax * 360` |
| Força do pulo | -820 px/s | `p.vy = -820` |
| Gravidade | 1600 px/s² | `p.vy += 1600 * dt` |
| Velocidade terminal | 1200 px/s | `if (p.vy > 1200) p.vy = 1200` |

### Dimensões do Player

| Propriedade | Valor |
|-------------|-------|
| `w` (largura) | 36 |
| `h` (altura) | 56 |

### Ordem de Resolução

1. **Input** → `ax` (aceleração horizontal)
2. **Espelhamento** → se `p.mirrored`, inverte `ax`
3. **Pulo** → se `jump` pressionado e `onGround`, `vy = -820`
4. **Gravidade** → `vy += 1600 * dt`
5. **Integração X** → `x += vx * dt` → `resolveX()`
6. **Integração Y** → `y += vy * dt` → `resolveY()` com `onGround = false`
7. **Bounds** → clamp horizontal, queda no void → respawn + dano

## Colisão (AABB)

Referência: `ARQ:2045–2080`

### overlap() — detecção genérica

```javascript
const overlap = (a,b)=> a.x < b.x+b.w && a.x+a.w > b.x && a.y < b.y+b.h && a.y+a.h > b.y;
```

### resolveX() — colisão horizontal

Para cada plataforma que sobrepõe o player:
- Se `vx > 0`: empurra para esquerda da plataforma
- Se `vx < 0`: empurra para direita
- Zera `vx`

Também resolve contra **Plataformas Solidárias** (nuvens já resolvidas).

### resolveY() — colisão vertical

Para cada plataforma que sobrepõe o player:
- Se `vy > 0` (caindo): posiciona no topo → `onGround = true`
- Se `vy < 0` (subindo): posiciona abaixo
- Zera `vy`

Também resolve contra Plataformas Solidárias.

### Plataformas Solidárias

Quando uma Nuvem de Toxicidade é resolvida (puzzle), ela vira uma plataforma sólida:

```javascript
function solvedRect(c){
  return { x: c.x - 10, y: c.y + c.h - 22, w: c.w + 20, h: 22 };
}
```

## Câmera

Referência: `ARQ:1810–1811`

```javascript
const camTarget = clamp(p.x - VW*0.4, 0, L.width - VW);
L.camX += (camTarget - L.camX) * Math.min(1, dt*6);
```

- **Offset**: player fica a 40% da largura da tela (esquerda)
- **Smooth**: interpola com fator `dt*6` (~6× por segundo)
- **Limites**: `[0, levelWidth - VW]`

## Build Level Runtime

Referência: `ARQ:1673–1718`

`buildLevelRuntime(cfg)` converte a configuração estática do nível em estado runtime:

```javascript
const L = {
  cfg,                    // config original
  width: cfg.width,
  camX: 0,
  res: 100,               // resiliência (%)
  time: 0,                // tempo decorrido
  timeLeft: cfg.objectives.timeLimit || 0,
  puzzlesSolved: 0,
  printsGot: 0,
  printsCount: cfg.prints.length,
  keys: G.keys,
  player: { x, y, w:36, h:56, vx:0, vy:0, onGround:false, facing:1, runT:0, hitFlash:0, mirrored:false },
  platforms: [],          // ground + posts + unstable/glitch (built from cfg)
  clouds: [],             // copiadas de cfg com id, solved, glitch
  enemies: [],            // makeEnemy() para cada cfg.enemy
  obstacles: [],          // copiados com estado runtime (broken, t, removed)
  prints: [],             // {x,y,w:22,h:22,got,archive,id}
  powerups: [],           // {type,x,y,w:22,h:22,taken}
  projectiles: [],        // tiros de bots/boss
  fallers: [],            // caixas caindo (obstacle falling)
  ally: null,             // {x,y,t} — aliado temporário
  powerActive: null,      // {type,t} — power-up com duração
  shielded: false,
  notifTimer: 0,
  goal: { x:cfg.goalX, y:FLOOR-160, w:60, h:160 },
  glitchT: 0,
  boss: null,             // makeBoss() se cfg.boss existe
  onFinish: null,
  started: false,
};
```

### Montagem de plataformas

- `cfg.ground` → plataformas `kind:'ground'` (h:80)
- `cfg.platforms` → plataformas `kind:'post'` (h:22, estilo bolha de chat)
- `obstacle unstable` → plataforma `kind:'unstable'` (h:18, com ref ao obstacle)
- `obstacle glitch` → plataforma `kind:'glitch'` (sólida até ser quebrada)

## Boss Runtime

Referência: `ARQ:1721–1730`

```javascript
function makeBoss(spec){
  return {
    kind: spec.kind,      // 'fake' | 'admin' | 'hater'
    name: spec.name,
    hp: spec.hp,
    maxHp: spec.hp,
    phase: 1,             // 1: dodge, 2: collect, 3: puzzle final
    x: 1900, y: 250,
    w: 180, h: 200,
    t: 0, fireT: 0,
    spawnedFinal: false,
  };
}
```

### Fases do Boss

Referência: `ARQ:2189–2223`

| Fase | Condição de entrada | Comportamento |
|------|---------------------|---------------|
| 1 | Início | Dispara 3 projéteis a cada 1.0s. Transiciona após 8s. |
| 2 | `time > 8` | Dispara 2 projéteis a cada 1.6s. Transiciona quando `printsGot >= objective.prints`. |
| 3 | Prints coletados | Para de disparar agressivamente. A Nuvem final já está no nível — jogador resolve o puzzle. |

## Dano de Resiliência

Referência: `ARQ:2082–2100`

`damageRes(amt)`:
1. Se `shielded` e `amt > 1`: absorve, remove escudo
2. Reduz `L.res` em `amt`
3. Se `amt > 1`: flash vermelho + som de dano
4. Se `L.res <= 0`: game over

## Animação do Player

- `runT`: contador de tempo para animação de corrida (`Math.sin(p.runT)*6` para pernas)
- `hitFlash`: timer de flash de dano (0.35s)
- `facing`: 1 (direita) ou -1 (esquerda), controla offset dos olhos
