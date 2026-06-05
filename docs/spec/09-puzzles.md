# 09 — Puzzles

## Visão Geral

O jogo tem **6 tipos de puzzle** (A–F), cada um com construtor dinâmico e verificação de resposta. Todos aparecem no "Painel de Reparação" (overlay `#puzzleScreen`).

Referência: `ARQ:820–1124`

## Host do Puzzle

Referência: `ARQ:2173–2186`

```javascript
const PuzzleHost = {
  _check: null,
  onCheck(fn){ this._check = fn; }
};
```

- `PuzzleHost.onCheck(fn)`: cada puzzle registra sua função de verificação
- `$('#pzCheck')` (botão "Verificar") chama `PuzzleHost._check()`
- Cada `build(host, ctx, onResult)` recebe:
  - `host`: `$('#pzBody')` — container DOM do puzzle
  - `ctx`: `{data, id, world}` — dados de contexto
  - `onResult(success, info)`: callback com resultado

### Resultado

- `onResult(true, {perfect: true})` → puzzle resolvido, nuvem desarmada
- `onResult(false, {penalty: 6})` → erro, perde 6 de resiliência

---

## A — Conectar Nós de Comunicação

Referência: `ARQ:824–885`

**Objetivo**: Ordenar 4 nós na sequência correta da rede de apoio.

### Nós (fixos)

| Índice | Role | Nome | Descrição |
|--------|------|------|-----------|
| 0 | PESSOA ATINGIDA | Vítima | Quem está sofrendo o ataque. Reconhecer é o primeiro passo. |
| 1 | QUEM VÊ | Testemunha | Amigo, colega ou seguidor. Apoia e não espalha a ofensa. |
| 2 | NA PLATAFORMA | Moderador | Equipe do app/rede. Recebe denúncia, remove conteúdo, bloqueia. |
| 3 | OFFLINE | Responsável | Família, escola, conselho tutelar — agem na vida real e protegem. |

### Ordem Correta

`[0, 1, 2, 3]` — Vítima → Testemunha → Moderador → Responsável

### Mecânica

- 4 cards posicionados aleatoriamente (posição embaralhada com `sort(Math.random()-0.5)`)
- Player clica em sequência — cada clique adiciona ao array `order`
- Clicar em nó já selecionado remove ele e todos posteriores
- SVG wires são desenhados entre nós selecionados
- Verificação: compara array `order` com `[0,1,2,3]`
- Erro: highlight no primeiro errado + shake + reset

---

## B — Desconstruir a Frase Ofensiva

Referência: `ARQ:902–954`

**Objetivo**: Classificar cada palavra como ATAQUE ou FATO.

### Frases (PHRASES_B)

Referência: `ARQ:739–756`

| # | Situação | Texto |
|---|----------|-------|
| 0 | No feed da turma | "Ninguém te aguenta nessa sala, vi seu nome na lista do trabalho." |
| 1 | Um colega comenta | "Olha o nojento ali, tirou 7 na prova de matemática." |
| 2 | No story de uma colega | "Some daqui idiota, a foto que postou tem 12 curtidas." |
| 3 | Em um grupo do WhatsApp | "Tá feia hoje hein, sua aula começa às 8." |

### Formato das Palavras

Cada palavra: `"texto:classe"` onde classe é `a` (ataque) ou `f` (fato).

**Exemplo (frase 0):**
| Palavra | Classe Real |
|---------|------------|
| Ninguém | ataque |
| te | ataque |
| aguenta | ataque |
| nessa, | ataque |
| sala, | ataque |
| vi | fato |
| seu | fato |
| nome | fato |
| na | fato |
| lista | fato |
| do | fato |
| trabalho. | fato |

### Mecânica

- Clique alterna: `neutral` → `ataque` → `fato` → `neutral`
- Visual: ataque = fundo magenta com line-through; fato = fundo verde
- Verificação: todas devem estar classificadas (zero neutral) e corretas
- Erro: shake nas palavras erradas + contagem de erros

---

## C — Identificar a Lei

Referência: `ARQ:956–990`

**Objetivo**: Escolher a lei brasileira aplicável à situação.

### Questões (LEI_QS)

Referência: `ARQ:758–783`

| # | Situação | Lei Correta |
|---|----------|-------------|
| 0 | Vazaram prints de conversa privada sem consentimento | Lei Carolina Dieckmann (12.737/2012) |
| 1 | Alunos humilham outro em comentários, escola não age | Lei 13.185/2015 |
| 2 | Agressor ameaça vazar foto íntima se não receber dinheiro | Lei 14.811/2024 + Código Penal |
| 3 | Colega de 13 anos sofre humilhação constante em apps | ECA |

