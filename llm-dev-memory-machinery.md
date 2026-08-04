---
schema_version: "0.29"
class: core-satellite
status: draft-pre-validacao
data: 2026-07-28
satellite_of: llm-dev-memory.md
---

# Método Keelson Memory — Maquinaria (satélite)

> **Satélite de [`llm-dev-memory.md`](llm-dev-memory.md), aberto sob demanda.** Reúne o design da **maquinaria de manutenção** da memória — o que o agente precisa ao **construir ou mexer** nela, não no dia a dia de escrever código. Abra quando for: desenhar **automação/hooks**, decidir um **subagente**, tratar **concorrência/merge** entre branches, montar **`doc-tests`** ou **instrumentação/métricas**, ou a skill **`wiki-update`**. O núcleo (mapa de `docs/`, contratos) e o roteador dos satélites vivem no [`llm-dev-memory.md`](llm-dev-memory.md). Status geral desta camada: **desenho, pré-validação.**

## Concorrência: várias sessões, uma memória (desenho, pré-validação)

Trabalho em paralelo — vários agentes, worktrees, sessões simultâneas — toca a memória em dois momentos distintos. As respostas para o primeiro já existiam espalhadas pelas peças acima; esta seção as costura e fecha o que faltava para o segundo.

**Durante o trabalho (mesma árvore ou mesma branch):** o `now/` já é escopado por branch, com o protocolo de session-id + anexar-nunca-sobrescrever para a colisão rara na mesma frente (ver "`now/<branch>.md`"). E tudo o que entra nas casas permanentes (log, ADR, edição de `docs/`) passa pelo funil `_drafts/` — um arquivo por rascunho, aprovação humana — que **serializa por construção** as escritas concorrentes: dois agentes nunca disputam o mesmo arquivo permanente; disputam, no máximo, a atenção do guardião. O gargalo humano é deliberado — é o ponto de controle, não um defeito.

**No merge (branches longas):** a regra de verdade — **o que a branch escreve na camada congelada (glossário, invariantes, ADRs, architecture as-built) é *candidato*; canônico é o que chega à main.** O `now/` é verdade-por-branch por definição; o `log/` faz **união** no merge (append-only: as duas histórias sobrevivem); a numeração de ADR já tem a regra de renumeração manual (ver a "Limitação conhecida" em `docs/decisions/`). A peça que faltava é o **lint de merge**: no merge para a main, checagem *mecânica* (termo definido duas vezes, número de ADR duplicado + referências órfãs após a renumeração, invariantes textualmente contraditórios) e *semântica* (o mesmo termo novo definido de formas divergentes pelas duas branches — o `ICC` nascendo em paralelo; decisões em tensão). Não é mecanismo novo: é o lint semântico de glossário já desenhado (ver "Subagentes"), com um gatilho a mais — o merge. Quem fecha é o guardião: no processo, o merge para a main é uma transição de estado com guardrail próprio (ver [`llm-dev-flow.md`](llm-dev-flow.md), "Guardrails").

**O que não construir (pés no chão):** nenhum lock, nenhuma infraestrutura de coordenação distribuída — o git *é* a camada de coordenação; o padrão só adiciona a camada semântica que o git não tem. Status: desenho com evidência parcial (o protocolo do `now/` e o funil `_drafts/` já existiam; o lint de merge não rodou em projeto nenhum) — não formalizar automação disso antes de medir, como sempre.

## Promoção de conteúdo: regra de decisão humana

Existem dois tipos de escrita no wiki, com níveis de automação muito diferentes:

- **Incremental, com gatilho objetivo** (entrada de log a partir de um fix, ADR a partir de uma decisão clara): pode ser auto-rascunhada para aprovação de 1 clique — ver Automação.
- **Promoção estrutural** (um brief DRAFT de `docs/tasks/` vira seção oficial de `docs/architecture.md`, ou um relatório de `docs/reports/` vira base de uma nova funcionalidade): é **sempre uma decisão humana explícita**, nunca auto-sugerida. Não há gatilho objetivo para "isto amadureceu o suficiente" — é julgamento editorial. O agente só participa quando pedido ("ajuda a sintetizar esse brief numa página").

