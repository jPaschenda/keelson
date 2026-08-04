---
schema_version: "0.4"
class: application-guide
tool: Claude Code
adapta:
  - memory 0.23
  - flow 0.14
  - player 0.6
manufacturer_reference: llm-dev-claude.log.md (capturas datadas — auto-descrição + observável; o leitor mantém)
clouple: llm-dev-claude.log
verified_in: 2026-07-23
validation: por-linha (ver coluna Confiança; default do guia — campo)
status: draft-pre-validacao
data: 2026-07-22
---

# Método Keelson — Application Guide: Claude Code

> **O que é:** a camada fina que liga os **papéis** da trilogia (memory / flow / player) aos **mecanismos
> concretos do Claude Code**. É o adaptador — não um tutorial de Claude Code, nem um resumo da doc da
> Anthropic. Para cada coisa que o método precisa, este guia diz *como o Claude Code entrega* e *onde ela
> não consegue* (os buracos).

## Sobre esta classe (contrato — todo application guide segue este molde)

- **Frontmatter carrega duas coordenadas + validação.** `adapta:` = quais versões do padrão este guia
  implementa (coordenada do método). `verificado_em:` = contra qual estado da ferramenta foi conferido
  (coordenada da ferramenta). `validacao:` é agora **por linha** (coluna Confiança abaixo), não um selo único
  pro guia inteiro. As duas coordenadas são o que torna auditável "só mexemos no guia".
- **O corpo é um *mapeamento por papel*, não um resumo.** As **linhas** são as necessidades do método
  (estáveis); as **células** são os mecanismos da ferramenta (mudam com a ferramenta). Resumir a doc do
  fabricante seria uma cópia pior que envelhece no dia seguinte — o valor está no *mapeamento* e nos
  *buracos*.
- **Aponta para a evidência externa, não a copia.** A verdade da ferramenta **não é uma doc baixada** — é
  **captura datada**: a auto-descrição da ferramenta + o observável de um comando (`--help`/schema), morando
  no **log frio** [`llm-dev-claude.log.md`](llm-dev-claude.log.md). Este guia quente aponta; o leitor mantém
  as capturas; re-deriva quando a ferramenta muda (ver o fim). Separar quente/frio evita reinstalar o
  monólito dentro do guia.
- **Confiança por linha (obrigatória).** Cada linha carrega `campo` / `observável` / `desk`. O observável
  (`--help`/schema) **ganha só na superfície sintática que toca** (nomes de flag/comando/campo); a metade
  semântica não observada **nasce `desk`**. É o "não emprestar credibilidade do forte pro fraco" na
  granularidade da célula.

## Mapeamento por papel

| Necessidade do método (papel) | Mecanismo no Claude Code | Confiança | Buraco / nota |
|---|---|---|---|
| **Tier 0 — arquivo raiz, sempre carregado** | `CLAUDE.md` na raiz; **carga incondicional** (injetado no início de toda sessão). Também aninhado (`sub/CLAUDE.md`) e de usuário (`~/.claude/CLAUDE.md`) | **campo** | Carga sempre incondicional → o *Tier 0 seed* é garantido de graça; não há modo glob-scoped a vigiar. Risco oposto: por sempre entrar, incha fácil → o "teste de inclusão" é 100% disciplina manual |
| **"Aponta, não re-explica" — ponteiro/import** | `@caminho/relativo` dentro do `CLAUDE.md` importa outro arquivo | **campo** | Mecanização literal do princípio. Cuidado: `@import` **puxa o conteúdo para o contexto** — importar demais recria o monólito por outra porta |
| **Tier 1 — índice/contratos/retomada baratos** | Sem "índice" nativo. Você mantém `wiki/index.md`, `glossary.md`, `invariants.md`, `now/` como arquivos comuns e os referencia do `CLAUDE.md` (por `@import` ou ponteiro) | **campo** | O Claude Code não distingue Tier 1 de Tier 2 — a separação por custo é convenção sua, não imposta pela ferramenta |
| **Tier 2 — conteúdo sob demanda** | Arquivos em `docs/`, acessados por `Read`/`Grep`/`Glob` quando o agente decide (ou você aponta) | **campo** | Nenhum — o acesso seletivo é o comportamento default do agente |
| **Automação de manutenção** | **Hooks** em `settings.json`: `SessionStart` (carregar `now/`), `Stop`/`SubagentStop` (rascunhar log, atualizar `now/`), `PostToolUse` (disparar rascunho ao tocar caminhos do `watch.json`), `UserPromptSubmit` | **desk** | Nomes/eventos por doc/schema (observável), mas o *disparo de fim a fim* não foi rodado aqui. Hooks são comando/shell — a lógica *semântica* (o que virou rascunho) segue do agente/humano |
| **Subagentes — veículo de execução** | Subagentes via `Task` e agentes definidos em `.claude/agents/`: fan-out de leitura, execução isolada | **campo** | Alinha com "subagente é veículo, não 4ª camada". Cada spawn nasce sem contexto → serve para leitura/execução, não para carregar disciplina |
| **Skills — automação empacotada, disparada** | **Skills** em `.claude/skills/` — instruções empacotadas e invocáveis. Onde vivem a skill "update do application guides", um lint de saúde, o bootstrap | **desk** | Gatilho humano (ou hook) — casa com "vigilante só se instrumentado" (player). A skill de update é **desenho, não rodada** ainda |
| **Referência do fabricante (evidência externa)** | Capturas datadas — `claude --help`/schema (observável) + auto-descrição da ferramenta — no log frio [`llm-dev-claude.log.md`](llm-dev-claude.log.md) | **—** | O leitor mantém as capturas; o guia aponta, não copia. Ao mudar, re-deriva (loop abaixo) |

