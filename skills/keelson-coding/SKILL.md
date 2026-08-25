---
class: skill
name: keelson-coding
status: draft-para-testar
description: Implementa (coding) um artefato já aterrissado do Método Keelson — tasks-fase<N>.md (feature), fix-<slug>.md (bug) ou tweak-<slug>.md (ajuste sub-especificado), genérica sobre os três — levando de NOT_BUILT até a soleira NOT_BUILT_CODED de BUILT. Descobre o alvo, confirma, executa task a task (red-first proporcional pra feature, piso obrigatório pra fix, sem red pra tweak; verificação mecânica antes da semântica; gatilho de reescrita-de-teste sempre checado) e para no handoff sem virar a chave de BUILT. Se reinvocada sobre um artefato já em NOT_BUILT_CODED com veredito de keelson-review-session registrado em wiki/log/, muda de modo: detecta o veredito, confirma com o humano, e só então commita e grava Feature state: BUILT. Use quando o usuário mandar codificar/implementar uma fase, um fix ou um tweak já aterrissado, ou pedir para finalizar/promover depois de uma revisão. NÃO use para aterrissar (isso é keelson-phase-landing, keelson-fix ou keelson-tweak) nem para promover PILOT/PROD (isso é keelson-field-validation).
---

# Coding

Implementa um artefato **já aterrissado** — a transição `NOT_BUILT` → **soleira** `NOT_BUILT_CODED` de `BUILT`,
descrita em `.keelson/llm-dev-flow.md` ("Sub-degraus", "Sessões", "Guardrails"). **Genérica sobre o artefato:**
`tasks-fase<N>-<slug>.md` (vindo de `keelson-phase-landing`), `fix-<slug>.md` (vindo de `keelson-fix`) ou
`tweak-<slug>.md` (vindo de `keelson-tweak`) — a disciplina é a mesma, com uma diferença no rigor do
*red*-first (ver Passo 3). **Precondição:** o artefato já foi aterrissado (tabela de aterrissagem preenchida,
causa-raiz+reprodução no caso de fix, ou escopo+antes/depois no caso de tweak). Se não existe, **isto não é
coding ainda** — rode `keelson-phase-landing`/`keelson-fix`/`keelson-tweak` primeiro. Você escreve código. Se
reinvocada sobre um artefato que já cruzou a soleira com um veredito `PRONTO` registrado (ver Passo 1), a
sessão muda de papel: não codifica mais, **executa a promoção** — sempre mediante confirmação humana explícita
(Passo 2b). Fora esse caso, você não vira a chave de `BUILT` sozinha.

## Escopo — o que esta skill faz e onde para

- Cobre **uma** fronteira: `NOT_BUILT` → **soleira** `NOT_BUILT_CODED` de `BUILT` (código escrito, testes
  verdes).
- **Não aterrissa** (`keelson-phase-landing`/`keelson-fix`). **Não promove `BUILT` sozinha** — só depois de
  detectar um veredito `PRONTO` registrado em `wiki/log/` (Passo 1) e de confirmação humana explícita (Passo
  2b). Nunca promove `PILOT`/`PROD` (isso é `keelson-field-validation`).
- **Bug fora do próprio escopo não se conserta inline.** Se, codificando, você tropeçar num bug que não é o que
  esta task descreve (de outra fase, ou achado incidental), **pare** e aponte pra `keelson-fix` — não improvise
  o conserto dentro desta sessão.
- Não escreve nem edita `brief` nem `plan`.

## Passo 1 — Auto-descoberta do alvo (antes de escrever código)

- **Alvo:** `tasks-fase<N>-<slug>.md`, `fix-<slug>.md` **ou** `tweak-<slug>.md`, aterrissado. Cruze três
  sinais: os artefatos em `docs/specs/<slug>/`, `docs/fixes/` ou `docs/tweaks/` (fix/tweak normalmente arquivo
  único; e o `Feature state` no frontmatter), `wiki/now/<branch>.md`, e o `git status`. O `Feature state`
  encontrado decide o modo desta sessão:
  - **Ainda não `NOT_BUILT_CODED`** (checkboxes/plano com itens **abertos**) → modo **coding**, segue abaixo
    normalmente.
  - **Já `NOT_BUILT_CODED`** (soleira cruzada, itens fechados) → modo **promoção**, não coding. Veja o bullet
    "Detecção de promoção" logo abaixo — **não** volte a escrever código sem antes checar essa branch.
- **Precondição de aterrissagem:** confirme que a tabela de aterrissagem (feature), a causa-raiz+reprodução
  (fix) ou o escopo+antes/depois (tweak) já estão escritos. Se não (ou o arquivo não existe), **pare** e avise:
  precisa aterrissar antes (`keelson-phase-landing`/`keelson-fix`/`keelson-tweak`).
