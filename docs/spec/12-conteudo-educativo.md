# 12 — Conteúdo Educativo

## Acervo de Provas (Archive)

Referência: `ARQ:683–725`

20 prints colecionáveis que desbloqueiam cards educativos no Acervo. Cada print está associado a um `archiveId` (p1–p20).

### Prints Completos

| ID | Título | Tag | Conteúdo |
|----|--------|-----|----------|
| p1 | 37% no Brasil | DADO · UNICEF | Cerca de 37% dos adolescentes brasileiros já sofreram cyberbullying segundo a UNICEF. Não é raro — é estatística pública. |
| p2 | 1 em cada 5 crianças | DADO · TIC KIDS | O TIC Kids Online Brasil aponta que 1 em cada 5 crianças e adolescentes já passou por alguma forma de violência digital. |
| p3 | Crítica não é ataque | CONCEITO | Crítica fala de fato ("você atrasou três vezes"). Ataque fala da pessoa ("você é um lixo"). Os dois soam parecidos — separar é o primeiro passo. |
| p4 | Lei 13.185/2015 | LEI | Programa de Combate à Intimidação Sistemática: define bullying e cyberbullying e obriga escolas a prevenir e intervir. |
| p5 | Print é prova | TÁTICA | Antes de bloquear, tire print da ofensa, com data e perfil visíveis. Sem prova, a denúncia perde força. |
| p6 | Não responda no ódio | TÁTICA | Responder a um ataque com outro alimenta a Nuvem. Bloqueie, guarde provas e leve para um adulto de confiança. |
| p7 | Lei 14.811/2024 | LEI | Tornou hediondo o crime de induzir/auxiliar suicídio ou automutilação contra crianças e adolescentes, e reforçou a proteção em escolas e redes. |
| p8 | ECA | LEI | O Estatuto da Criança e do Adolescente garante proteção integral — inclusive online. Família, escola e Estado têm dever legal de agir. |
| p9 | Perfil fake é crime | CONCEITO | Criar perfil falso para humilhar ou ameaçar pode caracterizar falsa identidade, injúria, calúnia ou ameaça. Não é "só uma brincadeira". |
| p10 | Vazamento sem consentimento | CONCEITO | Compartilhar foto, áudio ou conversa íntima de alguém sem permissão é crime previsto na Lei Carolina Dieckmann e no ECA — e abre processo civil por danos. |
| p11 | Disque 100 | CANAL | Direitos Humanos. Recebe denúncias de violência contra crianças e adolescentes — inclusive cyberbullying e sextorsão. Funciona 24h. |
| p12 | SaferNet | CANAL | Denúncia anônima de crimes na internet em new.safernet.org.br/denuncie. Também oferece canal de ajuda e orientação para vítimas. |
| p13 | CVV 188 | CANAL | Centro de Valorização da Vida. Apoio emocional 24h, gratuito, sigiloso. Para você ou para alguém que precisa de escuta agora. |
| p14 | Polícia Federal · Ciência | CANAL | A PF mantém canal específico para crimes cibernéticos: sextorsão, ameaça, exploração sexual, vazamento. Procure gov.br/pf. |
| p15 | Rede de apoio | CONCEITO | Você não resolve sozinho. A ordem que funciona: vítima → testemunha → moderador da plataforma → responsável offline (família, escola, conselho tutelar). |
| p16 | Anonimato online não te protege | CONCEITO | Plataformas guardam IP, logs e metadados. Investigações cibernéticas identificam autores mesmo em fóruns anônimos quando há denúncia formal. |
| p17 | Sextorsão | ALERTA | Se alguém ameaça vazar foto íntima sua, NÃO pague, NÃO mande mais nada. Guarde provas, bloqueie e denuncie em SaferNet/PF imediatamente. |
| p18 | Depoimento · Beatriz, 14 | HISTÓRIA | "Quando criaram um perfil zoando minha foto, eu queria sumir. Foi a coordenadora da escola que sentou comigo, tirou print de tudo e ligou para o conselho tutelar. Passou." |
| p19 | Empatia primeiro | CONCEITO | Antes de "tinha que fazer X", escute. "Eu acredito em você" e "isso não é culpa sua" pesam mais que qualquer estratégia. |
| p20 | Acionar adulto não é "X9" | CONCEITO | Pedir ajuda a um adulto de confiança não é dedurar — é estratégia. Ódio digital é grande demais para uma pessoa sozinha enfrentar. |

