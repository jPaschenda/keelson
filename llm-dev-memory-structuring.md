---
schema_version: "0.29"
class: core-satellite
status: draft-pre-validacao
data: 2026-07-28
satellite_of: llm-dev-memory.md
---

# Método Keelson Memory — Estruturação (satélite)

> **Satélite de [`llm-dev-memory.md`](llm-dev-memory.md), aberto sob demanda.** Reúne o que o agente precisa ao **adotar ou estruturar** o método num projeto — não no dia a dia de escrever código. Abra quando for: desenhar ou enxugar o **Tier 0**; ingerir uma **fonte de domínio** (`docs/domain/`) ou tratar uma fonte que também é **artefato de runtime**; configurar o **Obsidian**; ou planejar a **adoção em degraus**. O núcleo (mapa de `docs/`, contratos) e o roteador dos satélites vivem no [`llm-dev-memory.md`](llm-dev-memory.md).

## O que faz um bom Tier 0

O Tier 0 — o arquivo raiz que o agente carrega em **toda** sessão — já existe na maioria dos projetos e cumpre o papel. Mas "existir" não é "estar bom", e a seção "Arquitetura" do núcleo ([`llm-dev-memory.md`](llm-dev-memory.md)) só deu a regra na negativa (*não inche*). Aqui vai a versão positiva: o que faz um Tier 0 bom, do ponto de vista de quem o lê — o agente, no começo de cada sessão.

**O fato que define tudo: o Tier 0 é o único arquivo que paga aluguel toda sessão.** Todo o resto é sob demanda — o índice da Tier 1 é barato, mas ainda é escolha; a Tier 2 só entra quando o índice aponta. O Tier 0 é lido **independentemente** de ser relevante para a tarefa de agora. O custo dele não é "tamanho", é **tamanho × toda sessão**: uma linha útil 1 em 20 sessões ainda custa 20 leituras.

Disso decorre o **teste de inclusão** — a pergunta única que decide o que entra:

> *Se esta linha não estivesse aqui, o agente erraria por padrão — de um jeito que ele não recuperaria só lendo a Tier 1 no momento em que o assunto aparecesse?*

Se a resposta é "ele descobriria no `index.md` na hora", **não é Tier 0**. O Tier 0 é para o que se precisa saber **antes de saber o que se está fazendo** — os defaults aplicados antes de consultar qualquer coisa.

**Princípios (todos derivam do fato acima):**