## Automação por raio de explosão

Zero envolvimento do programador é indesejável para tudo que vira "verdade citável depois" — um LLM reconstruindo o "porquê" de uma mudança sem ter participado dela alucina com facilidade. O objetivo certo é reduzir o custo do humano de **escrever** para **aprovar um diff pequeno**.

| Camada | Reversível? | Automação segura |
|---|---|---|
| `wiki/now/<branch>.md` | Sim — sobrescrito toda sessão | 100% automático, sem aprovação, **sem LLM** (template puro a partir de `git diff`) |
| `wiki/log/*.md`, `docs/decisions/` | Não — citado depois como fato | Rascunho automático em `wiki/_drafts/` + aprovação de 1 clique |
| `docs/*.md` (edição in-place) | Não — vira referência confiável | Rascunho automático + aprovação de 1 clique |
| Promoção estrutural (`tasks/`/`reports/` → `docs/`) | Não — decisão editorial | **100% humana**, nunca auto-sugerida |
| Checagem de obsolescência (link quebrado, símbolo renomeado) | N/A — mecânica | 100% automático, sem LLM, roda em CI |

### Hook de `Stop`, em dois estágios

**Pré-requisito antes de implementar**: verificar num spike curto (~10 min) que os hooks `Stop` e `SessionStart` existem e se comportam como esperado no harness específico em uso — o desenho abaixo assume isso, mas não foi validado ainda. Não vale desenhar mais automação em cima disso sem confirmar primeiro.

`Stop` dispara a cada turno do agente, não só no fim da sessão — uma chamada de LLM em todo turno seria cara e lenta. Desenho em dois estágios:

**Estágio 1 — sempre, sem LLM, custo ~0** (script de shell/git puro):
1. `git diff --stat` contra um marcador salvo no início da sessão (`SessionStart` grava o HEAD).
2. Sem diff → sai, custo zero.
3. Regenera `wiki/now/<branch>.md` por template puro (branch, últimos commits, arquivos tocados, timestamp) — **sem chamar LLM**. Git já sabe "o quê" e "quando"; síntese não é necessária pra isso.

**Estágio 2 — condicional, com LLM, só quando o gatilho casa**:
4. Verifica se o diff toca caminhos "quentes" ou se mensagens de commit batem com um padrão — regras lidas de `wiki/watch.json`, schema mínimo: ```json { "hot_paths": ["MT5Server/**/*.mqh", "LocalHub/src/services/*.js", "docs/decisions/**"], "commit_patterns": ["^fix", "^revert", "bug", "race", "incident"] } ```
5. Se sim: chamada headless (não-interativa) rascunha uma entrada de log/ADR em `wiki/_drafts/`.
6. **Throttle**: no máximo 1 chamada a cada N minutos mesmo se o gatilho casar todo turno, para não gerar rascunhos duplicados numa sessão de edição longa antes do commit.
7. **Trava de recursão**: a invocação headless do passo 5 seta uma variável de ambiente (ex.: `WIKI_HOOK_RUNNING=1`) que o próprio script do hook verifica no início — se já estiver setada, sai sem fazer nada. Evita que uma sessão disparada pelo hook dispare o mesmo hook de novo.

**Complemento no `SessionStart`**: ao abrir sessão nova, checar `wiki/_drafts/` e avisar se há rascunhos pendentes de revisão — fecha o loop sem exigir que o humano lembre de procurar.

**Regra dura**: nunca commit automático e silencioso de conteúdo persistente (`log/`, `docs/decisions/`, `docs/*.md`). `wiki/now/` é a única exceção deliberada.

## Subagentes: veículo de execução, não uma quarta camada de automação

