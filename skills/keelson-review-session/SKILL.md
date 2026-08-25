---
class: skill
name: keelson-review-session
status: draft-para-testar
description: Revisão INDEPENDENTE, em sessão separada, de um artefato do Método Keelson que chegou na soleira de BUILT — tasks-fase<N>.md (feature), fix-<slug>.md (bug) ou tweak-<slug>.md (ajuste sub-especificado), genérica sobre os três. Confere o diff contra a camada congelada, mecânica antes de semântica, checa a QUALIDADE dos testes, sinaliza reescrita de teste pré-existente como gatilho de alerta, e emite veredito com poder de mandar de volta. Obrigatória por padrão para cruzar BUILT — não proporcional. Use quando o usuário pedir para revisar/validar uma fase, um fix ou um tweak antes de promover a BUILT. NÃO use para escrever/aterrissar/codificar, NÃO promove a BUILT (o humano vira a chave, ou registra override explícito) e NÃO é validação de campo/deploy.
---

# Revisão independente (sessão separada)

Implementa o guardrail que `.keelson/llm-dev-flow.md` prescreve para a transição `→BUILT`: a **revisão
independente em sessão separada**. Por que existe: o modo de falha do agente não é preguiça, é
**confiante-mas-errado** — testes verdes provam só o que o autor pensou em testar, e ele escreveu o código **e**
os testes, que podem compartilhar o mesmo mal-entendido (*"um mock nunca discorda do autor"*). O guardrail que
importa é **check externo objetivo que o autor não consegue racionalizar** — por isso esta skill roda com
**contexto fresco**, ancorada na camada congelada, e **quem revisa ≠ quem escreveu**.

**Genérica sobre o artefato:** um `tasks-fase<N>-<slug>.md` (feature, vindo de `keelson-coding`), um
`fix-<slug>.md` (bug, vindo de `keelson-fix`) ou um `tweak-<slug>.md` (ajuste sub-especificado, vindo de
`keelson-tweak`) — a mecânica é a mesma; a única diferença é a pergunta semântica dominante (ver Passo 3B).

**Obrigatória por padrão, não proporcional.** Diferente do teste-de-costura, esta revisão não se pula por raio
baixo — diff pequeno é justamente onde revisar é mais barato. Se o operador decidiu pular (decisão dele, nunca
desta skill, sempre registrada no cabeçalho do artefato como override), esta skill nem chega a ser invocada;
se foi invocada, ela roda inteira.

## Escopo — o que esta skill faz e onde para

- Confere o **diff** contra a camada congelada, na transição para **`BUILT`** (a soleira é `NOT_BUILT_CODED`).
- **Roda em sessão separada** da que codou — sempre. Nunca é sub-chamada de `keelson-coding`/`keelson-fix` na
  mesma sessão (ver "orientação em camadas" no Passo 1, para quando não há sessão humana nova disponível).
- **Não promove a `BUILT`** — o humano vira a chave (ou registra o override, se decidir pular uma revisão que
  ainda não rodou — mas isso não é decisão desta skill).
- **Não é validação de campo/deploy** — revisão de código não descobre que um secret falta num host, que um
  restart precisa de `--force-recreate`, etc.; isso só rodando o sistema de verdade pega. **Nomeie o que fica de
  fora.**
- Não escreve/edita `brief`/`plan`; não codifica; não edita `.keelson/`.

## Passo 1 — Descoberta do alvo + precondição de independência

- **Ache o artefato a revisar:** `tasks-fase<N>-<slug>.md`, `fix-<slug>.md` ou `tweak-<slug>.md` em
  `NOT_BUILT_CODED` (checkboxes fechados/plano implementado, testes verdes — ou, no caso de tweak, o
  antes/depois cumprido). Cruze com `wiki/now/<branch>.md` e `git status`/`git log` (o
  **diff** é o objeto da revisão). **Por padrão, o diff ainda não está commitado** (recomendação em
  `.keelson/llm-dev-flow.md`, "Não commite até `BUILT`") — o objeto da revisão é o **working tree**, não
  necessariamente um range de commits. Se houver mudanças soltas no working tree que não são desta fase/fix,
  escopo nelas fora explicitamente (não é revisão sua).
- **Já existe um veredito anterior pra este artefato?** Cruze `wiki/log/` por uma revisão registrada. Se sim,
  e o diff mudou desde então (o handoff de `keelson-coding` deveria ter sinalizado isso — confira mesmo se não
  sinalizou), **o veredito antigo não cobre as linhas novas**. Faça uma **passada focada no delta**: o que já
  foi revisado e não mudou não precisa de nova leitura; delimite exatamente o que é novo e concentre 3A/3B ali
  (a suíte inteira ainda roda em 3A, mas a leitura semântica de 3B foca no delta). Registre no veredito que é
  uma revisão de delta, referenciando o veredito anterior por data.