---

## Distribuição de Prints por Nível

| Nível | Prints |
|-------|--------|
| 1-1 | p3, p5, p15 |
| 1-2 | p4, p6, p1, p2 |
| 1-3 | p18, p3, p5, p15 |
| 1-B | p9, p5, p11 |
| 2-1 | p9, p10, p6, p19 |
| 2-2 | p19, p20, p15, p13 |
| 2-3 | p10, p9, p17, p7, p11 |
| 2-B | p5, p11, p12, p14 |
| 3-1 | p16, p9, p3, p6 |
| 3-2 | p4, p19, p2, p7, p20 |
| 3-3 | p7, p17, p14, p11, p13, p8 |
| 3-B | p7, p11, p12, p13, p14 |

### Contagem por Tipo

| Tipo | Quantidade | IDs |
|------|-----------|-----|
| CONCEITO | 6 | p3, p9, p10, p15, p16, p19, p20 |
| LEI | 3 | p4, p7, p8 |
| TÁTICA | 2 | p5, p6 |
| CANAL | 4 | p11, p12, p13, p14 |
| DADO | 2 | p1, p2 |
| ALERTA | 1 | p17 |
| HISTÓRIA | 1 | p18 |

---

## Desbloqueio de Archive

Referência: `ARQ:727–731`

```javascript
function unlockArchive(id){
  if (SAVE.archiveSeen.has(id)) return;
  SAVE.archiveSeen.add(id);
  toast(`Acervo desbloqueado: ${ARCHIVE.find(a=>a.id===id)?.title || id}`, 'good');
}
```

Quando um print é coletado no nível (`ARQ:1920–1931`):
1. Verifica se é novo: `!SAVE.collectedPrints.has(pr.id)`
2. Adiciona ao set global: `SAVE.collectedPrints.add(pr.id)`
3. Incrementa total: `SAVE.totalPrints += 1`
4. Chama `unlockArchive(pr.archive)` para desbloquear o card

### Renderização do Archive

Referência: `ARQ:2870–2882`

Grid de 4 colunas com cards. Cards não desbloqueados:
- Classe `.locked` (35% opacidade)
- Título prefixado com "🔒"
- Descrição genérica: "Colete o Print correspondente nas fases para revelar este conteúdo."

---

## Canais de Denúncia (Tela de Vitória)

Referência: `ARQ:512–516`

A tela final (WIN) mostra 4 canais:

| Canal | Descrição | Contato |
|-------|-----------|---------|
| SaferNet Brasil | Denúncia anônima de crimes digitais | new.safernet.org.br/denuncie |
| Disque 100 | Direitos Humanos. Violência contra crianças e adolescentes | Ligue 100 |
| CVV — Valorização da Vida | Apoio emocional 24h, gratuito e sigiloso | Ligue 188 · cvv.org.br |
| Polícia Federal · Ciência | Canal para crimes cibernéticos | gov.br/pf — Cibernético |

---

## Conteúdo da Tela "Sobre o Cyberbullying"

Referência: `ARQ:583–605`

Texto principal:
> Cyberbullying é a agressão repetida no ambiente digital — em redes sociais, chats e fóruns. Ele se espalha rápido e atinge a vítima 24h por dia. No Brasil, dados da UNICEF apontam que cerca de **37% dos adolescentes** já passaram por isso, e o TIC Kids Online Brasil mostra que **1 em cada 5 crianças** já vivenciou alguma forma de violência digital.

Cards informativos:

| Tema | Conteúdo |
|------|----------|
| ECA | Garante proteção integral. Dever da família, sociedade e Estado. |
| Lei 13.185/2015 | Programa de Combate à Intimidação Sistemática. Obriga escolas a intervir. |
| Lei 14.811/2024 | Crime hediondo: indução ao suicídio/automutilação contra menores. |
| Onde denunciar | SaferNet, Disque 100, CVV 188, Conselho Tutelar, PF. |
