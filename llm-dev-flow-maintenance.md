---
schema_version: "0.21"
class: core-satellite
status: draft-pre-validacao
data: 2026-07-28
satellite_of: llm-dev-flow.md
---

# Método Keelson Flow — Manutenção (satélite)

> **Satélite de [`llm-dev-flow.md`](llm-dev-flow.md), aberto sob demanda — modo bug.** A maior parte da vida de um sistema é passada **vivo**, mantendo verdadeiro o que já roda; ali o trabalho dominante deixa de ser construir para a frente. Este satélite cobre a correção de bug **e o refino de algo sub-especificado (tweak)** — como os dois encaixam no fluxo, **sem mecanismo novo**. Abra quando o trabalho for **consertar ou refinar** (não construir para a frente). O frame forward (brief→plan→tasks, eixos, escada, guardrails) e o roteador vivem no núcleo [`llm-dev-flow.md`](llm-dev-flow.md).

## Manutenção — a seta de volta operacional (PILOT → PROD)

Até aqui o fluxo levou uma feature `brief`→`plan`→`tasks` escada acima, até um sistema vivo. Mas a maior parte da *vida* de um sistema é passada **vivo** — e ali o trabalho dominante não é construir para a frente, é **manter verdadeiro o que já roda**. Keelson cobre isso **sem mecanismo novo**: manutenção é a **seta de volta** (a mesma das "Hipóteses a validar") disparando nos **degraus vivos — `PILOT` e `PROD`** —, onde código que já roda contradiz a camada congelada. Funcionalidade nova sobre um sistema vivo é só um novo ciclo `brief`→`plan`→`tasks`; esta seção trata da outra metade: **os bugs, e as limitações assumidas em que eles às vezes se transformam.**

O reframe: **um bug é uma suposição refutada, descoberta num degrau vivo.** A escada já antecipa isso na letra (o degrau `PROD`: *"aprendizado novo vira novo spec/ADR por supersede, não edição do congelado"*). Manutenção só **nomeia e instrumenta** o que a escada já implica — e estende de `PROD` para `PILOT`, porque a partir de `PILOT` o congelamento já está ligado (o brief é quase-imutável), então a disciplina anti-spec-rot já vale: divergência descoberta **não** se conserta editando o congelado em silêncio.

### A correção nasce já aterrissada

Repare no que é uma investigação de causa-raiz de bug: ler o current-state (via `wiki/index.md`→ `architecture.md`→código) e reconciliar uma expectativa contra o que o código realmente faz. **Isso é uma aterrissagem** — estruturalmente a mesma atividade (ver "Aterrissagem" no núcleo [`llm-dev-flow.md`](llm-dev-flow.md)). A única diferença: a aterrissagem forward reconcilia uma expectativa **de design** (o § do brief); a causa-raiz reconcilia uma expectativa **de comportamento que não está se confirmando na solução** (a reprodução do bug).

Consequência: o artefato de correção **nasce já aterrissado**. Ele não tem a fase divergente do brief, porque a realidade — a falha — já escreveu a spec dele. A **reprodução é o critério de aceite**: o alvo objetivo que o agente não escreveu (o antídoto ao confiança-mas-errado, ver "Guardrails" no núcleo [`llm-dev-flow.md`](llm-dev-flow.md)), e executável de graça. Por isso ele vive no lado `plan`/`tasks`, não no lado `brief`.

### O artefato: `fix-<slug>.md`, no balde `docs/fixes/`

`docs/fixes/` é o **irmão corretivo** de `docs/specs/`: onde `specs/` é o tier prescritivo **forward** (o que *vai ser construído*), `fixes/` é o tier prescritivo **corretivo** (como se *conserta o presente defeituoso*). Não é um registro de bugs — isso seria um tracker, e o dono da lista aberta é externo (ver `known-issues.md` e "não duplicar o dono", abaixo); `fixes/` guarda os **planos de correção em voo**. O dono da estrutura de pasta é o [`llm-dev-memory.md`](llm-dev-memory.md); aqui vive só o encaixe no fluxo.