1. **Roteador, não biblioteca.** O maior valor do Tier 0 é dizer *onde* o conhecimento mora, não *ser* o conhecimento. Um bom Tier 0 é quase todo ponteiro (arquitetura em `docs/architecture.md`, decisões em `docs/decisions/`, retomada em `wiki/now/`). Todo fato que ele **contém** em vez de **apontar** vai envelhecer no único lugar que não dá para deixar de ler. *Anti-padrão nomeado — o mini-índice.* A forma mais tentadora de biblioteca disfarçada de roteador é uma **tabela de "arquivos-chave"** (key-files, entry-points) dentro do Tier 0: ela *parece* roteamento — são ponteiros! —, e é justamente por isso que engana. É o roteador se fantasiando de biblioteca: duplica o `wiki/index.md` (que **já é** o mapa, na Tier 1 barata) e **cresce com o código** (fere o princípio 3). O índice é o mapa; o Tier 0 **aponta para ele**, não reimplementa um mini-índice no arquivo que não dá para deixar de ler. (É o caso que expõe o limite de "quase todo ponteiro": nem todo ponteiro salva — um *maço* de ponteiros que espelha outro arquivo é biblioteca, não rota.)
2. **Defaults e proibições, não conhecimento.** O conteúdo legítimo é o que precisa ser verdade antes de qualquer leitura: idioma, "rode os testes com X", "nunca faça Y". Guardrails que, se o agente não souber da estaca zero, ele *já violou* quando os fosse descobrir. *A fronteira do comando: comando-do-dia fica; manobra multi-passo aponta.* O comando **atômico** que se invoca reflexivamente toda sessão (`npm test`, o runner certo, o lint) é default legítimo — errá-lo é violar por padrão, e o agente não se recupera lendo o índice "na hora" porque nem sabe que devia procurar. Já a **manobra operacional multi-passo** (o roteiro de deploy, a rotação de chave, o restart do watchdog) **não** paga aluguel toda sessão: não se faz deploy a cada turno, e quando se faz há tempo de abrir a receita. Ela é *conhecimento que muda* — mora num **runbook** (`docs/runbooks/`, Tier 2) e o Tier 0 só **aponta**. Foi a ausência desse balde que fez roteiros operacionais gravitarem para o Tier 0 (no OptiFlux, 115 linhas de "Comandos"): sem lar digno para a manobra imperativa, ela caía no único arquivo sempre-carregado.
3. **Baixa rotatividade é sinal de saúde.** Um bom Tier 0 fica meses quase intocado enquanto o projeto anda rápido — porque o movimento do projeto é registrado na Tier 1/2. Churn no Tier 0 é cheiro de conhecimento (que muda) vazando para dentro dele.
4. **Um diff contra os priors do agente, não um tutorial.** O agente chega sabendo nada deste projeto e tudo sobre software em geral. O Tier 0 não deve ensinar o que é um ADR ou como o git funciona — deve dizer os **desvios específicos deste projeto** em relação ao default. Toda linha que conta algo que o agente suporia sozinho é aluguel desperdiçado.
5. **Ordena pelo custo-se-esquecido.** Mesmo num arquivo curto a leitura decai de cima para baixo; a regra mais cara de violar vai no topo.

**O segundo roteamento — para o *método*, não só para o *projeto*.** Os ponteiros do princípio 1 levam ao conhecimento do **projeto** (arquitetura, decisões, retomada). Mas o agente frio também não sabe, por default, **como se trabalha aqui**: que existe um `brief→plan→tasks`, um caminho de correção de bug, uma disciplina de entrega/registro por sessão. Isso é um *diff contra os priors* (princípio 4) tão legítimo quanto qualquer proibição — o agente que não o conhece **já improvisou** o processo antes de descobrir que havia um. Então o Tier 0 carrega um segundo maço de ponteiros: para a **trilogia do método**, que o projeto adotante mantém pinada em **`.keelson/`** (ver [`llm-dev-package.md`](llm-dev-package.md)) — o fluxo (`llm-dev-flow.md`, incluindo a **correção de bug**), o jogador (`llm-dev-player.md`) e este tabuleiro (`llm-dev-memory.md`). São ponteiros, não conteúdo: passam no teste de inclusão porque **apontam, não copiam**. E são **relativos ao repo** (`.keelson/...`) — um Tier 0 que aponta para o disco do autor do método só funciona na máquina dele.

**A forma do Tier 0: preâmbulo de navegação no topo, projeto depois.** Os dois roteamentos acima (o `index.md` e a trilogia em `.keelson/`), somados aos poucos contratos Tier-1 *always-on* — o glossário, os `invariants.md`, o `known-issues.md` e o ponteiro de retomada `now/` —, formam um **preâmbulo de navegação** que mora no **topo** (no cabeçalho, ou colado a ele). O cabeçalho é onde se organiza *como se opera aqui*; do preâmbulo para baixo vêm as coisas **específicas do projeto** — e essa zona abre pelas **proibições turn-1** (o conteúdo mais caro de violar, princípio 5), seguidas dos comandos-do-dia e das convenções. Um mapa de contratos perdido num rodapé é um mapa que o agente frio não vê **antes de agir**.

