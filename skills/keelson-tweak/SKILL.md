---
class: skill
name: keelson-tweak
status: draft-para-testar
description: Cobre a landing leve de um tweak do Método Keelson — um ajuste que não viola nada prometido, só estava sub-especificado — e escreve o tweak-<slug>.md em docs/tweaks/. Roda o Eixo 0 primeiro (isso violou algo prometido?); se sim, redireciona pra keelson-fix em vez de forçar um tweak. Descobre o alvo, confirma escopo + critério de aceite soft (antes/depois, sem red — nada quebrou) + confirmação de que nada congelado é tocado, escreve o tweak-doc com gate humano, e opcionalmente codifica na mesma sessão se o raio permitir. Use quando o usuário pedir um ajuste/refino/polish que não é bug nem feature nova, ou quando keelson-fix descobrir na triagem que não está diante de um bug. NÃO revisa (isso é sempre keelson-review-session, sempre sessão separada), NÃO promove BUILT/PILOT/PROD sozinha, e NÃO aceita tocar invariante/decisão congelada (isso não é tweak — aponta pra keelson-plan-init se já existe brief VALIDATED, ou keelson-brief-prep se não existe brief nenhum ainda).
---

# Tweak — refino de algo sub-especificado (não bug, não feature)

Implementa o modo-tweak descrito em `.keelson/llm-dev-flow-maintenance.md`: a porta irmã de `keelson-fix` para
quando nada foi prometido-e-violado, só sub-especificado — um ajuste dentro do que já existe, sem abrir
capacidade nova. Roda o mesmo **Eixo 0** que `keelson-fix` roda, começando pela porta oposta: aqui a suspeita
de partida é "não é bug", e o teste confirma ou corrige isso antes de qualquer trabalho.

## Escopo — o que esta skill faz e onde para

- Cobre a fronteira **ajuste sub-especificado → `tweak-<slug>.md` aterrissado** (e, opcionalmente, codificado).
- **Nunca invoca `keelson-review-session`** como sub-chamada da mesma sessão — mesma disciplina de
  independência de `keelson-fix`.
- **Não faz deploy** (`keelson-deploy`) nem valida em campo (`keelson-field-validation`).
- **Se o Eixo 0 revelar que algo FOI violado** — é bug de verdade, não tweak — você **não** força isso a virar
  `tweak-<slug>.md`. Para e aponta pra `keelson-fix` (ver Passo 1).
- **Se o ajuste toca invariante ou decisão congelada** — não é tweak, é mudança de spec de verdade. Para e
  aponta pra `keelson-plan-init` (revisão versionada, se já existe brief `VALIDATED` cobrindo a área) ou
  `keelson-brief-prep` (se não existe brief nenhum ainda — faz a preparação, colhendo o que você já levantou
  como evidência), conforme o caso.
- **Cerimônia proporcional:** um ajuste pequeno demais pra valer documento (2px de espaçamento, um rótulo) não
  ganha `tweak-<slug>.md` — vira só o ajuste + uma linha no `log/`, mesma régua de "bug trivial" do
  `keelson-fix`.

## Passo 1 — Eixo 0 + descoberta

- **Rode o Eixo 0** (`.keelson/llm-dev-flow-maintenance.md`, "Eixo 0"): o que o usuário quer ajustar **viola
  algo prometido** (código diverge de brief/invariante/ADR)? Se sim, **isto não é tweak** — pare e recomende
  `keelson-fix`, sem prosseguir.
- **Toca invariante ou decisão congelada?** Se sim, também não é tweak — pare e recomende `keelson-plan-init`
  (revisão versionada, se já existe brief `VALIDATED` cobrindo a área) ou `keelson-brief-prep` (se não
  existe brief nenhum ainda).
- **Reúna o pedido:** o que o usuário quer refinar, ou o achado que `keelson-fix` trouxe (Eixo 0 dele disparou
  e redirecionou pra cá).
- **Cheque duplicidade:** já existe um `tweak-<slug>.md` em voo pra este mesmo ajuste (`docs/tweaks/`, `git
  status`)? Se sim, continue aquele — não abra um segundo.

## Passo 2 — Escopo + critério de aceite (a landing leve)

Diferente de `keelson-fix`, não há causa-raiz nem reprodução — nada quebrou. O que substitui isso:

- **Escopo do ajuste:** o que muda, exatamente — descrito como antes/depois, não como um teste.
- **Confirmação de que nada congelado é tocado:** releia os §§ do brief relevantes, invariantes, ADRs. Se o
  ajuste fica dentro do que já foi prometido, confirme isso explicitamente — é essa confirmação, não um teste,
  que segura a fronteira tweak/feature-nova.

## Passo 3 — Gate humano (PARE aqui — obrigatório)

Pare e apresente:

