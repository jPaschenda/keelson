---
title: Keelson — Pacote de distribuição
package_version: "0.32"
data: 2026-08-24
status: draft-pre-validacao
---

# Método Keelson — pacote de distribuição

> A lista do que vai na caixa + como instalar. **Primeira versão — vai mudar muito.** Para *o que é* o
> método, comece pelo [`llm-dev-README.md`](llm-dev-README.md).

**Versão do pacote: 0.32 (2026-08-24)** — pré-validação; primeira ferramenta suportada: Claude Code.

## Conteúdo

| Arquivo                                                      | Papel                                             | Versão |
| ------------------------------------------------------------ | ------------------------------------------------- | ------ |
| [`llm-dev-README.md`](llm-dev-README.md)                     | front-door: mapa, classes, firewall, glossário    | 0.8    |
| [`llm-dev-user-guide.md`](llm-dev-user-guide.md)             | user guide: o *como* operacional (instalar, rodar, manter) | 0.1 |
| [`llm-dev-package.md`](llm-dev-package.md)                   | este manifesto: o que vai na caixa + instalação   | 0.32   |
| [`llm-dev-memory.md`](llm-dev-memory.md)                     | core: tabuleiro (memória) — núcleo fino           | 0.30   |
| [`llm-dev-memory-structuring.md`](llm-dev-memory-structuring.md) | core-satellite: memória — estruturação/adoção     | 0.29   |
| [`llm-dev-memory-machinery.md`](llm-dev-memory-machinery.md) | core-satellite: memória — maquinaria/automação    | 0.30   |
| [`llm-dev-flow.md`](llm-dev-flow.md)                         | core: jogo (fluxo) — núcleo fino                  | 0.27   |
| [`llm-dev-flow-maintenance.md`](llm-dev-flow-maintenance.md) | core-satellite: fluxo — manutenção/bug/tweak      | 0.21   |
| [`llm-dev-player.md`](llm-dev-player.md)                     | core: jogador (humano) — contrato de fronteira    | 0.9    |
| [`llm-dev-claude.md`](llm-dev-claude.md)                     | application guide: Claude Code                    | 0.4    |
| [`llm-dev-claude.log.md`](llm-dev-claude.log.md)             | log frio do guia: capturas datadas                | 0.1    |
| [`llm-dev-migration.md`](llm-dev-migration.md)               | playbook de migração (classe `playbook`)          | 0.5    |
| [`llm-dev-prompt-bootstrap.md`](llm-dev-prompt-bootstrap.md) | prompt: projeto novo                              | 0.4    |
| [`llm-dev-prompt-migration.md`](llm-dev-prompt-migration.md) | prompt: projeto existente                         | 0.3    |
| `skills/keelson-brief-prep/`                                  | skill: preparação mecânica do brief (nunca a intenção) — colhe evidência, estrutura DRAFT | 0.1 |
| `skills/keelson-plan-init/`                                  | skill: cria o plan (ou revisa por versão, plano VALIDATED); ponteiro de mão dupla com backlog.md | 0.4 |
| `skills/keelson-phase-landing/`                              | skill: aterrissagem de fase → escreve tasks-faseN | 0.3    |
| `skills/keelson-coding/`                                     | skill: coding (fase, fix ou tweak) → soleira de BUILT, e promoção a BUILT (reinvocada) | 0.8 |
| `skills/keelson-review-session/`                              | skill: revisão independente (fase, fix ou tweak) → BUILT | 0.8 |
| `skills/keelson-deploy/`                                      | skill: implanta em homologação/produção → PILOT/PROD (soleira) | 0.4 |
| `skills/keelson-field-validation/`                           | skill: validação de campo (alta fidelidade, orienta) | 0.2 |
| `skills/keelson-fix/`                                         | skill: descoberta/reprodução/causa-raiz de bug → fix-<slug>.md | 0.8 |
| `skills/keelson-tweak/`                                       | skill: landing leve de ajuste sub-especificado → tweak-<slug>.md | 0.2 |
| `skills/keelson-application-guides-update/`                  | skill: re-deriva o application guide (obs.-ancorado) | 0.3 |
| `skills/keelson-metrics-snapshot/`                            | skill: snapshot mecânico de saúde da wiki/docs    | 0.1    |
| `skills/keelson-wiki-update/`                                 | skill: reconcilia sessão com wiki/decisions       | 0.1    |

## Onde o pacote mora no projeto adotante: `.keelson/`