Um subagente (sessão isolada, disparada de dentro da principal, que faz um trabalho e devolve só um resumo) **não é uma nova forma de automação** ao lado do hook, da skill e da decisão humana — é um *veículo* para a mesma lógica. A pergunta útil não é "criar um agent para o wiki?", e sim *qual tarefa é agent-shaped*. A lente que decide é a própria tese do padrão — **economia de token na sessão principal**: um subagente roda a leitura pesada (git, `index.md`, `glossary.md`) num contexto descartável e devolve só o resultado, quarentenando esse volume fora da sessão que escreve código. O cold-start, que normalmente é o custo de um agente, aqui é vantagem — reconciliação e lint de wiki *não querem* o contexto da sessão de código.

### Qual tarefa é agent-shaped (decomposição pelo raio de explosão)

| Tarefa | Veículo certo | Por quê |
|---|---|---|
| Regen de `now/<branch>.md` | **Shell puro, sem LLM** | Template de `git diff`; não há nada para "sintetizar". Subagente aqui seria overkill absurdo. |
| Reconciliar + rascunhar log/ADR (`wiki-update`) | **Subagente como *motor* da skill/hook** | Read-heavy; o ganho é isolar essa leitura da sessão principal. Mas continua invocado *de dentro* do `wiki-update`/Estágio 2 — não é um agent de usuário separado. |
| Lint semântico / divergência de glossário | **Subagente dedicado** (melhor encaixe) | Fan-out sobre `docs/` inteiro, não-interativo, devolve só achados. É a verificação semântica do glossário — a única parte que é trabalho de LLM. |
| Promoção estrutural (`tasks/`/`reports/` → `docs/`) | **100% humana, nunca agente** | Julgamento editorial sem gatilho objetivo; já marcada como decisão humana em "Promoção de conteúdo". |

Os dois extremos da tabela — `now/` (shell puro) e promoção estrutural (humana) — **nunca** viram agente; o espaço real é o meio: reconciliação (como motor de algo já existente) e lint semântico (como artefato próprio).

### O melhor candidato: lint semântico de glossário

É o encaixe mais limpo — fan-out de leitura que devolve pouco. A validação em dois níveis do glossário já é a especificação dele: a mecânica (termo duplicado, link quebrado) roda em hook/CI sem LLM; a **semântica** (um doc redefine ou usa um termo divergindo do glossário) é o que o subagente varre, devolvendo achados para decisão humana — nunca corrige sozinho. Generaliza o `mind-lint` do padrão pessoal, aplicado a vocabulário; o precedente de campo que ele teria pego é a própria divergência de `ICC`/`perna` da seção "`glossary.md`".

**Quando promover: não agora.** Desenho, **não implementado** — os hooks nem foram validados no harness e o `wiki-update` só existe em um projeto. O gatilho para promover não é "seria elegante", é **medir** que a fricção real ("o `wiki-update` lê demais e polui a sessão de código") aparece nos números da instrumentação. Instrumentar antes de formalizar — a disciplina do resto do documento.

## Validação de "doc-tests", em camadas

Executar de fato comandos citados nos docs (ex.: `docker compose up -d mt5server` contra produção) é inseguro/impraticável. Validação em camadas, da mais barata à mais arriscada:

1. **Sintaxe/config** (sem executar nada): blocos YAML/JSON/`.env` comparados contra o schema real (ex.: toda variável citada existe em `.env.example`?). Zero risco, 100% automatizável.
2. **Referência** (grep): caminhos de arquivo e símbolos citados ainda existem na árvore atual — pega renomeações e arquivos movidos.
3. **Execução seletiva**: só comandos de baixo risco e sem efeito colateral (`npm test`, `npm run lint`) rodam de fato em CI. Comandos de infra/produção ganham anotação `unsafe-to-execute: true` no frontmatter e ficam restritos à camada 2.
4. **Gatilho reverso**: quando um script citado muda de nome/local em `package.json`/`scripts/`, o CI flagra docs que ainda citam o nome antigo.

