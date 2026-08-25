---
schema_version: "0.27"
class: core
status: draft-pre-validacao
data: 2026-08-01
extraido_de: llm-dev-memory.md v0.11
---

# Método Keelson Flow — Desenvolvimento Humano & LLM

> **O que é, em uma linha:** Desenvolvimento de Software Colaborativo Humano & LLM em que uma **memória compartilhada viva** (a wiki), que **congela por evidência**, mantém o código gerado **no rumo da spec sem recair no waterfall**.

> **Status: experimental — validando no campo.** Este documento descreve o processo como ele emergiu na prática; endurece para prescrição depois que um ciclo completo (spec → código → promoção) rodar até o fim.

---

## Relação com os padrões irmãos — a trilogia, uma costura limpa

Este processo **usa** a wiki, mas não **é** a wiki. A separação segue a mesma regra que o processo prega ("não duplicar o dono"), e a dependência é de **mão única**:

- **[`llm-dev-memory.md`](llm-dev-memory.md) = a memória** (substantivos, estado, *onde* o conhecimento mora): tiers por custo de carregar, `docs/`, ADR, `index`/`glossary`/`log`/`now`, convenções de página, automação de manutenção, instrumentação, Obsidian. É o **tabuleiro**. **Funciona sozinha** — um projeto pode adotar só a wiki sem este processo.
- **`llm-dev-flow.md` (este doc) = o fluxo** (verbos, transições, *como* o trabalho anda e *quem* faz): `brief`→`plan`→`tasks`, os eixos de estado, a escada de evidência, o gradiente de congelamento, hipóteses-a-validar, os rituais, a divisão de trabalho humano/LLM. É o **jogo**.
- **[`llm-dev-player.md`](llm-dev-player.md) = o humano** (papéis e disciplinas: as três vigílias, as três entregas por sessão, o que não se delega). É o **jogador** — o terceiro elemento da trilogia. Toda linha "quem fecha: humano" da tabela de guardrails aponta, no fundo, para ele.

> **Agnóstico a ferramenta.** Um dos três padrões do núcleo: fala em **papéis**, não em produtos. As especificidades de cada ferramenta vivem nos *application guides* — ver [`llm-dev-README.md`](llm-dev-README.md), o firewall.

O processo referencia a wiki o tempo todo (colhe o glossário, registra no log, cataloga no index); a wiki não precisa saber que o processo existe. Se algum dia os dois divergirem sobre um fato de estrutura de arquivo, **a wiki é o dono** — aqui só se descreve *quando* e *por que* mexer nesses artefatos.

## Por que um processo, e não só uma wiki

A wiki resolve **memória** (o LLM relê barato, não perde contexto entre sessões). Mas manter o código *na direção certa* ao longo de muitas sessões curtas é um problema de **fluxo**, não de memória — e é aí que entra o SDD.

E há a recíproca: **é o fluxo que mantém a memória viva.** A wiki funciona sem o processo, mas não se conserva sozinha — é cada sessão do fluxo que reescreve o `now/`, deposita no `log/` e faz as decisões evoluírem por supersede; documento parado apodrece. A fronteira memória↔fluxo é de mão dupla: a memória dá ao trabalho onde pisar, e o trabalho, em troca, é o que a mantém de pé.

A tese central, que amarra tudo: **num time humano, congelar decisões é overhead que se minimiza; no dev Humano+LLM, a camada congelada é a memória compartilhada que *habilita* sessões pequenas.** O humano carrega contexto na cabeça entre sessões; o agente relê a wiki fria toda vez. Logo, **SDD-LLM congela para impedir *deriva*, não mudança.** O processo inteiro é a disciplina de decidir *o que* congelar, *quando*, e com base em *qual evidência* — sem virar waterfall.

---

## Os três artefatos: `brief` → `plan` → `tasks-faseN` (em `docs/specs/<slug>/`)

O `llm-dev-memory.md` organiza o **presente** (`docs/`), o **passado** (`wiki/log/`) e o **porquê das escolhas** (`docs/decisions/`). Falta o **futuro/prescritivo**: o que *deve ser construído*. É o nicho do SDD, e na prática ele emerge sozinho como "briefs de design" em `docs/specs/`. Este processo dá nome e disciplina a isso, em três artefatos:

| Doc | Papel | Análogo na indústria |
|---|---|---|
| `brief-<slug>.md` | **O QUÊ + PORQUÊ + restrições de design.** A especificação. Versionado, com histórico de revisões | `spec.md` (spec-kit), `requirements.md`+`design.md` (Kiro) |
| `plan-<slug>.md` | **O COMO.** Plano de implementação faseado, com **gates** de passagem entre fases (critérios objetivos, não datas). Referencia o brief por seção, não duplica. Versionado — uma lacuna de campo pode abrir uma fase nova por revisão versionada (ver abaixo), mesmo já `VALIDATED` | `plan.md` |
| `tasks-fase<N>-<slug>.md` | **Quebra fina, uma por fase.** O detalhamento executável de **uma** fase do plano; criado *just-in-time* quando o gate da fase anterior fecha | `tasks.md` (mas um por fase) |

Princípio herdado do glossário: o `plan` **referencia** o brief por § em vez de copiar; o brief é a fonte única. Mesmo raciocínio de "não duplicar o dono".

## Dois eixos de estado ortogonais (no frontmatter, nunca no nome do arquivo)

O erro clássico de SDD é confundir "spec pronta" com "feature pronta". São coisas diferentes e precisam de eixos separados:

- **`Doc Status`** — maturidade do *documento*: `DRAFT` → `VALIDATED` → `ARCHIVED` (ou `SUPERSEDED`, se um brief novo substitui o antigo antes de ser construído).
- **`Feature state`** — **evidência que o *código* já acumulou.** Não é escopo, não é audiência: é uma **escada de evidência** (a lição real do TRL — cada degrau é definido pelo que já foi *demonstrado*, não por calendário nem intenção), e é ela que diz quanto já pode virar chão firme para o agente:

