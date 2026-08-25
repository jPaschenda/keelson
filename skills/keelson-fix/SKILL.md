---
class: skill
name: keelson-fix
status: draft-para-testar
description: Cobre a descoberta, reprodução e causa-raiz de um bug do Método Keelson — a aterrissagem corretiva — e escreve o fix-<slug>.md em docs/fixes/. Roda o Eixo 0 primeiro (isso violou algo prometido?); se não, redireciona pra keelson-tweak em vez de forçar um fix. Descobre o defeito, reproduz (red, piso sempre), investiga a causa-raiz, triagem (trivial? dono contradito? raio? escopo nunca implementado?), escreve o fix-doc com gate humano, e opcionalmente codifica na mesma sessão se o raio permitir. Use quando o usuário reportar um bug, ou quando outra skill (keelson-coding, keelson-review-session, keelson-field-validation) tropeçar num bug fora do próprio escopo e apontar pra cá. NÃO revisa (isso é sempre keelson-review-session, sempre sessão separada), NÃO promove BUILT/PILOT/PROD sozinha, NÃO força uma lacuna de escopo grande demais a virar fix (aponta pra keelson-plan-init se já existe brief VALIDATED pra área, ou keelson-brief-prep se não existe brief nenhum ainda), e NÃO força um ajuste sub-especificado a virar fix (aponta pra keelson-tweak).
---

# Fix — descoberta, reprodução e causa-raiz de bug

Implementa o modo-bug descrito em `.keelson/llm-dev-flow-maintenance.md`: a **seta de volta operacional** que dispara
nos degraus vivos (`PILOT`/`PROD`) quando código que já roda contradiz a camada congelada. Um bug é uma
**suposição refutada** — e investigar a causa-raiz **é uma aterrissagem** (a mesma atividade de
`keelson-phase-landing`, só que reconciliando um comportamento observado contra o esperado, em vez de um brief
contra o código). Você é o orquestrador dessa aterrissagem corretiva: descobre, reproduz, investiga, triA, e
escreve o `fix-<slug>.md`. Pode, se o raio permitir, continuar e codificar na mesma sessão — mas **nunca**
revisa o próprio trabalho.

## Escopo — o que esta skill faz e onde para

- Cobre a fronteira **defeito observado → `fix-<slug>.md` aterrissado** (e, opcionalmente, codificado).
- **Nunca invoca `keelson-review-session`** como sub-chamada da mesma sessão — a independência que a revisão
  promete é justamente ser outra sessão; nenhuma skill pode se auto-conceder isso.
- **Não faz deploy** (`keelson-deploy`) nem valida em campo (`keelson-field-validation`).
- Se o bug for **trivial**, você recomenda o caminho leve (commit + linha no `log/` + teste de regressão) e
  **para** — não escreve `fix-<slug>.md` nenhum. Cerimônia proporcional ao raio, sempre.
- Se a investigação revelar que **não é bug nenhum** — o requisito do brief sempre esteve certo, só nunca foi
  cumprido, e o gap é grande demais pra um conserto — você **não** força isso a virar `fix-<slug>.md`. Registra
  o achado e aponta pra `keelson-plan-init` (ver Passo 3).
- Se o **Eixo 0** (Passo 1) revelar que **nada foi violado**, só sub-especificado — você **não** força isso a
  virar `fix-<slug>.md` nem investiga causa-raiz de algo que não está errado. Para e aponta pra `keelson-tweak`.
- **Achado real que não vira `fix-<slug>.md`** — porque o humano adia a decisão no Passo 4, ou porque foi um
  achado secundário notado dentro de outro artefato (outra "Validação de campo", outra Superfície de
  incerteza) — **nunca fica só em prosa, enterrado.** Vira uma linha em `wiki/known-issues.md` (ver Passo 4 e
  Passo 5). `known-issues.md` é sobre o **sistema** que o projeto constrói — não confundir com o
  `known-issues.md` do próprio Método Keelson, que é escopo de `keelson-wiki-update`.

## Passo 1 — Eixo 0, descoberta e reprodução (red é piso, sempre)

- **Rode o Eixo 0 primeiro** (`.keelson/llm-dev-flow-maintenance.md`, "Eixo 0"): o que foi reportado **viola
  algo prometido** (código diverge de brief/invariante/ADR)? Se **não** — nada foi violado, só está
  sub-especificado — **isto não é bug**. Pare aqui e recomende `keelson-tweak`, sem investigar causa-raiz de
  algo que não está errado.
- **Reúna o defeito:** o que o usuário reportou, ou o achado que outra skill trouxe (`keelson-field-validation`,
  `keelson-review-session`, `keelson-coding` — qualquer uma que tenha tropeçado num bug fora do próprio escopo).
