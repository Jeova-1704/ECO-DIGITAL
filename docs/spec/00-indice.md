# Eco Digital — Especificação Técnica Completa

> **OpenSpec v1.0** — extraído do código-fonte `Eco Digital.html` (3049 linhas)
> Gerado em: junho de 2026

---

## Índice de Documentos

| # | Arquivo | Descrição |
|---|---------|-----------|
| 01 | [Visão Geral](01-visao-geral.md) | Propósito, público-alvo, contexto educativo, referências legais |
| 02 | [Arquitetura](02-arquitetura.md) | Estrutura single-file, organização do JS, dependências, IIFE |
| 03 | [Engine](03-engine.md) | Game loop, física, colisão, câmera, transformação Canvas |
| 04 | [Telas e Estados](04-telas-e-estados.md) | State machine com 10 telas, transições, navegação |
| 05 | [Sistemas](05-sistemas.md) | Save, Audio (Web Audio API), HUD, Toast |
| 06 | [Mapa-Múndi](06-mapa-mundi.md) | World map: nós, paths, avatar, painel de fase |
| 07 | [Mecânicas](07-mecanicas.md) | Resiliência, medalhas, progressão, controles |
| 08 | [Níveis](08-niveis.md) | 12 níveis com configurações completas (ground, platforms, clouds, enemies...) |
| 09 | [Puzzles](09-puzzles.md) | 6 tipos com dados completos (frases, leis, ordens, empatia...) |
| 10 | [Inimigos](10-inimigos.md) | 4 tipos: comportamento, IA, renderização |
| 11 | [Power-ups e Obstáculos](11-powerups-e-obstaculos.md) | 4 power-ups + 5 tipos de obstáculo |
| 12 | [Conteúdo Educativo](12-conteudo-educativo.md) | 20 prints do acervo, leis, canais de denúncia, estatísticas |
| 13 | [Renderização](13-renderizacao.md) | Pipeline Canvas, temas visuais, efeitos (parallax, glitch) |
| 14 | [Paleta Visual](14-paleta-visual.md) | Design tokens, CSS variables, tipografia, estilos |
| 15 | [UI/UX](15-ui-ux.md) | Overlays HTML, responsividade, acessibilidade, teclado |

---

## Convenções

- **Referências de linha**: `ARQ:1234` indica a linha 1234 de `Eco Digital.html`
- **IDs de nível**: formato `MUNDO-FASE` (ex: `1-1`, `2-B` para boss)
- **VW/VH**: viewport virtual do canvas — 1600×900 pixels lógicos
- **FLOOR**: constante Y=820, linha do chão padrão
- **dt**: delta time em segundos (clampado a 0.033 = ~30fps mínimo)