Esse mapa de contratos **não é o mini-índice** (princípio 1): o mini-índice é uma tabela de *arquivos de código* que duplica o `index.md` e cresce com o repo; o preâmbulo é um conjunto **fixo** de contratos que se precisa conhecer *antes de saber o que se está fazendo*. O `known-issues.md` ganha ponteiro direto pela mesma razão do seu par `invariants.md`: o agente frio não pode **construir sobre um bug conhecido**, e não descobriria isso só chegando ao `index.md` na hora — porque não saberia que devia procurar. O ponteiro é **estável embora o arquivo seja transiente**: aponta-se para o `known-issues.md`; é o *conteúdo* dele que entra e sai.

**O modo de falha é o monólito — e é um paradoxo.** O Tier 0 tem atração gravitacional: é o único arquivo que todos sabem que será lido, então a tentação constante é "põe aqui pra não perder". Passado certo tamanho, isso se inverte: um paredão de texto é **escaneado**, não lido — e o que deveria ganhar destaque é justamente o que se enterra. Um Tier 0 de 40 linhas é lido; um de 400 é *skimmado*. Há um orçamento implícito, e ele é pequeno. Corolário: quando algo importante nasce, o reflexo "documenta no Tier 0 pra não perder" é o errado — o certo é **uma linha de ponteiro** para onde a coisa mora (*aponta, não re-explica*); duplicar no arquivo sempre-carregado é a pior duplicação de todas.

**Enxugar um Tier 0 que já inchou é uma mini-migração — cobertura antes do corte.** A seção acima diz o que faz um Tier 0 *nascer* bom; mas quando o monólito já existe — um Tier 0 battle-hardened de centenas de linhas —, reduzi-lo não é deletar à vontade. É a mesma retirada disciplinada de um doc redundante, com o mesmo risco real de perda: cada fato do Tier 0 é **grepado contra o destino** (o `index.md`, o `architecture.md`, os `invariants.md`, um `runbook`) *antes* de sair; **o não-coberto migra primeiro**, e só então se corta. Cortar sem conferir cobertura é perder informação no único arquivo cuja falta ninguém nota na hora. O procedimento passo-a-passo vive na Fase A do playbook de migração ([`llm-dev-migration.md`](llm-dev-migration.md)) — aqui fica só o princípio: **de-bloat de Tier 0 monolítico segue o funil coverage-first.**

| Pergunta | Se sim → | Se não → |
|---|---|---|
| O agente erraria por default sem isto? | candidato a Tier 0 | Tier 1/2 |
| Ele descobriria na hora, lendo o índice? | Tier 1 (aponta) | pode ficar no Tier 0 |
| Isto muda com o projeto? | Tier 1/2 (envelhece lá) | Tier 0 aguenta |
| É desvio dos priors do agente, ou tutorial genérico? | desvio fica; tutorial sai | — |

**"Sempre carregado" é uma suposição, não uma garantia.** Este padrão assume o Tier 0 como incondicional. Há ferramentas cujo arquivo-raiz é **carregado condicionalmente** (por glob, ou sob demanda do modelo). Onde isso existe, é *aliada*: empurrar o que é específico-de-caminho para fora da camada sempre-carregada é exatamente o que o padrão prega, agora mecanizado. Mas exige garantir uma **semente incondicional** (o *Tier 0 seed*): mesmo 5 linhas — idioma, proibições, ponteiros — carregadas toda sessão, ou se perde a razão de ser do Tier 0 ("defaults antes de saber a tarefa"). O *como* cada ferramenta expressa o Tier 0 (o nome do arquivo, se a carga é incondicional ou por glob) é assunto do *application guide*, não deste documento.

### Exceção: fonte que também é artefato de runtime (fronteira código↔doc)

Nem todo documento de referência pode — ou deve — ser movido para `docs/`. Alguns são, ao mesmo tempo, **dependência de runtime do código**: uma base de conhecimento RAG carregada por um agente, um template de prompt lido em execução, um schema/config que é doc e insumo. Mover esse arquivo para a pasta de current-state por estética de organização **quebra o caminho que o código espera**.