- **Reproduza.** Diferente do forward (onde *red*-first é proporcional ao raio), aqui é **piso**: a reprodução
  vem de graça de um bug real, então o custo é quase zero e sempre se justifica. A reprodução **é** o critério
  de aceite — um alvo que você não inventou.
- **Cheque duplicidade:** já existe um `fix-<slug>.md` em voo pra este mesmo defeito (`docs/fixes/`,
  `wiki/known-issues.md`, `git status`)? Se sim, continue aquele — não abra um segundo.

## Passo 2 — Causa-raiz (a aterrissagem corretiva)

Leia o current-state (`wiki/index.md` → `architecture.md`/`data-model.md`, e o código real) e reconcilie o
comportamento observado contra o esperado — a mesma disciplina de `keelson-phase-landing`, aplicada ao
comportamento em vez de ao requisito. Não pare no sintoma: a causa-raiz é o que vai virar o critério de "o teste
denunciaria o código antigo".

## Passo 3 — Triagem (dois eixos ortogonais + trivialidade + escopo)

- **Trivial?** (typo, texto de GUI, off-by-one óbvio) → **pare aqui.** Recomende: commit + uma linha de
  incidente no `log/` + o teste de regressão. Sem `fix-<slug>.md`, sem mais nenhum passo desta skill.
- **Escopo nunca implementado?** A causa-raiz não é "código diverge do esperado" nem "requisito estava errado"
  (isso é o Eixo 1, abaixo) — é algo que deveria estar coberto e nunca foi cumprido, grande demais pra um
  conserto. → **pare aqui, não escreva `fix-<slug>.md`** em nenhum dos dois casos abaixo — mas distinga qual é,
  porque o próximo passo muda:
  - **A lacuna é de um brief que já existe e está `VALIDATED`** (o requisito estava lá, nunca foi construído) —
    registre o achado em `docs/specs/backlog.md`, apontando pro brief § e pro que achou a lacuna. Próximo passo:
    `keelson-plan-init` — revisão versionada do plano (ver `.keelson/llm-dev-flow.md`, "Revisão versionada do
    `plan`").
  - **Não existe brief nenhum cobrindo essa área** (o achado é sobre um assunto nunca especificado, não uma
    promessa não cumprida de um brief existente) — registre o achado em `docs/specs/backlog.md` do mesmo jeito,
    mas o próximo passo é `keelson-brief-prep` (faz a preparação do brief novo, colhendo esta investigação
    como evidência) — **não** `keelson-plan-init`, que exige uma precondição (`VALIDATED`) que ainda não existe.
- **Eixo 1 — dono contradito:** nenhum (código divergiu do as-designed, sem culpa de decisão) / invariante-ADR
  (decisão congelada estava errada → supersede) / brief (requisito estava errado → revisão versionada).
- **Eixo 2 — raio de explosão:** decide a cerimônia e se você pode colapsar os próximos passos na mesma sessão.

## Passo 4 — Gate humano (PARE aqui — obrigatório)

Pare e apresente:

> 🔧 **Fix — descoberta**
> - **Defeito:** {resumo curto}
> - **Reprodução:** {comando/passo que falha, red confirmado}
> - **Causa-raiz:** {resumo}
> - **Dono contradito:** {nenhum / invariante-ADR / brief} · **Raio:** {baixo/médio/alto}
> - **Trivial?** {não — se fosse, já teria parado no Passo 3}
>
> Confirma a abordagem de conserto? Isso vira `Doc Status: VALIDATED` do `fix-<slug>.md`.

**Regra dura:** não escreva o `fix-<slug>.md` como `VALIDATED` sem essa confirmação — nasce `DRAFT`, sobe pra
`VALIDATED` só depois do humano aprovar a abordagem.

**Se o humano adiar** ("real, mas não agora") em vez de confirmar: não escreva `fix-<slug>.md` nenhum — o
achado ainda não tem chão pra virar artefato de conserto. Em vez disso, adicione uma linha em
`wiki/known-issues.md` (defeito + reprodução + ponteiro pra esta sessão/`log/`), pra ele não desaparecer sem
rastro. Isso fecha quando alguém decidir tratá-lo — aí sim nasce o `fix-<slug>.md`.

## Passo 5 — Escrever o `fix-<slug>.md`

- **Arquivo:** `docs/fixes/fix-<slug>.md` (arquivo único, sem subpasta — promove pra `docs/fixes/<slug>/
  {fix-<slug>.md, tasks-<slug>.md}` só se de fato crescer o irmão `tasks-<slug>.md`, just-in-time. Não crie a
  subpasta antecipando o crescimento).
- **Cabeçalho**, no formato endurecido por campo (ver `.keelson/llm-dev-flow-maintenance.md`, "O artefato"):
  `Version` · `Data` · `Doc Status` com o motivo inline · `Feature state` (`NOT_BUILT_LANDED` neste ponto) com
  motivo inline se houver desvio · `Achado em` (qual sessão/skill achou) · `Rastreado também em` (ponteiro pro
  `tasks-fase<N>` § se a causa mora numa fase anterior, `wiki/log/`, `known-issues.md`).
- **Corpo:** Defeito (o QUÊ, dado pela realidade) → Reprodução (= critério de aceite) → Causa-raiz → Plano de
  conserto → Risco de não corrigir.
- **Feche com a Superfície de incerteza.**
- **Se, escrevendo a Superfície de incerteza ou uma "Validação de campo", você notar um defeito diferente do
  que está consertando** — não deixe-o só mencionado em prosa. Aplique a regra dura 2 (nova triagem, Passo 3)
  ou, se não há fôlego pra investigar agora, registre-o direto em `wiki/known-issues.md` com o ponteiro pra
  este `fix-<slug>.md`.

## Passo 6 — Opcional: codificar na mesma sessão

Se o raio permitir, você pode continuar e implementar o conserto agora — mesma regra de "features pequenas
colapsam" (`llm-dev-flow.md`, "Sessões"). Se preferir (raio maior, ou você quer uma cabeça diferente pra
codificar), pare aqui e entregue pra uma sessão de `keelson-coding` separada, apontando pro `fix-<slug>.md`
`VALIDATED`. Se codificar:

