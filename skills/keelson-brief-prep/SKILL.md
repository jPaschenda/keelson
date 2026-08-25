---
class: skill
name: keelson-brief-prep
status: draft-para-testar
description: Faz a preparação mecânica de um brief-<slug>.md do Método Keelson quando uma investigação (keelson-fix, keelson-tweak, achado de backlog) revela decisão de desenho genuína e não existe brief nenhum cobrindo aquela área ainda — sem nunca escrever a intenção. Descobre se já existe brief em voo pro mesmo slug/área (não duplica), confirma com o humano que isto é território de brief e não de plan/fix/tweak, cria a estrutura do arquivo (Doc Status: DRAFT, cabeçalho, seções O QUÊ/PORQUÊ/Restrições marcadas como pendência do humano) e colhe evidência já produzida em docs/specs/backlog.md, wiki/known-issues.md e fix-<slug>.md/tweak-<slug>.md relacionados para dentro da seção "Hipóteses a validar" (cada item com fonte e nível de evidência, nunca invenção), gravando o ponteiro de mão dupla no item de backlog quando a origem for um. Use quando keelson-fix ou keelson-tweak apontarem "brief novo" e nenhum existir ainda pra área, ou quando o usuário quiser abrir uma frente de brief a partir de achados já espalhados pelo repo. NÃO escreve a prosa de O QUÊ/PORQUÊ nem qualquer hipótese sem fonte já existente no repo — isso é sempre do humano (ver llm-dev-player.md, "o agente pode redigir a spec, mas não pode querê-la"). NÃO promove Doc Status pra VALIDATED (gate humano sempre, revisão adversarial). NÃO é keelson-plan-init (que só roda depois de um brief VALIDATED) nem keelson-wiki-update (que reconcilia wiki/index.md e docs/decisions/, não docs/specs/).
---

# Brief — preparação mecânica (nunca a intenção)

O método deixa a vigília de **Intenção** deliberadamente fora de automação: "o agente pode redigir a spec, mas
não pode *querê-la*" (`llm-dev-player.md`). Isso é correto e não muda aqui. Mas deixou um vão real: quando uma
investigação (`keelson-fix`, `keelson-tweak`) roda o Eixo 0 e conclui "isto é decisão de desenho de verdade",
o único nome que o vocabulário de skills oferecia era `keelson-plan-init` — que exige exatamente o que ainda
não existe (um brief `VALIDATED`). Sem um nome melhor, o handoff apontava pra lá mesmo sabendo que a
precondição falhava — o agente reaproveitando o skill mais próximo por falta de opção, não por ser a rota
certa. Esta skill fecha esse vão sem cruzar a fronteira: faz a **preparação** (arquivo, estrutura, evidência já
dita em outro lugar do repo), nunca a **intenção** (a prosa do O QUÊ/PORQUÊ).

## Escopo — o que esta skill faz e onde para

- Cobre a fronteira **decisão de desenho descoberta em campo, sem brief nenhum cobrindo a área → estrutura de
  `brief-<slug>.md` em `DRAFT`**, com evidência já existente colhida e organizada.
- **Nunca escreve O QUÊ / PORQUÊ / Restrições de design** — essas seções nascem como pendência explícita
  (`⏳ pendente — humano escreve`), mesmo que o material colhido pareça sugerir uma resposta óbvia. Arriscar um
  rascunho dessas seções é exatamente o que a vigília de Intenção proíbe — não é uma diferença de grau, é a
  fronteira.
- **Nunca promove `Doc Status` pra `VALIDATED`** — isso segue exigindo revisão adversarial (arquitetura,
  segurança) + checklist de completude/ambiguidade, sempre humana (ver `llm-dev-flow.md`, seção do gate do
  brief).
- **Não é `keelson-plan-init`** — aquela só roda depois de `VALIDATED`; esta roda antes, quando nem `DRAFT`
  existe ainda.
- **Não é `keelson-wiki-update`** — aquela reconcilia `wiki/index.md`/`docs/decisions/`; esta toca só
  `docs/specs/<slug>/brief-<slug>.md`.
