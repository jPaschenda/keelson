---
schema_version: "0.8"
class: front-door
status: draft-pre-validacao
data: 2026-07-28
---

# Método Keelson — leia primeiro

> **A porta de entrada do Método Keelson — uma metodologia para desenvolvimento com LLM.** Comece por aqui: este documento diz o que cada peça é, em que
> ordem lê-las, e a regra que mantém tudo no lugar.

## O problema

Um agente de IA trabalha no seu código em **sessões curtas e repetidas**, e chega a cada uma sem memória da
anterior. Sem estrutura, ou ele relê o projeto inteiro toda vez (caro, lento) ou trabalha às cegas (erra). A
metodologia **Keelson** resolve isso com uma **memória viva** — documentação que o próprio agente mantém e que
**congela por evidência** — organizada para custar o mínimo de tokens e de atenção do humano.

## Os três padrões do núcleo (agnósticos a ferramenta de desenvolvimento)

A metodologia Keelson basea-se em uma trilogia; cada peça é um papel, versionada de forma independente:

- **[`llm-dev-memory.md`](llm-dev-memory.md) — o tabuleiro.** Onde o conhecimento mora. "Memória viva" é
  a pasta **`wiki/`** + pasta é o **`docs/`** + fontes de runtine.  O `wiki/` contem a indexação (índice + log) e das fontes. Observando-se que no caos de fontes de runtime os artefato podem ficar dentro do código e são apenas indexados no wiki/. Há uma organização de Tiers por custo de carregar, ADR, glossário,  invariantes.
- **[`llm-dev-flow.md`](llm-dev-flow.md) — o jogo.** Como o trabalho anda: `brief→plan→tasks`, transições
  de estado, guardrails proporcionais que mantêm o código no rumo da spec sem recair no waterfall.
- **[`llm-dev-player.md`](llm-dev-player.md) — o jogador.** O papel do humano: o que não se delega, o que se
  entrega ao agente a cada sessão, a rotina com tudo engrenado.

Essas três peças não mencionam nenhuma ferramenta específica de desenvolvimento com LLM, como Claude Code, Kimi Code,  Codex, Gemini, Cursor ou outra. Isto é de propósito para manter os três padrões no núcleo do método agnósticos .

## A camada fina, por ferramenta de desenvolvimento: os *application guides*

O núcleo é agnóstico; cada ferramenta de código (Claude Code, Cursor, Codex, Gemini…) tem suas
especificidades. Um **application guide** é a camada fina que liga os papéis do método aos mecanismos de
**uma** ferramenta — e absorve a evolução tecnológica dela **sem tocar o núcleo**.

- Hoje: **[`llm-dev-claude.md`](llm-dev-claude.md)** (Claude Code), o único com validação de campo — e o
  **molde de referência** da classe (o contrato de como um guia é estruturado vive no topo dele).
- Um guia **não resume** a doc do fabricante (seria uma cópia pior, que envelhece no dia seguinte). Ele
  **mapeia**: para cada necessidade do método, como a ferramenta a entrega — e onde ela não consegue.
- Ele aponta para a **referência do fabricante** — a evidência externa sobre a ferramenta, que **você
  mantém**. Não é uma doc baixada: é **captura datada** (a auto-descrição da ferramenta + o observável de um
  comando como `--help`/schema).
- Essas capturas moram no **log frio** do guia — **`llm-dev-<ferramenta>.log.md`** (append-only) —, separado
  do guia quente para não reinstalar o monólito. Cada linha do mapa carrega sua **confiança**
  (`campo`/`observável`/`desk`). Ao mudar a ferramenta, rode a skill `keelson-application-guides-update` para
  re-derivar o guia; um **teste de fora em sessão separada** carimba a re-verificação.

## A regra que mantém as camadas limpas: o firewall

> **Os três padrões do núcleo falam em papéis e capabilities — nunca em produtos.** Nenhum nome de
> ferramenta (Claude, Cursor, Codex, Gemini…) aparece em `memory` / `flow` / `player`. Se aparecer, aquilo
> pertence a um *application guide*.

É testável por `grep`, e é o que garante que o núcleo permaneça agnóstico e que a churn de ferramenta caia só
na camada fina — não no que você já validou.

