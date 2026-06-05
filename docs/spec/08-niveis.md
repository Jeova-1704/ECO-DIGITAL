# 08 — Níveis

## Visão Geral

12 níveis organizados em 3 mundos × 4 fases (3 regulares + 1 boss).

Referência: `ARQ:1329–1616`

## Constantes e Helpers

```javascript
const FLOOR = 820;
function P(x,y,w){ return [x,y,w] }
```

## Formato de Configuração

Cada nível segue esta estrutura:

```javascript
{
  id: 'M-F',           // M=mundo, F=fase (1-3 ou B)
  world: N,            // 1, 2 ou 3
  name: string,        // nome de exibição
  theme: string,       // feed|chat|forum|boss-feed|boss-chat|boss-forum
  isBoss: boolean,     // true para fases boss
  desc: string,        // descrição curta
  width: number,       // largura total do nível (px)
  spawn: [x, y],       // posição inicial do player
  ground: [[x,y,w],...],       // blocos de chão
  platforms: [[x,y,w],...],    // plataformas flutuantes (posts)
  clouds: [{x,y,w,h,pz,data}], // nuvens de toxicidade
  enemies: [{type,x,y,...}],    // inimigos
  obstacles: [{type,...}],       // obstáculos
  prints: [[x,y,archiveId],...],// prints colecionáveis
  powerups: [{type,x,y}],       // power-ups
  goalX: number,       // posição X da saída
  objectives: {
    puzzles: N,         // puzzles mínimos para completar
    prints: N,          // prints para medalha de ouro
    dontLoseRes: bool,  // resiliência >= 50% para prata
    timeLimit: N        // 0 = sem limite, N = segundos
  },
  boss: {kind,hp,name} // apenas para fases boss
}
```

---

## MUNDO 1 — Rede Social Pública (Feed)

### 1-1: Comentários Tóxicos

| Propriedade | Valor |
|-------------|-------|
| Tema | `feed` |
| Largura | 4400px |
| Spawn | [100, 740] |
| GoalX | 4280 |
| Objetivos | 2 puzzles, 3 prints, dontLoseRes, sem timer |

**Ground:**
| X | Y | W |
|---|---|---|
| 0 | 820 | 700 |
| 820 | 820 | 520 |
| 1480 | 820 | 460 |
| 2080 | 820 | 600 |
| 2820 | 820 | 520 |
| 3460 | 820 | 940 |

**Platforms (14):** 420,640,160 · 700,540,140 · 960,500,160 · 1240,440,160 · 1560,560,160 · 1840,500,160 · 2200,480,140 · 2400,380,160 · 2660,480,160 · 2980,540,160 · 3220,460,140 · 3500,540,160 · 3760,460,140 · 4040,500,160

**Clouds (2):**
| # | X | Y | W | H | Puzzle | Data |
|---|---|---|---|---|--------|------|
| 1 | 720 | 680 | 110 | 130 | A (Rede de Apoio) | — |
| 2 | 1380 | 640 | 110 | 160 | B (Frase) | 0 |

**Enemies (2):** Troll x=900, Troll x=2300

**Prints (3):** p3 [320,760], p5 [760,580], p15 [1300,420]

---

### 1-2: Compartilhamento em Massa

| Propriedade | Valor |
|-------------|-------|
| Largura | 4800px |
| Spawn | [100, 740] |
| GoalX | 4660 |
| Objetivos | 3 puzzles, 4 prints, dontLoseRes, sem timer |

**Clouds (3):** B(data:1) x=1080, C(data:0) x=2300, A x=3380

**Enemies (3):** Troll x=1500, Bot x=1800/y=300, Troll x=2900

**Obstacles (1):** Glitch x=2580 (desbloqueia ao resolver cloud 1)

**Prints (4):** p4, p6, p1, p2

**Powerups (1):** Shield x=920

---

### 1-3: Viralização

| Propriedade | Valor |
|-------------|-------|
| Largura | 5000px |
| Spawn | [100, 740] |
| GoalX | 4700 |
| Objetivos | 3 puzzles, 4 prints, dontLoseRes, **timer: 90s** |

**Clouds (3):** B(data:2) x=1500, D(data:0) x=2500, E(data:0) x=3500

**Enemies (3):** Bot x=900/y=420, Bot x=2100/y=380, Troll x=3300

**Obstacles (6):** Unstable platforms em x=700,1180,1680,2180,2680,3180

**Prints (4):** p18, p3, p5, p15

**Powerups (1):** Freeze x=1300

---

### 1-B: Chefão · O Perfil Fake

| Propriedade | Valor |
|-------------|-------|
| Largura | 2200px |
| Spawn | [100, 740] |
| GoalX | 2100 |
| Objetivos | 1 puzzle, 3 prints, dontLoseRes=false, sem timer |
| Boss | kind: fake, hp: 3 |

**Ground:** um bloco só [0,820,2200]

**Clouds (1):** C(data:1, bossFinal:true) x=1900 — nuvem final do boss

**Prints (3):** p9, p5, p11

**Powerups (2):** Shield x=900, Freeze x=1400

---

## MUNDO 2 — Grupo de Mensagens (Chat)

### 2-1: Apelidos Cruéis

| Propriedade | Valor |
|-------------|-------|
| Largura | 4600px |
| Objetivos | 3 puzzles, 4 prints, dontLoseRes, sem timer |

**Clouds (3):** B(data:3) x=1080, B(data:0) x=2140, A x=3060