O `fix-<slug>.md` é **dominantemente um plano**, não um brief. Carrega, comprimidos:
- no **cabeçalho**, o QUÊ dado pela realidade — o *defect statement* + a **reprodução** (que é o critério de aceite);
- no **corpo**, o COMO — a **causa-raiz** e o **plano de conserto**.

É **autocontido**: não referencia um brief separado, porque o brief-equivalente é a própria reprodução. Colapsa `brief`+`plan`+`tasks` num doc pela mesma regra que faz "features pequenas colapsarem". **Nasce arquivo único, sem subpasta** — `docs/fixes/fix-<slug>.md` — e só **se de fato crescer o irmão `tasks-<slug>.md`** é que promove, *just-in-time*, pra `docs/fixes/<slug>/{fix-<slug>.md, tasks-<slug>.md}` (espelhando `specs/`). Dado de campo (11 fixes reais, OptiFlux, até 2026-08-17): nenhum precisou do irmão — a subpasta antecipada só atrapalhava referenciar o arquivo (path repete o slug duas vezes), sem nunca pagar o benefício que a justificava. Não pré-provisione a subpasta.

**Template do cabeçalho** (endurecido pelos 3 primeiros fixes reais do OptiFlux — não é mais desenho): `Version` (bump a cada seção nova, o doc cresce por sessão) · `Data` · `Doc Status` **com o motivo inline** ("VALIDATED — operador aprovou a abordagem X") · `Feature state` **com o motivo inline quando há desvio** (ex.: "BUILT — promovido sem revisão independente separada, aceito pelo operador, raio Y") · `Achado em` (qual sessão/skill achou — `keelson-field-validation`, `keelson-review-session`, etc.) · `Rastreado também em` (ponteiro pro `tasks-fase<N>` § se a causa mora numa fase anterior, `wiki/log/`, `known-issues.md`). O corpo cresce por seção conforme sessões diferentes tocam o mesmo doc (Defeito/Reprodução/Causa-raiz → Plano pendente de validação → Conserto aplicado → Reteste de campo → Deploy) — um `fix-<slug>.md` real span várias sessões, não uma só.