> 🪶 **Tweak — descoberta**
> - **Ajuste:** {resumo curto}
> - **Antes:** {estado atual} · **Depois:** {estado desejado}
> - **Confirmação:** nada prometido foi violado; nada congelado é tocado
> - **Trivial?** {não — se fosse, já teria parado no Escopo}
>
> Confirma o ajuste? Isso vira `Doc Status: VALIDATED` do `tweak-<slug>.md`.

**Regra dura:** não escreva o `tweak-<slug>.md` como `VALIDATED` sem essa confirmação — nasce `DRAFT`, sobe pra
`VALIDATED` só depois do humano aprovar o escopo.

## Passo 4 — Escrever o `tweak-<slug>.md`

- **Arquivo:** `docs/tweaks/tweak-<slug>.md` (arquivo único, sem subpasta — mesma regra do `fix-<slug>.md`:
  promove pra `docs/tweaks/<slug>/{tweak-<slug>.md, tasks-<slug>.md}` só se de fato crescer o irmão, just-in-time).
- **Cabeçalho**, mesmo formato do `fix-<slug>.md` (ver `.keelson/llm-dev-flow-maintenance.md`, "O artefato"):
  `Version` · `Data` · `Doc Status` com o motivo inline · `Feature state` (`NOT_BUILT_LANDED` neste ponto) ·
  `Achado em` · `Rastreado também em`.
- **Corpo:** Escopo (o QUÊ) → Antes/depois (= critério de aceite) → Confirmação de não-toque em congelado →
  Risco de não fazer (se houver).
- **Feche com a Superfície de incerteza.**

## Passo 5 — Opcional: codificar na mesma sessão

Se o raio permitir, continue e implemente agora — mesma regra de "features pequenas colapsam"
(`llm-dev-flow.md`, "Sessões"). Se codificar:

1. **Sem *red*-first** — nada quebrou, não há o que provar "errado" antes de mudar. O critério de aceite é o
   antes/depois do Passo 2.
2. **Gatilho mecânico, sempre cheque:** este tweak **reescreveu o valor esperado de um teste pré-existente**?
   Se sim, marque isso explicitamente no handoff — mesmo raciocínio do lado fix: um tweak que precisa reescrever
   prova existente pode não ser tão inofensivo quanto pareceu no Eixo 0.

**Recomendação: deixe o diff sem commit**, mesma regra do resto do método — commit acontece uma vez, em
`BUILT` (ver `.keelson/llm-dev-flow.md`, "Recomendação: não commite até `BUILT`").

## Handoff — pare sempre antes da revisão

- **Se parou no Passo 1** (Eixo 0 achou bug, ou achou toque em congelado), não há `tweak-<slug>.md` — o handoff
  aponta direto pra `keelson-fix` ou `keelson-plan-init`, conforme o caso, sem os passos abaixo.
- Estado final desta sessão: `tweak-<slug>.md` em `NOT_BUILT_LANDED` (se parou no Passo 4) ou `NOT_BUILT_CODED`
  (se codificou no Passo 5) — **nunca `BUILT`**, essa promoção exige `keelson-review-session`, sempre sessão
  separada, **obrigatória por padrão** (override só humano, explícito, registrado no cabeçalho).
- **Se o gatilho mecânico do Passo 5 disparou**, isso vai no topo do handoff.
- **Aponte os próximos passos, nesta ordem, sem pular nenhum:** `keelson-review-session` (padrão, mesma régua
  do lado fix) → `keelson-deploy` (implantação) → `keelson-field-validation` (campo, produz `PILOT`/`PROD`).

## Regras duras (não viole)

1. **Nunca invoque `keelson-review-session` dentro desta sessão** — a independência é sessão separada, sem
   exceção estrutural.
2. **Se o Eixo 0 achar que algo foi violado, não force um `tweak-<slug>.md`** — redirecione pra `keelson-fix`,
   sem improvisar o conserto aqui.
3. **Se o ajuste toca invariante ou decisão congelada, não é tweak** — redirecione pra `keelson-plan-init`
   (revisão versionada) ou brief novo.
4. **Trivial não ganha `tweak-<slug>.md`** — proporcionalidade, sempre.
5. **Não edite o brief (`VALIDATED`)** nem `.keelson/`; divergência de fato vira revisão versionada, nunca
   edição silenciosa.
6. **Não invente** fato/dado/contexto — se faltar, pare e pergunte.

## Nota de maturidade

Skill nova, `draft-para-testar`. Nasce de dado de campo real, não de desenho especulativo: pelo menos 4 casos
de "isso não é bug" apareceram no OptiFlux entre 2026-08-13 e 2026-08-16 — um ajuste de layout tratado 100%
informal (sem critério de aceite escrito), outros registrados só como nota dentro do cabeçalho de um fix-doc
não relacionado, sem artefato próprio. O Eixo 0 (`.keelson/llm-dev-flow-maintenance.md`) e esta skill nomeiam o
que já estava acontecendo ad hoc. O que ainda falta medir é a automação (Eixo 0 + landing) rodando como skill
de verdade, não a distinção conceitual em si. *Instrumentar antes de formalizar.*