1. **Red confirmado antes** (já é o Passo 1) → implemente → **green**.
2. **Gatilho mecânico, sempre cheque:** o fix **reescreveu o valor esperado de um teste pré-existente** (não só
   adicionou teste novo)? Se sim, **marque isso explicitamente no handoff** — é a assinatura clássica de
   confiante-mas-errado (o mesmo agente que escreveu o fix reescreveu a prova de que está certo) e eleva a
   recomendação de revisão, **mesmo em raio baixo**.
3. **Fix que muda comportamento corrige o `architecture.md` junto** (spec-rot ao contrário).

**Recomendação: deixe o diff sem commit**, mesmo se codificou nesta sessão (Passo 6) — a revisão roda contra o
working tree; commit acontece uma vez, em `BUILT` (ver `.keelson/llm-dev-flow.md`, "Recomendação: não commite
até `BUILT`"). **Se esta sessão editou um diff que já tinha veredito de `keelson-review-session`** (ex.:
endereçando um achado não-bloqueante que a própria revisão recomendou), sinalize isso no handoff — o veredito
antigo não cobre as linhas novas (ver `.keelson/llm-dev-flow.md`, "Um veredito não estica sozinho").

## Handoff — pare sempre antes da revisão

- **Se parou no Passo 1 por Eixo 0** (nada foi violado), não há `fix-<slug>.md` — o handoff aponta direto pra
  `keelson-tweak`, sem os passos abaixo.
- **Se parou no Passo 3 por escopo nunca implementado**, não há `fix-<slug>.md` — o handoff aponta direto pra
  `keelson-plan-init` (se já existe brief `VALIDATED` cobrindo a área) ou `keelson-brief-prep` (se não
  existe brief nenhum ainda), sem os passos de revisão/deploy/campo abaixo (não fazem sentido pra um achado que
  virou item de backlog, não código).
- **Se o Passo 4 foi adiado**, não há `fix-<slug>.md` — o handoff confirma a linha nova em
  `wiki/known-issues.md`, sem os passos abaixo.
- Estado final desta sessão: `fix-<slug>.md` em `NOT_BUILT_LANDED` (se parou no Passo 5) ou `NOT_BUILT_CODED`
  (se codificou no Passo 6) — **nunca `BUILT`**, essa promoção exige `keelson-review-session`, sempre sessão
  separada, **obrigatória por padrão** (override só humano, explícito, registrado no cabeçalho — nunca decisão
  desta skill).
- **Se o gatilho mecânico do Passo 6 disparou**, isso vai no topo do handoff, não enterrado na Superfície de
  incerteza.
- **Aponte os próximos passos, nesta ordem, sem pular nenhum:** `keelson-review-session` (padrão — ou, raio
  baixo sem o gatilho mecânico, a orientação em camadas de sub-agente que ela descreve) → `keelson-deploy`
  (implantação) → `keelson-field-validation` (campo, produz `PILOT`/`PROD`). Omitir `keelson-deploy` da lista
  já aconteceu uma vez em campo no lado feature — não repita no lado fix.