Os dois eixos de estado, comprimidos:
- **`Doc Status`:** `DRAFT` → `VALIDATED` (o humano aprova *a abordagem de conserto* — gate humano, como todo `VALIDATED`) → `ARCHIVED`.
- **`Feature state`:** a **mesma escada completa** do núcleo (ver `llm-dev-flow.md`, "Sub-degraus"), não uma versão reduzida — `NOT_BUILT` (só o defect statement) → `NOT_BUILT_LANDED` (reprodução *red* + causa-raiz + plano, escritos por `keelson-fix`) → `NOT_BUILT_CODED` (fix *green* — *red*-antes-do-fix é **piso sempre**, não proporcional ao raio, diferente do forward) → `BUILT` (revisão independente aprovou, ou override registrado — ver "O guardrail do fix", abaixo) → `BUILT_DEPLOYED_PILOT`/`BUILT_DEPLOYED_PROD` (`keelson-deploy`) → `PILOT` (raro — entrega gradual/*canary* da correção) / `PROD` (correção em produção), via `keelson-field-validation`.

**Destino terminal:** ao chegar em `PROD`, `fix-<slug>.md` (+ `tasks`) vai para `docs/archive/`, exatamente como `brief`/`plan`/`tasks` de uma feature — deixa de ser current-state e vira registro histórico do conserto.

### Proporcionalidade — nem todo bug vira documento

A espinha de sempre (rigor proporcional ao raio de explosão) vale aqui com força: **bug trivial não ganha `fix-<slug>.md`.** Um typo, um texto errado na GUI, um off-by-one óbvio = commit + uma linha de incidente no `log/` + o teste de regressão. O documento só se paga quando há **incerteza de causa-raiz**, **múltiplos consertos possíveis**, ou **raio alto** que exige o humano assinar a abordagem antes do código. Escrever um plano de correção para consertar um rótulo de botão é a mesma cerimônia-à-toa que a seção "o que não fazer" combate.

### Eixo 0 — antes de tudo: isso violou algo prometido?

Nem toda mudança sobre código já existente é bug. Antes dos dois eixos abaixo (que já pressupõem bug), a
pergunta que decide se você está no lugar certo:

- **Algo prometido foi violado** (código diverge de brief/invariante/ADR)? → é bug, segue para os Eixos 1/2.
- **Nada foi violado, só está sub-especificado** (o comportamento cumpre o prometido, mas o detalhe nunca foi
  pensado com cuidado — um painel apertado, uma resposta que funciona mas nunca foi refinada)? → **não é bug**,
  é **tweak**. Pare a triagem de bug aqui e vá para "O artefato: `tweak-<slug>.md`", abaixo.
- **Toca invariante ou decisão congelada?** Nem bug nem tweak — é mudança de spec de verdade. Se já existe
  brief `VALIDATED` cobrindo a área, é revisão versionada dele — `keelson-plan-init`. **Se não existe brief
  nenhum pra essa área**, o achado não tem a precondição que `keelson-plan-init` exige (`VALIDATED`) — o
  próximo passo nomeado é `keelson-brief-prep`: faz a preparação do brief novo (arquivo, evidência já
  levantada), nunca a intenção — o QUÊ/PORQUÊ seguem sendo trabalho humano (ver `llm-dev-player.md`).

Este é o **mesmo teste** que `keelson-fix` e `keelson-tweak` rodam primeiro, cada uma a partir da porta em que
o usuário bateu — o resultado do teste importa, não qual das duas skills foi chamada (ver "Sessões de
manutenção", abaixo). Errar a porta custa uma frase no Passo 1, não uma investigação inteira.

### O artefato: `tweak-<slug>.md`, no balde `docs/tweaks/`

`docs/tweaks/` é o terceiro irmão de `docs/specs/`/`docs/fixes/`: onde `specs/` constrói o futuro e `fixes/`
conserta o presente defeituoso, `tweaks/` **refina o presente que já cumpre o prometido**. Nasce vazio, como
os outros dois.

Diferença central pro `fix-<slug>.md`: **não há reprodução, porque nada quebrou.** O critério de aceite é
**antes/depois soft** — o estado atual e o estado desejado, descritos, sem *red* (não existe um teste que prove
o sistema "errado", porque não está). O que substitui o gate de reprodução é a **confirmação explícita de que
nada congelado é tocado** — é essa confirmação, não um teste, que impede um tweak disfarçado de mudar uma
decisão de verdade.

**Mesma regra de arquivo do `fix-<slug>.md`:** nasce único, sem subpasta — `docs/tweaks/tweak-<slug>.md` — e só
promove pra `docs/tweaks/<slug>/{tweak-<slug>.md, tasks-<slug>.md}` se de fato crescer o irmão `tasks`,
*just-in-time*. Não pré-provisione a subpasta.

Reusa a **mesma escada completa** do núcleo (`NOT_BUILT` → `NOT_BUILT_LANDED` → `NOT_BUILT_CODED` → `BUILT` →
`BUILT_DEPLOYED_PILOT`/`BUILT_DEPLOYED_PROD` → `PILOT`/`PROD`) e as **mesmas skills genéricas**
(`keelson-coding`, `keelson-review-session`, `keelson-deploy`, `keelson-field-validation`) — nenhuma ganha uma
versão "de tweak", só mais um valor possível de artefato de entrada. Revisão independente continua
**obrigatória por padrão** (mesma regra do fix) — "nada quebrou" não é motivo pra pular o guardrail nº 1, é só
motivo pra não ter *red*-first.

### Triagem — dois eixos ortogonais + depósitos

Quando um bug entra (Eixo 0 já confirmou que é bug), duas perguntas independentes o classificam:

**Eixo 1 — qual dono congelado a realidade contradisse** (decide o que é *superseded*/revisado). É a versão-em-produção da classificação de três vias da aterrissagem:

