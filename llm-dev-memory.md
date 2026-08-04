---
schema_version: "0.29"
class: core
status: draft-pre-validacao
data: 2026-07-28
supersede: 0.12
---

# Método Keelson Memory

> Inspirado em parte no padrão [LLM Wiki](llm-wiki.md), considerando uma especialização para documentação de projetos de software.
> Onde o LLM Wiki organiza conhecimento pessoal a partir de fontes externas (PDFs, artigos), o Keelson Memory organiza o conhecimento de um projeto de código a partir de si mesmo — arquitetura, decisões, incidentes — com um objetivo adicional que o padrão original não precisa resolver: **coordenar a atualização dinâmica da documentação de um projeto de software, reduzindo a necessidade de envolvimento do programador**, porque aqui o "leitor" mais frequente é um agente de IA trabalhando em sessões curtas e repetidas sobre o mesmo código.

> **Versão: 0.29 — pré-validação.** Conceito maduro, sem piloto instrumentado rodado até o fim. O esquema de versionamento e o histórico vivem em [`changelog/`](changelog/); as perguntas em aberto, em [`wiki/known-issues.md`](wiki/known-issues.md).

---

## A ideia central

A maioria dos projetos de software já acumula documentação em `docs/`: arquitetura, padrões de API, backlog. O problema não é a ausência de documentação — é que ela tende a **misturar dois papéis diferentes no mesmo arquivo**: "como o sistema funciona hoje" (deveria ser estável, editado in-place) e "o que aconteceu e por quê, em que data" (deveria ser cronológico, append-only, ou uma decisão citável e imutável). Quando os dois se misturam, o arquivo de referência só cresce — cada incidente e cada decisão viram um parágrafo novo na cauda — até virar caro demais para carregar numa sessão e difícil de saber qual seção ainda é verdade.

O Keelson Memory separa isso explicitamente em três destinos — referência estável, decisão imutável, incidente cronológico — e adiciona uma camada fina de índice para que um agente (ou um humano) nunca precise ler tudo para saber onde procurar.

## `docs/` × `wiki/` — a divisão de papéis

Dois diretórios com papéis distintos: **`docs/`** guarda o *conteúdo* — páginas de referência e decisões (current-state); **`wiki/`** guarda só *índice + log de incidentes + retomada* (o mecanismo Karpathy puro). As páginas moram sempre em `docs/`; o `wiki/` nunca as contém.

> **Agnóstico a ferramenta.** Um dos três padrões do núcleo: fala em **papéis**, não em produtos. As especificidades de cada ferramenta vivem nos *application guides* — ver [`llm-dev-README.md`](llm-dev-README.md), o firewall.

## Arquitetura: três camadas, ordenadas por custo de carregar

```
<projeto>/
  <tier-0>               # Tier 0 — arquivo raiz sempre carregado (nome por ferramenta; ver application guide)
  wiki/                  # Tier 1 barato — índice + glossário + invariantes + known-issues + retomada; log/ é lido sob demanda
    index.md
    glossary.md          # contrato de vocabulário canônico — um termo = uma definição
    invariants.md        # contrato de comportamento — o que o sistema sempre/nunca faz
    known-issues.md      # par transiente do invariants — o que está quebrado/em tratamento AGORA
    now/                 # ponteiro de retomada — o "sempre carregado barato"
      main.md
      <branch>.md
    log/                 # append-only; consultado sob demanda, não carregado inteiro
      2026-07.md
    _drafts/             # rascunhos aguardando aprovação (ver satélite -machinery)
    watch.json           # config: quais caminhos disparam rascunho automático
  docs/                  # Tier 2 — conteúdo, carregado só quando o índice aponta pra lá
    architecture.md
    decisions/
      0001-titulo.md
    specs/               # prescritivo forward — o que vai ser construído (brief/plan/tasks)
    fixes/               # prescritivo corretivo — como se conserta o presente (fix-<slug>)
    reports/
    ...
```

A regra de ouro: um agente nunca deveria ler `docs/*.md` inteiro sem antes ter passado por `wiki/index.md`. E o **Tier 0** nunca deveria crescer para absorver o que já cabe em `wiki/index.md` — ele é para regras críticas e comandos, não para conhecimento profundo.