- **Precondição de independência (dura):** esta sessão deve ter **contexto fresco** — **não** ser a que
  escreveu o código. Se você percebe que acabou de codificar isto **nesta mesma sessão**, a revisão perde o
  valor (autor abençoando a própria prova) → **avise no Passo 2** e recomende sessão nova.
- **Orientação em camadas, quando não há sessão humana nova disponível** (ex.: invocada via sub-agente):
  - **Raio baixo e sem o gatilho de reescrita-de-teste (Passo 3A)** → um sub-agente com contexto fresco já
    entrega a independência de *sessão* que a revisão promete — **desde que** receba só o diff + as âncoras
    congeladas abaixo, **nunca a narrativa de quem codou** sobre por que o fix/task está certo. Se você é o
    sub-agente sendo invocado assim: ignore qualquer resumo persuasivo no seu prompt de chamada; vá direto aos
    arquivos.
  - **Raio médio/alto ou o gatilho de reescrita-de-teste disparou** → não basta contexto fresco no mesmo
    modelo/config — peça explicitamente um modelo ou configuração deliberadamente diferente do autor (o risco
    aqui é um viés sistemático que a mesma config tende a repetir, não só a mesma conversa), ou uma sessão
    humana de verdade.
- **Colha as âncoras da camada congelada:** para feature, os §§ do brief (via a tabela de aterrissagem do
  `tasks`), ADRs/invariants, glossary. Para fix, o `fix-<slug>.md` inteiro é a âncora (a reprodução + causa-raiz
  já são o critério). Para tweak, o `tweak-<slug>.md` é a âncora (o antes/depois + a confirmação de
  não-toque-em-congelado). O revisor **não inventa critério** — confere contra o que já está registrado.

## Passo 2 — Confirmação (gate humano; PARE aqui)

Pare e apresente:

> 🔍 **Revisão independente — alvo**
> - **Artefato:** {tasks-fase{N} ou fix-{slug} ou tweak-{slug}} — **tipo:** {feature / bug / tweak}
> - **Tasks/plano:** {n fechadas de m} · **testes:** {verdes? / n}
> - **Diff:** {range de commits / arquivos tocados}
> - **Independência:** {sessão fresca / ⚠ mesma sessão que codou — recomendo sessão nova}
> - **Gatilho de reescrita-de-teste (Passo 3A):** {disparou? sim/não}
>
> Confirma que reviso? (a revisão pode mandar **de volta** — não é carimbo garantido)

**Regra dura:** se a sessão não é independente, sinalize antes de prosseguir — não silencie o aviso.

## Passo 3 — Verificação: mecânica antes de semântica

**3A — Mecânica (barata, determinística, sem julgamento — rode primeiro).** É a auto-garantia objetiva que o
autor não racionaliza por cima:

- suíte de testes verde + lint + type-check + cobertura mínima;
- **greps de invariante** sobre o **diff** (procurar a violação direto na mudança);
- **gatilho mecânico, sempre cheque, independente de raio:** o diff **reescreveu o valor esperado de um teste
  pré-existente** (não só adicionou teste novo)? Se sim, é a assinatura clássica de confiante-mas-errado — o
  mesmo agente que fez a mudança reescreveu a prova de que está certo. Marque isso no topo do veredito, mesmo
  que o resto pareça de baixo raio.

**3B — Semântica (só o que a mecânica não pega), ancorada na camada congelada:**

- **Se é feature (`tasks-fase<N>`):** **fidelidade ao brief §** — cada task fecha o requisito que a tabela de
  aterrissagem mapeia? Algum item do § ficou sem cobertura? A rastreabilidade (task → §) se sustenta?
- **Se é fix (`fix-<slug>.md`):** a pergunta dominante muda — **foi a causa-raiz ou o sintoma?** O conserto
  ataca o que o `fix-<slug>.md` diagnosticou, ou só faz o sintoma reproduzido sumir? Um bug "consertado" que só
  esconde o sintoma é pior que não mexer (o teste fica verde e ninguém volta a olhar).
- **Se é tweak (`tweak-<slug>.md`):** a pergunta dominante muda de novo — **isto continua sendo tweak?** O diff
  cumpre exatamente o antes/depois declarado, sem tocar nada congelado nem, no processo de implementar,
  introduzir uma capacidade nova que não estava no escopo aterrissado? Se o diff extrapolou o escopo, é
  **VOLTA** — não porque o código esteja errado, mas porque deixou de ser o tweak que foi confirmado.