| Nível | Evidência que existe | O que isso autoriza a congelar |
|---|---|---|
| **`NOT_BUILT`** | nenhuma — só a spec | nada (no máximo os 1-2 invariantes declarados) |
| **`BUILT`** | máquina + autor: o código existe e passa na própria verificação automatizada (dev + testes internos) | os contratos/interfaces que o código provou → **primeiros ADRs**; `architecture.md` começa como as-built |
| **`PILOT`** | humanos reais, controlado: rodou como sistema vivo com usuários reais em baixo risco (dogfood / alpha / friendly pilot) | o núcleo de design + o glossário; brief vira quase-imutável |
| **`PROD`** | realidade: em produção para a audiência real, sob risco real | congelamento duro; aprendizado novo vira **novo spec/ADR (supersede)**, não edição do congelado (ver o satélite [**Manutenção**](llm-dev-flow-maintenance.md)) |

**Ortogonais a esta escada — não são níveis dela:** *MVP* é recorte (propriedade do `plan`: quão fina é a primeira fatia vertical — convive com qualquer nível); *alpha / beta / GA* são sub-rótulos penduráveis sob `PILOT`/`PROD` (`PILOT:alpha`, `PROD:beta`) quando um projeto precisa de granularidade fina — a escada núcleo permanece com 4 degraus. A *fase do plano* **não** entra aqui: ela é a própria unidade que sobe a escada (ver logo abaixo).

### Sub-degraus — nomeando o que as sessões já produzem

A escada continua com **4 degraus**. Mas cada transição entre eles é feita por mais de uma sessão (ver "Sessões", abaixo), e o que cada sessão entrega de fato hoje só existe em prosa — sem rótulo, cada agente descobre o "estado real" relendo o `tasks`. Nomear resolve isso, sem inventar degrau novo — só sub-rotulando **dentro** de `NOT_BUILT` e de `BUILT`, no mesmo espírito de `PILOT:alpha`/`PROD:beta` acima:

| Valor de `Feature state` | O que já aconteceu | Quem produz |
|---|---|---|
| `NOT_BUILT` | só a spec | — |
| `NOT_BUILT_LANDED` | aterrissado — `tasks-fase<N>` existe, tabela de aterrissagem preenchida | `keelson-phase-landing` (ou, pro modo-bug, `keelson-fix`) |
| `NOT_BUILT_CODED` | código escrito, testes verdes — a soleira de `BUILT` | `keelson-coding` |
| `BUILT` | revisão independente aprovou (ver "Guardrails", abaixo — é o que de fato cruza o portão) | `keelson-review-session` |
| `BUILT_DEPLOYED_PILOT` | implantado em homologação/baixo risco, aguardando validação de campo | `keelson-deploy` |
| `PILOT` | validação de campo confirmou em homologação/baixo risco | `keelson-field-validation` |
| `BUILT_DEPLOYED_PROD` | implantado em produção, aguardando validação de campo | `keelson-deploy` |
| `PROD` | validação de campo confirmou em produção real | `keelson-field-validation` |

**Sintaxe deliberadamente diferente de `PILOT:alpha`:** aqui o valor é um **token só, com `_`**, não `nível:rótulo` — porque este campo é lido por **precondição de skill** ("só execute se `state = X`"), e frontmatter é YAML: um valor com `:` sem aspas é frágil pra comparação exata. `alpha`/`beta`/`GA` são anotação livre, opcional; os sub-degraus acima são gate duro.

Manter o estado no cabeçalho, não no nome do arquivo (`brief-x - DRAFT.md` é anti-padrão — renomear quebra links e histórico de `git`).

### Onde cada eixo mora — e por que `tasks` é por fase

Uma feature não sobe a escada de evidência inteira de uma vez: ela é construída **fase a fase**, e fases diferentes têm níveis de evidência diferentes ao mesmo tempo (a Fase 0 pode estar `BUILT` — ou até em produção, inerte — enquanto a Fase 1 nem começou). Um `tasks` único por feature não conseguiria carregar um `Feature state` coerente no frontmatter. Daí a regra de **onde cada eixo mora**:

- **`brief` e `plan`** carregam só o **`Doc Status`** (maturidade do documento de design). Um de cada por feature.
- **`tasks-faseN`** carrega **os dois eixos** — é o único artefato que acumula ambos: o **`Doc Status`** (`DRAFT`→`VALIDATED`→`ARCHIVED`) marca se a **tabela de aterrissagem** já foi aprovada por um humano (o gate entre `keelson-phase-landing`/`keelson-fix` e `keelson-coding`); o **`Feature state`** é a escada de evidência do código em si. Os dois avançam em ritmos diferentes e não se substituem: `VALIDATED` sozinho não implica `BUILT`, e um `tasks` pode ficar `VALIDATED` por várias sessões de coding enquanto o `Feature state` sobe devagar. **`VALIDATED` trava só a tabela de aterrissagem** (os já-existe/colisão/lacuna-de-HOW) — o resto do arquivo (checkboxes das tasks) continua mudando normalmente durante o coding; não é o mesmo "quase-imutável" que vale pro `brief`.
- O **estado da feature inteira** é a **fronteira** entre as fases (ex.: "Fase 0 `BUILT`, Fase 1 `NOT_BUILT`") — **calculado**, nunca gravado no frontmatter de `brief`/`plan` (seria duplicar o dono, que é sempre o `tasks-faseN`). Ver "Como a `wiki/` se acomoda ao processo", abaixo, pra onde esse cálculo aparece.

Consequências práticas:

- **Um `tasks-faseN.md` por fase, criado *just-in-time*** — quando o gate da fase anterior fecha. Detalhar uma fase futura antes viola o gradiente de congelamento (congelaria decisões que ainda não têm chão de código).
- **Estrutura em pasta, com nomes cheios:** cada feature vive em `docs/specs/<slug>/`, com `brief-<slug>.md`, `plan-<slug>.md` e os `tasks-fase<N>-<slug>.md`. O slug **se repete** no nome do arquivo **de propósito** — o grafo do Obsidian rotula cada nó pelo nome do arquivo, e nomes genéricos (`brief.md`, `plan.md`) tornam os nós de *todas* as features indistinguíveis no grafo. Paga-se a repetição no caminho para ganhar legibilidade no grafo (que é artefato de primeira classe aqui — ver `llm-dev-memory-structuring.md`, "Obsidian"). A pasta usa slug **ASCII** (sem acentos), o que mantém os links markdown limpos, sem `%`-encoding. Migração pode ser incremental — uma feature de cada vez.
- **Promoção acompanha a fase:** o current-state de cada fase migra para `architecture.md` quando *ela* atinge `BUILT`; o `brief`/`plan` só vão para `docs/archive/` quando a feature **inteira** atinge `PROD`.

