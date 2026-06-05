# 05 — Sistemas

## 1. Save

Referência: `ARQ:625–636`

O sistema de save é **em memória** (sem `localStorage`). Dados se perdem ao recarregar a página.

```javascript
const SAVE = {
  unlocked: {'1-1': true},         // níveis desbloqueados
  completed: {},                    // id -> {medals: 0..3, prints, bestRes}
  totalPrints: 0,                   // total de prints coletados (global)
  collectedPrints: new Set(),       // IDs globais de prints já coletados
  worldComplete: {1:false, 2:false, 3:false},
  archiveSeen: new Set(),           // archive items desbloqueados
  finalSeen: false,                 // se viu tela de vitória
};
```

### Funções Auxiliares

| Função | Linha | Descrição |
|--------|-------|-----------|
| `isUnlocked(id)` | 634 | Verifica se nível está desbloqueado |
| `isComplete(id)` | 635 | Verifica se nível foi completado |
| `medalsFor(id)` | 636 | Retorna medalhas (0–3) do nível |

### Progressão

Ao completar um nível, `onLevelCompleted(id)` (`ARQ:1626`):
1. Desbloqueia o próximo nível na sequência linear de `LEVELS`
2. Se o nível é boss, marca `worldComplete[world] = true`
3. Se é o boss final (`3-B`), marca `SAVE.finalSeen = true`

---

## 2. Audio

Referência: `ARQ:639–680`

Sistema de áudio **100% sintetizado** via Web Audio API. Nenhum arquivo de áudio.

### Inicialização

```javascript
let ctx = null;  // AudioContext — criado lazy
```

O `AudioContext` é criado na primeira chamada de `ensure()` (`ARQ:641`), geralmente quando o usuário clica "Jogar" (requer interação do usuário para autoplay policy).

### Sons Sintetizados

| Função | Tipo | Parâmetros | Uso |
|--------|------|-----------|-----|
| `tone()` | Genérico | freq, dur, type, vol, sweep, delay | Base para todos os outros |
| `notification()` | Square sweep | 1320→1120Hz, 60ms + 990Hz, 50ms | Bot dispara projétil, ruído de toxicidade |
| `damage()` | Sawtooth sweep | 220→100Hz + 110→50Hz | Player toma dano |
| `pickup()` | Triangle | 880Hz + 1320Hz | Coleta print |
| `gold()` | Triangle arpeggio | 880/1108/1318/1760Hz | Print dourado |
| `jump()` | Triangle sweep | 520→700Hz | Pulo |
| `solved()` | Triangle arpeggio | 523/659/784/1046Hz | Puzzle resolvido |
| `wrong()` | Square | 200Hz, 120ms | Resposta errada no puzzle |
| `open()` | Sine | 220Hz + 330Hz | Abre puzzle |
| `bossHit()` | Sawtooth + Square | 90→50Hz + 60Hz | Boss atingido |
| `victory()` | Triangle arpeggio | 523/659/784/1046/1318Hz | Vitória final |
| `powerup()` | Sine arpeggio | 659/784/988/1318Hz | Coleta power-up |
| `shield()` | Sine | 540Hz + 720Hz | Escudo absorve dano |

### Ruído Estático (Static)

Referência: `ARQ:655–663`

Um **loop de ruído branco** toca continuamente com volume variável, simulando interferência de toxicidade:

```javascript
function startStatic() {
  // Cria buffer de 1s de ruído branco
  const buf = ctx.createBuffer(1, ctx.sampleRate, ctx.sampleRate);
  const d = buf.getChannelData(0);
  for (let i=0; i<d.length; i++) d[i] = (Math.random()*2-1) * 0.25;
  noiseNode = ctx.createBufferSource();
  noiseNode.buffer = buf;
  noiseNode.loop = true;
  noiseGain = ctx.createGain();
  noiseGain.gain.value = 0.0001;  // praticamente mudo
  noiseNode.connect(noiseGain).connect(ctx.destination);
  noiseNode.start();
}

function setStatic(level) {
  // level: 0..0.06 — volume do ruído
  noiseGain.gain.linearRampToValueAtTime(Math.max(0.0001, level), ctx.currentTime + 0.15);
}
```

A intensidade é calculada pela proximidade do player a nuvens não-resolvidas (`ARQ:2017–2025`):

```javascript
const intensity = clamp(1 - closest/700, 0, 1);
AUDIO.setStatic(inSilence ? 0 : intensity * 0.06);
```

