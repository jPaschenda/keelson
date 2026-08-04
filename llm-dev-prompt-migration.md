---
schema_version: "0.3"
class: prompt
adapta:
  - memory 0.26
  - flow 0.15
  - player 0.7
status: draft-pre-validacao
data: 2026-07-28
---

# Método Keelson — Prompt de migração (projeto existente)

> **Como usar:** exemplo **genérico**. Antes, **vendorize o pacote do método em `.keelson/`** na raiz do
> projeto (cópia pinada — ver `llm-dev-package.md`). Depois cole o bloco abaixo numa sessão da sua ferramenta.
> Adapte os `<placeholders>`. Ele apenas *dispara e guarda* o playbook — quem manda é o `llm-dev-migration.md`.

```
Você vai migrar este projeto EXISTENTE para o Método Keelson, sem quebrar o que já funciona.

Os arquivos do método estão vendorizados em .keelson/ na raiz. Leia primeiro:
1. .keelson/llm-dev-README.md — mapa e firewall.
2. .keelson/llm-dev-memory.md — as Tiers e os baldes de docs/.
3. .keelson/llm-dev-migration.md — o PLAYBOOK que você vai seguir à risca.
4. .keelson/llm-dev-<ferramenta>.md — o encaixe na sua ferramenta (ex.: llm-dev-claude.md).

REGRA-MÃE: incremental, não big-bang. NÃO reescreva arquivos grandes de uma vez.

Siga o playbook nesta ordem, PARANDO nos pontos marcados para eu confirmar:
1. Rede de segurança: trabalhe numa branch. Capture o baseline de métricas ANTES de mover nada
   (wiki/metrics/AAAA-MM-DD-metric.md refletindo o estado atual, pré-método).
2. CLASSIFICAÇÃO — CHECKPOINT: varra os docs existentes e me apresente uma tabela "artefato → balde do
   método" (current-state / ADR / backlog / report / fonte de domínio / bugs-conhecidos-em-tratamento →
   known-issues / postmortem → log+ADR / fonte-de-runtime-NÃO-mover / notas). NÃO mova nada ainda — espere eu
   confirmar.
3. Depois de eu confirmar: crie o esqueleto (wiki/, docs/) e enxugue o Tier 0 para uma semente.
4. Extraia só os ADRs ÓBVIOS do monólito de arquitetura; deixe o resto onde está.
5. git mv dos arquivos claramente mal-colocados — mas SÓ depois de grepar referências de entrada (nada
   aponta pra ele por path de código?). Indexe as fontes-de-runtime onde estão, sem mover.
6. Pare. O resto do monólito é drenado aos poucos, por uso, em sessões futuras.
   
Ao terminar: mostre a árvore criada e o conteúdo do Tier 0

Se você não tiver certeza de um fato, dado ou contexto para responder à minha solicitação, não invente nenhuma informação. Em vez disso, pare imediatamente e me faça as perguntas necessárias para preencher as lacunas.
```