- **Se já existe brief (`DRAFT` ou `VALIDATED`) pra mesma área/slug** — não duplica. Aponta pro existente e, no
  máximo, oferece colher a evidência nova pra dentro da seção "Hipóteses a validar" dele (ainda sob confirmação
  do humano de que aquele achado pertence ali).
- **Cerimônia mínima, não zero.** Mesmo a preparação pede confirmação humana antes de escrever qualquer
  arquivo — não é um passe mecânico livre como o "estrutural" de `keelson-wiki-update` (baixo raio por natureza
  ali; um brief novo, mesmo vazio, é o primeiro artefato de uma feature possivelmente grande, e o nome do
  slug/pasta já é uma decisão que merece confirmação).

## Passo 1 — Isto é mesmo território de brief?

- **De onde veio o gatilho?** Normalmente o Eixo 0 de `keelson-fix`/`keelson-tweak`
  (`.keelson/llm-dev-flow-maintenance.md`, "Eixo 0") apontou pra cá — achado que toca decisão congelada ou
  escopo nunca implementado, **e** nenhum brief existente cobre a área. Pode também ser o usuário pedindo
  direto, sem skill anterior.
- **Confirme com o humano, explicitamente:** isto é decisão de desenho (o quê e o porquê), não um ajuste ou
  conserto pontual? Se a resposta for "ainda não sei" — essa também é uma resposta válida. Pare e sugira
  continuar investigando em conversa normal antes de abrir o arquivo. Abrir cedo demais é o mesmo erro que
  abrir tarde demais: cerimônia sem substância ainda por trás.
- **Cheque duplicidade:** já existe `docs/specs/<slug>/brief-<slug>.md` (qualquer `Doc Status`) pra esta mesma
  área? (`git status`, `docs/specs/`, `wiki/index.md`). Se sim, pare — vá pro parágrafo "Se já existe" no
  Escopo, acima, em vez de continuar os passos abaixo.
- **Descubra o slug**, confirme com o humano — normalmente já sugerido pelo contexto (nome do subsistema, do
  achado). É o nome que vira pasta; ASCII, mesma convenção de `keelson-plan-init`.

## Passo 2 — Colher evidência já dita (nunca inventar)

Percorra o que já existe e **aponte a fonte de cada item** — isto é busca e organização, não julgamento:

- `docs/specs/backlog.md` — itens relacionados ao slug/área.
- `wiki/known-issues.md` — entradas relacionadas, especialmente as de raio alto.
- `docs/fixes/fix-<slug-relacionado>.md` / `docs/tweaks/tweak-<slug-relacionado>.md` — achados/causas-raiz que
  tocam a mesma área.
- `wiki/log/` — incidentes relacionados.
- Se a origem foi uma sessão de `keelson-fix`/`keelson-tweak` desta mesma conversa, a análise que ela já
  produziu (uma tabela de achados, uma leitura de conjunto) é a fonte mais rica — não rederive, cite.

Cada achado vira uma linha candidata pra "Hipóteses a validar" (Passo 3), carregando o **nível de evidência que
já tem hoje** — "confirmado por consulta read-only em produção" é diferente de "achado por leitura de código,
nunca testado". Não infle o nível do que já existe.

## Passo 3 — Escrever a estrutura (nunca a prosa de intenção)

- **Arquivo:** `docs/specs/<slug>/brief-<slug>.md`.
- **Cabeçalho:** `Version: 0.1` · `Data` · `Doc Status: DRAFT` · `Achado em` (a sessão/skill que motivou) ·
  `Rastreado também em` (ponteiro pros itens de `backlog.md`/`known-issues.md` colhidos no Passo 2).