## Glossário do conjunto

| Termo                        | O que é                                                                                                                                                                                                                                                 |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Keelson**                  | a metodologia inteira — o guarda-chuva da trilogia (memória/fluxo/jogador); a peça naval que enrijece a quilha e dá **governo** ao barco. Na prática é um métodologia que suporta o desenvolvimento de software utilizando um LLM (llm-dev)             |
| **tabuleiro / memória**      | `llm-dev-memory.md` — onde o conhecimento mora (`docs/` + `wiki/` + fontes de runtime)                                                                                                                                                                  |
| **jogo / fluxo**             | `llm-dev-flow.md` — como o trabalho anda                                                                                                                                                                                                                |
| **jogador / humano**         | `llm-dev-player.md` — o que o humano não delega                                                                                                                                                                                                         |
| **application guide**        | camada fina que liga papéis→mecanismos de UMA ferramenta específica; **mapeia, não resume**                                                                                                                                                             |
| **referência do fabricante** | a evidência externa sobre a ferramenta que **você** mantém — não uma doc baixada, mas **capturas datadas** (auto-descrição + observável) no log frio `llm-dev-<ferramenta>.log.md`; o guia aponta, não copia                                            |
| **captura**                  | uma entrada datada no log frio do guia: a auto-descrição da ferramenta (metade semântica, nasce `desk`) + o observável de um comando (`--help`/schema, ganha na superfície dele). A evidência que embasa as células do guia                             |
| **Tier 0 seed**              | a semente de contexto carregada **incondicionalmente** toda sessão (defaults + proibições mínimos), garantida mesmo em ferramentas de carga condicional                                                                                                 |
| **playbook**                 | procedimento reusável **do método** — agnóstico, serve em qualquer projeto (ex.: [`llm-dev-migration.md`](llm-dev-migration.md), `classe: playbook`). *Distinto de `runbook`* (que é do projeto)                                                        |
| **runbook**                  | manobra operacional de **um projeto** que adota o método: roteiro imperativo (deploy, rotação de chave) + guardrails, no balde `docs/runbooks/` (substitui o antigo `docs/rules/`). *Distinto de `playbook`* (que é do método)                          |
| **`.keelson/`**              | a pasta na raiz do projeto adotante onde o **pacote do método** é *vendorizado* (cópia pinada); irmão do `.claude/`. O Tier 0 aponta para a trilogia por **path relativo** (`.keelson/llm-dev-flow.md`). Ver [`llm-dev-package.md`](llm-dev-package.md) |

## Por onde começar

- **Projeto novo (greenfield):** rode o **[`llm-dev-prompt-bootstrap.md`](llm-dev-prompt-bootstrap.md)** —
  cria o esqueleto (um bom Tier 0, `wiki/`, `docs/`) e instala as convenções, para que todo conteúdo novo já
  nasça no lugar certo.
- **Projeto que já roda, já possui alguma documentação gerada:** rode o
  **[`llm-dev-prompt-migration.md`](llm-dev-prompt-migration.md)**, que segue o playbook
  **[`llm-dev-migration.md`](llm-dev-migration.md)** — migração **incremental, não big-bang**.
- **Para entender o porquê** (o julgamento por trás das regras): o livro **"Confiante e Errado"**. Para o **como
  operacional** (instalar, rodar, chamar as skills): o **[User Guide](llm-dev-user-guide.md)**. *(companheiros)*

## Estado e versão

**Pré-validação.** O conceito é maduro; falta um piloto instrumentado rodado até o fim. Primeira ferramenta
suportada: **Claude Code**. Cada documento é versionado de forma independente (esquema `0.x` → `1.0` após
validação de campo → `2.0` em mudança estrutural), com changelog próprio.

## Versionamento e histórico

A versão corrente está no frontmatter (`schema_version`). O **histórico de mudanças** deste documento vive
fora dele — em [`changelog/llm-dev-README.md`](changelog/llm-dev-README.md) —, para o padrão ficar limpo
como artefato de distribuição. O esquema de numeração (0.x → 1.0 → 2.0) e o índice geral estão em
[`changelog/README.md`](changelog/README.md).
