---
schema_version: "0.5"
class: playbook
adapta:
  - memory 0.26
  - flow 0.15
  - player 0.7
status: draft-pre-validacao
data: 2026-07-28
---

# Método Keelson — Playbook de migração

> **Quando usar:** você já tem um projeto que roda — código, docs espalhados, um Tier 0 que provavelmente já
> virou monólito — e quer trazê-lo para o método **sem quebrar o que funciona**. Se o projeto é novo, este
> não é o seu doc (use o prompt de bootstrap). A diferença é de natureza: greenfield *cria do nada*;
> migração *classifica o que já existe* e encosta no destrutivo (mover arquivo, quebrar link). O critério de
> sucesso também muda — aqui é **"não quebrar"**, não "criar".
>
> Agnóstico a ferramenta. Onde um passo tem forma específica (ex.: *como* se enxuga o Tier 0 no Claude
> Code), o detalhe vive no *application guide* — ver [`llm-dev-README.md`](llm-dev-README.md).

## A diretriz-mãe: incremental, não big-bang

O modo de falha nº 1 é abrir o `architecture.md` de 1128 linhas e tentar reclassificar tudo numa sessão —
isso **trava a migração antes de começar**. Em vez disso: extraia agora só os ganhos óbvios, deixe o resto
onde está, e garanta que **todo conteúdo novo já nasça no lugar certo**. O monólito é drenado aos poucos,
seção por seção, só quando ela for tocada por outro motivo. Migração é um **gradiente, não um evento**.

## Antes de tocar em nada: rede de segurança + baseline