## Congelamento: um gradiente, não um evento — e por que ele *habilita* a agilidade

Waterfall congela tudo no início; ágil puro nunca congela. Nenhum dos dois serve ao dev com LLM. A síntese é um **gradiente**: *congela o que já foi aprendido, mantém fluido o que ainda vai ser aprendido — e a fronteira sobe junto com o `Feature state`* (a coluna "o que isso autoriza a congelar" acima é literalmente essa fronteira, degrau a degrau).

A inversão que torna isso não-burocrático já foi enunciada como tese: a camada congelada é a memória compartilhada que *permite* sessões pequenas. Os ADRs, o `architecture.md` congelado e o glossário são a memória de "isto já foi decidido, não relitigue" — sem eles, a cada sessão o agente pode redesenhar o que já estava resolvido. Você *quer* congelar agressivamente — mas só o que a evidência já sustenta, e movendo a fronteira para cima conforme sobe de nível.

## Hipóteses a validar — a seta de volta contra o waterfall

O que faz o spec-kit cheirar a waterfall não é ter spec, é o fluxo ser uma **seta só** (`spec→plan→código`) sem retorno. A seta de volta é uma convenção simples no brief: uma seção **"Hipóteses a validar"**, cada hipótese marcada com o **nível de evidência que vai testá-la**:

```
H3 (validar em PILOT): "Faixa Central é a primeira tática real certa."
   Se refutada → revisão versionada do brief + entrada no log/. Se confirmada → vira invariante/ADR.
```

Quando o nível é atingido, a hipótese ou **confirma** (congela — vira invariante/ADR) ou **refuta** (dispara revisão versionada do brief + `log/`). Isso torna a spec honesta sobre o que ainda *não* sabe, e casa com o versionamento que o brief já tem — a revisão é um bump + log, não uma reescrita silenciosa. É o mecanismo que reconcilia "SDD garante direção" com "requisitos amadurecem com o sistema".

## Revisão versionada do `plan` — quando uma fase nova nasce depois do `VALIDATED`

O brief já tinha esse mecanismo (acima); o campo mostrou que o `plan` precisa dele também. Uma lacuna descoberta em produção — um `fix-<slug>.md` que aponta uma peça do brief nunca implementada, um item de `backlog.md`, uma hipótese refutada — pode exigir uma **fase que o plano original não previu**, mesmo com o `plan` já `VALIDATED`. A resposta não é reescrever o plano, nem abrir um `plan-<slug>.md` do zero (a feature já tem plano; o que falta é uma fase): é o mesmo gesto do brief, aplicado ao plano.

- **Bump de versão** (`V0.1`→`V0.2`), com uma entrada curta no topo do documento — o que mudou, por quê, apontando a origem (fix-doc §, item de backlog, brief §) **por ponteiro**, sem repetir o desenho.
- **Fases já `VALIDATED`/`BUILT` não se tocam.** A fase nova se insere, e as fases seguintes só são **renumeradas** se nenhuma delas já tiver um `tasks-faseN` materializado — a criação *just-in-time* (acima) é o que torna essa renumeração segura. Se uma fase seguinte já tem `tasks-faseN` no chão, não renumeie: anexe a fase nova ao final, ou como sub-fase.
- **A fase nova nasce marcada, inline, como pendente de revisão humana** — não é um `Doc Status` novo no frontmatter (isso degradaria o documento inteiro por causa de uma fase só); é uma anotação escopada só a ela. O `Doc Status` do plano **permanece `VALIDATED`** durante a revisão — as fases antigas continuam congeladas, e é só a fase nova que espera o humano.
- **A Superfície de incerteza da revisão é escopada só a ela**, não ao plano inteiro — o mesmo princípio de "um veredito não estica sozinho" (ver "Guardrails", abaixo), agora aplicado a uma edição em vez de a uma revisão.

Não nasce skill nova para isso — é o segundo caminho da mesma skill que cria o plano do zero (ver `keelson-plan-init`), porque a precondição que muda é só "o plano já existe e está `VALIDATED`", não a natureza do trabalho.

## Aterrissagem: reconciliar a spec com o código antes da quebra-fina

> *(Nome do passo ajustável — "aterrissagem" porque é onde a spec, que pairava no plano de design, pousa sobre o código real.)*

Transformar uma fase do `plan` em `tasks` **não é transcrição**. O brief e o plano carregam suposições implícitas sobre o código que já existe — e essas suposições precisam ser **confrontadas contra o código real antes** de virarem task. Este é o momento em que as "hipóteses a validar" deixam de ser abstratas: o agente lê o current-state (via `wiki/index.md` → `architecture.md`/`data-model.md`, e o próprio código) e reconcilia, requisito a requisito, o que o brief pede contra o que já está lá.

Cada delta cai em um de **três tipos** (nomeados a partir da primeira aplicação real — OptiFlux, Fase 0):

1. **Já existe → reaproveitar.** O requisito já está implementado, total ou parcialmente. A "task" vira *validar/testar o que existe*, não construir — e, corolário, **não** construir um caminho paralelo que duplique o existente (regra "não duplicar o dono"). *(Ex.: um mecanismo de idempotência que o §11 pedia já existia; a decisão foi reusar e recusar um endpoint novo que o duplicaria.)*
2. **Colisão → decisão humana.** O requisito bate contra algo existente (um nome de coluna, um contrato, uma feature ativa). Levanta-se o achado, **o humano decide**, e a decisão vira registro no `tasks` + `log/` — **nunca** uma edição silenciosa do brief (`VALIDATED`, quase-imutável). *(Ex.: a coluna de ownership que o brief pedia colidia com uma já usada por outra feature → coluna nova, decidido pelo operador.)*
3. **Lacuna de HOW → elaborar no `tasks`.** O brief especifica o QUÊ mas não o COMO executável — o que é *esperado*, é a fronteira brief↔tasks. O COMO é inventado no `tasks`, referenciando o § do brief, sem tocá-lo. *(Ex.: "não arquivar se a tática não é terminal" não dizia como o serviço saberia disso remotamente → uma tabela auxiliar, definida no tasks.)*

**Artefato:** uma tabela curta no topo do `tasks` (requisito × já existe? × o que a task faz) torna a reconciliação legível e auditável — e de quebra é a espinha de rastreabilidade (cada task → § do brief).