Regra: *fonte que também é artefato de runtime fica onde o código a espera; o wiki a **indexa** no lugar em vez de movê-la.* O `wiki/index.md` ganha uma entrada apontando para o caminho real, marcando o acoplamento de código explicitamente, e o `glossary.md` extrai dela as definições de domínio — mas o arquivo não migra. No OptiFlux, os `LocalHub/RAG/*` são esse caso: um deles é `require`-ado por `AgentCommander.js` via caminho relativo; foram catalogados numa seção "Base de domínio / especificação" do índice, sem mover. Esse tipo de fonte-de-especificação-que-é-também-código é o gancho natural para a camada prescritiva (Spec-Driven), formalizada em [`llm-dev-flow.md`](llm-dev-flow.md).

### `docs/domain/` — fonte externa, estável, congelada por autoridade de fora (guardrail)

Irmã da exceção acima — as duas são *fontes*, não current-state — mas por um motivo oposto: aquela não migra porque o **código** a prende; esta ganha lugar próprio porque a sua **autoridade vem de fora do projeto**. É a documentação estável do domínio da solução — **regulamentos, legislação, normas, artigos de referência, fundamentos do campo** — que não é artefato de engenharia (fonte-de-verdade *interna*, escrita pelo time), não é spec (o que o time decidiu construir) nem decisão (um porquê que o time possui). É o `raw/` do [`llm-wiki.md`](llm-wiki.md) no seu sentido literal — fonte externa, curada, imutável — que este padrão tinha descartado ao assumir que toda fonte-de-verdade nasce dentro do projeto. Do ponto de vista do software, funciona como **guardrail**: restringe o que o sistema tem permissão de ser.

É o **terceiro tipo de fonte normativa**, ao lado dos dois internos que já existem — e o que o distingue é a *fonte da autoridade*:

| Fonte normativa | Autoridade | Encerra-se por |
|---|---|---|
| ADR / `invariants.md` | interna (o time decidiu) | outra decisão sua |
| `glossary.md` | interna (o time convencionou) | revisão sua |
| **Fonte de domínio** | **externa** | você não a encerra — só cumpre; muda quando a autoridade de fora muda |

**Estrutura** (mini-Karpathy dentro de `docs/`: cru e sintetizado fisicamente separados):

```
docs/domain/
  source/                 # fontes cruas, imutáveis, append-only — nunca editadas (PDF/binário ok)
    resolucao-x-v3.pdf
  resolucao-x.md          # página-resumo: síntese + extração de conformidade (mantida pelo LLM)
```

**Três camadas de conteúdo**, na mesma lógica cru→síntese→promoção do padrão pessoal:

1. **Fonte crua** (`domain/source/`) — a evidência imutável. Não carrega metadado (é PDF/binário) e nunca é editada.
2. **Página-resumo** (`docs/domain/*.md`) — o digest para os dois leitores (humano e agente), sempre apontando para a fonte primária. Para o conteúdo **vinculante**, não basta resumir: faz a **extração de conformidade** — é o que a separa de um livro de biblioteca e a torna guardrail de verdade: > *Resolução X, Art. 12 → reter registros de execução por 5 anos → `data-model.md §retenção` / teste `T` / invariante `INV-014`.*
3. **Promoção seletiva para os contratos**, sob os **mesmos critérios rígidos de entrada** do resto do padrão (senão glossário e invariantes incham e ficam inúteis): conteúdo definicional → `glossary.md` (a fonte de domínio vira mais um *dono* de verbete); conteúdo vinculante → `invariants.md` (vira um **terceiro tipo de dono** de invariante, ao lado do ADR e do código). O resume é o digest completo; os contratos recebem só o que passa a régua — um não substitui o outro (mesmo mecanismo de `architecture.md`→`invariants.md`).

**Papéis (`papel:` no frontmatter do resume) — conjunto, não exclusivo.** `referencia` (educa o agente → alimenta o glossário) e/ou `guardrail` (restringe o código → alimenta os invariantes). **Documentos mistos são normais**, não exceção — uma mesma fonte pode alimentar os dois ao mesmo tempo; o papel vive, na prática, em cada *extração*, não no documento inteiro. Só os itens de `guardrail` recebem o link-de-enforcement do exemplo acima.

