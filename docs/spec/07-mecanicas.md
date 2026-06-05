# 07 — Mecânicas

## 1. Resiliência Emocional

Referência: `ARQ:1678`, `ARQ:2082–2100`

A resiliência é a **barra de vida** do jogador. Começa em 100 e diminui com:

| Fonte de Dano | Quantidade | Condição |
|---------------|-----------|----------|
| Tocar inimigo | 35 | Por colisão (com bounce) |
| Projétil (bot/boss) | 20 | Por hit |
| Caixa caindo (falling) | 25 | Por hit |
| Zona de silêncio | 15 × dt | Dano contínuo por segundo |
| Toxidade da nuvem | 40 × dt | Dano contínuo quando dentro da nuvem |
| Puzzle errado | 6 | Por tentativa incorreta |
| Queda no void | 35 | Player cai abaixo de VH+200 |
| Timeout | 100 | Timer chega a 0 (morte instantânea) |

### Escudo de Empatia

Se `L.shielded === true`, o primeiro dano com `amt > 1` é absorvido. O escudo é removido após absorver um hit.

### Game Over

Quando `L.res <= 0`, `gameOver()` é chamado (`ARQ:2965`):
- Muda para tela OVER
- Para o ruído estático
- Opções: Recomeçar fase ou Voltar ao mapa

### Respawn

Ao cair no void (y > VH+200), o player volta ao spawn com dano de 35.

---

## 2. Medalhas

Referência: `ARQ:2901–2911`

Ao completar um nível, o sistema calcula medalhas:

| Medalha | Requisitos |
|---------|-----------|
| **Bronze** (1) | Completou o nível (chegou ao goal + puzzles mínimos) |
| **Prata** (2) | Bronze + Resiliência ≥ 50% (se `dontLoseRes`) |
| **Ouro** (3) | Prata + Coletou todos os prints do nível |

### Lógica Detalhada

```javascript
let medals = 1; // bronze sempre
// Prata: se nível exige dontLoseRes e res >= 50, OU se não exige
if (L.res >= 50 || !cfg.objectives.dontLoseRes) medals = Math.max(medals, 2);
// Ouro: se coletou todos os prints
if (L.printsGot >= cfg.objectives.prints) medals = 3;
// Clamp prata: se exige dontLoseRes e res < 50, limita a bronze
if (cfg.objectives.dontLoseRes && L.res < 50) medals = Math.min(medals, 1);
// Clamp ouro: se exige dontLoseRes e res < 50, limita a prata
if (medals === 3 && cfg.objectives.dontLoseRes && L.res < 50) medals = 2;
```

Boss fights sempre têm `dontLoseRes: false`, então prata é automática.

---

## 3. Progressão e Desbloqueio

Referência: `ARQ:625–640`, `ARQ:1626–1640`

### Desbloqueio Inicial

Apenas o nível `1-1` começa desbloqueado:
```javascript
unlocked: {'1-1': true}
```

### Desbloqueio Sequencial

Ao completar qualquer nível, o próximo na lista linear `LEVELS` é desbloqueado:

```javascript
function onLevelCompleted(id){
  const idx = LEVELS.findIndex(l => l.id === id);
  const next = LEVELS[idx + 1];
  if (next) SAVE.unlocked[next.id] = true;
}
```

Ordem de desbloqueio:
```
1-1 → 1-2 → 1-3 → 1-B → 2-1 → 2-2 → 2-3 → 2-B → 3-1 → 3-2 → 3-3 → 3-B
```

### Mundo Completo

Se o nível completado é boss (`isBoss`), marca `SAVE.worldComplete[world] = true`.

### Final do Jogo

Se o boss final (`3-B`) é completado, `SAVE.finalSeen = true` e a tela de vitória (WIN) é exibida.

---

## 4. Condição de Vitória do Nível

Referência: `ARQ:2885–2954`

Para completar um nível, o player deve:

1. **Chegar ao goal** (overlap com `L.goal`)
2. **Resolver puzzles suficientes**: `L.puzzlesSolved >= cfg.objectives.puzzles`
3. **Boss derrotado** (se aplicável): `L.boss.defeated === true`

Se qualquer condição falhar, o player é empurrado para trás e recebe toast de aviso.

---

## 5. Controles

Referência: `ARQ:2993–3030`

### Keymap

```javascript
const keymap = {
  ArrowLeft:  'left',
  ArrowRight: 'right',
  ArrowUp:    'jump',
  KeyA:       'left',
  KeyD:       'right',
  KeyW:       'jump',
  Space:      'jump',
  KeyE:       'use',
  Enter:      'use',
  KeyM:       'mute',
  Escape:     'esc'
};
```

### Ações por Tela

| Tela | Tecla | Ação |
|------|-------|------|
| **PLAY** | `←` `→` ou `A` `D` | Mover |
| | `Espaço` ou `W` | Pular |
| | `E` | Interagir (nuvem/barreira) |
| | `M` | Mute toggle |
| | `Esc` | Voltar ao mapa |
| **MAP** | `←` `→` | Mover entre nós |
| | `E` | Selecionar fase |
| | `Esc` | Voltar ao título |
| **PUZZLE** | (clique no DOM) | Interação com puzzle |
| | `Esc` | Recuar do puzzle |

### Prevenção de Scroll

`Space`, `ArrowUp`, `KeyW` têm `e.preventDefault()` para evitar scroll da página.

---

## 6. Timer

Referência: `ARQ:1775–1782`, `ARQ:2240–2245`

Níveis com `timeLimit > 0` têm countdown:

```javascript
if (L.timeLeft > 0) {
  L.timeLeft -= dt;
  if (L.timeLeft <= 0) {
    L.timeLeft = 0;
    damageRes(100); // morte instantânea
  }
}
```

O timer é exibido no HUD central com `.warn` (vermelho) quando < 15s.

Níveis com timer: `1-3` (90s), `2-2` (75s).

---

## 7. Espelho (Mirror)

Referência: `ARQ:1858–1863`

Quando o player entra em uma zona de espelho (`obstacle mirror`):

```javascript
p.mirrored = true;   // inverte controles horizontais
$('#mirrorFx').classList.add('on');  // overlay visual
```

O input é invertido em `updatePlay`:
```javascript
if (L.player.mirrored) ax = -ax;
```

Ao sair da zona, o espelho é removido. Simboliza "ver o cyberbullying do outro lado".