**Enemies (3):** Fake x=900, Troll x=1700, Bot x=2400/y=360

**Obstacles (1):** Falling x=1500, period=1.6s, width=250

**Powerups (1):** Shield x=680

---

### 2-2: Exclusão do Grupo

| Propriedade | Valor |
|-------------|-------|
| Largura | 4800px |
| Objetivos | 3 puzzles, 4 prints, dontLoseRes, **timer: 75s** |

**Clouds (3):** E(data:1) x=1400, F(data:0) x=2400, D(data:0) x=3400

**Enemies (2):** Fake x=1900, Hater x=2900/y=360 range=500

**Obstacles (7):** 6× Unstable (x=640..3140), 1× Silence x=2000 w=600 h=300

**Powerups (1):** Ally x=760

---

### 2-3: Prints Vazados

| Propriedade | Valor |
|-------------|-------|
| Largura | 5000px |
| Objetivos | 4 puzzles, 5 prints, dontLoseRes, sem timer |

**Clouds (4):** C(data:0) x=1080, F(data:1) x=2120, D(data:0) x=3060, B(data:1) x=3960

**Enemies (3):** Bot x=1500/y=380, Hater x=2600/y=320 range=480, Fake x=3380

**Obstacles (1):** Glitch x=2540 (key: c1)

**Powerups (1):** Gold x=1700

---

### 2-B: Chefão · Administrador Omisso

| Propriedade | Valor |
|-------------|-------|
| Largura | 2400px |
| Objetivos | 1 puzzle, 4 prints, dontLoseRes=false, sem timer |
| Boss | kind: admin, hp: 4 |

**Clouds (1):** D(data:0, bossFinal:true) x=2080

**Obstacles (1):** Silence x=600 w=600 h=300

**Prints (4):** p5, p11, p12, p14

**Powerups (2):** Freeze x=900, Shield x=1400

---

## MUNDO 3 — Fórum Anônimo

### 3-1: Anonimato Tóxico

| Propriedade | Valor |
|-------------|-------|
| Largura | 4800px |
| Objetivos | 3 puzzles, 4 prints, dontLoseRes, sem timer |

**Clouds (3):** A x=1080, C(data:2) x=2140, F(data:0) x=3060

**Enemies (3):** Hater x=1500/y=360 invisibleUntil:print, Hater x=2600/y=320 invisibleUntil:print, Fake x=3380

**Obstacles (1):** Mirror x=1700 w=600

**Powerups (2):** Shield x=700, Freeze x=2400

---

### 3-2: Discurso de Ódio

| Propriedade | Valor |
|-------------|-------|
| Largura | 5000px |
| Objetivos | 4 puzzles, 5 prints, dontLoseRes, sem timer |

**Clouds (4):** E(data:0) x=1080, B(data:2) x=2120, A x=3060, D(data:0) x=3960

**Enemies (4):** Troll x=1500, Hater x=2600/y=320, Fake x=3380, Bot x=4200/y=380

**Obstacles (2):** Silence x=1900, Falling x=2800

**Powerups (2):** Ally x=1500, Gold x=3700

---

### 3-3: Sextorsão e Ameaças

| Propriedade | Valor |
|-------------|-------|
| Largura | 5200px |
| Objetivos | 4 puzzles, 6 prints, dontLoseRes, sem timer |

**Clouds (4):** C(data:2) x=1080, D(data:0) x=2120, E(data:1) x=3060, F(data:1) x=3960

**Enemies (4):** Troll x=1500, Hater x=2600/y=320 invisibleUntil:print, Fake x=3380, Hater x=4200/y=340 invisibleUntil:print

**Obstacles (3):** Silence x=1500, Falling x=3000, Glitch x=4400 (key: c3)

**Powerups (2):** Shield x=800, Gold x=3200

---

### 3-B: Chefão · O Hater Coletivo

| Propriedade | Valor |
|-------------|-------|
| Largura | 2600px |
| Objetivos | 1 puzzle, 5 prints, dontLoseRes=false, sem timer |
| Boss | kind: hater, hp: 5 |

**Clouds (1):** C(data:2, bossFinal:true) x=2300

**Prints (5):** p7, p11, p12, p13, p14

**Powerups (3):** Shield x=700, Freeze x=1300, Ally x=1700

---

## Resumo de Dificuldade

| Nível | Puzzles | Prints | Timer | Inimigos | Obstáculos especiais |
|-------|---------|--------|-------|----------|---------------------|
| 1-1 | 2 | 3 | — | 2 Trolls | — |
| 1-2 | 3 | 4 | — | 1 Troll + 1 Bot | Glitch |
| 1-3 | 3 | 4 | 90s | 2 Bots + 1 Troll | 6× Unstable |
| 1-B | 1 | 3 | — | Boss | — |
| 2-1 | 3 | 4 | — | Fake + Troll + Bot | Falling |
| 2-2 | 3 | 4 | 75s | Fake + Hater | 6× Unstable + Silence |
| 2-3 | 4 | 5 | — | Bot + Hater + Fake | Glitch |
| 2-B | 1 | 4 | — | Boss | Silence |
| 3-1 | 3 | 4 | — | 2 Haters (invisíveis) + Fake | Mirror |
| 3-2 | 4 | 5 | — | Troll + Hater + Fake + Bot | Silence + Falling |
| 3-3 | 4 | 6 | — | Troll + 2 Haters (invisíveis) + Fake | Silence + Falling + Glitch |
| 3-B | 1 | 5 | — | Boss | — |