### Opções por Questão

Cada questão tem 3 opções (1 correta + 2 distratores). Exemplo (questão 0):

| Opção | Lei | Descrição | Correta? |
|-------|-----|-----------|----------|
| A | Lei Carolina Dieckmann (12.737/2012) | Invasão de dispositivo e divulgação não autorizada | Sim |
| B | Código de Defesa do Consumidor | Relações de consumo | Não |
| C | Lei Maria da Penha | Violência doméstica | Não |

### Mecânica

- Seleção simples (radio-style)
- Verificação: se correta, "LEI CORRETA"; se errada, highlight vermelho + "LEI INCORRETA"

---

## D — Ordem da Denúncia

Referência: `ARQ:992–1041`

**Objetivo**: Ordenar 5 passos da denúncia na sequência correta.

### Passos (ORDER_QS)

Referência: `ARQ:785–794`

| Índice | Passo |
|--------|-------|
| 0 | Tirar print da ofensa |
| 1 | Bloquear o agressor |
| 2 | Denunciar dentro da plataforma |
| 3 | Contar para um responsável (família/escola) |
| 4 | Acionar canal externo (SaferNet, Disque 100, PF) |

### Mecânica

- Steps embaralhados visualmente
- Clique em sequência para numerar
- Clique em step já numerado: remove ele e posteriores
- Verificação: `order.every((v,i) => v === i)`
- Erro: shake no primeiro errado + reset

---

## E — Empatia em Diálogo

Referência: `ARQ:1043–1076`

**Objetivo**: Escolher a resposta mais empática para acolher a vítima.

### Questões (EMPATIA_QS)

Referência: `ARQ:796–809`

| # | Situação | Resposta Correta |
|---|----------|-----------------|
| 0 | Amiga chorando conta que sofre cyberbullying | "Eu acredito em você. Não é sua culpa. Vamos guardar prints e procurar alguém para te ajudar." |
| 1 | Colega se isolou depois de virar alvo de zoeira | "Se quiser conversar, tô aqui. Sem pressão, no seu tempo." |

### Opções por Questão

Cada questão tem 3 opções com classificações:

| Tipo | Descrição |
|------|-----------|
| `empathic` | Resposta correta — acolhe, valida e propõe ação |
| `minimize` | Minimiza o sofrimento ("é só piada", "todo mundo passa") |
| `neutral` | Bem-intencionada mas evasiva ou decides pelo outro |

---

## F — Reconstruir Mensagem de Apoio

Referência: `ARQ:1078–1124`

**Objetivo**: Montar uma mensagem positiva com palavras corretas, excluindo distratores.

### Mensagens (MSG_RECON)

Referência: `ARQ:811–818`

| # | Mensagem Alvo | Distratores |
|---|---------------|-------------|
| 0 | "Você não está sozinho, eu acredito em você." | sozinho, idiota, culpa |
| 1 | "Isso não é sua culpa, vamos pedir ajuda juntos." | drama, para, sempre |

### Mecânica

- Dois containers: "SUA MENSAGEM" (target) e "PALAVRAS DISPONÍVEIS" (bank)
- Clique em palavra do bank → move para target
- Clique em palavra do target → remove de volta para bank
- A ordem das palavras no target importa
- Verificação: `built === q.target` (string exata)
- Palavras embaralhadas com `sort(Math.random()-0.5)`

---

## Distribuição de Puzzles por Nível

| Nível | Cloud 1 | Cloud 2 | Cloud 3 | Cloud 4 |
|-------|---------|---------|---------|---------|
| 1-1 | A | B(0) | — | — |
| 1-2 | B(1) | C(0) | A | — |
| 1-3 | B(2) | D(0) | E(0) | — |
| 1-B | C(1) | — | — | — |
| 2-1 | B(3) | B(0) | A | — |
| 2-2 | E(1) | F(0) | D(0) | — |
| 2-3 | C(0) | F(1) | D(0) | B(1) |
| 2-B | D(0) | — | — | — |
| 3-1 | A | C(2) | F(0) | — |
| 3-2 | E(0) | B(2) | A | D(0) |
| 3-3 | C(2) | D(0) | E(1) | F(1) |
| 3-B | C(2) | — | — | — |

O número entre parênteses é o índice `data` (seleciona variação do puzzle).