## Regras duras (não viole)

1. **Nunca invoque `keelson-review-session` dentro desta sessão** — a independência é sessão separada, sem
   exceção estrutural.
2. **Bug fora do próprio escopo não se conserta improvisando** — se, investigando este bug, você achar um
   segundo bug não relacionado, pare e trate como uma nova triagem (Passo 3), não um adendo silencioso.
3. **Trivial não ganha `fix-<slug>.md`** — proporcionalidade, sempre.
4. **Red é piso, não proporcional** — diferente do forward, aqui não se pula.
5. **Não edite o brief (`VALIDATED`)** nem `.keelson/`; divergência de requisito vira revisão versionada do
   brief (Eixo 1), nunca edição silenciosa.
6. **Não invente** fato/dado/contexto — se faltar, pare e pergunte.
7. **Lacuna de escopo grande demais não vira `fix-<slug>.md` forçado** — pare no Passo 3 e aponte pra
   `keelson-plan-init` (brief já existe) ou `keelson-brief-prep` (brief não existe ainda).
8. **Achado real que não vira `fix-<slug>.md` — adiado no Passo 4, ou secundário dentro de outro artefato —
   nunca fica só em prosa.** Vira linha em `wiki/known-issues.md`, sempre.
9. **Se o Eixo 0 achar que nada foi violado, não force um `fix-<slug>.md`** — redirecione pra `keelson-tweak`
   sem investigar causa-raiz de algo que não está errado.

## Nota de maturidade

Skill em `draft-para-testar` — nasce nesta leva, mas a maquinaria que ela empacota **já não é desenho**: 3
`fix-<slug>.md` reais rodaram no OptiFlux antes desta skill existir (`docs/fixes/optidash-fleet-actions-silent-error/`,
`orch-market-pair-mismatch/`, `orch-tatica-unitaria-volta-sinal/`), e o template de cabeçalho acima foi
extraído deles, não inventado. O que ainda falta medir é a **automação** (descoberta + gate) rodando como
skill, não a disciplina em si. *Instrumentar antes de formalizar.*

**Dado de campo novo:** a primeira sessão real que investigou um achado de campo (Fase 4, Camada de
Orquestração) descobriu, na causa-raiz, que não era bug — era o diálogo real que o brief §2.1 sempre pediu e
nunca foi construído. A skill (ainda sem esta regra escrita) improvisou corretamente: não escreveu
`fix-<slug>.md`, registrou o item no `backlog.md`, apontou pra `keelson-plan-init`. O Passo 3 acima só
nomeia o que ela já fez sozinha.

**Segundo dado de campo:** uma sessão de `keelson-wiki-update` (a skill de reconciliação genérica, sem escopo
de sistema) notou, por conta própria, dois achados do mesmo dia — o bug de legibilidade acima e um
não-determinismo separado — vivendo só dentro do `log/`, sem linha em `known-issues.md`, e sinalizou isso como
fora do próprio escopo dela. A regra dura 8 fecha essa lacuna aqui, não em `keelson-wiki-update` — achado de
sistema é escopo de quem investiga sistema.

**Terceiro dado de campo:** a mesma triagem do Fase 4 GUI que motivou a regra 7 também expôs o caso oposto —
um achado que **não era bug nenhum**, era um ajuste de layout (320px→450px) sem nada prometido violado. Sem
Eixo 0 nem `keelson-tweak`, esse achado foi tratado 100% informal, sem critério de aceite escrito, pendurado na
mesma leva de revisão de dois bugs de verdade. O Eixo 0 (regra dura 9, `.keelson/llm-dev-flow-maintenance.md`)
fecha essa lacuna nomeando a porta certa.

**Quarto dado de campo — primeiro uso real do caminho trivial (OptiFlux, 2026-08-22):** dois pares de itens já
pré-classificados como "trivial" em `wiki/known-issues.md` (falhas de suíte por fixture de tempo não
congelada, colidindo com lógica de calendário BRT do produto). O Eixo 1 reconheceu a classificação existente e
pulou o `fix-<slug>.md` inteiro — mas não pulou a disciplina: reproduziu o *red* de cada caso isoladamente
(arquivos de repro descartáveis, nunca commitados) antes de corrigir, confirmando o mecanismo exato descrito no
known-issues antes de tocar código. Suítes completas rodadas, sem quebra; linhas removidas do known-issues;
entrada registrada em `wiki/log/`. Primeira confirmação de que o atalho trivial (Passo 1, "conserto trivial sai
do fluxo inteiro") preserva a disciplina de verificação mesmo pulando a cerimônia do documento.