**Precisão sobre o "barato" da Tier 1**: "quase sempre vale carregar inteiro" vale para o índice, os dois contratos, o `known-issues.md` e o ponteiro de retomada (`now/`). O `log/` mora no mesmo diretório, mas *não* segue essa regra — como é *append-only* e cresce sem limite, carregá-lo inteiro a cada sessão seria justamente o desperdício que a arquitetura combate. O log é **consultado sob demanda** (por isso o sharding mensal: abre-se só o mês relevante); quem carrega barato o "onde paramos" é o `now/`.

## Os satélites deste padrão — abra sob demanda

Este documento é o **núcleo sempre-carregado** da memória: o mapa de *onde o conteúdo mora* e as *regras dos contratos* — o que se precisa em **toda** sessão. A profundidade por atividade vive em dois satélites, abertos só quando o modo de trabalho pede — é o princípio de tier deste próprio padrão, aplicado a ele mesmo. São ponteiros poucos e estáveis, e o gatilho é a **atividade**, não um resumo do conteúdo:

- **[`llm-dev-memory-structuring.md`](llm-dev-memory-structuring.md)** — abra ao **adotar ou estruturar** o método num projeto: desenhar/enxugar o Tier 0, ingerir uma fonte de domínio (`docs/domain/`), tratar uma fonte que também é artefato de runtime, configurar o Obsidian, ou escalar a adoção em degraus.
- **[`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md)** — abra ao **construir ou mexer na maquinaria** de memória: automação/hooks, subagentes, concorrência e merge entre branches, `doc-tests`,instrumentação/métricas, ou a skill `wiki-update`.

Se você está **escrevendo código ou spec**, o núcleo abaixo basta — nenhum satélite é necessário.

## `docs/` — conteúdo, esqueleto pré-definido

| Arquivo/pasta | Papel | Regra |
|---|---|---|
| `architecture.md` | Visão geral, fluxos, invariantes | **Editado in-place** — nunca acumula narrativa datada nem decisão |
| `architecture - <subsistema>.md` (ou `architecture/`) | Satélites por subsistema | Criado quando o principal passaria de ~300-400 linhas ou cobriria mais de um subsistema |
| `api-standards.md` | Contrato de API | Editado in-place, opcional dependendo da necessidade do projeto. |
| `data-model.md` | Modelo de dados | Editado in-place, opcional dependendo da necessidade do projeto. |
| `test-strategy.md` | Estratégia de testes | Editado in-place, opcional dependendo da necessidade do projeto. |
| `project-state.md` | Snapshot operacional (deploy, ambiente) — opcional; **o estado**: o que está no ar, quais versões | Editado in-place, sem tabela de histórico. Não confundir com `runbooks/`: aqui é *o que está no ar* (substantivo); lá é *como se muda esse estado* (verbo). Opcional dependendo da necessidade do projeto. |
| xxx-xxx.md | Documentação técnica específica do projeto. | De acordo com as características técnicas do projeto, opcionalmente outros arquivos podem ser incluídos no `docs/` |
| **`decisions/NNNN-titulo.md`** | **Architecture Decision Records** — decisões arquiteturais citáveis | **Imutável após aceito**; só o campo `Status` muda (`accepted` → `superseded by NNNN`) |
| **`specs/`** | **Tier prescritivo/forward** — o que ainda vai ser construído: `brief-<slug>.md` (o QUÊ+PORQUÊ), `plan-<slug>.md` (o COMO, faseado), `tasks-<slug>.md` (quebra por fase), + `backlog.md` | Governado pelo processo — ver [`llm-dev-flow.md`](llm-dev-flow.md). Cada doc carrega dois eixos de estado no frontmatter |
| **`fixes/`** | **Tier prescritivo/corretivo** — como se conserta o presente defeituoso: `fix-<slug>.md` (defeito+reprodução no cabeçalho + causa-raiz + plano de conserto), evoluindo para `tasks` quando proporcional. Irmão corretivo do `specs/` (forward × correção) | Governado pelo processo — ver [`llm-dev-flow-maintenance.md`](llm-dev-flow-maintenance.md). **Nasce já aterrissado**; carrega os dois eixos comprimidos; arquiva em `archive/` ao atingir `PROD` |
| **`archive/`** | Destino terminal: brief/plan/tasks de features já `IMPLEMENTED`, cujo current-state já foi promovido para `architecture.md` | Registro de design histórico — não é current-state; fora do caminho de leitura normal |
| `reports/` | Pré-síntese: relatórios externos, auditorias | Fontes primárias ainda não digeridas |
| **`domain/`** | **Fontes de domínio** — regulamentos, legislação, normas, artigos e fundamentos externos; o *terceiro tipo de fonte* (autoridade **externa**, congelada por quem está fora do projeto). `domain/source/` guarda a fonte crua imutável; a página-resumo (síntese + extração de conformidade) mora na raiz de `domain/` | Ver satélite [`llm-dev-memory-structuring.md`](llm-dev-memory-structuring.md). A fonte crua nunca é editada; o controle temporal e a curadoria vivem na página-resumo, que promove seletivamente para `glossary.md` (definições) e `invariants.md` (obrigações) |
| **`runbooks/`** | **Manobras operacionais do desenvolvimento** — roteiros *imperativos* de como se opera este sistema (deploy, rotação de chave, restart de serviço) **+ os guardrails colados a cada passo** ("só rotaciona na janela dupla"). Absorve o resíduo fino de política mole que não é invariante nem ADR ("PR precisa de 1 review") como inquilino minoritário, prefixo `policy-*.md` | Balde escolhido por **âncora, não forma**: procedimento ancorado numa *operação* entra aqui; procedimento ancorado num *subsistema* (troubleshooting de um módulo — sintoma→causa→fix) fica **satélite de `architecture.md`**, mesmo sendo procedural. Cresce devagar; seção em `wiki/index.md`, sem índice próprio até ~10 arquivos. **Substitui `rules/`, aposentado na v0.24** |
| **`general/`** | Miscelânea: entregáveis/artefatos que não se encaixam em nenhuma outra categoria (manuais de usuário, identidade visual, mockups, ferramentas auxiliares) | Não é conhecimento sintetizado — só existe para não poluir a raiz de `docs/`; não precisa de convenção de conteúdo, é catch-all deliberado |

> **Duas fontes têm tratamento próprio no satélite [`llm-dev-memory-structuring.md`](llm-dev-memory-structuring.md)** — a *exceção runtime-artifact* (fonte que o código carrega) e a `docs/domain/` (regulamento/norma externa) — porque instanciá-las é atividade de estruturação, não de código.

### `docs/decisions/` — ADR (Architecture Decision Record), separado de incidentes

Formato (Nygard/MADR): **Título, TL;DR, Status, Contexto, Decisão, Consequências**. A convenção de TL;DR na primeira linha (ver "Convenções de página" abaixo) vale para ADRs também, mesmo não sendo parte do formato Nygard/MADR original — aprendido na prática (v0.2.1 esqueceu de aplicar aos dois primeiros ADRs do OptiFlux, corrigido depois). Numeração sequencial, nunca reaproveitada. Diferente de um incidente: uma decisão é referenciada por ID por anos, então precisa ser estável e imutável — misturá-la num log cronológico mensal perde essa propriedade.

**Imutável ≠ eterna**: imutável é o *registro*, não a validade da decisão — uma decisão não prescreve por envelhecer. O que a encerra é sempre *outra* decisão (revogação ou refinamento), tomada quando a evidência do desenvolvimento confirma ou desmente as bases que a sustentavam, e documentada com o mesmo rigor. Mecanicamente é só o campo `Status`: o ADR antigo nunca é reescrito nem apagado, passa a `superseded by NNNN` e continua legível. Congela-se o registro, não o julgamento.

Critério de quando vale um ADR (para não virar ADR de tudo): *"um novo desenvolvedor precisaria disso para não reabrir um debate já resolvido, ou repetir um erro já corrigido?"* Se sim, ADR. Refatoração rotineira, não.

Exemplos reais já identificados no OptiFlux, hoje enterrados como parágrafo de explicação técnica dentro de `architecture.md`, e que deveriam ser ADRs citáveis:
- *"TCP_Client NUNCA chama `SocketRead`"* — decisão de arquitetura PUSH-only motivada por bug do Wine.
- *"CANCEL ≠ RETIRE, nunca reaproveitar um pelo outro"* — separação deliberada de comandos.

**Limitação conhecida — colisão de numeração entre branches**: duas branches paralelas podem criar `NNNN` iguais de forma independente (ex.: duas features criando `0007-x.md` cada uma). Não vale resolver com infraestrutura — é aceito como um conflito de merge comum, resolvido por renumeração manual na hora do merge, igual a qualquer outro conflito de arquivo novo.

## O processo (`brief`→`plan`→`tasks`) — em `llm-dev-flow.md`

A camada prescritiva/forward — o fluxo brief`→`plan`→`tasks`, os eixos de estado, a escada de evidência, o gradiente de congelamento, a disciplina anti-spec-rot — é o **[`llm-dev-flow.md`](llm-dev-flow.md)**, que **usa** esta memória (dependência de mão única: o processo usa a wiki; a wiki funciona sem o processo). O que permanece aqui, porque o processo o consome como substrato: o esqueleto de `docs/` (incluindo `specs/` e `archive/`), o formato de **ADR**, e o `glossary.md` (colhido no `VALIDATED`).

## `wiki/` — índice, log de incidentes e retomada de sessão

### `index.md`

Catálogo de `docs/`, não duplica conteúdo. Cada entrada tem: resumo de uma linha; **"Consultar para" / "NÃO consultar para"**; última atualização; e ponteiros para o **glossário** e os **invariantes** (`wiki/glossary.md`/`wiki/invariants.md`, seções próprias abaixo). Ganha também uma seção própria **"Decisões (ADR)"**, catalogando `docs/decisions/` por número + status — porque ADRs acumulam e precisam de índice próprio, assim como as páginas de referência. E uma seção **"Manutenção"** — o `known-issues.md` e os `fix-<slug>` em voo de `docs/fixes/` —, para o agente frio ver num relance o que está quebrado e o que está em conserto.

### `glossary.md` — contrato de vocabulário canônico

Artefato Tier-1 de primeira classe (barato, quase sempre vale carregar junto com o `index.md`). É o **único** lugar onde cada termo de domínio tem sua definição canônica. Nasceu como uma seção "glossário de entidades" dentro do `index.md` (v0.7) e foi promovido a arquivo próprio quando ficou claro, no campo, que ele é o *mecanismo anti-divergência* que faz o resto do padrão fechar — não um apêndice do índice.

O raciocínio: um agente trabalha em sessões curtas e repetidas; sem um vocabulário fixo, ele re-deriva ou diverge conceitos entre sessões, e docs diferentes acabam definindo o mesmo termo de formas sutilmente incompatíveis. Isso não é hipótese — no OptiFlux o mesmo termo `perna`/`leg` era usado com granularidades incompatíveis entre RAG, EA e schema do DB, e a sigla `ICC` estava expandida de **duas formas diferentes** em dois documentos (`Índice de Cobertura de Caixa` no código/`FinancialService.js`, mas `...de Custódia` no índice do wiki). Divergência de conceito entre docs vira bug; o glossário existe para eliminá-la.

Regras:

- **Um termo = uma definição.** Se um doc define um termo diferente do glossário, é o doc que está errado (ou o glossário desatualizou — nesse caso corrige-se aqui primeiro e propaga).
- **O glossário não duplica o dono.** Cada verbete é uma linha curta + link para o documento (ou o código) que **possui** o tratamento profundo. Onde há evidência em código, o dono é o código: no OptiFlux, a taxonomia de execução tem como dono `data-model.md §Taxonomia` e os termos financeiros têm como dono `FinancialService.js` (cada fórmula referencia sua seção de origem). O glossário fixa o significado e aponta; não vira uma segunda cópia divergível.
- **Todo caminho é link de verdade** (não menção em backtick) — o glossário é um hub natural do grafo do
  Obsidian: cada termo puxa uma aresta para o dono.

**Validação em dois níveis: verificação mecânica e semântica** (a razão de ser do glossário, não um extra):

1. *Verificação mecânica* (barata, roda em hook/CI sem LLM): nenhum termo definido duas vezes; sem verbetes duplicados; todo link do glossário resolve para um arquivo/âncora existente.

2. *Verificação semântica* (o que o operador realmente quer — "não haver divergência de conceito"): detectar quando um doc usa ou redefine um termo de forma inconsistente com o glossário. Isso **é** trabalho de LLM — o lugar dele é a skill `wiki-update`/lint, não um hook mecânico. É a evolução natural do `mind-lint`do padrão pessoal, aplicada a vocabulário.

### `invariants.md` — contrato de comportamento

Artefato Tier-1, **par do glossário**: se o `glossary.md` é o contrato de *vocabulário* (o que os termos significam), o `invariants.md` é o contrato de *comportamento* (o que o sistema **sempre/nunca** deve fazer). São os dois lados da mesma moeda anti-deriva — um impede deriva *conceitual*, o outro *comportamental* — e ambos alimentam os guardrails de revisão (ver `llm-dev-flow.md`, "Guardrails por transição").

É uma lista *terse* e grep-able dos invariantes duros do projeto. Cada invariante é **uma regra checável + link para o dono** — normalmente um ADR, às vezes uma regra inviolável no código ou em `architecture.md`. Não re-argumenta a decisão (isso é do ADR); só enuncia a regra numa linha e aponta.

O que **qualifica** como invariante (senão a lista incha): uma regra cuja violação é bug ou quebra de segurança/correção, que vale em todo o código e ao longo do tempo, e que **um agente poderia violar sem perceber**. Teste: *"uma violação silenciosa disto causaria um incidente real?"* Se sim, invariante. Convenção local de time → `docs/runbooks/` (política mole, como inquilino minoritário `policy-*.md`; `rules/` foi aposentado na v0.24). Decisão pontual sem regra contínua → fica só como ADR.

Regras (espelham o glossário):

- **Um invariante = uma afirmação checável + o dono.** Frasear como regra dura ("X **nunca** Y", "**sempre**
  Z"), não como parágrafo.
- **Não duplica o dono.** O ADR (ou o código) carrega o *porquê*; o `invariants.md` é o *checklist operacional* dele. Quando um ADR é aceito com uma regra "nunca/sempre", extrai-se a linha para cá, apontando de volta — mesmo ritmo de promoção do glossário. **Não é espelho 1:1 dos ADRs**: nem todo ADR vira invariante, e há invariantes que vêm do código, não de um ADR formal.
- **Marcar o nível de checagem** (alimenta as duas verificações do guardrail): *mecânica* quando a regra nomeia um token grep-able (um hook confere sozinho); *semântica* quando exige julgamento (revisor LLM/humano).
- Mantido **in-place** (current-state, editável — não append-only): quando um ADR é superseded, seu invariante é atualizado ou removido.

Formato (exemplo real do OptiFlux):

| Invariante | Checagem | Dono |
|---|---|---|
| O EA **nunca** chama `SocketRead` (telemetria é PUSH-only) | mecânica — grep `SocketRead` no código do EA | ADR 0001 |
| Cálculo financeiro **sempre** usa `strategy_legs.price` (execução real), nunca `spread_entry/exit` (meta) | semi-mecânica | `FinancialService.js` (regra inviolável) |
| `orch_tactic_id` **nunca** ganha `FOREIGN KEY` (a tática dona vive fora do Core) | mecânica — grep no schema | ADR 0003 (a extrair) |

É a nossa versão da "constitution" do spec-kit / dos steering do Kiro — mas **viva** (mantida pelos rituais de promoção, não curada à mão em paralelo) e **ponteira** (não duplica os ADRs). Por isso o critério de revisão não envelhece: reflete o que o sistema realmente é.

### `known-issues.md` — ponteiro do que está quebrado/em tratamento (transiente)

Artefato Tier-1, **par transiente do `invariants.md`**. Se o `invariants.md` é o contrato do que o sistema **sempre/nunca** faz (durável, current-state), o `known-issues.md` é o ledger do que está **quebrado ou limitado agora e em tratamento** nos degraus vivos (`PILOT`/`PROD`). Existe por um leitor específico: o **agente que começa frio** — para ele não gastar sessão redescobrindo um bug já conhecido, nem construir sobre comportamento bugado achando que é correto.

O tracker externo (issue tracker do projeto) é dono da **lista completa + triagem** (aberto/atribuído/prioridade/backlog/fechado); este arquivo **não o duplica** — é a **projeção curada para o agente**, in-repo e *grep-able*, com só o subconjunto que muda **como o código deve ser escrito agora**. É a mesma relação que o `invariants.md` tem com os ADRs: projeção, não cópia.

Regras (espelham o `invariants.md`, com uma inversão-chave — transiente, não durável):

- **Puramente transiente, mantido in-place.** A linha **sai** quando o problema resolve — seja por **fix embarcado** (ver [`llm-dev-flow-maintenance.md`](llm-dev-flow-maintenance.md)), seja por **decisão *won't-fix***. Não acumula histórico — isso é do `log/`.
- **Nada durável mora aqui.** Todo aprendizado assenta nas casas existentes: a **lição** → ADR (o critério já cobre *"repetir um erro já corrigido"*); a **limitação assumida** → ADR (é decisão com rationale) e/ou`invariants.md` ("o sistema **nunca** faz X") e/ou nota no `architecture.md` (o as-built inclui seus limites); o **incidente** → `log/`. Reter o já-resolvido aqui incharia o arquivo e mataria o valor *grep-able* — o mesmo modo de falha que o `invariants.md` teria guardando invariantes já *superseded*.- **Não é um Jira.** Sem *assignee*, sem prioridade, sem fluxo de status/colunas. Só: o defeito/limitação conhecida + o **raio** + um **ponteiro** (para o item do tracker, a entrada de `log/`, ou o `fix-<slug>.md`  que o trata).
- **Sai por cobertura.** Antes de remover uma linha, o mesmo reflexo do *coverage-first*: o que precisava assentar já assentou no destino durável?

Formato (uma linha por problema, *terse* e *grep-able*):

| Problema | Raio | Estado / ponteiro |
|---|---|---|
| GUI mostra saldo desatualizado após cancelamento (cache não invalida) | baixo | em tratamento → `docs/fixes/cache-saldo/` |
| Motor não aceita ordens fracionadas | médio | em avaliação — decidir fix × limitação assumida |

Catalogado no `wiki/index.md` sob a seção **"Manutenção"** (par da "Especificações (SDD)"). A curadoria — o que entra, quando uma linha já pode sair — é do **jogador** (ver [`llm-dev-player.md`](llm-dev-player.md), vigília da Estrutura). Status: desenho, pré-validação — reaproveita as camadas existentes (Tier-1 *grep-able*, promoção seletiva, coverage-first), sem mecanismo novo.

### `log/AAAA-MM.md`

Cronológico, append-only, sharded por mês desde o início. **Só incidentes/diagnóstico** — decisões arquiteturais vão para `docs/decisions/`. Cada entrada referencia commit/PR quando aplicável. Formato de prefixo consistente, parseável com grep:

```
## [AAAA-MM-DD] incidente | Título curto
```

### `now/<branch>.md`

Ponteiro efêmero de retomada de sessão, **escopado por branch** — não um arquivo único — porque concorrência entre sessões paralelas é real (duas sessões, dois agentes, ou agente+humano trabalhando ao mesmo tempo). Cada branch tem seu próprio arquivo; branches diferentes nunca colidem. Para o caso raro de duas sessões na *mesma* branch simultaneamente: cada escrita carrega `session-id` + timestamp; se o arquivo foi tocado há poucos minutos por um `session-id` diferente, a escrita **anexa** uma nota
em vez de sobrescrever silenciosamente. Sobrescrito (não acumulado) a cada sessão relevante — é o único elemento do sistema que pode ser 100% automático sem aprovação humana (ver "Automação" no satélite [`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md)). Bônus: o diretório `wiki/now/` vira, de graça, uma visão de "quem está trabalhando em quê agora".

### `_drafts/`

Rascunhos gerados automaticamente (entradas de log ou ADR candidatas) aguardando aprovação de 1 clique.

Nunca commitados diretamente pelo hook — ver "Automação" no satélite [`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md). **Ciclo de vida explícito**: aprovado → move o
conteúdo para o destino final (`wiki/log/AAAA-MM.md` ou `docs/decisions/NNNN-titulo.md`) e apaga o rascunho; rejeitado → apaga sem mover. Um rascunho nunca fica pendurado indefinidamente — `SessionStart`avisa sobre pendências justamente para forçar essa decisão logo na próxima sessão.

## Convenções de página

- **TL;DR na primeira linha** após o título de cada página de `docs/` — permite `grep -A1 "^# "` em todo `docs/` e ter uma visão quase completa do sistema sem abrir nada.
- **Frontmatter leve**: tags + "verificado contra commit `<hash>` em `<data>`" — permite lint mecânico de obsolescência sem reler código.
- **Regra de tamanho**: página passou de ~300-400 linhas ou passou a cobrir mais de um subsistema → divide em satélite + entrada no índice.
- **Fronteira dura**: o wiki (`wiki/` e `docs/`) nunca sintetiza o que `git log`/`git blame` já responde. Antes de escrever qualquer coisa, perguntar: "isso se perde se eu não escrever, ou o código já conta essa história sozinho?"
- **Todo caminho de arquivo é um link de verdade, nunca só menção em backtick**: `wiki/index.md` e demais páginas devem referenciar outras páginas via `[texto](caminho/relativo.md)`, não `` `caminho/relativo.md` `` solto. Achado de campo: um índice cheio de menções em backtick não gera nenhuma aresta no grafo do Obsidian — o catálogo existe em texto mas o vault fica com um grafo pobre,  a maioria das páginas aparecendo como órfã (ou nem parecendo, se `showOrphans` estiver desligado). Isso mina justamente o objetivo de usar o índice como hub visual de navegação.
- **Filtro de rigor**: toda convenção emprestada de um framework estabelecido (ADR, Diataxis, etc.) só entra se puder ser justificada numa frase de custo-benefício testável no projeto real — não porque "é o padrão certo". Esta é uma ferramenta de campo de batalha, não um exercício acadêmico; os dois exemplos de ADR acima passam nesse filtro porque resolvem dor já observada no OptiFlux.

## O que não entra na memória (segurança)

As demais seções tratam do que a memória contém; esta trata do que ela **não pode conter** — e é fácil de esquecer, porque toda a disciplina do padrão empurra na direção contrária (registre, escreva, alimente).

Dois princípios:
- **A memória guarda o *conhecimento* do sistema, nunca os seus *segredos*.** A wiki é relida sessão após sessão e enviada, a cada leitura, para o provedor do modelo — o que entra nela, viaja. Segredos, credenciais, chaves e dados pessoais de usuários não pertencem a `wiki/` nem a `docs/`: segredo mora em cofre (vault, `.env` fora do versionamento), e o documento, quando precisa, **aponta** ("a credencial vive em tal cofre") sem nunca copiar o valor.
- **A memória é uma superfície de ataque.** O agente executa o que lê com confiança — inclusive o que não deveria estar lá: uma instrução errada (ou plantada) num documento carregado será seguida com a mesma convicção que uma decisão legítima. As mitigações já existem no padrão e ganham aqui a leitura de segurança: o **raio de explosão** (nada entra nas casas permanentes sem aprovação humana — ver "Automação") e a **curadoria do guardião** (saber *de onde veio* cada coisa que a memória afirma — ver [`llm-dev-player.md`](llm-dev-player.md), vigília da estrutura).

Status honesto: o princípio está de pé; a prática endurecida (quarentena, auditoria de proveniência) ainda é jovem no campo — uma fronteira ainda aberta.

## Versionamento e histórico

A versão corrente está no frontmatter (`schema_version`). O **histórico de mudanças** deste documento vive fora dele — em [`changelog/llm-dev-memory.md`](changelog/llm-dev-memory.md) —, para o padrão ficar limpo como artefato de distribuição. O esquema de numeração (0.x → 1.0 → 2.0) e o índice geral estão em [`changelog/README.md`](changelog/README.md). 

As perguntas em aberto do método, que antes fechavam este arquivo, migraram para [`wiki/known-issues.md`](wiki/known-issues.md).

## Nota final

Este documento é intencionalmente abstrato e incompleto — descreve o padrão, não uma implementação fechada. A estrutura exata de `docs/`, o conteúdo dos hooks, o formato das páginas — tudo isso deveria ser co-evoluído em conversa com o agente, dentro de cada projeto específico, e não copiado cegamente. Use como ponto de partida.