- **Colisões pendentes:** varra por **colisões/decisões não resolvidas** que ficaram para o usuário. Se houver,
  elas **bloqueiam** — traga-as no Passo 2.
- **Detecção de promoção (só quando o alvo já está em `NOT_BUILT_CODED`):** esta invocação não é coding, é
  **promoção**. Busque em `wiki/log/` uma entrada `revisão | <slug-do-artefato> ...` (o formato exigido de
  `keelson-review-session`, com o token literal `PRONTO`/`VOLTA`) e confira se o diff do working tree mudou
  desde a data daquela entrada (mesmo grep de frescor que `keelson-review-session` já faz no próprio Passo 1).
  - **`PRONTO`, diff inalterado** → vá direto para o **Passo 2b** (pule o Passo 2 de coding).
  - **`VOLTA`** → os achados listados viram tasks a reabrir; siga no modo coding normal (Passo 2 → 3) para
    endereçá-los.
  - **`PRONTO`, mas o diff mudou desde então** → o veredito não cobre o delta novo
    (`.keelson/llm-dev-flow.md`, "Um veredito não estica sozinho"); avise no Passo 2b e recomende uma nova
    passada de `keelson-review-session` focada no delta, em vez de promover por cima de código que a revisão
    não viu.
  - **Nenhuma entrada encontrada, mas o operador afirma que a revisão aconteceu** (ex.: sessão externa, não
    documentada aqui) → sem rastro auditável, isto é **override**
    (`.keelson/llm-dev-flow.md`, "override explícito, nunca silencioso"). Peça ao operador a justificativa
    curta que vai inline no `Feature state: BUILT` — leve isso para o Passo 2b.

Se algum parâmetro não sair com confiança, deixe em branco para o Passo 2 — não invente.

**Se o Passo 1 identificou modo promoção, pule este Passo 2 e vá direto para o Passo 2b, mais abaixo.**

## Passo 2 — Confirmação (gate humano; PARE aqui)

Pare e apresente:

> 🔨 **Coding — alvo**
> - **Artefato:** {tasks-fase{N}-{slug} ou fix-{slug} ou tweak-{slug}} — **tipo:** {feature / bug / tweak}
> - **Itens abertos:** {n de m}
> - **Colisões pendentes:** {nenhuma / lista}
>
> Confirma que começo a implementar? Se houver colisão pendente, decida antes — não implemento por cima dela.

**Regra dura:** não comece a codificar sem confirmação. Colisão pendente **bloqueia**: resolva (a decisão vira
registro no artefato + `wiki/log/`) antes do Passo 3.

## Passo 3 — Implementação (após confirmação)

Trabalhe **item a item**, marcando conforme fecha. Respeite a classificação já feita na aterrissagem
(já-existe / lacuna-de-HOW, para feature; a causa-raiz, para fix).

Verificação, na ordem:

1. **Red-first.** **Feature:** proporcional ao raio de explosão — escreva o teste a partir do critério de
   aceite, **confirme que falha**, e só então implemente; não em toda task trivial. **Fix:** **piso sempre**,
   não proporcional — a reprodução já veio de graça na aterrissagem corretiva, então o custo do *red* é quase
   zero e sempre se justifica (ver `.keelson/llm-dev-flow-maintenance.md`, "O guardrail do fix"). **Tweak:**
   **sem *red*** — nada quebrou, então não há um estado "errado" pra provar antes; o critério de aceite é o
   antes/depois escrito na landing.
2. **Mecânica antes de semântica:** ao fim de cada cluster, rode testes + lint/types + **greps de invariante**
   — barato e determinístico — antes de qualquer avaliação por leitura.
3. **Teste-de-costura por fronteira.** Uma **fronteira** = ponto onde o código depende de um **contrato que não
   possui** (outro processo/serviço, SO/arquivos, dispositivo, motor de banco, protocolo, módulo de outra
   equipe, lib externa) e que os testes costumam substituir por um **dublê** (mock/stub/fixture/simulador). Para
   **cada fronteira que sustenta peso**, escreva **um** teste que exercita o **contraparte real** in-suite — não
   só o dublê. Um dublê escrito pela mesma mão concorda consigo por construção; só o contraparte real discorda
   (é o que pega o bug de costura **antes** da produção). *(Ex.: motor de banco real, servidor/middleware real,
   terminal/protocolo real de trading, sistema de arquivos real, dispositivo real — a lista muda por sistema.)*
   O que genuinamente **não** roda in-suite (host/produção/dispositivo/mercado vivo) você **não** finge testar:
   declara na Superfície de incerteza como `field-validation-required`.
