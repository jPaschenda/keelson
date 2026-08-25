---
class: skill
name: keelson-wiki-update
status: draft-para-testar
description: Reconcilia mudanças recentes de código/docs com wiki/ e docs/decisions/ deste projeto — regenera wiki/now/<branch>.md, rascunha entradas de incidente/decisão em wiki/_drafts/ para aprovação, e atualiza o catálogo wiki/index.md. Use quando o usuário pedir "atualiza o wiki", "sincroniza o wiki", "roda o wiki-update", ou depois de uma sessão com edições manuais de docs/ ainda não refletidas na wiki. NÃO use para o fluxo SDD por fase (keelson-plan-init/phase-landing/phase-coding/phase-review/field-validation tocam docs/specs/ e o ciclo brief→plan→tasks; esta toca wiki/ e docs/decisions/, sob demanda, sem amarra a fase) nem para auditoria de saúde (isso é keelson-metrics-snapshot).
---

# Reconciliação do wiki (wiki-update)

Skill **sob demanda**, não hook automático — a ponte nomeada entre a fase 1 (100% manual) e uma fase 2 de
automação ainda não validada (ver `.keelson/llm-dev-memory-machinery.md`, "Skill complementar: `wiki-update`").
Reconcilia o estado real do repositório com `wiki/` e `docs/decisions/` depois de uma sessão de edição manual —
o jeito nomeado de dizer "reconcilia isso com o wiki" que faltava mesmo antes de qualquer hook existir.

## Escopo — o que esta skill faz e onde para

- Reconcilia **uma sessão** (ou o período desde a última reconciliação) com `wiki/index.md`,
  `wiki/now/<branch>.md`, e propõe rascunhos para `wiki/log/` e `docs/decisions/`.
- **Não** é `keelson-metrics-snapshot` (que audita saúde/crescimento — evento próprio, não por sessão).
- **Não** é nenhuma das skills de fase (`keelson-plan-init`/`phase-landing`/`phase-coding`/`phase-review`/
  `field-validation`) — essas tocam `docs/specs/` e o ciclo `brief→plan→tasks`; esta toca `wiki/` e
  `docs/decisions/`, roda a qualquer momento, sem amarra a uma fase.
- **Não commita nada** — escritas de maior risco ficam em rascunho até aprovação humana explícita.

## Passo 1 — Descobrir o escopo da mudança

Branch atual (`git branch --show-current`) e `git diff`/`git log` desde a referência mais recente disponível —
a entrada mais recente em `wiki/log/AAAA-MM.md`, ou o timestamp de `wiki/now/<branch>.md`, o que for mais
recente. **Se nada mudou, avisar e não fazer nada.**

## Passo 2 — Classificar cada mudança relevante

Não é preciso tratar toda mudança — só o que é arquiteturalmente significativo ou digno de registro;
refatoração rotineira não gera nada:

- **Estrutural** (arquivo novo/movido/renomeado em `docs/`, categoria nova) → atualizar `wiki/index.md`
  **diretamente** — baixo raio de explosão, reversível via git, erro aqui é má rota, não falsa verdade. Não
  precisa de aprovação prévia, mas relatar no resumo final.
- **Incidente** (bug corrigido, investigação, condição de corrida, postmortem) → rascunhar uma entrada em
  `wiki/_drafts/log-<timestamp>.md`, formato `## [AAAA-MM-DD] incidente | Título curto`, referenciando
  commit/PR quando existir. **Nunca escrever direto em `wiki/log/`.**
- **Decisão** (escolha arquitetural com trade-off, "por que X e não Y") → rascunhar um ADR em
  `wiki/_drafts/adr-<timestamp>.md`, formato Nygard/MADR (Título, TL;DR, Status, Contexto, Decisão,
  Consequências), com o próximo número sequencial livre em `docs/decisions/`. **Nunca escrever direto em
  `docs/decisions/`.**
- **Promoção estrutural** (um brief de `docs/specs/` ou relatório de `docs/reports/` virando página oficial)
  → **nunca fazer automaticamente** — é decisão humana explícita; no máximo, mencionar que o documento parece
  maduro o suficiente para essa conversa acontecer.

## Passo 3 — Regenerar `wiki/now/<branch>.md` (sempre por último)

Branch, últimos commits, arquivos tocados, "em que se estava trabalhando" (síntese curta), "próximo passo" (se
inferível da conversa/diff). **Sobrescreve o conteúdo anterior** — é efêmero por design, não pede aprovação.

## Passo 4 — Rascunho, nunca commit

Toda escrita em `docs/decisions/`, `wiki/log/` ou edição em `docs/*.md` fica como rascunho em `wiki/_drafts/`
até o humano aprovar explicitamente — só então mover para o destino final e apagar o rascunho.

## Passo 5 — Resumo final ao usuário

O que foi atualizado **diretamente** (índice, `now.md`) vs. o que está **aguardando aprovação** em
`wiki/_drafts/` (um resumo de uma linha por rascunho).

## Regras duras (não viole)

1. **Estrutural pode escrever direto** (`index.md`) — baixo raio, reversível, erro é má rota, não falsa
   verdade.
2. **Incidente/decisão NUNCA direto** — sempre `wiki/_drafts/`, aprovação de 1 clique antes do destino final.
3. **Promoção estrutural nunca automática** — decisão humana; no máximo, sinalize que está madura.
4. **`now.md` sempre regenerado por último**, sobrescreve sem pedir aprovação (efêmero por design).
5. **Nunca commita nada.**
6. **Não edita `.keelson/`** nem o núcleo/trilogia.

## Nota de maturidade

Skill em `draft-para-testar`, mas **já rodando de verdade** — não é só desenho. Nasceu como proposta em
`memory-machinery.md` ("Skill complementar: `wiki-update`"), foi implementada primeiro no adotante (OptiFlux) e
é **promovida à fonte nesta leva** — a seta de volta na direção contrária às demais desta sessão (adotante →
fonte, não fonte → adotante). Resolve a pergunta em aberto da v0.2.1 sobre skills equivalentes a
`mind-ingest`/`mind-query`/`mind-lint`.
