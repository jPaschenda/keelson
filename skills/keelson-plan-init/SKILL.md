---
class: skill
name: keelson-plan-init
status: draft-para-testar
description: Cria o plan-<slug>.md a partir de um brief VALIDATED (transição brief→plan) do Método Keelson — ou revisa por versão um plan já VALIDATED quando uma lacuna de campo (fix-doc, backlog, hipótese refutada) pede uma fase nova. Descobre o brief/plano-alvo, verifica a precondição certa pra cada caminho, confirma com o usuário e escreve ou revisa o plano faseado com gates objetivos, referenciando o brief (e a origem da lacuna, na revisão) por §, sem detalhar tasks. Se a origem for um item de docs/specs/backlog.md, grava também o ponteiro de volta nele (mão dupla), evitando que o backlog fique desatualizado sobre o que já está em andamento. Use quando o usuário mandar criar/lançar/iniciar o plano de uma feature, ou revisar um plano existente por causa de algo achado em campo. NÃO use para escrever/validar o brief, aterrissar fase (keelson-phase-landing) nem codificar (keelson-coding).
---

# Init do plan (ou revisão versionada)

Cria o **plano de implementação** de uma feature — a transição `brief` `VALIDATED` → `plan` `DRAFT`, descrita
em `.keelson/llm-dev-flow.md` (§"três artefatos", §"Sessões") — ou, quando o plano **já existe e está
`VALIDATED`**, faz a sua **revisão versionada** (§"Revisão versionada do `plan`" do mesmo doc), abrindo uma
fase nova que o plano original não previu. O plano é o **COMO**: fases com **gates objetivos**, referenciando
o brief por §. Você **não** detalha as tasks das fases (isso é a aterrissagem, just-in-time) nem vira a chave
de `VALIDATED` — do plano inteiro, ou da fase nova numa revisão — isso é sempre do usuário.

**Dois caminhos, uma escolha de precondição:**
- **(a) Plano novo:** brief `VALIDATED`, sem `plan-<slug>.md` ainda.
- **(b) Revisão versionada:** brief **e** plano ambos `VALIDATED`, e uma lacuna de campo (um `fix-<slug>.md`
  que apontou algo do brief nunca implementado, um item de `backlog.md`, uma hipótese refutada) pede uma fase
  que nenhuma fase existente cobre.

## Escopo — o que esta skill faz e onde para

- Cobre **duas** fronteiras: (a) `brief` `VALIDATED` sem plano → escreve `plan-<slug>.md` (`DRAFT`); (b)
  `brief` e `plan` ambos `VALIDATED`, lacuna de campo → revisão versionada do plano existente (bump de versão,
  fase nova, fases seguintes renumeadas só se nenhuma já tiver `tasks-faseN` materializado).
- **Não** escreve nem valida o `brief`; **não** aterrissa fase (`keelson-phase-landing`); **não** codifica
  (`keelson-coding`).
- **Não detalha as tasks** de nenhuma fase — elas nascem just-in-time na aterrissagem.
- **Na revisão (b), não reescreve nem retoca fase já `VALIDATED`/`BUILT`** — só a fase nova é editável.
- **Se a origem (brief ou lacuna) corresponder a um item de `docs/specs/backlog.md`, anota o ponteiro de volta
  nele** — na mesma passada que escreve/revisa o plano, nunca como tarefa separada (ver Passo 3).

## Passo 1 — Auto-descoberta do alvo (antes de escrever ou revisar)

**Caminho (a) — plano novo:**
- **`<slug>` / `<feature>`:** ache o `brief-<slug>.md` em `docs/specs/<slug>/` que **ainda não tem
  `plan-<slug>.md`**. `<feature>` = título H1 do brief. Cruze com `wiki/index.md` (seção "Especificações (SDD)")
  e `wiki/now/<branch>.md`.
- **Precondição — brief `VALIDATED`:** leia o `Doc Status` no frontmatter do brief.
  - `DRAFT` → **pare**: a spec ainda não fechou; o plano não deve nascer sobre um brief instável.
  - inexistente → **pare**: é preciso um brief antes (fronteira anterior).