**O controle temporal mora na página-resumo — nunca na fonte.** Duas razões que se somam: a fonte costuma ser um PDF (não carrega frontmatter) e, mesmo em texto, editá-la violaria a imutabilidade. Então a camada mantida (o resume) é a **dona única** de tudo o que se afirma *sobre* a fonte, inclusive a data — a ficha de catálogo carrega os metadados do livro, não o livro. Frontmatter do resume:

```yaml
---
title: Resolução X — retenção e custódia
tipo: fonte
papel: [referencia, guardrail]
fonte: docs/domain/source/resolucao-x-v3.pdf
fonte_publicada: 2024-03-15      # do corpo do doc; "sem data na fonte" se ausente — nunca inventar
fonte_vigencia: 2024-06-01       # se o doc disser
capturado_em: 2026-07-21         # quando ingerimos — sempre conhecível
verificado_em: 2026-07-21        # última confirmação de que ainda é a versão vigente
status: vigente                  # vigente | superseded-by <slug>
---
```

Regra honesta das datas: `capturado_em` é sempre preenchível; `fonte_publicada`/`fonte_vigencia` vêm do corpo do documento e **ficam vazias quando ele não se data** — nunca se fabrica uma data.

**Congelado ≠ esquecido.** É a ponta *mais congelada* do gradiente de congelamento (ver [`llm-dev-flow.md`](llm-dev-flow.md)) — muda na escala da legislação, não do sprint; congelada por uma autoridade fora do projeto inteiro. Mas é justamente onde a obsolescência é mais perigosa e menos notada. Mitigação, reaproveitando maquinaria existente: o `verificado_em` + **um gatilho de idade no lint semântico** já desenhado ("esta fonte tem N anos — foi superseded?"). O frescor é checagem **semântica/humana, não mecânica**: não se diffa um PDF escaneado contra a realidade regulatória — o lint *lembra*, não *prova*.

**Versionamento — append-only.** Nova versão da fonte entra como **arquivo novo** em `source/` (a antiga permanece — nunca se apaga evidência); o resume atualiza `fonte:` e `status:` e registra a supersessão. É a mesma disciplina "imutável ≠ eterna" dos ADRs, aplicada à evidência externa.

O resume é catalogado no `wiki/index.md` sob a seção **"Base de domínio"** (a mesma já cunhada na exceção de runtime acima). A responsabilidade de *sourcing* e curadoria de `domain/source/` — que fonte entra, qual versão vale, quando reconferir — é do **jogador**, não do agente (ver [`llm-dev-player.md`](llm-dev-player.md), vigília da Estrutura). Status honesto: desenho, pré-validação — reaproveita as camadas existentes (cru→resume→contratos, lint semântico, gradiente de congelamento), sem mecanismo novo; ainda não instanciado num projeto.

## Obsidian

Se o projeto já usa Obsidian sobre `docs/` (comum), o vault normalmente está na raiz do projeto — o que inclui `node_modules`, `dist`, `coverage` etc. por padrão. Recomendado:

- **Excluir ruído**: `Settings → Files & Links → Excluded files` — adicionar pastas de build/dependência.
- **Aterrissar em `wiki/index.md`**: fixar via plugin `bookmarks` (core) ou, com um plugin de comunidade, `Homepage`, para abrir `index.md` automaticamente.
- **`showOrphans: true`** (`.obsidian/graph.json`) recomendado enquanto o wiki é novo — a maioria das páginas ainda não está interligada, e com `showOrphans: false` elas simplesmente somem do grafo em vez de aparecerem como nós desconectados (o que ao menos sinalizaria "isso precisa de link"). Revisar depois que o vault amadurecer e a maioria das páginas já estiver referenciada pelo índice.

## Por que isso funciona

