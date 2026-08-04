---
class: skill
name: keelson-plan-init
status: draft-para-testar
description: Cria o plan-<slug>.md a partir de um brief VALIDATED (transição brief→plan) do Método Keelson. Descobre o brief que ainda não tem plano, verifica que está VALIDATED, confirma com o usuário e escreve o plano faseado com gates objetivos, referenciando o brief por §, sem detalhar tasks. Use quando o usuário mandar criar/lançar/iniciar o plano de uma feature. NÃO use para escrever/validar o brief, aterrissar fase (keelson-phase-landing) nem codificar (keelson-phase-coding).
---

# Init do plan

Cria o **plano de implementação** de uma feature — a transição `brief` `VALIDATED` → `plan` `DRAFT`, descrita
em `.keelson/llm-dev-flow.md` (§"três artefatos", §"Sessões"). O plano é o **COMO**: fases com **gates
objetivos**, referenciando o brief por §. **Precondição:** o `brief` existe e está `VALIDATED` (spec fechada).
Você **não** detalha as tasks das fases (isso é a aterrissagem, just-in-time) nem vira a chave de `VALIDATED`
do plano (a revisão é do usuário).

## Escopo — o que esta skill faz e onde para

- Cobre **uma** fronteira: `brief` `VALIDATED` → escreve `plan-<slug>.md` (`DRAFT`).
- **Não** escreve nem valida o `brief`; **não** aterrissa fase (`keelson-phase-landing`); **não** codifica
  (`keelson-phase-coding`).
- **Não detalha as tasks** de nenhuma fase — elas nascem just-in-time na aterrissagem.

## Passo 1 — Auto-descoberta do brief-alvo (antes de escrever o plano)

- **`<slug>` / `<feature>`:** ache o `brief-<slug>.md` em `docs/specs/<slug>/` que **ainda não tem
  `plan-<slug>.md`**. `<feature>` = título H1 do brief. Cruze com `wiki/index.md` (seção "Especificações (SDD)")
  e `wiki/now/<branch>.md`.
- **Precondição — brief `VALIDATED`:** leia o `Doc Status` no frontmatter do brief.
  - `DRAFT` → **pare**: a spec ainda não fechou; o plano não deve nascer sobre um brief instável.
  - inexistente → **pare**: é preciso um brief antes (fronteira anterior).
- **Sem plano ainda:** confirme que `plan-<slug>.md` não existe. Se existir em `DRAFT`, o modo é
  **continuar/atualizar**, não sobrescrever.

Se algum parâmetro não sair com confiança, deixe em branco para o Passo 2 — não invente.

## Passo 2 — Confirmação (gate humano; PARE aqui)

Pare e apresente:

> 🗺️ **Plan Init — alvo**
> - **Feature:** {feature}
> - **Slug:** {slug}
> - **Brief:** docs/specs/{slug}/brief-{slug}.md — Doc Status: {VALIDATED / DRAFT / não encontrei}
> - **Plano existente:** {não / DRAFT — continuar}
>
> Confirma que escrevo o plano? Se o brief ainda está DRAFT, ele precisa fechar (VALIDATED) antes.

**Regra dura:** não escreva o plano sem confirmação, e **nunca** sobre um brief `DRAFT`.

## Passo 3 — Escrever o plan (após confirmação)

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
3. **Feche com a Superfície de incerteza:** "o que assumi / onde posso estar errado / o que não verifiquei".

## Handoff — plano em DRAFT

- O plano nasce `DRAFT`. **Não vire a chave de `VALIDATED`** — a promoção é do usuário, por revisão semântica
  (sanidade de fases/gates/dependências, coerência com o brief).
- Aponte no handoff o que o revisor deve cutucar (gates frágeis, dependências incertas, fases grandes demais).

## Regras duras (não viole)

1. **Não edite o brief** (`VALIDATED`) — divergência vira ADR ou entrada no `wiki/log/`.
2. **Nunca escreva o plano sobre um brief `DRAFT`** — a spec precisa ter fechado.
3. **Não detalhe tasks** — respeita o gradiente de congelamento (é a aterrissagem que quebra fino).
4. **Não edite nada em `.keelson/`** — método vendorizado, pinado.
5. **Não invente** fato/dado/contexto — se faltar, pare e pergunte.
6. **Não vire a chave de `VALIDATED`** do plano sozinho.

## Nota de maturidade

Skill em `draft-para-testar`. A disciplina do plano (faseamento por gates objetivos, referência ao brief por §,
tasks just-in-time) é padrão do `flow.md`; a **automação** (descoberta + gate) é pré-validação. Promova ou pode
conforme o que o **rastro do projeto** (`wiki/log/`, o `tasks-fase<N>`, ou um testemunho de fase) registrar
sobre o que a skill pegou/deixou passar — *instrumentar antes de formalizar*.
