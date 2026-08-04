---
schema_version: "0.17"
class: core-satellite
status: draft-pre-validacao
data: 2026-07-28
satellite_of: llm-dev-flow.md
---

# Método Keelson Flow — Manutenção (satélite)

> **Satélite de [`llm-dev-flow.md`](llm-dev-flow.md), aberto sob demanda — modo bug.** A maior parte da vida de um sistema é passada **vivo**, mantendo verdadeiro o que já roda; ali o trabalho dominante deixa de ser construir para a frente. Este satélite cobre a correção de bug — como ela encaixa no fluxo, **sem mecanismo novo**. Abra quando o trabalho for **consertar** (não construir para a frente). O frame forward (brief→plan→tasks, eixos, escada, guardrails) e o roteador vivem no núcleo [`llm-dev-flow.md`](llm-dev-flow.md).

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

É **autocontido**: não referencia um brief separado, porque o brief-equivalente é a própria reprodução. Colapsa `brief`+`plan`+`tasks` num doc pela mesma regra que faz "features pequenas colapsarem" — e, quando a correção é grande, **evolui para `tasks`** (`docs/fixes/<slug>/{fix-<slug>.md, tasks-<slug>.md}`, espelhando `specs/`), *just-in-time*.

Os dois eixos de estado, comprimidos:
- **`Doc Status`:** `DRAFT` → `VALIDATED` (o humano aprova *a abordagem de conserto* — gate humano, como todo `VALIDATED`) → `ARCHIVED`.
- **`Feature state` (a escada, no pequeno):** `NOT_BUILT` (reprodução *red*) → `BUILT` (fix *green*) → `PILOT` (raro — entrega gradual/*canary* da correção) → `PROD` (correção em produção).

**Destino terminal:** ao chegar em `PROD`, `fix-<slug>.md` (+ `tasks`) vai para `docs/archive/`, exatamente como `brief`/`plan`/`tasks` de uma feature — deixa de ser current-state e vira registro histórico do conserto.

### Proporcionalidade — nem todo bug vira documento

A espinha de sempre (rigor proporcional ao raio de explosão) vale aqui com força: **bug trivial não ganha `fix-<slug>.md`.** Um typo, um texto errado na GUI, um off-by-one óbvio = commit + uma linha de incidente no `log/` + o teste de regressão. O documento só se paga quando há **incerteza de causa-raiz**, **múltiplos consertos possíveis**, ou **raio alto** que exige o humano assinar a abordagem antes do código. Escrever um plano de correção para consertar um rótulo de botão é a mesma cerimônia-à-toa que a seção "o que não fazer" combate.

### Triagem — dois eixos ortogonais + depósitos

Quando um bug entra, duas perguntas independentes o classificam:

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
- **Alto raio → sessão independente** pergunta *"foi a causa-raiz ou o sintoma?"*, não *"o teste passou?"*.
- **Fix que muda comportamento corrige o `architecture.md` junto** — é o spec-rot ao contrário: manutenção é onde o as-built se re-acerta; um fix que deixa o `architecture.md` descrevendo o comportamento antigo cria uma mentira citável na camada as-built.

### O ponteiro do que está em tratamento — `known-issues.md`

Para o **agente que começa frio** não retrabalhar o que já se conhece — nem construir sobre comportamento bugado achando que é correto —, a memória mantém um **`wiki/known-issues.md`**: o ledger *grep-able* do que está **quebrado ou limitado agora e em tratamento** nos degraus vivos. É o **par transiente do `invariants.md`** (um diz o que é sempre verdade; o outro, o que está quebrado agora). **Puramente transiente:** a linha sai quando o problema resolve — por fix embarcado **ou** por decisão *won't-fix* —, e nada durável mora ali (a lição vai para ADR/`invariants.md`/`architecture.md`/`log/`). Não é um tracker: a lista completa + triagem é do issue tracker externo (**não duplicar o dono**); este é a projeção curada para o agente. Formato e regras: [`llm-dev-memory.md`](llm-dev-memory.md).

### Sessões de manutenção

Segue a regra geral: **uma sessão = uma transição de estado de um artefato.** Uma sessão de manutenção move o `fix-<slug>.md` por uma transição nomeável — reproduzir → achar a causa-raiz → `VALIDATED` a abordagem, ou um degrau da escada. A *entrega* humana no modo-fix é a **reprodução curada** (ver [`llm-dev-player.md`](llm-dev-player.md)).

> **Status: desenho, pré-validação.** Nenhum bug rodou por este caminho ainda. Ele se sustenta por **reusar** a aterrissagem, a escada de evidência, o `log/`, o ADR, o `invariants.md` e o gate humano — adicionando só dois artefatos (`docs/fixes/` + `wiki/known-issues.md`), do mesmo jeito que a `docs/domain/` entrou. Endurece com o campo, pela regra de sempre.