O projeto que adota o método **vendoriza** este pacote numa pasta **`.keelson/`** na raiz do repo — uma cópia
**pinada** (a versão fica registrada; atualizar é um *re-vendor* deliberado, não um link vivo). É o irmão do
`.claude/`: sinaliza "isto é o método, não é conhecimento do *projeto*", e por isso **não** entra em `wiki/`
nem em `docs/` (que são a memória do projeto).

Com isso, o Tier 0 e os artefatos apontam para o método por **path relativo** (`.keelson/llm-dev-flow.md`) —
nunca para o disco de quem escreveu o método (que só existe na máquina dele). É a regra que fecha o
[`llm-dev-memory.md`](llm-dev-memory.md) ("O que faz um bom Tier 0", o *segundo roteamento*): o Tier 0 roteia
para o conhecimento do projeto (via `wiki/index.md`) **e** para a trilogia do método (via `.keelson/`).

### Exceção: skills rodam de `.claude/skills/`, não de `.keelson/`

O harness (Claude Code) só descobre skills em `.claude/skills/` (projeto), `~/.claude/skills/` (usuário) ou
via plugin — **nunca** varre `.keelson/`. As skills são, portanto, o único artefato do pacote que **não** é
lido de `.keelson/`: enquanto os docs são lidos ao vivo por path relativo, as skills são **config executável
do harness** e precisam viver onde ele as procura. Instalar uma skill = **copiar** `skills/<nome>/` para
`.claude/skills/<nome>/`.

**Política (pré-validação): instale direto em `.claude/skills/`** — a fonte é o próprio pacote. Pinar uma
cópia em `.keelson/skills/` e recopiá-la no re-vendor é possível, mas só paga quando as skills começarem a
divergir por adotante; até lá é sincronização a mais. Em qualquer caso, a versão que **roda** é a de
`.claude/skills/`.

## O que VOCÊ fornece

- **A referência do fabricante — você a *gera*, não a baixa.** Não existe "manual congelável" das ferramentas
  de código; então a evidência da ferramenta vem de **captura**: você roda a skill `keelson-application-guides-update`,
  que pede a **auto-descrição** da ferramenta, ancora no **observável** (`--help`/schema) e grava a captura
  datada no **log frio** `llm-dev-<ferramenta>.log.md` (append-only). O guia quente aponta para esse log; um
  **teste de fora em sessão separada** carimba a re-verificação. Espelha o `docs/domain/source/` do método
  (evidência externa fisicamente separada da síntese) — só que a fonte é a própria ferramenta se descrevendo,
  não um arquivo baixado.

## Instalação / por onde começar

1. **Vendorize o pacote** em `.keelson/` na raiz do projeto (ver "Onde o pacote mora" acima). Os prompts de
   instalação já fazem isso e geram o Tier 0 apontando para lá por path relativo.
2. Leia `llm-dev-README.md` (o mapa) e o `llm-dev-user-guide.md` (o *como* operacional — instalar, rodar, manter).
3. **Projeto novo** → rode `llm-dev-prompt-bootstrap.md`. **Projeto existente** → `llm-dev-prompt-migration.md`.
4. (Claude Code) Instale as skills copiando cada `skills/<nome>/` para `.claude/skills/<nome>/` (ver
   "Exceção: skills rodam de `.claude/skills/`" acima). Skills atuais, em 4 famílias: **fluxo forward**
   (`keelson-brief-prep` — preparação mecânica do brief, quando nenhum existe ainda pra área — `keelson-plan-init`,
   `keelson-phase-landing`, `keelson-coding`, `keelson-review-session`), **implantação e
   campo** (`keelson-deploy`, `keelson-field-validation`), **manutenção/bug/tweak** (`keelson-fix` e
   `keelson-tweak` — duas portas de entrada simétricas que rodam o mesmo teste de fronteira antes de tudo, e
   ambas reusam `keelson-coding`/`keelson-review-session`/`keelson-deploy`/`keelson-field-validation`, e podem
   apontar pra `keelson-brief-prep` quando o Eixo 0 achar decisão de desenho sem brief cobrindo a área), e
   **manutenção do método/wiki** (`keelson-application-guides-update`, `keelson-metrics-snapshot`,
   `keelson-wiki-update`).

## Fora do pacote (companheiros — ver `llm-dev-roadmap.md`)

- O livro **"Confiante e Errado"** (o *porquê* / julgamento) — o único artefato pago; O livro atualmente é vendido na Leanpub em https://leanpub.com/confiante-errado

Observação: User Guide **faz parte do pacote** (ver Conteúdo), o livro apenas aponta para ele.