| Dono contradito | O que a realidade revelou | Movimento |
|---|---|---|
| **Nenhum** | o as-designed estava certo; o código divergiu | corrige o código — **sem** supersede |
| **Invariante / ADR** | uma decisão congelada estava errada | **supersede** do ADR + atualiza `invariants.md` |
| **Brief** | o requisito estava errado (hipótese refutada em degrau vivo) | **revisão versionada** do brief (bump + `log/`) |

**Eixo 2 — raio de explosão** (decide a cerimônia): um bug cosmético e um que corrompe dado financeiro entram pela mesma porta e saem por guardrails diferentes — a mesma tabela de "Guardrails por transição" (núcleo [`llm-dev-flow.md`](llm-dev-flow.md)).

**Os depósitos** (ortogonais aos eixos — não são escolha *ou/ou*): todo bug não-trivial deixa um **incidente no `log/`** (que já é "só incidentes/diagnóstico"); um bug cuja lição merece não-repetir deixa um **ADR** (o critério de ADR já cobre *"repetir um erro já corrigido"* — não é preciso ser "arquitetural"); um bug que estabelece um "nunca mais" comportamental deixa uma linha em **`invariants.md`**, tendo o **teste de regressão** como sua checagem mecânica.

### O guardrail do fix — o teste de regressão *red* é o piso

O vício central do agente (confiança-mas-errado) é **pior corrigindo bug** do que construindo feature, por três razões: (1) ele "conserta" o **sintoma** sem a **causa-raiz** e o teste fica verde; (2) ele escreve a reprodução **e** o fix — o mesmo mal-entendido compartilhado que já contamina testes; (3) regressão verde prova que **o sintoma sumiu**, não que a causa foi resolvida nem que nenhum bug novo entrou. Daí:

- **Teste-*red*-antes-do-fix é a DoD *default* do bug** — não proporcional como no forward, mas **piso**: num bug a reprodução vem de graça, então o custo do *red* é quase zero e sempre se justifica.
- **A reprodução nasce do observável de produção** (o log real, o *trace*, o dado corrompido) — um alvo externo que o agente não inventou.
- **Revisão independente é obrigatória por padrão, feature ou fix** (ver `llm-dev-flow.md`, "Revisão independente é obrigatória por padrão") — não só em alto raio. A pergunta que ela faz aqui é *"foi a causa-raiz ou o sintoma?"*, não *"o teste passou?"* — e isto é **mais** importante em fix do que em feature nova, porque o padrão-armadilha do modo-bug é justamente reescrever a própria prova (ver acima). Pular exige override humano explícito, registrado no cabeçalho (nunca a skill decidindo sozinha). **Gatilho mecânico que não perdoa, independente de raio:** se o fix **reescreveu o valor esperado de um teste pré-existente** (não só adicionou teste novo), a revisão deixa de ser opcional na prática — é a assinatura exata de "o autor também escreveu o que prova que ele está certo".
- **Fix que muda comportamento corrige o `architecture.md` junto** — é o spec-rot ao contrário: manutenção é onde o as-built se re-acerta; um fix que deixa o `architecture.md` descrevendo o comportamento antigo cria uma mentira citável na camada as-built.

### O ponteiro do que está em tratamento — `known-issues.md`

Para o **agente que começa frio** não retrabalhar o que já se conhece — nem construir sobre comportamento bugado achando que é correto —, a memória mantém um **`wiki/known-issues.md`**: o ledger *grep-able* do que está **quebrado ou limitado agora e em tratamento** nos degraus vivos. É o **par transiente do `invariants.md`** (um diz o que é sempre verdade; o outro, o que está quebrado agora). **Puramente transiente:** a linha sai quando o problema resolve — por fix embarcado **ou** por decisão *won't-fix* —, e nada durável mora ali (a lição vai para ADR/`invariants.md`/`architecture.md`/`log/`). Não é um tracker: a lista completa + triagem é do issue tracker externo (**não duplicar o dono**); este é a projeção curada para o agente. Formato e regras: [`llm-dev-memory.md`](llm-dev-memory.md).

