# 10 — Inimigos

## Visão Geral

4 tipos de inimigos com comportamentos distintos. Todos compartilham a interface base:

Referência: `ARQ:1126–1258`

```javascript
function makeEnemy(spec){
  return Object.assign({
    x:0, y:0, w:36, h:36, vx:0, vy:0,
    dead:false, frozen:0, neutralized:false,
    seedX:0, type:spec.type
  }, spec);
  e.seedX = e.x;
}
```

### Propriedades Comuns

| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `x`, `y` | number | Posição atual |
| `w`, `h` | number | Dimensões (hitbox AABB) |
| `seedX` | number | Posição X original (referência para patrulha) |
| `seedY` | number | Posição Y original (apenas Bot) |
| `dead` | boolean | Se foi destruído |
| `frozen` | number | Timer de congelamento (power-up freeze) |
| `neutralized` | boolean | Se foi pacificado (visual fantasma) |
| `hostile` | boolean | Se ficou agressivo (apenas Fake) |
| `invisibleUntil` | string | Condição para revelar (ex: `'print'`) |
| `revealed` | boolean | Se já foi revelado |
| `range` | number | Alcance de patrulha/perseguição |

### Comportamento Universal

- Se `dead` ou `neutralized`: sem update
- Se `frozen > 0`: decrementa timer, sem movimento
- Colisão com player: causa 35 de dano (se não shielded), com bounce

---

## Troll

Referência: `ARQ:1147–1152`, `ARQ:1199–1207`

### Comportamento

Patrulha horizontal em linha reta:

```javascript
e.x += e.dir * 90 * dt;
if (e.x > e.seedX + (e.range||180)) e.dir = -1;
if (e.x < e.seedX) e.dir = 1;
```

- **Velocidade**: 90 px/s
- **Alcance padrão**: 180px
- **Direção**: alterna ao atingir limites

### Visual