Na **Zona de Silêncio**, o ruído é zero (simulando o silêncio opressor).

### Mute

`toggleMute()` (`ARQ:677`): alterna `muted`. Se mutado, zera `noiseGain`.

---

## 3. HUD

Referência: `ARQ:149–361` (HTML), `ARQ:2225–2269` (JS)

O HUD é renderizado em **HTML** (não no canvas), sobreposto ao canvas com `pointer-events: none`.

### Componentes

| Componente | ID | Descrição |
|------------|-----|-----------|
| Barra de Resiliência | `#resBar` | Barra verde → vermelha (low) → dourada (shielded) |
| Prints da fase | `#printCount` / `#printTotal` | "3 / 5 prints da fase" |
| Acervo total | `#totalPrints` | Total global de prints |
| Timer | `#timer` | Countdown se nível tem `timeLimit` |
| Power ativo | `#powerActive` | Label + segundos restantes |
| Mini-mapa | `#minimap` | Posição do player, nuvens e meta |
| Hint | `#hint` | "Pressione E para..." (perto de nuvem/barreira) |

### updateHUD()

Chamada a cada frame em `updatePlay()` (`ARQ:2242`):

1. Atualiza largura da barra de resiliência (`L.res + '%'`)
2. Toggle classes `.low` (res < 35) e `.shielded`
3. Atualiza contadores de prints
4. Atualiza timer (se aplicável), toggle `.warn` (time < 15s)
5. Atualiza power ativo (label + segundos)
6. Re-renderiza mini-mapa (remove dots antigos, cria novos)

### Mini-mapa

Referência: `ARQ:169–174` (HTML), `ARQ:2256–2268` (JS)

Estrutura: barra horizontal (`width: 44%`, `height: 22px`) com:
- `.mi-player` — ponto verde (#9af2d3) com glow
- `.mi-cloud` — ponto magenta (toxic) ou verde (solved)
- `.mi-goal` — barra verde à direita

Posições: `left = (entity.x / level.width * 100) + '%'`

---

## 4. Toast

Referência: `ARQ:185–188` (CSS), `ARQ:2271–2278` (JS)

Notificação flutuante temporária (1.3s), centralizada no topo:

```javascript
function toast(msg, kind){
  const el = $('#toast');
  el.textContent = msg;
  el.className = 'toast show ' + (kind || '');
  clearTimeout(toast._to);
  toast._to = setTimeout(() => el.className = 'toast', 1300);
}
```

Classes de cor: `.good` (borda verde) ou `.bad` (borda vermelha).

### Toasts Emitidos

| Mensagem | Contexto | Tipo |
|----------|----------|------|
| `+1 Print de Segurança` | Coleta print | good |
| `Print Dourado · +5 prints` | Coleta power-up gold | good |
| `Escudo de Empatia ativo` | Coleta shield | good |
| `Inimigos congelados por 5s` | Coleta freeze | good |
| `Aliado Temporário ativo` | Coleta ally | good |
| `Escudo de Empatia absorveu o ataque` | Shield absorve dano | good |
| `Escudo de Empatia absorveu` | Shield absorve dano em damageRes | good |
| `Plataforma Solidária criada` | Puzzle resolvido | good |
| `Barreira de glitch desfeita` | Puzzle de barreira resolvido | good |
| `{bossName} foi silenciado pela denúncia` | Boss derrotado | good |
| `Acervo desbloqueado: {title}` | Print novo desbloqueia archive | good |
| `Resolva pelo menos N puzzle(s)` | Chega ao goal sem puzzles suficientes | bad |
| `Encerre o chefão pelo Painel de Denúncia` | Boss não derrotado | bad |
| `Fase bloqueada` | Tenta entrar em nível bloqueado | bad |
| `Som mudo` / `Som ligado` | Toggle mute | good |

---

## 5. Efeitos Visuais (DOM)

### Flash de Dano

Referência: `ARQ:179`, `ARQ:2093–2096`

`#flash`: overlay com gradiente radial vermelho. Ativado por 140ms ao tomar dano:

```javascript
$('#flash').classList.add('on');
setTimeout(() => $('#flash').classList.remove('on'), 140);
```

### Espelho (Mirror)

Referência: `ARQ:181–183`

`#mirrorFx`: overlay com gradiente magenta→verde. Ativado quando player está na zona de espelho:

```javascript
if (inZone && !p.mirrored) { p.mirrored = true; $('#mirrorFx').classList.add('on'); }
```