### Sessões de manutenção — e as skills que as cobrem

Segue a regra geral: **uma sessão = uma transição de estado de um artefato.** Uma sessão de manutenção move o `fix-<slug>.md` por uma transição nomeável — reproduzir → achar a causa-raiz → `VALIDATED` a abordagem, ou um degrau da escada. A *entrega* humana no modo-fix é a **reprodução curada** (ver [`llm-dev-player.md`](llm-dev-player.md)).

- **`keelson-fix`** roda o Eixo 0 primeiro, depois cobre descoberta + reprodução + causa-raiz + triagem (os dois
  eixos de bug) + escreve o `fix-<slug>.md`; opcionalmente continua e codifica na mesma sessão se o raio
  permitir (mesma regra de "features pequenas colapsam", `llm-dev-flow.md`, "Sessões"). Para sempre antes da
  revisão.
- **`keelson-tweak`** é a porta irmã: mesma forma, artefato diferente. Roda o Eixo 0 também; se ele achar que
  algo **foi** violado (é bug de verdade), redireciona pra `keelson-fix` em vez de forçar um `tweak-<slug>.md`.
  **Duas portas, um só teste** — não importa qual delas o usuário lembrou de chamar, o Eixo 0 corrige o rumo no
  primeiro passo, antes de qualquer investigação custosa.
- **`keelson-review-session`**, **`keelson-deploy`**, **`keelson-field-validation`** são as mesmas skills do
  fluxo forward, genéricas sobre o artefato — não existe uma versão "de fix" nem "de tweak" delas. A única
  diferença de comportamento pro lado fix: a pergunta semântica dominante da revisão vira "causa-raiz ou
  sintoma?" (ver "O guardrail do fix", acima), e o *red*-first é piso sempre, não proporcional. Pro lado tweak,
  não há *red*-first (nada quebrou) — a revisão ancora no critério antes/depois do `tweak-<slug>.md`.
- Qualquer skill do fluxo forward que tropeçar num bug **fora do próprio escopo** (de outra fase, ou achado incidental) não conserta inline — aponta para `keelson-fix` (regra dura em `llm-dev-flow.md`, "Sessões").
- **`keelson-brief-prep`** é o terceiro nome que o Eixo 0 pode produzir: quando o achado é decisão de
  desenho legítima, mas **nenhum brief cobre a área ainda**. Prepara a estrutura do `brief-<slug>.md` (evidência
  colhida, intenção sempre pendente) e para. Não é uma porta de entrada própria — ninguém chama esta skill
  reportando um bug direto — só o destino de saída quando o Eixo 0, rodando a partir de `keelson-fix` ou
  `keelson-tweak`, aterrissa em "brief novo" em vez de "brief já existe".

> **Status: endurecendo com o campo.** Já rodou — 3 casos reais no OptiFlux (`docs/fixes/optidash-fleet-actions-silent-error/`, `orch-market-pair-mismatch/`, `orch-tatica-unitaria-volta-sinal/`), incluindo um cujo template de cabeçalho endureceu o formato acima. `keelson-fix`/`keelson-deploy` (as skills) nascem `draft-para-testar` — a maquinaria do modo-bug em si (aterrissagem reusada, escada de evidência, `log/`, ADR, `invariants.md`, gate humano) já não é mais desenho.
>
> **O Eixo 0 e o `tweak-<slug>.md` nascem de dado de campo, não de desenho especulativo.** Pelo menos 4 casos
> reais de "isso não é bug" apareceram no OptiFlux entre 2026-08-13 e 2026-08-16 (um layout de painel, ajustes
> aplicados à parte numa "sessão dedicada de melhorias de GUI", uma melhoria não-bloqueante registrada no
> cabeçalho de um fix-doc) — o suficiente para decidir onde a categoria mora, em vez de esperar mais.
> `keelson-tweak` nasce `draft-para-testar`, como toda skill nova neste método.