**Por que importa:** sem a aterrissagem, a quebra-fina vira transcrição cega, e as suposições erradas sobre o código só explodem no meio da implementação — como bug ou retrabalho. A aterrissagem é barata (uma passada de leitura guiada pelo `wiki/index.md`) e paga caro: na primeira aplicação real, pegou uma colisão de nome que teria virado bug e evitou construir um endpoint redundante. É também o gatilho natural para confrontar as hipóteses do brief cujo nível-dono é a fase que está sendo aterrissada.

## Ciclo de vida e os dois rituais

**No `VALIDATED` (spec fecha, antes do código): colher o vocabulário para o `glossary.md`.** É o momento de maior valor do mecanismo anti-divergência — um brief grande introduz muitos termos que vão *definir* o subsistema; travá-los agora impede que o código os reintroduza com desvio depois. O dono dos termos no glossário é, temporariamente, o próprio brief; o glossário marca isso e aponta a migração futura. (Formato e regras do glossário: ver [`llm-dev-memory.md`](llm-dev-memory.md), seção "glossary.md".)

**Promover e arquivar — "a spec queima ao ser promovida".** Isto é gradual, acompanhando a escada de evidência, não um único evento:
1. **A partir de `BUILT`**, o *current-state* (como o subsistema funciona **agora**) começa a ser escrito em `architecture.md` (+ satélite se grande) e `data-model.md` — nasce as-built e endurece a cada nível; o dono dos termos no glossário passa do brief para esses arquivos.
2. Decisões transversais que precisam ser citáveis fora da feature viram **ADRs** (extração preguiçosa — só o que será citado de fora, não 15 ADRs de uma vez), progressivamente, conforme sobrevivem ao contato.
3. **Ao atingir `PROD`** (feature provada), `brief`/`plan`/`tasks` vão para `docs/archive/` — deixam de ser current-state (viram registro de design histórico). **Não** podem ficar em `docs/specs/` fingindo estar pendentes depois de provados.

## O risco nº 1 — spec-rot — e como o processo o contém

Spec abandonado é pior que doc velho: doc velho é impreciso, spec obsoleto é uma **mentira citável**. A regra: a partir do `VALIDATED` com implementação em curso, o brief é **quase-imutável, como um ADR**. Divergência descoberta implementando **não** se conserta editando o brief em silêncio — vira um ADR (se muda uma decisão) ou um incidente no `log/`. O brief é o *as-designed*; o `architecture.md` é o *as-built*. Vão divergir, e tudo bem — desde que a divergência seja **rastreada**, nunca silenciosa.

## Manutenção (modo bug) — em `llm-dev-flow-maintenance.md`

Corrigir um bug é um **modo de trabalho próprio**, não o fluxo forward. Toda a maquinaria de manutenção — a correção que nasce já aterrissada, o artefato `fix-<slug>.md` em `docs/fixes/`, a triagem de dois eixos, o teste-regressão *red* como piso, o ponteiro `known-issues.md` — vive no satélite **[`llm-dev-flow-maintenance.md`](llm-dev-flow-maintenance.md)**. Abra-o quando o trabalho for **consertar o presente defeituoso**; para construir para a frente (brief/plan/tasks), o núcleo abaixo basta.

## Como a `wiki/` se acomoda ao processo

- **`index.md`** ganha uma seção "Especificações (SDD)" — tabela das features por `Doc Status` × `Feature state`, com ponteiros brief/plan/tasks. Um agente entrando frio vê o estado do roadmap num relance. O `Feature state` da **feature inteira**, nessa tabela, é o **mínimo entre as `tasks-faseN`** existentes, na ordem da escada (incluindo sub-degraus: `NOT_BUILT < NOT_BUILT_LANDED < NOT_BUILT_CODED < BUILT < BUILT_DEPLOYED_PILOT < PILOT < BUILT_DEPLOYED_PROD < PROD`) — é uma coluna **calculada** ao montar a tabela, nunca um campo gravado em `brief`/`plan`.
- **`glossary.md`** colhe o vocabulário no `VALIDATED` (ritual acima).
- **`log/`** registra os marcos: `DRAFT`→`VALIDATED`, cada gate de fase que passa, e cada divergência spec↔código.
- **`now/<branch>.md`** carrega o progresso fino durante a implementação ("Fase 2, faltam os testes de crash do gate 2→3").

## Conexão com a fronteira código↔doc

Alguns specs são também **artefato de runtime** (uma base de conhecimento RAG carregada por um agente — ver a exceção em [`llm-dev-memory.md`](llm-dev-memory.md), seção "`docs/`"). Esses não migram para `docs/specs/`: ficam onde o código os espera e a wiki os indexa. O SDD é a camada prescritiva; quando um spec é *também* código, as duas naturezas coexistem e a localização segue o código.

---

## Sessões: uma sessão = uma transição de estado de um artefato

Uma pergunta que todo mundo que trabalha com LLM sente mas raramente resolve: *quando abrir uma sessão nova?* O processo responde de graça — porque os artefatos e seus eixos de estado **são** as fronteiras de sessão. A formulação:

> **Uma sessão = uma transição de estado de *um* artefato.**

- Sessão do **brief** → move o brief de `DRAFT` para `VALIDATED`. (Se nem `DRAFT` existe ainda — a decisão de
  desenho emergiu de uma investigação sem cobertura —, `keelson-brief-prep` faz a preparação mecânica
  primeiro; o conteúdo em si, o QUÊ/PORQUÊ, segue sem skill, ver [`llm-dev-player.md`](llm-dev-player.md).)
- Sessão do **plan** → move o plan de `DRAFT` para `VALIDATED`.
- Sessão de **aterrissagem** (`keelson-phase-landing`) → move o `tasks-fase<N>` a `NOT_BUILT_LANDED`.
- Sessão de **coding** (`keelson-coding`) → move o `tasks-fase<N>` a `NOT_BUILT_CODED` — a soleira de `BUILT`.
- Sessão de **revisão** (`keelson-review-session`, sempre sessão separada) → cruza o portão pra `BUILT`.
- Sessão de **deploy** (`keelson-deploy`) → move a `BUILT_DEPLOYED_PILOT`/`BUILT_DEPLOYED_PROD`.
- Sessão de **validação de campo** (`keelson-field-validation`) → move a `PILOT`/`PROD`.
- Sessão de **bug-fix** (`keelson-fix`) → mesma escada, começando já aterrissada (ver o satélite [**Manutenção**](llm-dev-flow-maintenance.md)).

