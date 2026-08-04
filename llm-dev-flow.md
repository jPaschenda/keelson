---
schema_version: "0.19"
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
| `plan-<slug>.md` | **O COMO.** Plano de implementação faseado, com **gates** de passagem entre fases (critérios objetivos, não datas). Referencia o brief por seção, não duplica | `plan.md` |
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

Manter o estado no cabeçalho, não no nome do arquivo (`brief-x - DRAFT.md` é anti-padrão — renomear quebra links e histórico de `git`).

### Onde cada eixo mora — e por que `tasks` é por fase

Uma feature não sobe a escada de evidência inteira de uma vez: ela é construída **fase a fase**, e fases diferentes têm níveis de evidência diferentes ao mesmo tempo (a Fase 0 pode estar `BUILT` — ou até em produção, inerte — enquanto a Fase 1 nem começou). Um `tasks` único por feature não conseguiria carregar um `Feature state` coerente no frontmatter. Daí a regra de **onde cada eixo mora**:

- **`brief` e `plan`** carregam o **`Doc Status`** (maturidade do documento de design). Um de cada por feature.
- **`tasks-faseN`** carrega o **`Feature state`** — a *fase* é a unidade que acumula evidência e percorre a escada no seu próprio documento. Um por fase.
- O **estado da feature inteira** é a **fronteira** entre as fases (ex.: "Fase 0 `BUILT`, Fase 1 `NOT_BUILT`").

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

- **`index.md`** ganha uma seção "Especificações (SDD)" — tabela das features por `Doc Status` × `Feature state`, com ponteiros brief/plan/tasks. Um agente entrando frio vê o estado do roadmap num relance.
- **`glossary.md`** colhe o vocabulário no `VALIDATED` (ritual acima).
- **`log/`** registra os marcos: `DRAFT`→`VALIDATED`, cada gate de fase que passa, e cada divergência spec↔código.
- **`now/<branch>.md`** carrega o progresso fino durante a implementação ("Fase 2, faltam os testes de crash do gate 2→3").

## Conexão com a fronteira código↔doc

Alguns specs são também **artefato de runtime** (uma base de conhecimento RAG carregada por um agente — ver a exceção em [`llm-dev-memory.md`](llm-dev-memory.md), seção "`docs/`"). Esses não migram para `docs/specs/`: ficam onde o código os espera e a wiki os indexa. O SDD é a camada prescritiva; quando um spec é *também* código, as duas naturezas coexistem e a localização segue o código.

---

## Sessões: uma sessão = uma transição de estado de um artefato

Uma pergunta que todo mundo que trabalha com LLM sente mas raramente resolve: *quando abrir uma sessão nova?* O processo responde de graça — porque os artefatos e seus eixos de estado **são** as fronteiras de sessão. A formulação:

> **Uma sessão = uma transição de estado de *um* artefato.**

- Sessão do **brief** → move o brief de `DRAFT` para `VALIDATED`.
- Sessão do **plan** → move o plan de `DRAFT` para `VALIDATED`.
- Sessão de **coding da fase N** → move o `tasks-fase<N>` de `NOT_BUILT` rumo a `BUILT`.

Por que isso é natural do ponto de vista do LLM — duas razões, ambas econômicas:

1. **Cada fronteira é uma virada *qualitativa* de contexto.** O que se quer carregado muda quase por inteiro entre os modos: o brief é **divergente** (domínio, RAG, requisitos, conversa); o plan é **estruturante** (o brief fixo + `architecture.md`/`data-model.md`); o coding é **convergente** (o `tasks-faseN` + um punhado de arquivos + o test runner). Misturá-los numa sessão só polui o contexto de cada modo. Sessões separadas mantêm o contexto enxuto.
2. **Cada fronteira deixa um artefato de handoff durável.** Como o agente começa frio a cada sessão, o que importa é retomar sem o histórico da sessão anterior. O artefato (brief `VALIDATED`, `tasks-faseN` com checkboxes) é o "flush" da memória de trabalho (a sessão) para a de longo prazo (a wiki/spec): **a fronteira de sessão é o ponto de descarga.**

A regra prática, que responde ao "quando abrir sessão nova?":

> **Termine a sessão quando o artefato dela atingir um estado nomeável** (uma transição de eixo, ou um gate) — não quando os tokens acabam. **Comece uma nova quando o *tipo* de contexto muda.** Nas fronteiras brief / plan / fase, os dois gatilhos coincidem.

Nuance — é um mapeamento de **tipos** de sessão, não uma contagem literal de 1:1:

- O brief costuma levar **várias** sessões até `VALIDATED` (refinamento iterativo). "Uma sessão de brief" é, na real, "uma fase de trabalho de brief".
- Uma **fase grande** se quebra em mais de uma sessão de coding (por cluster de tasks) — o `tasks-faseN` com checkboxes é o que permite pausar/retomar sem perder o fio.
- Features pequenas **colapsam** brief + plan numa sessão só.