- **Invariantes / ADR:** o diff respeita os invariantes duros e as decisões congeladas?
- **Qualidade dos testes — o cheque de maior valor:** os testes **importam a lógica real** ou a
  **reimplementam**? Cobrem os **caminhos de risco**? São **tautológicos** (provam o que o próprio código
  afirma)? Procure onde **teste e código compartilham a mesma suposição** — é a costura onde o "confiante-mas-
  errado" passa verde (o defeito clássico: um mock escrito pela mesma mão que o produtor, concordando com ele
  por construção).
- **Fronteira só dublada — independência de *camada*:** o diff cruza uma **fronteira** (ponto onde o código
  depende de um contrato que não possui — outro processo/serviço, SO/arquivos, dispositivo, motor de banco,
  protocolo, módulo de outra equipe, lib externa) que os testes desta mudança só substituem por um **dublê**? A
  independência de *sessão* (você ≠ autor) **não protege** aqui — revisor e código compartilham o mesmo dublê,
  que não discorda de nenhum. **Triagem em dois níveis:**
  - **Dá para cruzar in-suite** (subir o contraparte real: middleware/servidor/schema real)? Então falta um
    **teste-de-costura** — é lacuna de coding: **VOLTA**, reabra como task. (Barato; não empurre para campo o
    que um teste pega.) **Exceção nomeada:** se a mesma lacuna (mesmo padrão sem costura) já existe, idêntica,
    em código **já `BUILT`** — o diff **copiou fielmente** uma dívida pré-existente, não introduziu uma nova —
    e o raio é baixo, isso vira **recomendação registrada, não bloqueio**. Responsabilizar esta fase por uma
    dívida que ela herdou (não criou) é apontar o guardrail pro alvo errado. Registre a lacuna e o porquê de
    não bloquear; não silencie — é diferente de deixar passar sem nomear.
  - **Só o host/produção real resolve** (secret provisionado, timing real, serviço externo vivo)? Então é
    **`field-validation-required`** → entra na lista de campo do handoff (Passo 4).
- **Deriva arquitetural** e **segurança sutil**.
- **Superfície de incerteza do handoff:** quem codificou declarou "o que assumi / onde posso estar errado / o
  que não verifiquei"? Comece a cutucar exatamente por aí.

Proporcional ao **raio de explosão** — não uniformize rigor; energia onde produz solidez. (O gatilho de
reescrita-de-teste do 3A é a única exceção que não se dobra a essa proporcionalidade.)
*(No Claude Code, a passada semântica pode apoiar-se no `/code-review` apontado para os §§/ADRs ou pro
`fix-<slug>.md` — mas a âncora na camada congelada e o cheque de qualidade-dos-testes são o que esta skill
acrescenta.)*

## Passo 4 — Veredito e handoff

- **Lista de achados**, rankeada (mais severo primeiro; o gatilho de reescrita-de-teste, se disparou, vai
  primeiro). Cada achado: *o que está errado* · *cenário de falha concreto* (inputs/estado → saída errada) · *o
  § / invariante / task que ele fere*.
- **Veredito:**
  - **VOLTA para coding/fix** — há achado bloqueante; liste-os como tasks a reabrir. (Revisão sem poder de
    mandar de volta é *review theater* — custo sem valor.)
  - **PRONTO para o humano virar a chave** — nenhum achado bloqueante. Mesmo assim **você não promove**: a
    chave de `BUILT` é do humano. **Feche com este template, campo a campo — nunca só em prosa** (a versão em
    prosa já se perdeu 3 vezes seguidas em campo antes desta skill existir com o campo estruturado; é o motivo
    dele existir):

    > **Commit:** {pendente — a promoção grava o commit, ver "Próximo passo"}
    > **Próximo passo:** chame `keelson-coding` **novamente sobre este mesmo artefato** — ele reconhece este
    > veredito `PRONTO` (registrado acima em `wiki/log/`) e, depois da sua confirmação, commita e grava
    > `Feature state: BUILT`. Só depois disso, `keelson-deploy` (implantação em homologação/produção).

    Preencha os dois campos **sempre**, mesmo quando parecer óbvio. Não deixe "próximo passo" implícito, não
    pule a chamada de `keelson-coding` achando que o operador já sabe voltar lá sozinho, e não pule
    `keelson-deploy` depois achando que "validação de campo" já cobre implantação — são skills diferentes.