**Regra dura:** se, dentro de qualquer uma dessas sessões, aparecer um bug **fora do escopo dela** (de outra fase, ou achado incidental — ex.: `keelson-field-validation` tropeça num bug que não é o que estava validando), a sessão **não conserta inline**. Ela para, registra o achado, e aponta para `keelson-fix` — mesmo que pareça pequeno. Misturar o modo da sessão corrente com uma investigação de causa-raiz não pedida é o mesmo custo de poluição de contexto que a separação de sessões existe para evitar.

Por que isso é natural do ponto de vista do LLM — duas razões, ambas econômicas:

1. **Cada fronteira é uma virada *qualitativa* de contexto.** O que se quer carregado muda quase por inteiro entre os modos: o brief é **divergente** (domínio, RAG, requisitos, conversa); o plan é **estruturante** (o brief fixo + `architecture.md`/`data-model.md`); o coding é **convergente** (o `tasks-faseN` + um punhado de arquivos + o test runner). Misturá-los numa sessão só polui o contexto de cada modo. Sessões separadas mantêm o contexto enxuto.
2. **Cada fronteira deixa um artefato de handoff durável.** Como o agente começa frio a cada sessão, o que importa é retomar sem o histórico da sessão anterior. O artefato (brief `VALIDATED`, `tasks-faseN` com checkboxes) é o "flush" da memória de trabalho (a sessão) para a de longo prazo (a wiki/spec): **a fronteira de sessão é o ponto de descarga.**

A regra prática, que responde ao "quando abrir sessão nova?":

> **Termine a sessão quando o artefato dela atingir um estado nomeável** (uma transição de eixo, ou um gate) — não quando os tokens acabam. **Comece uma nova quando o *tipo* de contexto muda.** Nas fronteiras brief / plan / fase, os dois gatilhos coincidem.

Nuance — é um mapeamento de **tipos** de sessão, não uma contagem literal de 1:1:

- O brief costuma levar **várias** sessões até `VALIDATED` (refinamento iterativo). "Uma sessão de brief" é, na real, "uma fase de trabalho de brief".
- Uma **fase grande** se quebra em mais de uma sessão de coding (por cluster de tasks) — o `tasks-faseN` com checkboxes é o que permite pausar/retomar sem perder o fio.
- Features pequenas **colapsam** brief + plan numa sessão só.
- Um bug pequeno **colapsa** triagem + causa-raiz + coding numa sessão só de `keelson-fix` (mesma lógica — nenhum desses três precisa de sessão fresca entre si, só a **revisão** precisa).

E há uma transição que a sessão **nunca fecha sozinha, por construção**: `→ BUILT` exige revisão independente (ver "Guardrails", abaixo) — e revisão **é**, por definição, uma sessão diferente da que codou. Nenhuma skill pode se auto-conceder essa independência chamando outra skill dentro da mesma sessão; é o único portão da escada inteira onde colapsar sessões quebra a própria garantia que o portão promete. A sessão de coding (`keelson-coding`, ou `keelson-fix` quando colapsado) leva o artefato até a *soleira* do `BUILT` (`NOT_BUILT_CODED` — código escrito, testes verdes) e faz o handoff ali; deploy (`keelson-deploy`) e validação de campo (`keelson-field-validation`), ao contrário, não dependem de sessão fresca — podem rodar na mesma sessão da revisão ou do coding, à vontade, com o humano confirmando em cada portal.

---

## Guardrails por transição — *Definition of Done* proporcional

Uma transição de estado só "conta" quando passa por um guardrail. Mas o guardrail é **proporcional ao raio de explosão da transição** — ao tamanho da afirmação que ela faz. `BUILT` afirma pouco ("o código funciona isolado"); `PROD` num sistema com dinheiro real afirma muito. Uniformizar o rigor é desperdício de um lado e *review theater* do outro; proporcional é energia onde produz solidez.

**Por que a seção existe (o vício que ela ataca):** um LLM tende a *corrigir a própria prova*. Testes verdes provam só o que ele pensou em testar — e ele escreveu o código **e** os testes, que podem compartilhar o mesmo mal-entendido. O modo de falha do agente não é preguiça, é **confiança-mas-errado**. Logo, o guardrail que importa é **check externo objetivo que o autor não consegue racionalizar** — não "mais contexto para o agente ler" (ele raciocina por cima com a mesma confiança), e sim "mais coisa que *checa* o agente".

### As duas independências, e a regra do teste-de-costura

"Externo ao autor" tem **dois eixos**: **quem** verifica (autor → revisor independente → uso real) e **em que fidelidade** (dublê/mock → ambiente real). São ortogonais e **nenhum substitui o outro** — a revisão independente move só o *quem* (revisor fresco, mas mesma fidelidade de mock), então **não** pega a **suposição errada sobre o mundo** (o recurso existe? a mensagem chega? o contrato do outro serviço é esse?); essa classe só aparece cruzando a fronteira real. *(Linhagem: Segregação de Funções no eixo do quem, Fidelidade de Ambiente no eixo da fidelidade — aprofundamento no livro companheiro.)*

Disso saem **duas regras operacionais**, não uma teoria:

Antes, o que é uma **fronteira**: qualquer ponto onde o código depende de um **contrato que ele não possui** — outro processo ou serviço, o sistema operacional/arquivos, um dispositivo, um motor de banco, um protocolo/formato de fio, o módulo de outra equipe, uma biblioteca externa — e que os testes costumam **substituir por um dublê** (mock, stub, fixture, simulador, emulador). O dublê codifica *as suas suposições* sobre o outro lado; só o contraparte real pode discordar delas. (A definição é agnóstica a domínio de propósito — a *lista* de fronteiras muda por sistema.)

1. **Teste-de-costura — baixe a fidelidade para a esquerda, onde é barato.** Para **cada fronteira** que a fase cruza e que **sustenta peso**, **um** teste exercita o **contraparte real** in-suite — não o dublê. Antídoto ao *"o dublê nunca discorda do autor"*: um dublê escrito pela mesma mão concorda consigo por construção; só o contraparte real discorda, e pega barato e cedo a classe de bug que de outro modo só a produção revelaria. *(Exemplos de domínios distintos: o motor de banco real e o middleware real num serviço; o terminal/protocolo real num sistema de trading; o sistema de arquivos real num CLI; o dispositivo real num embarcado.)*
2. **`field-validation-required` — o resíduo caro, nomeado.** A fronteira cujo contraparte real **genuinamente não roda in-suite** — porque só existe no host/produção/dispositivo/mercado vivo (um secret provisionado, timing real, um serviço externo, o hardware) — **não** se finge testada: vira uma **lista explícita** no handoff, para a validação de alta fidelidade. É o eixo que a **escada de evidência** sobe (`BUILT` afirma só a camada de dublê; `PILOT`/`PROD` cruzam para a real). O revisor não valida campo — **nomeia onde é cego**.