E há transições que a sessão **não fecha sozinha**: `→ BUILT` normalmente exige revisão humana + deploy. A sessão de coding leva o `tasks-faseN` até a *soleira* do `BUILT` (código escrito, testes verdes) e faz o handoff ali — a promoção formal é do operador. (Mesmo com todos os testes verdes, uma fase pode se manter em `NOT_BUILT` — falta revisão + deploy + ADR; o agente não vira a chave sozinho.)

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

Régua de sempre: **proporcional ao raio de explosão** — costura para toda fronteira que sustenta peso, não para um wrapper trivial.

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
| tasks-faseN `→BUILT` | testes verdes + lint/types + greps de invariante | revisão **independente**: fidelidade ao §, invariantes/ADR, qualidade dos testes | agente-revisor (sessão separada) → **humano vira a chave** |
| `→PILOT/PROD` | deploy + health check | crash tests, simulação antes de risco real, runbooks; revisão de segurança | humano |
| merge da memória `→ main` | termo definido 2×; nº de ADR duplicado + refs órfãs pós-renumeração; invariantes textualmente contraditórios | o mesmo termo novo definido de formas *divergentes* pelas branches; decisões em tensão | **guardião** (humano) |

### O merge para a main é uma transição — e tem guardrail próprio *(desenho, pré-validação)*

Com trabalho em branches paralelas, há uma transição que as linhas clássicas da tabela não cobriam: o **merge da memória para a main**. A regra de verdade é da wiki (ver [`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md), "Concorrência"): o que a branch escreveu na camada congelada é *candidato*; canônico é o que chega à main (`now/` é verdade-por-branch; `log/` faz união). O guardrail do merge — última linha da tabela — é o lint semântico já desenhado na wiki, disparado no merge, e proporcional como os demais: um merge que só toca código dispensa a checagem semântica da memória. Quem fecha é o guardião. Status honesto: desenhado a partir das peças existentes, ainda não rodado sob paralelismo real.

### Convenções recomendadas (com atribuição; aplicadas por raio de explosão, nunca dogmáticas)

- **Critério de aceite testável** — no estilo **EARS** (*Easy Approach to Requirements Syntax*, Mavin et al., 2009): requisito escrito como `QUANDO <condição> O SISTEMA DEVE <comportamento>`. Dá ao agente um **alvo objetivo que ele não escreveu** — o antídoto mais direto ao "corrigir a própria prova". Aplicar aos requisitos que importam, não a todos.
- **Teste-primeiro que falha antes do código** — a fase *red* do **TDD** (Red-Green-Refactor, Beck, 2002): para tasks de alto raio de explosão, escrever o teste a partir do critério de aceite, **confirmar que falha**, e só então implementar. Mata o teste tautológico/falso-positivo (este projeto já viu um: teste que reimplementava a lógica em vez de importá-la). Proporcional, não para toda task trivial.
- **Revisão independente em sessão separada** — quem revisa ≠ quem escreveu, com contexto fresco (o modelo de sessão já entrega isso de graça). Ancorada na camada congelada. No harness pode ser o `/code-review` apontado para brief + ADRs; skill dedicada só **depois de medir o que ela pega**.
- **Revisão da *qualidade* dos testes** (não só "passam"): importam a lógica real em vez de reimplementá-la? cobrem os caminhos de risco? não são tautológicos? — ponto que o mercado sub-atende.

### Duas peças que faltavam (as mais valiosas na visão do próprio agente)

- **Índice fino de invariantes** — uma lista *terse* e grep-able dos invariantes duros (ex.: "o EA nunca chama `SocketRead`"; "cálculo financeiro sempre usa o preço real de execução"), cada um apontando para o ADR/dono. É artefato de *memória* (vive na `wiki/`, ao lado do glossário — ver `llm-dev-memory.md`, seção "`invariants.md` — contrato de comportamento"). **Não duplica os ADRs** (eles são o dono); é o *checklist operacional* deles, para o revisor e para os greps mecânicos. É a nossa versão da "constitution" — mas viva e ponteira, não estática.
- **Superfície de incerteza no handoff** — item da *Definition of Done* de **toda** transição: todo artefato entregue termina com *"o que assumi / onde posso estar errado / o que não verifiquei"*. Ataca o viés confiança-mas-errado na raiz — o humano e o próximo revisor sabem onde cutucar. Baratíssimo, e é o que o agente mais pediu de dentro do processo. (A sessão da Fase 0 fez isso meio por acaso — "pendente revisão, ADR 0003"; como norma, fica confiável.)

### O que **não** fazer (pés no chão)

- Não uniformizar rigor — gate pesado em mudança de baixo raio é desperdício.
- Não fazer *review theater* — revisão sem poder de mandar o artefato **de volta** é custo sem valor.
- Não adotar pipeline de N comandos nem uma `constitution.md` que duplique os ADRs.
- **Instrumentar antes de formalizar**: rodar a revisão independente por 2–3 fases, ver o que ela de fato pega, e só então investir em skill dedicada. Um guardrail só entra se, medido, produzir um resultado mais estável no fim — não porque a teoria o acha elegante.

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