## Instrumentação

O padrão se valida com **medida**, não com estética — e a medida sai do que já existe, sem construir nada novo: o histórico do `git` (taxa de crescimento vs. poda de cada página de referência) e os transcripts de sessão (qual arquivo é "quente" de verdade; quanto do contexto vem de cada fonte). Uma calibração que **tempera a tese central**: documentos são uma **fatia pequena** do contexto carregado — a maior parte é **releitura de código** —, então não superestime o peso dos docs na conta de tokens. As métricas *do praticante* (cold-start, relitigação, idade do `now/`, emendas no log, onde as violações são pegas) são do **jogador** (humano), não do agente — ver [`llm-dev-player.md`](llm-dev-player.md).

**Regra dura**: nenhuma automação (hooks, CI) se formaliza num projeto novo antes de rodar essa instrumentação retrospectiva pelo menos uma vez — medir é barato e evita cerimônia que não paga. E não se generaliza a assinatura de um único arquivo: antes de tratar a regra de split (~300-400 linhas) como universal, mede-se em mais de uma página real.

### Onde as métricas são armazenadas

Um número solto no chat se perde. Convenção: `wiki/metrics/AAAA-MM-DD-metric.md`, um arquivo por snapshot, sempre a mesma tabela de colunas (arquivo, commits, linhas +/-, saldo líquido, % nunca podado, linhas atuais, refs em transcripts) — para diffar dois snapshots ser trivial. A data vem primeiro (ordena no grafo do Obsidian); o **baseline** é simplesmente o snapshot de **data mais antiga** (marcá-lo no nome quebraria a ordenação). Captura manual por ora; **não** faz parte do `wiki-update` (que reconcilia uma sessão, não audita a saúde do projeto). **Mínimo obrigatório**: um snapshot antes de promover o projeto de 0.x → 1.0 (esquema em [`changelog/README.md`](changelog/README.md)); cadência contínua recomendada, semanal ou mensal conforme o ritmo. O critério de promoção a 1.0 deve ser **quantitativo** (definido antes: ex. "reduzir X% os tokens das 5 perguntas mais comuns"), não "o piloto pareceu melhorar".

## Skill complementar: `wiki-update` (sob demanda, sem hook)

Achado de campo: mesmo antes de qualquer automação (hooks ainda não implementados em nenhum projeto), a falta de uma forma explícita de **pedir** uma atualização do wiki já é fricção — o humano fez edições manuais e não tinha um jeito nomeado de dizer "reconcilia isso com o wiki". Proposta: uma skill `wiki-update`, invocada por comando (`/wiki-update` ou "atualiza o wiki"), como ponte entre a fase 1 (100% manual) e a fase 2 (hook automático, ainda não validada):

1. Roda `git diff`/`git log` desde a última entrada de `wiki/log/` ou desde o início da sessão.
2. Classifica: é incidente (→ rascunho em `wiki/log/`), decisão (→ rascunho em `docs/decisions/`), ou só uma mudança estrutural (arquivo novo/movido/renomeado) que só precisa atualizar o catálogo?
3. Regenera `wiki/now/<branch>.md` diretamente (raio de explosão baixo, mesmo critério do hook).
4. Atualiza `wiki/index.md` diretamente para mudanças estruturais (novo arquivo, rename, data) — raio de explosão baixo o suficiente (reversível via git, erro é má rota, não falsa verdade) para não exigir aprovação prévia, mas o resumo da execução deve listar o que mudou.
5. Para incidente/decisão: escreve em `wiki/_drafts/`, nunca commita direto — aprovação de 1 clique, igual ao desenho do hook.

Isso resolve a pergunta em aberto da v0.2.1 sobre skills equivalentes a `mind-ingest`/`mind-query`/ `mind-lint`. Ainda não implementada como arquivo de skill real em nenhum projeto — próxima etapa natural depois que a fase 1 estiver rodando por um tempo.