1. **Trabalhe numa branch.** Mover arquivo é reversível via git; a migração inteira é *candidata* até o
   merge (regra "candidato na branch / canônico na main" — ver [`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md),
   "Concorrência").
2. **Capture o baseline ANTES de migrar.** O primeiro snapshot (`wiki/metrics/AAAA-MM-DD-metric.md`) tem de
   refletir o estado *pré-método* — é contra ele que se mede se a migração melhorou custo/continuidade.
   Baseline coletado depois de já ter mexido não mede nada. (Ver `llm-dev-memory-machinery.md`, "Instrumentação"; o
   baseline é o snapshot de **data mais antiga**.)

## O trabalho é classificar, não criar

Migrar é, no fundo, olhar cada artefato existente e perguntar "que balde do método é esse?". Triagem:

| Artefato existente | Sinal | Vai para |
|---|---|---|
| Monólito de arquitetura (estado + história misturados) | edita in-place **e** acumula narrativa datada | separar: current-state → `docs/architecture.md`; decisões → ADR; incidentes → `wiki/log/` |
| Uma seção "decidimos X porque Y", citável | um porquê que o time **possui** | `docs/decisions/NNNN-*.md` (ADR) |
| Backlog / lista de tarefas | o que ainda vai ser feito | `docs/specs/` (+ `backlog.md`) |
| Lista de bugs conhecidos / known issues / limitações EM TRATAMENTO | defeito ainda aberto no sistema vivo | `wiki/known-issues.md` (ponteiro transiente — sai quando resolve; o aprendizado durável vai para ADR/`invariants.md`/`log/`) |
| Postmortem / registro de incidente já resolvido | história datada de um bug | `wiki/log/` (o incidente) + ADR (a lição, se merece não-repetir) |
| Relatório externo / auditoria não digerida | fonte primária ainda crua | `docs/reports/` |
| Regulamento / norma / legislação externa | autoridade **de fora** do projeto | `docs/domain/` (fonte crua em `source/`) |
| Roteiro operacional multi-passo (deploy, rotação de chave, restart de serviço) + comandos "de manobra" que hoje incham o Tier 0 | manobra **imperativa** que se *executa*, ancorada numa **operação** | `docs/runbooks/` — o Tier 0 guarda só o **comando-do-dia atômico** (`npm test`) e **aponta** para a manobra multi-passo. Troubleshooting ancorado num *subsistema* vai como satélite de `architecture.md`, não aqui |
| Doc que o **código carrega em runtime** (base RAG, template de prompt, schema) | há um `require`/path apontando pra ele | **NÃO move** — indexa no `wiki/index.md` onde está (exceção da fronteira código↔doc) |
| Notas ad-hoc já existentes (ex.: memória do agente) | conhecimento solto | migra/referencia — **não recria do zero** |

## Receita, em fases

- **A — Esqueleto, sem mover conteúdo.** **Vendorizar o pacote do método em `.keelson/`** na raiz do repo
  (cópia pinada — ver [`llm-dev-package.md`](llm-dev-package.md)); é o irmão do `.claude/`, e é para onde o
  Tier 0 vai apontar por path relativo. Criar `wiki/` (`index.md`, `now/`, `log/`, `known-issues.md` vazio) e
  as pastas de `docs/` (incluindo `runbooks/`, o balde de manobra operacional, e `fixes/`, o balde corretivo —
  ambos nascem vazios); **enxugar o Tier 0** para uma semente (ponteiros +
  proibições), tirando dele o conhecimento profundo que já cabe no `index.md`. Essa semente aponta em **duas
  direções**: para o conhecimento do projeto (via `index.md`) e para a **trilogia do método** em `.keelson/`
  (o "segundo roteamento" de [`llm-dev-memory.md`](llm-dev-memory.md), "O que faz um bom Tier 0"). A partir
  daqui, conteúdo novo nasce no lugar certo — que é metade da vitória.
  **O enxugue do Tier 0 é ele próprio uma mini-migração — cobertura antes do corte.** Um Tier 0 battle-hardened
  de centenas de linhas não se poda a olho: é a mesma disciplina de retirar um doc redundante (o funil do
  AGENTS.md), com o mesmo risco de perda silenciosa. Passo a passo:
  1. **Greppe cada fato** do Tier 0 contra o destino candidato — o **comando-do-dia atômico** *fica*; a
     **manobra multi-passo** vai para `runbooks/`; o "porquê" citável vira **ADR**; a regra dura vira
     `invariants.md`; o conhecimento profundo já está (ou vai) no `index.md`/`architecture.md`.
  2. **O não-coberto migra primeiro.** Só se corta do Tier 0 o que já tem casa **confirmada** no destino. Nada
     sai antes de estar coberto — cortar sem conferir cobertura é perder no único arquivo cuja falta ninguém
     nota na hora.
  3. **Só então corta**, deixando no Tier 0 o ponteiro de uma linha. Fim: a semente é defaults + proibições +
     ponteiros, e passa no teste de inclusão de [`llm-dev-memory.md`](llm-dev-memory.md) ("O que faz um bom
     Tier 0").
- **B — Colher os ADRs óbvios.** Extrair do monólito só as 2–3 decisões já claras (no OptiFlux: telemetria
  PUSH-only; RETIRE vs CANCEL) para `docs/decisions/`. Deixar o resto do arquivo como está.
- **C — Mover o barato e seguro.** `git mv` dos arquivos claramente mal-colocados (backlog, reports),
  **conferindo referências de entrada antes** (grep pelo nome do arquivo: nada aponta pra ele por path de
  código?). `git mv` preserva a história.
- **D — Indexar as fontes-de-runtime onde estão.** Os docs que o código `require`-a **não migram**; ganham
  entrada no `index.md` marcando o acoplamento explicitamente.
- **E — Drenar o monólito por uso.** Daqui pra frente, cada vez que uma seção do arquivo velho é tocada por
  outro motivo, ela é movida pro lugar certo. Nunca uma força-tarefa de reescrita.

## O que NÃO fazer

- Não reescrever o `architecture.md` inteiro de uma vez (trava tudo — a diretriz-mãe existe por isso).
- Não mover fonte-que-é-runtime "por estética de organização" — quebra o caminho que o código espera.
- Não renomear/mover sem **grepar as referências de entrada** primeiro.
- Não coletar o baseline **depois** de já ter migrado.
- Não tratar a migração como projeto com data de fim — ela termina quando o monólito secou, e isso é gradual.

## Papéis humano / LLM na migração

A **classificação** exige julgamento humano (esta seção é decisão ou current-state? este doc é fonte de
runtime?). O **LLM executa**: os moves, os `git mv`, os rascunhos de ADR, a atualização do `index.md`. O
**gatilho e a decisão** são do humano — como em todo o método, o LLM não puxa o próprio gatilho entre
sessões (ver [`llm-dev-player.md`](llm-dev-player.md)).

## Versionamento e histórico

A versão corrente está no frontmatter (`schema_version`). O **histórico de mudanças** deste documento vive
fora dele — em [`changelog/llm-dev-migration.md`](changelog/llm-dev-migration.md) —, para o padrão ficar limpo
como artefato de distribuição. O esquema de numeração (0.x → 1.0 → 2.0) e o índice geral estão em
[`changelog/README.md`](changelog/README.md).
