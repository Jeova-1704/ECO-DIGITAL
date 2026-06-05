# 01 — Visão Geral

## Nome

**Eco Digital: O Labirinto da Reputação** (v2)

## Tipo

Jogo educativo de plataforma 2D + puzzle, single-player, baseado em navegador.

## Público-Alvo

Adolescentes de **12 a 17 anos**, estudantes do ensino fundamental II e médio.

## Propósito Educacional

Ensinar sobre **cyberbullying** — reconhecimento, prevenção e denúncia — através de mecânicas de jogo. O jogador aprende a:

1. **Identificar** ataques digitais vs. fatos neutros
2. **Separar** crítica de agressão
3. **Coletar provas** (prints) antes de bloquear
4. **Conhecer as leis** brasileiras que protegem crianças e adolescentes
5. **Acionar canais reais** de denúncia e apoio
6. **Praticar empatia** ao acolher vítimas

## Contexto Narrativo

O **Espaço Digital** (feed, chat, fórum) está sendo coberto por **Nuvens de Toxicidade**. O jogador atravessa três mundos desarmando ataques, juntando provas e silenciando haters com **denúncia**, não com violência.

### Mundos

| # | Nome | Contexto | Tema |
|---|------|----------|------|
| 1 | Rede Social Pública | Posts, comentários, compartilhamentos | `feed` |
| 2 | Grupo de Mensagens | Conversas privadas, apelidos, exclusão | `chat` |
| 3 | Fórum Anônimo | Anonimato, discurso de ódio, ameaças | `forum` |

Cada mundo tem **3 fases regulares** + **1 chefão** (12 níveis no total).

## Marcos Legais Referenciados

| Lei | Descrição | Onde aparece no jogo |
|-----|-----------|---------------------|
| **ECA** (Estatuto da Criança e do Adolescente) | Proteção integral, inclusive online | Print p8, tela Sobre |
| **Lei 13.185/2015** | Programa de Combate à Intimidação Sistemática (bullying/cyberbullying) | Print p4, puzzle C |
| **Lei 14.811/2024** | Crime hediondo: indução ao suicídio/automutilação contra menores | Print p7, puzzle C |
| **Lei Carolina Dieckmann** (Lei 12.737/2012) | Invasão de dispositivo e divulgação não autorizada | Puzzle C |
| **Código Penal** (art. 158) | Extorsão (sextorsão) | Puzzle C |

## Canais de Denúncia no Jogo

| Canal | Descrição | Onde aparece |
|-------|-----------|-------------|
| **SaferNet Brasil** | Denúncia anônima de crimes digitais | Print p12, tela Vitória |
| **Disque 100** | Direitos Humanos, violência contra crianças/adolescentes | Print p11, tela Vitória |
| **CVV 188** | Apoio emocional 24h, gratuito, sigiloso | Print p13, tela Vitória |
| **Polícia Federal · Ciência** | Crimes cibernéticos (sextorsão, ameaça, exploração) | Print p14, tela Vitória |
| **Conselho Tutelar** | Proteção offline, medidas protetivas | Texto descritivo em prints |

## Dados Estatísticos Citados

| Dado | Fonte | Onde aparece |
|------|-------|-------------|
| 37% dos adolescentes já sofreram cyberbullying | UNICEF | Print p1, tela Sobre |
| 1 em cada 5 crianças passou por violência digital | TIC Kids Online Brasil | Print p2, tela Sobre |

## Formato de Distribuição

- **Arquivo único**: `Eco Digital.html` (~3049 linhas, HTML + CSS + JS inline)
- **Zero dependências externas**: sem CDN, sem frameworks, sem build
- **Funciona offline**: basta abrir no navegador
- **Canvas API nativa**: sem SVG, sem bibliotecas de diagrama

## Características Técnicas

- Viewport virtual: 1600×900 (aspect-ratio 16:9)
- Renderização: `<canvas>` com `image-rendering: pixelated`
- Áudio: Web Audio API sintetizado (sem arquivos de áudio)
- Input: teclado (sem suporte a touch/gamepad na versão atual)
- Persistência: em memória (sem localStorage — dados se perdem ao recarregar)
