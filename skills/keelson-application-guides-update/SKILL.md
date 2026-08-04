---
class: skill
name: keelson-application-guides-update
status: draft-para-testar
description: Re-deriva um application guide do Método Keelson (llm-dev-<ferramenta>.md) quando a ferramenta muda. Descobre o guia-alvo e o modo, confirma, pede a auto-descrição da ferramenta, ancora no observável (--help/schema), grava a captura datada no log frio (llm-dev-<ferramenta>.log.md) e finaliza com um teste de fora em sessão separada. Use quando o usuário souber que a ferramenta mudou, ou quando uma falha/drift acusar o guia. NÃO use para escrever docs do método (núcleo/trilogia) nem para atualizar o wiki do projeto (isso é keelson-wiki-update).
---

# Update do application guide

Re-deriva um application guide contra o **estado corrente da ferramenta**. **Re-mapeia, não re-resume** — e
**não baixa manual nenhum**: a verdade vem da **auto-descrição da ferramenta ancorada no observável**. O guia é
um **mapeamento por papel** — as **linhas** são as necessidades do método (estáveis), as **células** são os
mecanismos da ferramenta (mudam); você atualiza só as células.

## Escopo — o que esta skill faz e onde para

- Cobre a manutenção de **um** application guide (`llm-dev-<ferramenta>.md`) contra o estado corrente da ferramenta.
- Atualiza só as **células**; **nunca as linhas** (se uma linha nova parece necessária, é candidato ao núcleo —
  PARE e sinalize).
- **Não** toca no núcleo/trilogia, **não** atualiza o wiki do projeto (`keelson-wiki-update`), **não** edita `.keelson/`.

## Passo 1 — Descoberta do alvo e do modo (antes de mexer no guia)

- **Qual guia?** `llm-dev-<ferramenta>.md`. Leia o frontmatter (`adapta:`, `verificado_em:`) e o mapeamento por
  papel com a coluna **Confiança**. Se houver mais de um guia no projeto, identifique o da ferramenta que mudou.
- **Qual modo?** Dois gatilhos, e eles mudam o rigor:
  - **rotina** — o leitor **soube** que a ferramenta mudou (release, comportamento novo). É o gatilho do jogador
    (intenção + gatilho; não há vigilância ambiente que dispare sozinho). → **Passo 3A** (caminho barato).
  - **escalada** — uma **falha visível** ("o guia diz faça X, X não acontece") **ou drift instrumentado** acusou
    o guia. → **Passo 3B**.

Se o alvo ou o modo não estiverem claros, deixe em aberto para o Passo 2 — não presuma.

## Passo 2 — Confirmação (gate humano; PARE aqui)

Pare e apresente:

> 🧭 **Application Guide Update — alvo**
> - **Ferramenta / guia:** {ferramenta} — llm-dev-{ferramenta}.md
> - **verificado_em:** {data do frontmatter}
> - **Modo:** {rotina / escalada — se escalada, a linha suspeita}
>
> Confirma o alvo e o modo? (rotina e escalada aplicam rigor diferente)

**Regra dura:** não reescreva o guia sem confirmar — o modo determina o caminho.

## Passo 3A — Caminho barato (rotina)

1. **Leia** o guia quente (`llm-dev-<ferramenta>.md`): frontmatter (`adapta:`, `verificado_em:`) e o
   mapeamento por papel com a coluna **Confiança**.
2. **Auto-descrição:** peça à ferramenta que descreva **como ela entrega cada papel** (o molde das *linhas*)
   + a **linha aberta**: "o que mudou / o que é novo que eu não perguntei?". (Fecha o *unknown unknown* de
   uma entrevista fechada.)
3. **Âncora observável:** capture o barato e determinístico — `--help` e subcomandos, schema de settings,
   changelog do CLI. É a metade **atual** (a auto-descrição do modelo é congelada no training-cutoff dele).
4. **Reconcilie:** onde o observável contradiz a auto-descrição, **o observável ganha — só na superfície que
   ele toca** (nomes de flag/comando/campo). A metade **semântica** (o que um mecanismo *faz*, que o `--help`
   não diz) fica com a auto-descrição e **nasce `desk`**.