- **Sem plano ainda:** confirme que `plan-<slug>.md` não existe. Se existir em `DRAFT`, o modo é
  **continuar/atualizar**, não sobrescrever — e não é nenhum dos dois caminhos formais desta skill.
- **Correspondência com `docs/specs/backlog.md`:** releia o backlog — este brief nasceu de (ou coincide com)
  algum item aberto lá? É leitura semântica, não grep exato (a prosa do backlog não bate 1:1 com slug). Vários
  itens já carregam um sub-campo informal (`Status:`/`Plan:`/`Doc:`) — convenção já emergente em campo, não
  invente outra. Se houver correspondência clara, leve ao Passo 2; se não houver ou for ambíguo, não force.

**Caminho (b) — revisão versionada:**
- Se o usuário apontar uma lacuna de campo contra uma feature cujo brief **e** plano já estão `VALIDATED`,
  confirme primeiro que nenhuma fase existente do plano já cobre aquilo — se cobrir, isto não é o caso (b), é
  trabalho normal de aterrissagem/coding numa fase existente. Se não cobrir, identifique **onde a fase nova
  entra na numeração** (antes, entre, ou depois das fases existentes) e se alguma fase seguinte já tem
  `tasks-faseN` materializado (isso trava a renumeração — ver Passo 3).
- Leia a origem da lacuna por inteiro (o `fix-<slug>.md`, o item de `backlog.md`, a hipótese) — é ela que vai
  virar o ponteiro na fase nova, não uma paráfrase. **Se a origem for item de `backlog.md`, o ponteiro também
  vale de volta** (Passo 3) — no campo `Status:`/`Plan:`/`Doc:` que o item já usa, ou um `Status:` novo se não
  tiver nenhum.

Se algum parâmetro não sair com confiança, deixe em branco para o Passo 2 — não invente.

## Passo 2 — Confirmação (gate humano; PARE aqui)

**Caminho (a) — plano novo.** Pare e apresente:

> 🗺️ **Plan Init — alvo**
> - **Feature:** {feature}
> - **Slug:** {slug}
> - **Brief:** docs/specs/{slug}/brief-{slug}.md — Doc Status: {VALIDATED / DRAFT / não encontrei}
> - **Plano existente:** {não / DRAFT — continuar}
> - **Backlog:** {item (N) de docs/specs/backlog.md, {categoria} / nenhuma correspondência}
>
> Confirma que escrevo o plano? Se o brief ainda está DRAFT, ele precisa fechar (VALIDATED) antes.

**Regra dura:** não escreva o plano sem confirmação, e **nunca** sobre um brief `DRAFT`.

**Caminho (b) — revisão versionada.** Pare e apresente:

