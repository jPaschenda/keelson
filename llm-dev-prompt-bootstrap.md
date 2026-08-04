---
schema_version: "0.4"
class: prompt
adapta:
  - memory 0.27
  - flow 0.15
  - player 0.7
status: draft-pre-validacao
data: 2026-07-28
---

# Método Keelson — Prompt de bootstrap (projeto novo)

> **Como usar:** exemplo **genérico**. Antes, **vendorize o pacote do método em `.keelson/`** na raiz do
> projeto (a cópia pinada — ver `llm-dev-package.md`). Depois cole o bloco abaixo numa sessão da sua
> ferramenta. Adapte os `<placeholders>` (ferramenta, idioma). É um ponto de partida — ajuste ao seu projeto.
> Para projeto que já roda, use o `llm-dev-prompt-migration.md`.

```
Você vai preparar este projeto para o Método Keelson (memória viva para desenvolvimento com IA).

Os arquivos do método estão vendorizados em .keelson/ na raiz. Leia primeiro, nesta ordem:
1. .keelson/llm-dev-README.md — o mapa e o firewall.
2. .keelson/llm-dev-memory.md — as três Tiers e a seção "O que faz um bom Tier 0".
3. .keelson/llm-dev-<ferramenta>.md — como o método encaixa na sua ferramenta (ex.: llm-dev-claude.md).

Este é um projeto NOVO (greenfield): não há conhecimento a sintetizar ainda. Seu trabalho é criar o
esqueleto correto e instalar as convenções, para que todo conteúdo novo já nasça no lugar certo — NÃO
inventar arquitetura ou decisões que ainda não existem.

Crie:
1. O Tier 0 (arquivo raiz — na sua ferramenta, o arquivo sempre-carregado; no Claude Code é o CLAUDE.md):
   uma SEMENTE enxuta, com DUAS zonas.
   PRIMEIRO um PREÂMBULO DE NAVEGAÇÃO no topo (cabeçalho): (a) por onde começar — wiki/index.md; (b) a
   TRILOGIA DO MÉTODO em .keelson/, por path relativo — o fluxo (.keelson/llm-dev-flow.md; nova funcionalidade
   E correção de bug), o jogador (.keelson/llm-dev-player.md), o tabuleiro (.keelson/llm-dev-memory.md); (c) o
   mapa dos contratos Tier-1 always-on — wiki/glossary.md, wiki/invariants.md, wiki/known-issues.md (o que está
   quebrado agora, para não construir sobre bug) e wiki/now/<retomada>.
   DEPOIS a ZONA DO PROJETO: idioma <idioma do projeto>, as proibições turn-1 (o mais caro de violar vem
   primeiro), os comandos-do-dia (build/test), as convenções-desvio.
   Só o que passa no teste de inclusão; nada de conhecimento profundo, e nada de mini-índice de arquivos de
   código (isso é do wiki/index.md). Em dúvida sobre uma linha: "o agente erraria por default sem isto, de um
   jeito irrecuperável lendo o índice na hora?". Se não, fica de fora.
2. wiki/: index.md (catálogo, começa quase vazio), now/<branch>.md (ponteiro de retomada) se não há branch de versionamento usar main.md, log/ (vazio),
   glossary.md e invariants.md (contratos — começam vazios, só com o cabeçalho e a regra de entrada), e
   known-issues.md (ponteiro transiente do que está quebrado/em tratamento; começa vazio, par do invariants).
3. docs/: architecture.md (stub curto) e as pastas decisions/, specs/ (prescritivo forward — o que vai ser
   construído), fixes/ (prescritivo corretivo — correção de bug do presente; começa vazia), reports/, domain/,
   runbooks/ (manobra operacional — deploy, rotação de chave; começa vazia).

Ao terminar: mostre a árvore criada e o conteúdo do Tier 0, e me lembre de que a wiki se enche à medida que
eu trabalhar — ela nasce quase vazia de propósito.

Se você não tiver certeza de um fato, dado ou contexto para responder à minha solicitação, não invente nenhuma informação. Em vez disso, pare imediatamente e me faça as perguntas necessárias para preencher as lacunas.
```