- **Seções, cada uma explicitamente marcada:**
  - `## O QUÊ` — `⏳ pendente — humano escreve` (nunca preenchido por esta skill).
  - `## PORQUÊ` — `⏳ pendente — humano escreve`.
  - `## Restrições de design` — `⏳ pendente — humano escreve`.
  - `## Hipóteses a validar` — **esta sim vem pré-populada**, uma linha por achado do Passo 2, formato
    `- **{achado}** — nível de evidência: {o que já se sabe} — fonte: {onde}`. Cabeçalho da seção marca
    explicitamente: "rascunho organizado pela skill a partir de achados já registrados — o humano decide o que
    vira hipótese de verdade, o que descarta, e o que falta."
- **Se a origem for item de `docs/specs/backlog.md`:** grave o ponteiro de mão dupla — mesma disciplina de
  `keelson-plan-init` (regra dura 10, `llm-dev-flow.md`) — o item do backlog ganha uma linha `Brief:
  brief-<slug>.md (DRAFT)`.

## Passo 4 — Gate humano (PARE aqui — obrigatório)

Pare e apresente:

> 📋 **Brief — preparação concluída**
> - **Slug/área:** {nome}
> - **Evidência colhida:** {N itens, de onde}
> - **Ainda pendente (seu, não meu):** O QUÊ, PORQUÊ, Restrições de design
>
> O arquivo nasceu em `DRAFT`. As próximas sessões de brief são suas em conjunto comigo — eu redijo o que você
> ditar, mas não decido o quê nem o porquê.

**Regra dura:** não escreva nenhuma palavra nas seções O QUÊ/PORQUÊ/Restrições, mesmo que o material colhido no
Passo 2 pareça sugerir uma resposta óbvia. Sugerir é redigir-quando-convidado; preencher sem ser ditado é
querer a spec.

## Handoff

- **Estado final:** `brief-<slug>.md` em `DRAFT`, com a preparação + evidência organizada, sem uma palavra de
  intenção.
- **Próximo passo, sempre:** sessão(ões) de brief normal — conversa, não skill — até `VALIDATED` (ver
  `llm-dev-player.md`; `llm-dev-flow.md`, "Sessões: brief"). Dali em diante, `keelson-plan-init`.
- **Se o Passo 1 concluiu que não é território de brief ainda, ou que é cedo demais:** sem arquivo criado,
  handoff aponta pra continuar em conversa normal.

## Regras duras (não viole)

1. **Nunca escreva O QUÊ/PORQUÊ/Restrições de design** — mesmo rascunho, mesmo sugestão. Essas seções só têm
   conteúdo quando o humano ditou.
2. **Nunca promova `Doc Status` pra `VALIDATED`** — gate humano sempre, revisão adversarial.
3. **Não invente hipótese sem fonte já existente no repo** — toda linha de "Hipóteses a validar" cita de onde
   veio.
4. **Não duplique brief existente** — checagem de duplicidade é sempre o primeiro passo.
5. **Não decida se um achado pertence ao brief ou não** — na dúvida, marque como candidato e deixe o humano
   confirmar/descartar.
6. **Não edite `.keelson/` nem o núcleo/trilogia** (quando rodando num projeto adotante).

## Nota de maturidade

Skill nova, `draft-para-testar` — nasce de um vão real, não de desenho especulativo: uma investigação de bug
real (OptiFlux, reconciliação de `cash_flows`, 2026-08-24) revelou uma decisão de desenho genuína (o detector
do achado S2 alarma/bloqueia/só reporta? `cash_flows` ganha FK real, mudando o comportamento deliberado de
"dinheiro sobrevive ao delete"?) — e o único próximo passo nomeado no vocabulário do método
(`keelson-plan-init`) tinha uma precondição que não estava satisfeita (não existe brief pra reconciliação de
caixa ainda). O handoff de `keelson-fix` apontou pra lá mesmo assim, por falta de nome melhor. Esta skill nomeia
o vão; o primeiro uso real é o próprio caso de origem. *Instrumentar antes de formalizar.*

**Nota de batismo:** o desenho inicial chamava esta skill de "andaime"/`scaffold` — descartado por destoar do
resto do vocabulário do método (registro de construção civil, sem eco em nenhum outro nome de skill). `prep` é
o nome neutro escolhido no lugar: descreve a função (preparação mecânica) sem carregar metáfora nova.