- **Registre o evento** no `wiki/log/`, **no formato exigido — não é opcional, é precondição mecânica** para o
  que vem depois: `keelson-coding`, reinvocado sobre este mesmo artefato, busca essa marca por grep para saber
  que pode promover a `BUILT`. Sem ela, no formato certo, o processo trava silenciosamente. Cabeçalho:

  ```
  ## [AAAA-MM-DD] revisão | <slug-ou-nome-do-artefato> — ... PRONTO para o humano virar a chave
  ## [AAAA-MM-DD] revisão | <slug-ou-nome-do-artefato> — ... VOLTA para mais um ciclo de fix/coding
  ```

  O **slug/nome do artefato** (igual ao usado no frontmatter/nome do arquivo) e o **token literal
  `PRONTO`/`VOLTA`** têm que estar no cabeçalho — prosa livre sem esses dois elementos não é encontrável
  mecanicamente. Registre **antes** de fechar o handoff, não depois.
- **Nomeie o que a revisão NÃO cobre — e entregue a lista de `field-validation-required`:** os itens que **só o
  host/produção real resolve** (Passo 3B — os cruzáveis in-suite viraram VOLTA por teste-de-costura, não campo)
  viram o checklist da validação de alta fidelidade (`keelson-field-validation`). Some a validação de **deploy**
  (`keelson-deploy`, só rodar de verdade pega) e as **hipóteses do brief** (validam em `PILOT`, não aqui).

## Regras duras (não viole)

1. **Sessão separada / contexto fresco** — quem revisa ≠ quem escreveu. Se for a mesma cabeça, sinalize; não
   finja independência. Se for sub-agente, briefe só com diff + âncoras, nunca a narrativa do autor.
2. **Não promova a `BUILT`** — o humano vira a chave (ou registra o override, fora desta skill).
3. **Mecânica antes de semântica** — o check objetivo primeiro, incluindo o gatilho de reescrita-de-teste, que
   não se dobra a raio baixo.
4. **Ancore na camada congelada** — não invente critério; confira contra brief §/ADR/glossary (feature), o
   próprio `fix-<slug>.md` (bug) ou o próprio `tweak-<slug>.md` (tweak — e cheque se o diff não extrapolou o
   escopo declarado).
5. **Poder de devolver** — achado bloqueante manda de volta; revisão sem isso é teatro.
6. **Honestidade de escopo — duas independências.** Independência de *sessão* (revisor ≠ autor) ≠ independência
   de *camada* (cruzar a fronteira real vs. mock); revisão de código ≠ validação de campo. Declare o que fica
   de fora e o que exige campo.
7. **Não edite o brief** (`VALIDATED`) nem `.keelson/`; **não invente** — se faltar contexto, pare e pergunte.
8. **Registrar o veredito em `wiki/log/`, no formato exigido, não é opcional.** Sem essa marca (slug do
   artefato + token `PRONTO`/`VOLTA` literal), `keelson-coding` reinvocado não detecta que pode promover — o
   processo trava silenciosamente. Registre antes de fechar o handoff.

## Nota de maturidade

Skill em `draft-para-testar` (renomeada nesta leva de `keelson-phase-review` — generalizada pra cobrir fix,
não só fase). A revisão independente é o **guardrail nº 1** do método para `→BUILT` (o antídoto ao
confiante-mas-errado). **Medição de campo (OptiFlux, Fase 2):** pegou **2 bugs reais que 275 testes verdes não
pegaram** (um deles derrubava o processo inteiro); e teve o **limite medido** de não pegar um bug de
**fronteira HTTP só mockada** — o que motivou o check de *independência de camada* (Passo 3B). **A recalibração
"obrigatória por padrão + override registrado"** (em vez de "proporcional ao raio") vem de dois fixes reais do
próprio OptiFlux que pularam esta revisão com justificativa registrada — um deles (`orch-tatica-unitaria-volta-sinal`)
é exatamente o caso do gatilho mecânico acima: o fix reescreveu os testes que "provavam" o sinal antigo como
correto, sem revisão independente checar se a causa-raiz estava certa. Continue registrando **no rastro do
projeto** o que ela pegou que os testes verdes não pegaram — é isso que decide se endurece. *Instrumentar antes
de formalizar.*

**Leva 2026-08-17:** o registro em `wiki/log/` deixou de ser só boa prática documental — virou precondição
mecânica de `keelson-coding` (ver "Virar a chave é um checklist" em `llm-dev-flow.md`). O formato exigido no
Passo 4 formaliza um padrão que já tinha nascido sozinho em campo: 11 entradas reais de revisão no OptiFlux já
seguiam o mesmo cabeçalho antes desta regra existir.