O mesmo motivo do LLM Wiki pessoal — a manutenção é o que os humanos abandonam primeiro, e LLMs não cansam de atualizar um índice ou revisar uma referência cruzada. Mas em código, o ritmo é mais rápido: o conteúdo muda a cada commit, não a cada fonte nova. Isso exige duas coisas que o padrão pessoal não precisa: (1) uma fronteira dura contra duplicar o que o código já conta sozinho, e (2) um design de automação que trate o "porquê" como algo caro e citável, e o "onde estou agora" como algo barato e descartável.

**A memória não se conserva viva sozinha** (princípio): "viva" não é propriedade da estrutura — é conferida pela manutenção. O `now/` só aponta porque a sessão anterior parou para reescrevê-lo; o `log/` só cresce porque cada sessão deposita nele; documento parado apodrece. Quem anima o tabuleiro é o **fluxo de trabalho** — formalizado no [`llm-dev-flow.md`](llm-dev-flow.md) ou, sem ele, os rituais manuais — sob a vigilância do **jogador** ([`llm-dev-player.md`](llm-dev-player.md)). A dependência de *artefato* continua de mão única (a wiki não precisa do processo para existir); a dependência de *vitalidade* é do trabalho que passa por ela.

## Adoção em degraus

Generalização do princípio de migração incremental (ver o caso OptiFlux abaixo: "incremental, não big-bang"). A adoção é uma escada — cada degrau tem valor próprio e só se sobe o seguinte quando o anterior virou hábito:

1. **Matar o cold-start** (meia hora): `now/` + `log/` do mês + `index.md` mínimo. O retorno aparece na manhã seguinte, e é ele que compra a disciplina para o resto.
2. **Os dois contratos** (primeira semana): `glossary.md` colhido dos termos que o agente já usou torto; `invariants.md` a partir do teste de admissão. Colher dos mal-entendidos reais, não inventar por completude.
3. **O porquê** (primeiro mês): o primeiro ADR se escreve na primeira quase-relitigação — a dor recente garante um porquê que morde.
4. **O fluxo** (na primeira funcionalidade nova de porte): adotar o [`llm-dev-flow.md`](llm-dev-flow.md), estreando inteiro numa feature nova — nunca retrofitado em trabalho em andamento.

Em projeto legado, vale o princípio: **a memória se constrói por demanda, não por mutirão** — o as-built cresce uma página por vez, cada vez que uma sessão precisa entender um subsistema e deixa o entendimento escrito ao sair; histórico antigo fica no git.

## Relação com o `llm-wiki.md`

- `raw/` (pessoal) não tem equivalente direto aqui — o mais próximo é `docs/reports/` e `docs/tasks/`, mas eles não são imutáveis, são só "ainda não sintetizados".
- `wiki/` (pessoal, = índice + log + páginas) vira dois diretórios aqui: `docs/` (páginas + decisões) e `wiki/` (índice + log de incidentes + retomada de sessão).
- O papel de "schema" (o **Tier 0**) é o mesmo nos dois — mas aqui ele já existe na maioria dos projetos antes mesmo de se cogitar este padrão, então o trabalho é mais sobre reorganizar do que criar do zero.

## Caso de validação: OptiFlux (`D:/OptiFlux`)

O primeiro projeto adotante. Os ADRs concretos que ele rendeu (telemetria PUSH-only, RETIRE vs CANCEL) são os exemplos da seção "`docs/decisions/`" do núcleo ([`llm-dev-memory.md`](llm-dev-memory.md)). O princípio que o caso confirma: **migração incremental, não big-bang** — extrai-se só os ADRs já identificados e deixa-se o resto do `architecture.md` como está, e todo conteúdo novo já nasce no lugar certo (`wiki/log/`, `docs/decisions/`, edição in-place); reescrever tudo de uma vez é o que trava a migração antes de começar. Procedimento passo-a-passo: [`llm-dev-migration.md`](llm-dev-migration.md).