> 🗺️ **Plan — revisão versionada**
> - **Feature:** {feature}
> - **Plano:** docs/specs/{slug}/plan-{slug}.md — V{atual} → V{atual+1}
> - **Origem da lacuna:** {fix-doc / item de backlog / hipótese refutada}, apontando pro brief §{X}
> - **Ponteiro de volta no backlog:** {sim, item (N) / não se aplica}
> - **Onde a fase nova entra:** {posição proposta} · **Fases seguintes renumeadas:** {lista, ou "nenhuma —
>   nenhum `tasks-faseN` existente é afetado"}
>
> Confirma a revisão? Fases já `VALIDATED`/`BUILT` não são tocadas; só a fase nova nasce, marcada como
> pendente de revisão humana.

**Regra dura:** não escreva a revisão sem confirmação, e **nunca renumere** uma fase que já tem `tasks-faseN`
materializado — nesse caso, anexe a fase nova ao final ou como sub-fase, sem mexer na numeração existente.

## Passo 3 — Escrever ou revisar o plan (após confirmação)

**Caminho (a) — plano novo:**

1. **Leia o brief `VALIDATED`** (a fonte) e o **current-state** (`wiki/index.md` → `architecture.md`,
   `data-model.md`) — o suficiente para **fasear com realismo**. (A reconciliação profunda requisito × código
   é da aterrissagem, não daqui; aqui você só fasea com noção do que já existe.)
2. **Escreva `docs/specs/<slug>/plan-<slug>.md`** (`Doc Status: DRAFT`):
   - **Fases** de implementação, cada uma com um **GATE de passagem objetivo** — critério de **evidência**, não
     data (ex.: "gate 1→2: endpoint X responde e passa nos testes de contrato"). Liste fases + gates +
     **dependências**.
   - **Referencie o brief por §** — não copie; o brief é a fonte única.
   - **NÃO detalhe as tasks** de nenhuma fase (nascem na aterrissagem, just-in-time — detalhar já congela
     decisão sem chão de código).
   - Se fizer sentido, marque o **recorte de MVP** (quão fina é a primeira fatia vertical) — é propriedade do
     plano.
3. **Se o Passo 2 confirmou correspondência com `backlog.md`**, anote o ponteiro na própria linha do item —
   reaproveitando o campo `Status:`/`Plan:`/`Doc:` que ele já usa, ou criando um `Status:` se não tiver nenhum:
   `docs/specs/<slug>/plan-<slug>.md`. **Não reescreva a descrição do item nem o remova** — ele só sai do
   backlog quando a feature chegar a estado terminal (fora do escopo desta skill).
4. **Feche com a Superfície de incerteza:** "o que assumi / onde posso estar errado / o que não verifiquei".

**Caminho (b) — revisão versionada:**

1. **Leia o plano `VALIDATED` existente por inteiro** e a origem da lacuna (fix-doc/backlog/hipótese) — não
   fasea do zero, edita em cima do que já está congelado.
2. **Bump de versão** (`V{n}`→`V{n+1}`), com uma entrada curta no topo do documento: o que mudou, por quê,
   apontando a origem **por ponteiro** (fix-doc §, item de backlog, brief §) — sem repetir o desenho. **Se a
   origem for item de `backlog.md`, o ponteiro é de mão dupla:** grave também, na mesma passada, o ponteiro de
   volta na linha do item — campo `Status:`/`Plan:`/`Doc:` que ele já usa, ou um `Status:` novo — apontando
   `docs/specs/<slug>/plan-<slug>.md V{n+1}`.
3. **Insira a fase nova**, com **GATE de passagem objetivo** próprio — não herde um gate de uma fase antiga só
   porque é vizinha; derive um específico pra lacuna que motivou a revisão.
4. **Renumere as fases seguintes só se nenhuma delas já tiver `tasks-faseN` materializado.** Se já tiver, não
   renumere — anexe ao final ou como sub-fase (confirmado no Passo 2).
5. **Marque só a fase nova, inline, como "pendente de revisão semântica humana"** — não altere o `Doc Status`
   do plano como um todo, que **permanece `VALIDATED`**; as fases antigas continuam congeladas.
6. **Feche com a Superfície de incerteza escopada só a esta revisão** — não ao plano inteiro.

## Handoff

**Caminho (a):**
- O plano nasce `DRAFT`. **Não vire a chave de `VALIDATED`** — a promoção é do usuário, por revisão semântica
  (sanidade de fases/gates/dependências, coerência com o brief).
- Aponte no handoff o que o revisor deve cutucar (gates frágeis, dependências incertas, fases grandes demais).
- **Feche sempre com o campo estruturado:**
  > **Próximo passo:** depois que você validar este plano, `keelson-phase-landing` — aterrissa a Fase 0
  > (ou a primeira fase do plano) e escreve o `tasks-fase0-<slug>.md`.

**Caminho (b):**
- O plano **permanece `VALIDATED`** — só a fase nova está pendente. Aponte isso explicitamente no handoff, e
  **não vire a chave dessa fase sozinho** (mesma regra dura #6, agora por fase em vez de pelo documento
  inteiro).
- Se o `wiki/index.md` ainda não refletir a fase nova, sinalize — é trabalho de `keelson-wiki-update`, fora do
  escopo desta skill.
- **Feche sempre com o campo estruturado:**
  > **Próximo passo:** depois que você validar a fase nova, `keelson-phase-landing` — aterrissa ela e escreve
  > o `tasks-fase<N>-<slug>.md` correspondente.

## Regras duras (não viole)

1. **Não edite o brief** (`VALIDATED`) — divergência vira ADR ou entrada no `wiki/log/`.
2. **Nunca escreva o plano sobre um brief `DRAFT`** — a spec precisa ter fechado.
3. **Não detalhe tasks** — respeita o gradiente de congelamento (é a aterrissagem que quebra fino).
4. **Não edite nada em `.keelson/`** — método vendorizado, pinado.
5. **Não invente** fato/dado/contexto — se faltar, pare e pergunte.
6. **Não vire a chave de `VALIDATED`** — do plano inteiro (caminho a) ou da fase nova (caminho b) — sozinho.
7. **Na revisão versionada (b), nunca reescreva ou retoque fase já `VALIDATED`/`BUILT`** — só a fase nova é
   editável.
8. **Nunca renumere uma fase que já tem `tasks-faseN` materializado** — anexe, não renumere.
9. **Feche sempre com o campo estruturado "Próximo passo"** — prosa solta apontando pra `keelson-phase-landing`
   já foi ignorada em campo (mesma lição do lado `keelson-coding`/`keelson-fix` com `keelson-deploy`).
10. **Se a origem for item de `docs/specs/backlog.md`, o ponteiro é sempre de mão dupla** — nunca só o plano
    sabe do item. Reaproveite o campo que o item já usa (`Status:`/`Plan:`/`Doc:`) em vez de inventar sintaxe
    nova; não reescreva a descrição nem remova o item — ele sai do backlog só em estado terminal, fora do
    escopo desta skill.

## Nota de maturidade

Skill em `draft-para-testar`. A disciplina do plano (faseamento por gates objetivos, referência ao brief por §,
tasks just-in-time) é padrão do `flow.md`; a **automação** (descoberta + gate) é pré-validação. Promova ou pode
conforme o que o **rastro do projeto** (`wiki/log/`, o `tasks-fase<N>`, ou um testemunho de fase) registrar
sobre o que a skill pegou/deixou passar — *instrumentar antes de formalizar*.

**Dado de campo novo — caminho (b):** a primeira revisão versionada real (Fase 4→5, Camada de Orquestração)
foi improvisada sem esta seção existir — a skill reconheceu sozinha que não era criação de plano do zero (o
próprio `fix-<slug>.md` recomendava "nova fase, ou revisão versionada de uma fase já `VALIDATED`"), bumpou a
versão, renumerou fases sem `tasks-faseN` ainda materializado, e não virou a chave da fase nova sozinha. O
caminho (b) acima só nomeia o que ela já fez.

**Dado de campo novo — usabilidade:** essa mesma sessão fechou sem apontar `keelson-phase-landing` como
próximo passo explícito — o operador notou a ausência. Mesmo defeito, terceira ocorrência no método (já visto
do lado `keelson-coding`/`keelson-review-session` com `keelson-deploy`): prosa solta no meio do handoff não
sobrevive; campo estruturado, sim. Regra dura 9 e o "Próximo passo" nos dois caminhos fecham isso.

**Leva 2026-08-22 — ponteiro de mão dupla com `backlog.md`:** nasceu de drift real achado em campo (OptiFlux)
— dois itens do backlog já estavam implementados ou em fase avançada sem nenhuma linha refletir isso, porque
nada os ligava à fase real que os endereçava. Não corrige o passado (isso foi limpeza manual, fora desta
skill) — fecha a fonte do drift daqui pra frente: quando um plano nasce ou uma fase nova se abre a partir de
um item de backlog, o ponteiro se grava nos dois lados na mesma passada. O formato reaproveitado
(`Status:`/`Plan:`/`Doc:`) não foi inventado agora — já emergia organicamente em pelo menos 2 itens do backlog
real do OptiFlux antes desta regra existir, mesmo padrão de "formalizar o que já emergiu em campo" usado no
formato de veredito do `keelson-review-session`. Regra dura 10.