Régua de sempre: **proporcional ao raio de explosão** — costura para toda fronteira que sustenta peso, não para um wrapper trivial. **Exceção nomeada, aplicada na triagem de revisão** (detalhe operacional em `keelson-review-session`): se a lacuna de costura é **dívida herdada** — o diff copiou fielmente um padrão sem costura que já existe, idêntico, em código já `BUILT` — e o raio é baixo, isso vira recomendação registrada, não `VOLTA`. A fase não é responsável por uma dívida que herdou, só pela que introduziu.

### Revisão independente é obrigatória por padrão — com override humano explícito, nunca silencioso

Diferente do teste-de-costura (proporcional ao raio), a revisão independente que cruza `→BUILT` é **obrigatória por padrão**, feature ou fix — não "normalmente exigida". Diff pequeno é revisão barata e rápida; é exatamente onde pular custaria mais do que economiza. Isto **não é** um bloqueio incondicional: o operador pode **decidir explicitamente pular**, mas a decisão é sempre dele, nunca da skill, e fica **registrada** no cabeçalho do artefato promovido (o `Feature state: BUILT` carrega a justificativa inline — "promovido sem revisão independente separada, aceito pelo operador, motivo: X"), nunca um pulo silencioso.

**Gatilho mecânico, independente de raio:** se o diff **reescreve o valor esperado de um teste pré-existente** (não só adiciona teste novo), a recomendação de revisão sobe de força **mesmo em raio baixo** — é a assinatura clássica de confiante-mas-errado: o mesmo agente que escreveu o fix também reescreveu a prova de que está certo, e um teste que concordava com o comportamento antigo não vai discordar do novo por conta própria. Checável por grep no diff, sem julgamento.

**Sem sessão humana nova, uma orientação em camadas** (detalhe operacional mora em `keelson-review-session`, não duplicado aqui): raio baixo e sem o gatilho de reescrita-de-teste → um sub-agente com contexto fresco (só o diff + a camada congelada, nunca a narrativa do autor) já entrega a independência de *sessão* que a revisão promete. Raio médio/alto ou gatilho de reescrita-de-teste → sub-agente com modelo/config deliberadamente diferente do autor, ou sessão humana separada — porque aí o que se protege não é só "outra leitura", é um viés sistemático que o mesmo modelo, na mesma config, tende a repetir.

### Um veredito não estica sozinho — diff que muda depois do `PRONTO` não está coberto

O veredito de `keelson-review-session` cobre o diff **que existia no momento da revisão** — não o artefato pra
sempre. Se o código mudar depois de um `PRONTO` (mesmo endereçando uma recomendação **da própria revisão**,
mesmo que seja só um teste novo, sem tocar produção), as linhas novas **não** herdam o veredito antigo por
osmose. Isso vale a mesma régua da revisão em si: **obrigatória por padrão, override explícito permitido,
nunca silêncio.**

- **Padrão:** chame `keelson-review-session` de novo — mas não do zero. Uma **passada focada no delta**: o
  que já foi revisado (e não mudou) não precisa de nova leitura; só o que mudou desde o veredito anterior
  entra no escopo novo.
- **Override:** o humano pode decidir que o delta é pequeno/baixo-raio o bastante pra dispensar nova revisão
  (o exemplo mais comum: só teste novo, produção intocada) — mas é decisão **dele, nomeada**, não a skill
  presumindo que "o veredito de antes ainda vale". Registre no handoff/log, do mesmo jeito que qualquer outro
  override de revisão.
- **Quem detecta:** a skill que fez a mudança pós-veredito (`keelson-coding`, tipicamente) sinaliza isso
  explicitamente no handoff — "este diff mudou depois do veredito de {data}; as linhas novas não foram
  cobertas" — não deixa pra quem for virar a chave descobrir sozinho.

### Recomendação: não commite até `BUILT`

O handoff do coding (a soleira `NOT_BUILT_CODED`) fica, por padrão, **sem commit** — a revisão roda contra o
**working tree**, não contra um commit fechado. Se o veredito for `VOLTA`, o conserto continua no mesmo diff
não commitado: não existe um commit "ruim" pra reverter ou reescrever. O commit acontece **uma vez**, no
momento em que `BUILT` é atingido (veredito `PRONTO` + a chave virada, ou o override registrado) — um commit
limpo que representa o trabalho já revisado, não uma sequência com o ruído do ciclo de revisão embutido nela.
É o ponto de partida natural pra `keelson-deploy` em seguida.