4. **Gatilho mecânico, sempre cheque, independente de raio (mais crítico em fix):** você **reescreveu o valor
   esperado de um teste pré-existente** (não só adicionou teste novo)? Se sim, **marque isso explicitamente no
   handoff, em destaque** — é a assinatura clássica de confiante-mas-errado, e eleva a recomendação de revisão
   mesmo que o resto pareça de baixo raio.

**Se surgir um delta novo** (colisão, suposição que a aterrissagem não previu, ou **um bug fora do escopo desta
task**): **PARE e traga ao usuário** — o primeiro pode virar ADR ou exigir revisar a aterrissagem; o segundo
aponta pra `keelson-fix`. **Não** improvise nem edite o brief para acomodar.

## Handoff — a soleira de BUILT (fim do modo coding)

- Leve o artefato até a **soleira**: código escrito, **testes verdes**, itens fechados. **Pare aí.**
- **Recomendação: deixe o diff sem commit.** A revisão (`keelson-review-session`) roda contra o **working
  tree**, não contra um commit fechado — assim, se sair `VOLTA`, o conserto continua no mesmo diff, sem commit
  "ruim" pra reverter. Exceção legítima: fase grande, várias sessões — commits de checkpoint pra continuidade
  são aceitáveis, mas o estado que chega na revisão deve ser um diff revisável (ver `.keelson/llm-dev-flow.md`,
  "Recomendação: não commite até `BUILT`").
- **Escreva `Feature state: NOT_BUILT_CODED` no frontmatter do artefato.** Não basta descrever "chegamos na
  soleira" em prosa no handoff — o campo precisa refletir o estado de verdade, porque é dali que a próxima
  skill (e qualquer agente futuro lendo frio) deriva onde retomar. Descrever sem gravar é o mesmo erro que não
  ter feito o trabalho.
- **Se esta sessão editou um diff que já tinha um veredito de `keelson-review-session`** (ex.: endereçando um
  achado não-bloqueante que a própria revisão recomendou), **sinalize isso explicitamente no handoff**: "este
  diff mudou depois do veredito de {data}; as linhas novas não foram cobertas por aquele veredito." O veredito
  antigo não estica sozinho pra cobrir mudança nova — nem quando a mudança é exatamente o que a revisão pediu
  (ver `.keelson/llm-dev-flow.md`, "Um veredito não estica sozinho").
- **Não vire a chave de `BUILT` nesta sessão** — a promoção acontece numa invocação **futura**, depois de
  `keelson-review-session` rodar (sempre sessão separada), e só mediante a detecção + confirmação do Passo 2b.
- **Aponte os próximos passos, nesta ordem, sem pular nenhum mesmo que pareça óbvio:** `keelson-review-session`
  (revisão, sempre sessão separada) → chame `keelson-coding` **de novo** sobre este artefato para promover
  (Passo 2b/5) → `keelson-deploy` (implantação em homologação/produção) → `keelson-field-validation`
  (validação de campo, produz `PILOT`/`PROD`). Omitir um desses passos da lista já aconteceu em campo — não é
  opcional, mesmo quando parece implícito.
- **Se o gatilho de reescrita-de-teste do Passo 3 disparou, isso vai no topo do handoff.**
- Feche com a **Superfície de incerteza**: "o que assumi / onde posso estar errado / o que não verifiquei" —
  mais o que **não** foi coberto (itens que ficaram abertos e por quê).

## Passo 2b — Confirmação de promoção (gate humano; PARE aqui)

Só chega aqui quem o Passo 1 identificou como modo **promoção** (artefato já `NOT_BUILT_CODED`, com veredito
encontrado ou alegado). Pare e apresente:

> 🔑 **Coding — promoção a BUILT**
> - **Artefato:** {tasks-fase{N}-{slug} ou fix-{slug} ou tweak-{slug}}
> - **Veredito:** {PRONTO, `wiki/log/{data}` / sem rastro — override do operador}
> - **Diff:** {inalterado desde o veredito / arquivos tocados}
> - **Justificativa de override (se aplicável):** {texto do operador}
>
> Confirma que eu commito este diff e gravo `Feature state: BUILT`?

**Regra dura:** não commite nem grave `BUILT` sem esta confirmação explícita — mesmo com `PRONTO` óbvio e
diff inalterado. A decisão de virar a chave continua sendo do humano; esta skill só executa o gesto depois de
confirmado (`.keelson/llm-dev-flow.md`, "Virar a chave é um checklist, não uma metáfora"). Se o Passo 1 achou
"sem rastro auditável", **não prossiga sem a justificativa do operador** — ela é obrigatória, não opcional,
para preencher o campo acima.

## Passo 5 — Executar a promoção (após confirmação do Passo 2b)