## Os buracos (onde o Claude Code não expressa o papel)

O Claude Code dá os **mecanismos** (arquivo raiz sempre-carregado, imports, hooks, skills, subagentes,
busca seletiva). Mas a **memória viva em si — a separação por tiers, o congelamento por evidência, a poda —
não é nativa**: nenhuma dessas é imposta pela ferramenta. O guia liga os papéis aos mecanismos; **a
disciplina que faz a memória viva é do jogador** (ver `llm-dev-player.md`). Em particular:

- Não há fronteira imposta entre Tier 0/1/2 — a economia por custo de carregar é convenção, não trilho.
- `@import` resolve "apontar", mas nada impede o Tier 0 de inchar; o orçamento é vigiado por você.
- Hooks executam comandos; **não** julgam conteúdo — a decisão semântica volta para o agente/humano.

## Manter este guia vivo (loop de atualização)

A verdade da ferramenta muda na cadência do fabricante — quem a congela é ele, não você. O loop, **caminho
barato (default)**:

1. **Auto-descrição:** pergunte à ferramenta como ela entrega cada papel do método (o molde das *linhas*) +
   a linha aberta "o que mudou / o que não perguntei?".
2. **Âncora observável:** capture o barato e determinístico — `claude --help`, subcomandos, schema de
   `settings.json`, changelog do CLI.
3. **Reconcilie:** onde o observável contradiz a auto-descrição, **o observável ganha — na superfície dele**.
   A metade semântica não observada fica `desk`.
4. Atualize as **células** (nunca as *linhas*) + anexe a captura datada em
   [`llm-dev-claude.log.md`](llm-dev-claude.log.md) + bump de versão. O guia nasce **candidato**.
5. **Teste de fora, em sessão separada:** uma sessão nova exercita o **mecanismo real** das 2–3 linhas que
   sustentam peso (o Tier 0 seed *carrega* mesmo? uma checagem separada *dispara* mesmo?). A independência vem
   do **árbitro observável**, não da sessão nova (mesmo modelo). Passou → carimba `verificado_em:`. Falhou →
   escalada.

**Contra-alucinação (escalada, só sob evidência — não rotina):** falha visível **ou** drift instrumentado →
aterrissa a linha suspeita mais fundo + **diff** contra a última captura do log para localizar (*a ferramenta
mudou* vs *a descrição nasceu errada*).

Atualizar a fonte sem re-derivar o guia = o binding diverge em silêncio. **Esta maquinaria (teste em sessão
separada + escalada) é desenho pré-validação** — vale por dor medida, não como cerimônia obrigatória sobre um
guia que quase não faz churn. A skill que roda tudo isto: `keelson-application-guides-update`.

## Estado

`validacao: campo` — a metodologia é operada em Claude Code no dia a dia (inclusive na construção deste
próprio conjunto). O guia **como artefato escrito** é novo: **pré-validação**, primeira passada. Conferir os
nomes/comportamentos de hooks e caminhos contra a doc vigente antes de tratar como definitivo.

## Versionamento e histórico

A versão corrente está no frontmatter (`schema_version`). O **histórico de mudanças** deste documento vive
fora dele — em [`changelog/llm-dev-claude.md`](changelog/llm-dev-claude.md) —, para o padrão ficar limpo
como artefato de distribuição. O esquema de numeração (0.x → 1.0 → 2.0) e o índice geral estão em
[`changelog/README.md`](changelog/README.md).