- Corpo quadrado vermelho (#ff5d6c) com bordas arredondadas (8px)
- Olhos: dois quadrados pretos 5×5
- Boca: barra horizontal preta (rosto irritado)
- Sobrancelha: barra superior preta (expressão brava)
- Sombra abaixo

### Níveis onde aparece

1-1 (×2), 1-2 (×1), 1-3 (×1), 2-1 (×1), 3-2 (×1), 3-3 (×1)

---

## Bot (Bot de Spam)

Referência: `ARQ:1153–1162`, `ARQ:1208–1220`

### Comportamento

Movimenta-se em padrão sinusoidal e **dispara projéteis**:

```javascript
e.t += dt;
e.x = e.seedX + Math.sin(e.t*1.2) * (e.range||180);
e.y = (e.seedY||e.y) + Math.sin(e.t*2.4) * 30;
e.shoot += dt;
if (e.shoot > 1.6) {
  e.shoot = 0;
  world.projectiles.push({
    x: e.x+18, y: e.y+18,
    vx: (world.player.x < e.x ? -180 : 180),
    vy: 40, life: 4
  });
}
```

- **Movimento X**: senoidal, amplitude = range
- **Movimento Y**: senoidal, amplitude = 30px
- **Tiros**: a cada 1.6s, projétil na direção do player
- **Som**: `AUDIO.notification()` ao disparar

### Visual

- Corpo losango (quadrado rotacionado 45°) magenta (#c93b8a)
- Linha horizontal preta no centro (olho)
- Antena: linha fina + bola magenta no topo

### Níveis onde aparece

1-2 (×1), 1-3 (×2), 2-1 (×1), 2-3 (×1), 3-2 (×1)

---

## Fake (Perfil Fake)

Referência: `ARQ:1163–1172`, `ARQ:1221–1241`

### Comportamento

Começa amigável, fica hostil quando o player se aproxima:

```javascript
const dx = world.player.x - e.x;
const d = Math.abs(dx);
if (!e.hostile && d < 220) { e.hostile = true; e.flashT = 0.4 }
if (e.hostile) {
  e.x += (dx > 0 ? 1 : -1) * 70 * dt;  // perseguição lenta
}
```

- **Distância de ativação**: 220px
- **Flash**: 0.4s de transição visual
- **Velocidade de perseguição**: 70 px/s
- **Altura**: 40px (maior que outros inimigos)

### Visual

**Amigável (não hostil)**:
- Corpo verde (#67e8c5), bordas arredondadas
- Olhos + sorriso (arco para cima)
- Texto "OI?" flutuante acima

**Hostil**:
- Flash vermelho (#ff5d6c) por 0.4s, depois magenta (#c93b8a)
- Olhos + carranca (arco para baixo)
- Texto "FAKE!" vermelho acima

### Níveis onde aparece

2-1 (×1), 2-2 (×1), 2-3 (×1), 3-1 (×1), 3-2 (×1), 3-3 (×1)

---

## Hater (Hater Voador)

Referência: `ARQ:1173–1187`, `ARQ:1242–1257`

### Comportamento

Persegue o player em voo quando dentro do alcance, patrulha senoidal caso contrário:

```javascript
const dist = Math.hypot(dx, dy);
if (dist < (e.range||500) && !e.neutralized) {
  e.x += (dx/dist) * 80 * dt;
  e.y += (dy/dist) * 80 * dt;
} else {
  e.x = e.seedX + Math.sin(e.t) * 90;
}
```

- **Alcance de perseguição**: 500px (padrão)
- **Velocidade de perseguição**: 80 px/s
- **Patrulha**: sinusoidal, amplitude 90px
- **Dimensões**: 44×32 (mais largo que outros)
- **Pode ser invisível**: `invisibleUntil: 'print'` (mundo 3)

### Visual

- Corpo em forma de nuvem: retângulo arredondado magenta (#6c1e6a) + overlay mais claro (#c93b8a)
- Olhos: dois quadrados pretos 4×4
- Expressão ">:(" em vermelho quando não neutralizado

### Invisibilidade

Quando `invisibleUntil === 'print'`:
- Inimigo é desenhado como contorno tracejado com 15% de opacidade
- Revelado quando não há print perto dele (todos coletados ou nenhum disponível no raio)

```javascript
const nearestPrint = L.prints.find(pr =>
  !pr.got && Math.hypot((pr.x-e.x), (pr.y-e.y)) < 200
);
if (!nearestPrint) e.revealed = true;
```

### Aliado Temporário

Quando o player tem power-up `ally` ativo, haters são atraídos para o aliado em vez do player:

```javascript
if (L.ally && e.type === 'hater') {
  const ddx = L.ally.x - e.x, ddy = L.ally.y - e.y;
  const d = Math.hypot(ddx, ddy);
  e.x += (ddx/d) * 70 * dt;
  e.y += (ddy/d) * 70 * dt;
}
```

### Níveis onde aparece

2-2 (×1), 2-3 (×1), 3-1 (×2, invisíveis), 3-2 (×1), 3-3 (×2, 1 invisível)

---

## Projéteis

Referência: `ARQ:1157–1162` (criação), `ARQ:1880–1886` (update), `ARQ:2510–2518` (render)

Criados por Bots e Bosses:

```javascript
{x, y, vx, vy, life}
```

- Causam 20 de dano ao player
- Removidos quando `life <= 0` ou fora da viewport
- Renderizados como ponto magenta com "!" preto

## Resumo de Inimigos

| Tipo | Velocidade | Dano | Especial | Dimensões |
|------|-----------|------|----------|-----------|
| Troll | 90 px/s | 35 (contato) | Patrulha horizontal | 36×36 |
| Bot | Senoidal | 20 (projétil) | Dispara a cada 1.6s | 36×36 |
| Fake | 70 px/s | 35 (contato) | Fica hostil a 220px | 36×40 |
| Hater | 80 px/s | 35 (contato) | Persegue em voo, pode ser invisível | 44×32 |