**Não é regra absoluta:** uma fase grande, quebrada em várias sessões de coding ao longo de dias, pode
legitimamente precisar de commits de checkpoint pra continuidade (ver "Sessões" — "`tasks-faseN` com
checkboxes é o que permite pausar/retomar"). A recomendação vale pro **estado que vai pra revisão**: chegue
lá como um diff revisável, e trate o commit de `BUILT` como o registro histórico que importa — não os
checkpoints intermediários.

**Por que isso resolve, na prática, um problema já registrado:** a Fase 2 do OptiFlux teve um único commit
squash (`ddbfdd5f`, "T2.1-T2.20") apagar do histórico que houve revisão, um veredito `VOLTA` e achados
reabertos no meio — o `git log` mentia sobre a disciplina que de fato aconteceu. Não commitar até `BUILT`
elimina a classe inteira desse problema: não há ciclo de revisão pra um commit prematuro esconder.

### Virar a chave é um checklist, não uma metáfora

"Humano vira a chave" nomeou a decisão certa, mas por muito tempo não virou passo operacional — e por isso
caía num vão entre `keelson-review-session` (que só emite veredito, nunca escreve) e `keelson-deploy` (cuja
precondição já pressupõe `BUILT` pronto, e só confere se o commit *existente* está publicado).

Virar a chave, concretamente, são **dois atos**:

1. **Commitar o diff revisado** — o commit único e limpo que "não commite até `BUILT`" preparou.
2. **Gravar `Feature state: BUILT`** no frontmatter do artefato (`tasks-fase<N>.md`, `fix-<slug>.md` ou
   `tweak-<slug>.md`).

Os dois sempre juntos — um sem o outro deixa o artefato mentindo sobre o próprio estado: código commitado sem
o frontmatter dizer `BUILT`, ou frontmatter dizendo `BUILT` sem commit real por trás.

**A decisão é sempre humana; o gesto pode ser delegado.** A primeira versão desta regra dizia que os dois atos
não eram delegados a nenhuma skill, por medo de que automatizar o gesto tirasse o próprio ponto de decisão. Uso
de campo (OptiFlux) mostrou o custo real disso: o operador tinha que voltar à sessão de coding e, de próprio
punho, pedir a promoção — funciona para quem já conhece a dança, é opaco para quem não conhece. A correção não
é automatizar sem gate — é separar **decisão** de **execução**. `keelson-coding`, reinvocado sobre um artefato
já em `NOT_BUILT_CODED`, **detecta** um veredito `PRONTO` registrado em `wiki/log/` (formato exigido de
`keelson-review-session` — ver "Registrar o veredito", abaixo) e **pergunta**, num gate explícito, se deve
commitar e gravar `BUILT`. A decisão continua do humano — ele confirma ou não; a skill só evita que ele tenha
que redigitar o commit e editar o frontmatter à mão depois de já ter decidido. Sem o veredito registrado no
formato certo, ou sem a confirmação, `keelson-coding` não promove — cai no caminho de override (ver abaixo).

**Registrar o veredito não é opcional — é precondição mecânica da promoção.** `keelson-review-session` grava
o veredito em `wiki/log/` num cabeçalho padrão, com o **slug do artefato** e o token literal `PRONTO`/`VOLTA`
(detalhe em `keelson-review-session/SKILL.md`, Passo 4). Sem essa marca, no formato certo, `keelson-coding`
não tem como saber que pode promover — o processo trava silenciosamente, não porque a revisão não aconteceu,
mas porque ela não deixou rastro encontrável.

**Sem rastro (revisão em sessão externa, não documentada aqui), o caminho é o override já descrito acima** —
o operador fornece a justificativa curta que vai inline no `Feature state: BUILT` ("revisão sem rastro
auditável aqui, aceito pelo operador, motivo: X"), e `keelson-coding` só promove depois disso. É o mesmo
mecanismo de "override explícito, nunca silencioso" — só reconhecendo que "revisão aconteceu em sessão que
esta não consegue ver" é o caso normal de sessões independentes (o humano é a única ponte entre elas), não uma
exceção nova a formalizar.

**Defesa em profundidade:** `keelson-deploy` reverifica os dois atos na própria sondagem (Passo 1), em vez de
confiar que a etapa anterior fez certo — a mesma disciplina de "cada etapa reverifica a própria precondição"
que já rege o resto do fluxo.

### Dois níveis de verificação: mecânica antes de semântica

- **Verificação mecânica** (barata, determinística, sem LLM) — rode primeiro, é auto-garantia objetiva: suíte de testes, lint, type-check, cobertura mínima, e **greps de invariante** (procurar a violação direto no diff).
- **Verificação semântica** (LLM, só o que a mecânica não pega): fidelidade à spec, deriva arquitetural, qualidade dos testes, segurança sutil.

### O guardrail confere contra a camada congelada

O revisor **não inventa critério** — confere o diff contra o que a wiki já mantém vivo: **ADRs** (os invariantes), **glossário** (o vocabulário), **brief §item** (os requisitos, via a tabela da *aterrissagem*). É o braço de fiscalização de "congela para impedir deriva". Vantagem sobre `constitution.md`/steering estáticos (spec-kit/Kiro): nossos critérios são mantidos pelos rituais de promoção — **não envelhecem**.

Um **terceiro tipo de dono** entrou nessa camada congelada: a **fonte de domínio** — regulamento/legislação externa (ver [`llm-dev-memory.md`](llm-dev-memory.md), "`docs/domain/`"). Ela não muda o mecanismo: suas obrigações vinculantes entram no guardrail **como invariantes** (com o *dono* apontando para a página-resumo em vez de um ADR), então a verificação mecânica as pega por grep como qualquer outra. O que ela acrescenta é uma checagem **semântica de conformidade** no alto raio de explosão (`→PILOT/PROD` num domínio regulado): o diff não viola nenhuma obrigação externa vigente? Proporcional como as demais — só onde o não-cumprimento tem custo real.

### Tabela — guardrail por transição

| Transição | Mecânico | Semântico | Quem fecha |
|---|---|---|---|
| brief `→VALIDATED` | — | revisão adversarial (arquitetura, segurança); checklist de completude/ambiguidade | humano |
| plan `→VALIDATED` | — | sanidade de fases/gates/dependências; coerência com o brief | humano |
| tasks-faseN `→NOT_BUILT_CODED` | testes verdes + lint/types + greps de invariante | — (`keelson-coding`) | agente |
| `→BUILT` | (herda o mecânico acima) | revisão **independente**: fidelidade ao §, invariantes/ADR, qualidade dos testes, gatilho de reescrita-de-teste (`keelson-review-session`, sessão separada) | agente-revisor → **humano decide, `keelson-coding` executa** (detecta o veredito em `wiki/log/`, confirma, commita + grava `Feature state: BUILT`) (ou override registrado) |
| `→BUILT_DEPLOYED_PILOT`/`_PROD` | deploy + health check (`keelson-deploy`) | — | humano confirma o portal |
| `→PILOT`/`PROD` | evidência observada ao vivo (`keelson-field-validation`) | crash tests, simulação antes de risco real, runbooks; revisão de segurança | humano |
| merge da memória `→ main` | termo definido 2×; nº de ADR duplicado + refs órfãs pós-renumeração; invariantes textualmente contraditórios | o mesmo termo novo definido de formas *divergentes* pelas branches; decisões em tensão | **guardião** (humano) |

### O merge para a main é uma transição — e tem guardrail próprio *(desenho, pré-validação)*

Com trabalho em branches paralelas, há uma transição que as linhas clássicas da tabela não cobriam: o **merge da memória para a main**. A regra de verdade é da wiki (ver [`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md), "Concorrência"): o que a branch escreveu na camada congelada é *candidato*; canônico é o que chega à main (`now/` é verdade-por-branch; `log/` faz união). O guardrail do merge — última linha da tabela — é o lint semântico já desenhado na wiki, disparado no merge, e proporcional como os demais: um merge que só toca código dispensa a checagem semântica da memória. Quem fecha é o guardião. Status honesto: desenhado a partir das peças existentes, ainda não rodado sob paralelismo real.

### Convenções recomendadas (com atribuição; aplicadas por raio de explosão, nunca dogmáticas)

- **Critério de aceite testável** — no estilo **EARS** (*Easy Approach to Requirements Syntax*, Mavin et al., 2009): requisito escrito como `QUANDO <condição> O SISTEMA DEVE <comportamento>`. Dá ao agente um **alvo objetivo que ele não escreveu** — o antídoto mais direto ao "corrigir a própria prova". Aplicar aos requisitos que importam, não a todos.
- **Teste-primeiro que falha antes do código** — a fase *red* do **TDD** (Red-Green-Refactor, Beck, 2002): para tasks de alto raio de explosão, escrever o teste a partir do critério de aceite, **confirmar que falha**, e só então implementar. Mata o teste tautológico/falso-positivo (este projeto já viu um: teste que reimplementava a lógica em vez de importá-la). Proporcional, não para toda task trivial.
- **Revisão independente em sessão separada** — quem revisa ≠ quem escreveu, com contexto fresco (o modelo de sessão já entrega isso de graça). Ancorada na camada congelada. Skill dedicada: `keelson-review-session` — promovida depois de medida (Fase 2 do OptiFlux: 2 bugs reais que 275 testes verdes não pegaram). No harness sem a skill instalada, `/code-review` apontado para brief + ADRs cobre o mínimo.
- **Revisão da *qualidade* dos testes** (não só "passam"): importam a lógica real em vez de reimplementá-la? cobrem os caminhos de risco? não são tautológicos? — ponto que o mercado sub-atende.

### Duas peças que faltavam (as mais valiosas na visão do próprio agente)

- **Índice fino de invariantes** — uma lista *terse* e grep-able dos invariantes duros (ex.: "o EA nunca chama `SocketRead`"; "cálculo financeiro sempre usa o preço real de execução"), cada um apontando para o ADR/dono. É artefato de *memória* (vive na `wiki/`, ao lado do glossário — ver `llm-dev-memory.md`, seção "`invariants.md` — contrato de comportamento"). **Não duplica os ADRs** (eles são o dono); é o *checklist operacional* deles, para o revisor e para os greps mecânicos. É a nossa versão da "constitution" — mas viva e ponteira, não estática.
- **Superfície de incerteza no handoff** — item da *Definition of Done* de **toda** transição: todo artefato entregue termina com *"o que assumi / onde posso estar errado / o que não verifiquei"*. Ataca o viés confiança-mas-errado na raiz — o humano e o próximo revisor sabem onde cutucar. Baratíssimo, e é o que o agente mais pediu de dentro do processo. (A sessão da Fase 0 fez isso meio por acaso — "pendente revisão, ADR 0003"; como norma, fica confiável.)

### O que **não** fazer (pés no chão)

- Não uniformizar rigor — gate pesado em mudança de baixo raio é desperdício.
- Não fazer *review theater* — revisão sem poder de mandar o artefato **de volta** é custo sem valor.
- Não adotar pipeline de N comandos nem uma `constitution.md` que duplique os ADRs.
- **Instrumentar antes de formalizar**: o mesmo vale pra cada skill nova do fluxo — `keelson-deploy` e `keelson-fix` nascem nesta leva como desenho (`draft-para-testar`), sem promoção nenhuma, até o rastro do projeto (`wiki/log/`, um `tasks-fase<N>`/`fix-<slug>.md` real) mostrar o que pegam e o que deixam passar.

---

## Onboarding de um agente — como "explicar" isto ao LLM

A pergunta certa não é *"como eu re-explico brief→plan→tasks, eixos, etc. para o LLM a cada sessão?"* — é *"como faço o LLM descobrir isso sozinho?"*. **Você não re-explica; você aponta.** Se você sente que precisa re-explicar em prosa, é o sintoma de que o processo não está apontável num lugar único — e é justamente o que este documento resolve.

**Setup, uma vez por projeto:** o **Tier 0** do projeto tem uma linha que manda o agente começar pela wiki e seguir este processo. Ex.:

```
> Comece por `wiki/index.md`. Para trabalho de nova funcionalidade (brief/plan/tasks, estados,
> promoção), siga o processo em `.keelson/llm-dev-flow.md`.
```

**Ordem de leitura fria de um agente** (do mais barato/geral ao mais específico):
1. **Tier 0** → 2. `wiki/index.md` (mapa) → 3. `wiki/glossary.md` + `wiki/now/<branch>.md` (vocabulário + foco atual) → 4. o `brief`/`plan`/`tasks` da feature em questão, via a seção "Especificações (SDD)" do index.

**Prompt de continuação — minúsculo, porque o contexto vive nos artefatos:**

```
Continue o planejamento da <feature>. Leia wiki/index.md e siga o processo em llm-dev-flow.md.
Estamos na <Fase N> do plan. Primeiro faça a ATERRISSAGEM dessa fase (reconcilie os requisitos do
brief contra o código atual; classifique cada delta em já-existe / colisão / lacuna-de-HOW). Depois
escreva/atualize o tasks-faseN.md dessa fase, detalhando <§ do brief>. Não edite o brief (VALIDATED) —
divergência vira ADR ou entrada no log/.
```

As duas últimas frases são deliberadas: a aterrissagem evita transcrição cega do brief, e o lembrete de não editar o brief reforça a disciplina de spec-rot no próprio pedido — barato. Com o tempo, quando os hooks de `Stop`/`SessionStart` existirem (ver `llm-dev-memory-machinery.md`, "Automação"), até esses lembretes somem — o processo se auto-descreve.

## Versionamento e histórico

A versão corrente está no frontmatter (`schema_version`). O **histórico de mudanças** deste documento vive fora dele — em [`changelog/llm-dev-flow.md`](changelog/llm-dev-flow.md) —, para o padrão ficar limpo como artefato de distribuição. O esquema de numeração (0.x → 1.0 → 2.0) e o índice geral estão em [`changelog/README.md`](changelog/README.md).
