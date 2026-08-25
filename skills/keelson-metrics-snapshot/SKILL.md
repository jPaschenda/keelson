---
class: skill
name: keelson-metrics-snapshot
status: draft-para-testar
description: Gera um snapshot mecânico de saúde da wiki/docs do Método Keelson em wiki/metrics/AAAA-MM-DD-metric.md (tabela fixa: arquivo, commits, linhas +/-, saldo líquido, % nunca podado, linhas atuais, refs em transcripts). Escopo em duas camadas: arquivos gerados pelo método (fixo: index/glossary/invariants/known-issues/now/log) + páginas de projeto descobertas via os links do próprio wiki/index.md (o catálogo já declara o que é referência de primeira classe). Sinaliza órfãos (.md sob wiki/ ou docs/ não referenciados pelo index). Use quando pedir um snapshot de métricas, antes de promover o projeto 0.x→1.0, ou em cadência periódica. NÃO julga nem decide nada — só deriva e escreve; não edita nenhum doc existente.
---

# Snapshot de métricas (saúde da wiki/docs)

Deriva o snapshot mecânico descrito em `.keelson/llm-dev-memory-machinery.md` ("Onde as métricas são
armazenadas"): um arquivo `wiki/metrics/AAAA-MM-DD-metric.md` por captura, sempre a mesma tabela de colunas,
para diffar dois snapshots ser trivial. **Puramente mecânico** — sem julgamento, sem gate de fluxo: você não
decide nada, só deriva do `git` e escreve um arquivo novo.

## Escopo — o que esta skill faz e onde para

- Escreve **um arquivo novo** (ou atualiza o de hoje) em `wiki/metrics/` — nunca edita um doc existente.
- **Não** julga saúde nem recomenda ação — a leitura do snapshot é do humano (comparar com o anterior).
- **Não** é o `keelson-wiki-update` (que reconcilia uma sessão) nem faz parte dele — auditoria de saúde é
  evento separado, sob demanda.
- Baixo raio de explosão (arquivo novo, reversível via git) — o gate do Passo 2 é **informativo**, não um
  bloqueio pesado como nas skills de fase.

## Passo 1 — Escopo em duas camadas (o problema sem lista fixa)

Não há como prever todo `.md` de referência de um projeto adotante — mas dá para prever o que o **método**
gera, e o `wiki/index.md` já é o catálogo que declara o resto.

**Camada A — gerado pelo método (fixo, sempre medido):** `wiki/index.md`, `wiki/glossary.md`,
`wiki/invariants.md`, `wiki/known-issues.md`, cada `wiki/now/*.md`, cada `wiki/log/*.md`. (`log/` cresce por
natureza append-only — meça-o, mas não espere "poda" nele; é uma coluna com semântica diferente das demais,
não um bug.)

**Camada B — conteúdo do projeto (não previsível — descoberta, nunca hardcoded):** extraia os links markdown
de `wiki/index.md` que apontam para `.md` **dentro do repo** (tipicamente `docs/`: `architecture.md`,
`data-model.md`, `docs/decisions/*`, `docs/domain/*`, e o que mais o projeto catalogou). **Exclua**
deliberadamente `.keelson/` (é o pacote do método, versão pinada — saúde dele não é deste projeto) e
`manuscript/`/o livro (fora do escopo de doc-de-referência do projeto).

**Órfãos (achado de graça):** `.md` sob `wiki/`/`docs/` que existe mas **não** está linkado por `index.md` —
liste-os à parte no snapshot. Não é erro automático (pode ser intencional), mas um `.md` grande e crescendo
fora do catálogo é exatamente o tipo de coisa que a auditoria deveria expor.

## Passo 2 — Confirmação leve (informativa, não bloqueia)

Baixo raio de explosão — mostre o escopo achado e siga, salvo objeção:

> 📊 **Metrics Snapshot — escopo**
> - **Camada A (fixo):** {n arquivos}
> - **Camada B (via index.md):** {n arquivos} — {lista curta}
> - **Órfãos encontrados:** {n, ou nenhum}
>
> Prossigo com este escopo? (gera um arquivo novo, reversível — avise só se algo parecer errado)

## Passo 3 — Derivar as colunas (mecânico, por arquivo)

Para cada arquivo do escopo (A + B), via `git log --numstat` / `wc -l`:

- **commits** — nº de commits que tocaram o arquivo (histórico completo, ou desde o snapshot anterior mais
  recente — registre qual base foi usada).
- **linhas +/-** — soma de inserções/deleções (`git log --numstat`).
- **saldo líquido** — inserções − deleções no período.
- **linhas atuais** — `wc -l` do arquivo hoje.
- **% nunca podado** — `linhas atuais ÷ total de linhas já inseridas historicamente × 100`. Alto = quase nada
  podado (risco de acúmulo monótono); baixo = editado/podado de verdade.
- **refs em transcripts** — **melhor-esforço**: se os transcripts de sessão desta ferramenta estiverem
  acessíveis localmente, conte menções ao caminho do arquivo. Se não houver acesso portável, registre
  `— (não disponível nesta ferramenta)` na coluna inteira — **não invente** um número.

## Passo 4 — Escrever o snapshot

- **Nome:** `wiki/metrics/AAAA-MM-DD-metric.md` (data de hoje; data primeiro por ordenação no Obsidian). Se já
  existe um snapshot de hoje, **atualize-o** — não duplique.
- **Tabela única**, mesmas colunas sempre, Camada A e B juntas (marque a camada por linha) — é o que torna dois
  snapshots diretamente diferenciáveis.
- **Seção de órfãos** à parte, curta.
- **Baseline é implícito:** o snapshot de **data mais antiga** em `wiki/metrics/` — não marque isso no nome
  (quebraria a ordenação).

## Regras duras (não viole)

1. **Só deriva e escreve — não julga.** Nenhuma recomendação de ação no arquivo; a leitura é do humano.
2. **Nunca edita um doc existente** — só cria/atualiza o snapshot do dia em `wiki/metrics/`.
3. **Não mede `.keelson/`** nem o livro/manuscript — fora do escopo de saúde deste projeto.
4. **Não inventa `refs em transcripts`** — best-effort declarado ou "não disponível", nunca um número chutado.
5. **Camada B vem do `index.md`, nunca de convenção hardcoded** — é a única fonte que sabe o que o projeto
   estendeu além do que o método gera.

## Nota de maturidade

Skill em `draft-para-testar`. **Mínimo obrigatório do método**: um snapshot antes de promover 0.x→1.0; cadência
contínua (semanal/mensal) é recomendada, não obrigatória. Captura era manual até aqui — esta skill é a forma
nomeada de disparar corretamente. Origem: item (1) do backlog original de automação
(`docs/specs/Backlog.md`), reaberto depois que as skills de fluxo assentaram o padrão de
descoberta+escopo+gate-leve. *Instrumentar antes de formalizar*: se a Camada B via `index.md` mostrar-se
frágil em uso real (links que o parser não pega, etc.), ajuste aqui — não é lei gravada em pedra.
