# 04 — Telas e Estados

## State Machine

Referência: `ARQ:1661`

O jogo possui **11 estados** (telas), gerenciados por `G.screen`:

```
                    ┌──────────┐
          ┌────────►│  TITLE   │◄──────────────────┐
          │         └────┬─────┘                    │
          │              │ btnPlay                  │ btnMapTitle / btnFinalTitle
          │              ▼                          │
          │         ┌──────────┐                    │
          │    ┌───►│   MAP    │◄───────────────────┤
          │    │    └┬──┬──┬──┬┘                    │
          │    │     │  │  │  │                     │
          │    │     │  │  │  │ btnArchive          │
          │    │     │  │  │  ▼                     │
          │    │     │  │  │ ARCHIVE ───────────────┤
          │    │     │  │  │                        │
          │    │     │  │  │ btnStart / E           │
          │    │     │  │  ▼                        │
          │    │     │  │ INTRO                     │
          │    │     │  │  │ introStart             │
          │    │     │  │  ▼                        │
          │    │     │  │ PLAY ◄──────────────────┐│
          │    │     │  │ │\                       ││
          │    │     │  │ │ \ E (nuvem)            ││
          │    │     │  │ │  ▼                     ││
          │    │     │  │ │ PUZZLE ──(recuar)──────┘│
          │    │     │  │ │                         │
          │    │     │  │ │ res <= 0                │
          │    │     │  │ ▼                         │
          │    │     │  │ OVER ──(btnOverMap)───────┤
          │    │     │  │                          ││
          │    │     │  │ goal reached              │
          │    │     │  ▼                           │
          │    │     │ RESULT ──(resMap)────────────┤
          │    │     │  │                          ││
          │    │     │  │ id === '3-B'             │
          │    │     │  ▼                          │
          │    │     │ WIN ──(btnFinalMap)─────────┤
          │    │     │                             │
          │    │     │ btnHow / btnAbout           │
          │    │     ▼                             │
          │    │    HOW ──(btnHowClose)────────────┤
          │    │    ABOUT ──(btnAboutClose)────────┘
          │    │
          │    └─── (Esc / resRetry / btnRetry)
          │
          └──── (Esc from MAP)
```

## Transições

| De | Para | Gatilho | Linha |
|----|------|---------|-------|
| TITLE | MAP | `btnPlay` click | 2972 |
| TITLE | HOW | `btnHow` click | 2973 |
| TITLE | ABOUT | `btnAbout` click | 2974 |
| MAP | TITLE | `btnMapTitle` click / Esc | 2980, 3010 |
| MAP | INTRO | `pnStart` click / E no nó | 2848–2851, 3018 |
| MAP | ARCHIVE | `btnArchive` click | 2978 |
| INTRO | MAP | `introBack` click / Esc | 2982, 3005 |
| INTRO | PLAY | `introStart` click | 2983 |
| PLAY | PUZZLE | E perto de nuvem/barreira | 2122–2131 |
| PLAY | MAP | Esc | 3003 |
| PLAY | OVER | resiliência <= 0 | 2965–2969 |
| PLAY | RESULT | goal reached + puzzles ok | 2885–2954 |
| PLAY | WIN | goal reached + id === '3-B' | 2949–2951 |
| PUZZLE | PLAY | Puzzle resolvido / Recuar / Esc | 2147, 2184, 3004 |
| RESULT | MAP | `resMap` click | 2963 |
| RESULT | INTRO | `resRetry` / `resNext` click | 2956–2961 |
| OVER | MAP | `btnOverMap` click | 2986 |
| OVER | INTRO | `btnRetry` click | 2985 |
| WIN | MAP | `btnFinalMap` click | 2988 |
| WIN | ARCHIVE | `btnFinalArchive` click | 2989 |
| WIN | TITLE | `btnFinalTitle` click | 2990 |
| HOW | TITLE | `btnHowClose` click / Esc | 2975, 3006 |
| ABOUT | TITLE | `btnAboutClose` click / Esc | 2976, 3006 |
| ARCHIVE | MAP | `btnArchiveBack` click / Esc | 2979, 3007 |

## Overlays HTML

Cada tela é um `<section>` com classe `.overlay`. Apenas uma é visível por vez (`.show`).

Referência: `ARQ:2773–2789`

```javascript
function hideAllOverlays(){
  ['titleScreen','mapScreen','introScreen','puzzleScreen',
   'resultScreen','overScreen','victoryScreen','archiveScreen',
   'howScreen','aboutScreen']
    .forEach(id => { ... el.classList.remove('show') });
}
```

### Telas e IDs HTML

| Tela | ID HTML | Conteúdo |
|------|---------|----------|
| TITLE | `#titleScreen` | Logo, descrição, botões Jogar/Como Jogar/Sobre |
| MAP | `#mapScreen` | Canvas com nós + painel lateral de fase |
| INTRO | `#introScreen` | Card com nome, descrição e objetivos do nível |
| PLAY | (sem overlay) | Canvas renderiza o nível + HUD |
| PUZZLE | `#puzzleScreen` | Card com puzzle (6 tipos possíveis) |
| RESULT | `#resultScreen` | Medalhas, checklist de objetivos, botões |
| OVER | `#overScreen` | Tela de esgotamento emocional |
| WIN | `#victoryScreen` | Tela final com canais de denúncia |
| ARCHIVE | `#archiveScreen` | Grid 4 colunas com 20 cards educativos |
| HOW | `#howScreen` | Modal "Como Jogar" com grid de instruções |
| ABOUT | `#aboutScreen` | Modal "Sobre o Cyberbullying" com leis e dados |