1. **Commit escopado e limpo** do diff revisado — confira que o que vai ser commitado bate **exatamente** com
   o que a revisão já verificou (o mesmo "`git diff --stat` bate 1:1" que `keelson-review-session` já confere
   no próprio Passo 1). O working tree pode ter mudanças soltas de outra sessão/artefato **já staged** — não
   são suas; exclua-as explicitamente do commit, mesmo que já estivessem staged de antemão (`git reset --soft`
   antes de recommitar, se precisar desfazer um `add` largo demais). Mensagem representa o trabalho promovido,
   não o ruído do ciclo de revisão (`.keelson/llm-dev-flow.md`, "Recomendação: não commite até `BUILT`").
2. **Grave `Feature state: BUILT`** no frontmatter do artefato — com a justificativa inline se for promoção por
   override ("revisão sem rastro auditável aqui, aceito pelo operador em {data}, motivo: X").
3. **Aponte o próximo passo:** `keelson-deploy` (implantação em homologação/produção).

## Regras duras (não viole)

1. **Não edite o brief** (`VALIDATED`) — divergência vira ADR ou entrada no `wiki/log/`.
2. **Não edite nada em `.keelson/`** — método vendorizado, pinado.
3. **Não invente** fato/dado/contexto — se faltar, pare e pergunte.
4. **Só promove a `BUILT` depois de confirmação humana explícita** (Passo 2b) — nunca por conta própria, mesmo
   com `PRONTO` óbvio e diff inalterado; nunca promove `PILOT`/`PROD`.
5. **Não re-aterrisse por conta própria** — delta novo é sinal para o usuário, não para improviso.
6. **Bug fora de escopo não se conserta inline** — aponte pra `keelson-fix`, não improvise.
7. **Grave `Feature state: NOT_BUILT_CODED` no frontmatter, sempre** — não é opcional nem implícito; e liste
   `keelson-deploy` explicitamente entre revisão e campo nos próximos passos.
8. **Sem rastro de revisão em `wiki/log/` e sem justificativa de override do operador, não promove** — pare e
   peça a justificativa antes de preencher o Passo 2b.
9. **Escopo do commit é só os arquivos do artefato** — nunca arraste mudanças soltas de outra sessão/artefato
   pro commit de promoção, mesmo que já estejam staged.

## Nota de maturidade

Skill em `draft-para-testar` (renomeada nesta leva de `keelson-phase-coding` — generalizada pra cobrir fix, não
só fase). A disciplina de coding (task a task, red-first, mecânica-antes-de-semântica, handoff na soleira) é
padrão do `flow.md`; a **automação** (descoberta + gate) é pré-validação. **Já pegou um erro de campo na
primeira sessão real sob o desenho novo** (OptiFlux, Fase 4): o handoff descreveu a soleira em prosa mas não
gravou `Feature state: NOT_BUILT_CODED` no frontmatter, e a lista de próximos passos omitiu `keelson-deploy`
— os dois pontos acima (regra dura 7, e a lista explícita no Handoff) nasceram dessa correção. Promova ou pode
conforme o que o **rastro do projeto** (`wiki/log/`, o `tasks-fase<N>`/`fix-<slug>.md`, ou um testemunho de
fase) registrar sobre o que a skill pegou/deixou passar — *instrumentar antes de formalizar*.

**Leva 2026-08-17:** o Passo 2b/5 (detecção de promoção, confirmação, execução) nasceu de uma observação de
uso intenso em campo — o operador já fazia exatamente isso na mão (voltava à sessão de coding depois da
revisão e pedia a promoção explicitamente), mas dependia de tribal knowledge; formaliza o que já funcionava,
sem automatizar o ponto de decisão (`.keelson/llm-dev-flow.md`, "Virar a chave é um checklist, não uma
metáfora").

**Primeiro uso real (OptiFlux, `tweak-erro-aprovacao-tatica-causa-e-legibilidade`, 2026-08-18):** o núcleo do
mecanismo funcionou — detectou o `PRONTO` em `wiki/log/`, confirmou, promoveu, gravou `Feature state: BUILT`
referenciando o veredito, apontou os próximos passos certos. Mas achou um buraco real no Passo 5: nada
instruía a checar o **escopo** do commit contra o que a revisão de fato revisou, e o primeiro commit arrastou
~20 arquivos alheios que já estavam staged de outra sessão — a skill se auto-corrigiu com `git reset --soft`
antes de recommitar certo, mas não deveria depender disso. Regra dura 9 e o item 1 do Passo 5 (checagem de
escopo, espelhando o "`git diff --stat` bate 1:1" que `keelson-review-session` já fazia) nasceram dessa
correção — mesmo padrão da correção anterior (regra dura 7, campo anterior).
