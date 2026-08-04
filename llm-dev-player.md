---
schema_version: "0.9"
class: core
status: draft-pre-validacao
data: 2026-07-28
extraido_de: livro companheiro
---

# Método Keelson Player — O jogador (o humano)

> **O que é, em uma linha:** o terceiro padrão da trilogia — o papel do **humano** no desenvolvimento Humano+LLM, do ponto de vista do agente: o que **não se delega**, o que o jogador **entrega** ao agente a cada sessão, e o **veredito/triagem** que só ele dá.

> **Status: o mais jovem dos três padrões.** O `llm-dev-memory.md` e o `llm-dev-flow.md` foram destilados do campo e depois refinados; este nasceu do movimento inverso — o papel do humano foi destilado da prática e extraído para padrão. Ainda sem o desgaste de uso dos irmãos; endurece com o campo, pela mesma regra de sempre.

## A trilogia — relação com os padrões irmãos

- [`llm-dev-memory.md`](llm-dev-memory.md) = a **memória** (onde o conhecimento mora). É o **tabuleiro**.
- [`llm-dev-flow.md`](llm-dev-flow.md) = o **fluxo** (como o trabalho anda). É o **jogo**.
- `llm-dev-player.md` (este) = o **humano** (quem joga). É o **jogador**.

> **Agnóstico a ferramenta.** Um dos três padrões do núcleo: fala em **papéis**, não em produtos. As especificidades de cada ferramenta vivem nos *application guides* — ver [`llm-dev-README.md`](llm-dev-README.md), o firewall.

Dependência: este documento **usa** os outros dois — aponta para os artefatos da wiki e para as transições do processo; nenhum dos dois depende dele. Mas a recíproca de *manutenção* é real e é a razão de ele existir: tabuleiro e jogo não jogam sozinhos. Cada seta de volta do processo atravessa um julgamento humano, e uma wiki sem guardião apodrece com autoridade — vira uma fonte da verdade mentindo. **Sem jogador, o método vira teatro.**

## Por que um padrão para o humano

Três fatos sobre o agente definem, por subtração, o papel do lado de cá:

1. o modo de falha dele é **confiança-mas-errado** — não preguiça: ele apresenta o errado com a mesma convicção que o certo;
2. ele **não tem pele no jogo** — constrói a coisa errada com capricho, não sente o custo do bug, não conhece o usuário;
3. ele é **dócil** — proposto um caminho ruim com convicção, tende a ir junto e a achar bons argumentos.

Tudo o que o agente estruturalmente não pode garantir sobra para o humano. E há a assimetria de posição que sustenta o papel: todos os artefatos de continuidade (now, log, ADRs) são **próteses** de memória para um colaborador que não lembra. O humano é o único participante que de fato *lembra* — atravessa as sessões por fora delas, contínuo. Não é superioridade de inteligência; é **posição**.

## As três vigílias (o que não se delega)

| Vigília | Sobre o quê | Onde se exerce |
|---|---|---|
| **Intenção** | requisitos, o quê e o porquê — o agente pode redigir a spec, mas não pode *querê-la* | brief `→VALIDATED` é gate humano (process, guardrails) |
| **Estrutura** | arquitetura, decisões congeladas, e a **curadoria da memória**: auditar, podar, conferir o que ainda é verdade | ADRs; lint; o grafo como instrumento de visão (wiki, Obsidian) |
| **Veredito** | o "pronto" — o critério objetivo que o agente não escreveu; no alto risco, virar a chave | tabela de guardrails por transição (process): "quem fecha — humano" |

**Guardião ≠ microgerente.** O rigor é proporcional ao raio de explosão. Vigiar não é reler cada linha — é curar as fontes, julgar as fronteiras e deixar o agente correr no meio. Quem microgerencia paga duas vezes: o custo do trabalho braçal que o método existe para dispensar, e a atenção — escassa, e a única coisa que só o humano tem — que as três vigílias exigem.