5. **Atualize as células** (nunca as *linhas*). Para cada célula, **grave a Confiança**: `observável` (lastro
   em `--help`/schema), `campo` (validado em uso real) ou `desk` (só auto-descrição). Célula sem lastro
   observável **nasce `desk`** — obrigatório (não emprestar credibilidade do forte pro fraco).
6. **Anexe a captura datada** no log frio `llm-dev-<ferramenta>.log.md` (auto-descrição + observável do dia).
   Append-only: nunca reescreva captura anterior.
7. **Bump de versão** do guia + entrada no changelog (que célula mudou, com atribuição à captura). Se uma
   linha nova parece necessária, isso **não é do guia** — é candidato ao núcleo (memory/flow/player). Pare e
   sinalize ao humano. O guia agora é **candidato**.
8. **Teste de fora — em sessão separada** (§ Finalização). Só ele carimba `verificado_em:`.

## Passo 3B — Escalada (contra-alucinação; só sob evidência, não rotina)

Disparada por **falha visível** ("o guia diz faça X, X não acontece") **ou drift instrumentado**:

1. Aterrisse a **linha suspeita** mais fundo contra o observável / o campo.
2. **Diff** contra a última captura no log frio para localizar a causa: *a ferramenta mudou* (captura antiga
   ≠ comportamento atual) vs *a descrição nasceu errada* (nunca bateu). Corrija a célula e sua Confiança.

Depois de corrigida, a escalada converge para os passos 6–8 do caminho barato (captura + bump + teste de fora).

## Finalização — o teste de fora (sessão separada)

O passe que **constrói** o guia não pode ser o que o **abençoa** (senão é o construtor corrigindo a própria
prova). Emita a **spec** de um smoke test e a entregue a uma **sessão nova**:

- exercita o **mecanismo real** das **2–3 linhas que sustentam peso** (o Tier 0 seed *carrega* mesmo numa
  sessão limpa? uma checagem separada *dispara* mesmo?);
- a independência vem do **árbitro observável** (o comportamento real), **não** da sessão nova em si (o mesmo
  modelo relendo o guia e abençoando não vale — é auto-certificação);
- **gate proporcional:** só rode o teste de fora quando o update tocou uma linha **load-bearing**; renome
  cosmético de flag numa linha não-crítica dispensa (verificação mecânica basta);
- **passou** → carimba `verificado_em:` (candidato→merge); **falhou** → volta à escalada (Passo 3B).

No greenfield, o bootstrap dispara esse teste logo após montar o esqueleto → evidência no dia 1.

## Regras duras (não viole)

1. **Só as células, nunca as linhas.** As linhas são necessidades do método (estáveis). Linha nova necessária =
   candidato ao núcleo → PARE e sinalize ao humano.
2. **Quente ≠ frio.** A captura datada vai no log frio `llm-dev-<ferramenta>.log.md` (append-only), **nunca**
   empilhada dentro do guia — isso recria o monólito que enterra.
3. **Não re-resuma o fabricante** — o guia é mapeamento por papel, não resumo da doc.
4. **Célula sem lastro observável nasce `desk`** — não emprestar credibilidade do forte pro fraco.
5. **Só o teste de fora (sessão separada) carimba `verificado_em:`** — quem constrói não abençoa.
6. **Não invente** — onde o observável contradiz a auto-descrição, o observável ganha (só na superfície que toca).
7. **Não edite `.keelson/`** nem o núcleo/trilogia.

## Nota de maturidade

Skill em `draft-para-testar`. A maquinaria pesada (teste em sessão separada + escalada) é **desenho
pré-validação**; o núcleo que já entrega valor é o **caminho barato** (auto-descrição ancorada + capturas +
confiança por linha). O loop pesado ganha lugar **por dor medida** — não como cerimônia obrigatória sobre um
guia que quase não faz churn. *Instrumentar antes de formalizar.*