**Curar as fontes inclui as de fora.** Quando o domínio tem documentação que vincula o sistema — regulamento, legislação, norma (a *fonte de domínio*: [`llm-dev-memory.md`](llm-dev-memory.md), "`docs/domain/`") — o *sourcing* e a curadoria de `docs/domain/source/` são do **jogador**: que fonte entra, qual versão vale, quando reconferir se a autoridade de fora mudou. É o *sourcing* na sua forma mais literal, e cai inteiro na vigília da Estrutura — porque obrigação externa que ninguém reconfere é fonte da verdade envelhecendo em silêncio.

**E "de fora" inclui a própria ferramenta de desenvolvimento:** suas capabilities são uma autoridade externa como qualquer outra. Quando essa autoridade muda, reconferir não basta — o que dela depende precisa ser **re-derivado**, e disparar essa re-derivação é do jogador (**intenção + gatilho**): não há vigilância ambiente entre sessões que o faça sozinho. Rebaixar a fonte sem re-derivar o que a ela se liga é deixar o vínculo divergir calado. *(O mecanismo concreto dessa re-derivação — por ser específico de ferramenta — vive no application guide, não aqui; ver [`llm-dev-README.md`](llm-dev-README.md), o firewall.)*

## As três entregas (o que o jogador dá a cada sessão)

1. **O problema enquadrado** — o que resolver, por quê, e — tão importante quanto — o que **não** tocar.
2. **O contexto certo** — curadoria, não volume: a seleção do que *esta* tarefa precisa. A restrição que você conhecia e não disse não é erro do agente; é bug do enquadramento — seu.
3. **O "pronto" definido** — o critério objetivo, escrito antes, que o agente não escreveu (EARS quando o risco justifica). Sem ele, o agente corrige a própria prova.

Regra de delegação, numa linha: **delegue o *como* à vontade; não delegue o *se* nem o *porquê*.** Intenção, prioridade e "vale a pena fazer isto?" são do jogador.

E a economia que fecha a conta: memória bem curada paga o enquadramento em prestações — a sessão herda o glossário, os invariantes e o now de graça; ao jogador resta entregar só o incremento (esta tarefa, este critério, esta fronteira).

## A entrega muda no modo-manutenção

Corrigir um bug inverte a **primeira** entrega. Numa feature, é você que **enquadra o problema**; num bug, **a realidade o enquadra** — a falha observada no sistema vivo. Sua entrega deixa de ser "enquadrar" e passa a **curar a reprodução**: transformar o observável de produção (o log, o *trace*, o dado corrompido) no **alvo objetivo que o agente não escreveu** — o mesmo papel do "pronto definido", agora dado de graça pela falha, e o antídoto mais direto ao "corrigir a própria prova" quando o risco é justamente o agente consertar o *sintoma* e não a *causa*.

E a **triagem é sua**: qual dono congelado a realidade contradisse — o código (só corrige), um invariante/ADR (supersede) ou o próprio brief (revisão versionada)? Quando a resposta é **assumir uma limitação** em vez de codar, isso é uma **decisão** — vira ADR/invariante, nunca um encolher de ombros perdido no chat. O mecanismo completo (o artefato `fix-<slug>.md`, o `known-issues.md`) vive no satélite [`llm-dev-flow-maintenance.md`](llm-dev-flow-maintenance.md); aqui fica só o que **não se delega**: a reprodução curada e o veredito da triagem.

## Fronteira aberta

Este padrão descreve o jogador **no singular** — um humano, um agente. Vários humanos no mesmo tabuleiro (quem é guardião de qual vigília; quem arbitra um desacordo entre guardiões) é fronteira sem evidência de campo — declarada, não escondida, ao lado das demais perguntas abertas do método (multi-agente em paralelo, merge da memória, segurança adversarial, recuperação em escala).

## Versionamento e histórico

A versão corrente está no frontmatter (`schema_version`). O **histórico de mudanças** deste documento vive fora dele — em [`changelog/llm-dev-player.md`](changelog/llm-dev-player.md) —, para o padrão ficar limpo como artefato de distribuição. O esquema de numeração (0.x → 1.0 → 2.0) e o índice geral estão em [`changelog/README.md`](changelog/README.md).
