---
class: skill
name: keelson-phase-landing
status: draft-para-testar
description: Executa a ATERRISSAGEM de uma fase do Método Keelson e escreve o tasks-fase<N>. Descobre feature/slug/fase no workspace, confirma com o usuário, reconcilia os requisitos do brief contra o código real (já-existe / colisão / lacuna-de-HOW) e abre o tasks pela tabela de rastreabilidade. Use quando o usuário iniciar/lançar uma fase, pedir a aterrissagem de uma feature, ou mandar criar o tasks-faseN. NÃO use para escrever brief nem plan — só a fronteira plan→tasks.
---

# Aterrissagem de fase

Implementa a **aterrissagem** descrita em `.keelson/llm-dev-flow.md` (§"Aterrissagem"): a passagem de uma
fase do `plan` para o `tasks-fase<N>`. **Aterrissar não é transcrever o brief** — é reconciliar, requisito a
requisito, o que o brief pede contra o código que já existe, *antes* de virar task. Você é um orquestrador
técnico rigoroso: descobre o contexto, **confirma com o usuário**, reconcilia e escreve o tasks. Não codifica.

## Escopo — o que esta skill faz e onde para

- Cobre **uma** fronteira: `plan` → `tasks-fase<N>`. Aterrissa **uma** fase e escreve o seu `tasks`.
- **Para na soleira:** não começa a codificar (isso é a transição seguinte, `tasks`→`BUILT`).
- Não escreve nem edita `brief` nem `plan`.

## Passo 1 — Auto-descoberta de contexto (antes de perguntar ou escrever nada)

Inspecione o workspace com suas ferramentas e tente derivar cada parâmetro. **Se algum não sair com
confiança, deixe em branco** para o usuário preencher no Passo 2 — não invente.

- **`<slug>` / `<feature>`:** procure `plan-*.md` sob `docs/specs/**/` (padrão `docs/specs/<slug>/plan-<slug>.md`).
  `<feature>` = título H1 do plan.
- **`<N>` (fase-alvo):** a **próxima** fase cujo gate da fase anterior já fechou e que ainda **não** tem
  `tasks-fase<N>-<slug>.md` — ou que tem um **incompleto**. Cruze três sinais: a lista de fases/gates no
  `plan`, os `tasks-fase*-<slug>.md` existentes (e seus checkboxes), e `wiki/now/<branch>.md`. Um `tasks` em
  progresso no `git status` costuma ser a fase.
- **§§ do brief desta fase:** no `plan` (que referencia o brief por §), identifique **quais § do brief a Fase
  `<N>` cobre**. É o recorte que a aterrissagem vai reconciliar.
- **Check de gate:** confirme que o **gate da Fase `<N-1>` fechou**. Se não fechou, **não aterrisse** —
  aterrissar cedo congela decisão sem chão de código (viola o gradiente de congelamento). Pare e avise.

## Passo 2 — Confirmação (gate humano; PARE aqui — obrigatório)

Terminada a varredura, **pare a execução** e apresente o que descobriu, exatamente neste formato:

> 🔍 **Phase Landing — descoberta**
> - **Feature:** {feature descoberta ou "?"}
> - **Slug:** {slug ou "?"}
> - **Fase-alvo:** Fase {N} — {objetivo/gate da fase, tirado do plan}
> - **§§ do brief que a fase cobre:** {lista ou "?"}
> - **Gate da Fase {N-1}:** {fechado / não fechado / não encontrei}
>
> Confirma? Corrija o que estiver errado antes de eu prosseguir.

**Regra dura:** não avance para o Passo 3 sem **confirmação explícita** ou correção do usuário. Se o gate da
Fase `<N-1>` não fechou, **não** prossiga mesmo com confirmação — levante isso primeiro.

## Passo 3 — Aterrissagem (só após a confirmação)

Agora você **tem** `feature`/`slug`/`N`/§§ confirmados — **use os valores confirmados**, não reabra placeholders.

1. **Leia o current-state:** `wiki/index.md` → `architecture.md`, `data-model.md`, e o **código real** dos
   módulos que os §§ confirmados tocam.
2. **Reconcilie requisito a requisito** (cada § confirmado) contra o que já existe. Classifique cada delta:
   - **já-existe** → reaproveitar/validar; a task vira *testar o que há*. **NÃO** construa caminho paralelo que
     duplique o dono.
   - **colisão** → **PARE e traga a colisão ao usuário** para decisão; registre a decisão no `tasks` + rascunho
     em `wiki/log/`. **Nunca** edite o brief em silêncio.
   - **lacuna-de-HOW** → o COMO se inventa **no `tasks`**, referenciando o § do brief, sem tocá-lo.
3. **Confronte as hipóteses do brief** cujo nível-dono é esta fase.

## Passo 4 — Escrever o `tasks-fase<N>`

- **Arquivo:** `docs/specs/<slug>/tasks-fase<N>-<slug>.md`. **Se já existir, atualize/continue** — preserve os
  checkboxes já marcados; não sobrescreva o que já foi feito.
- **Abra pela TABELA DE ATERRISSAGEM** (é a espinha de rastreabilidade — cada task → § do brief):

  | Requisito (brief §) | Já existe no código? | O que a task faz | Rastreabilidade (§) |
  | :--- | :--- | :--- | :--- |
  | | | | |

- **Feche o arquivo com a Superfície de incerteza:** "o que assumi / onde posso estar errado / o que não verifiquei".

## Handoff — onde você para

- **Pare aqui.** Não comece a codificar — é a próxima transição.
- **Colisões pendentes de decisão humana bloqueiam o fechamento:** liste-as no topo do handoff.
- (Opcional, se o harness suportar) sugira ao usuário renomear a sessão para `fase{N}-{slug}` — mas **nunca
  execute o rename sozinho**.

## Regras duras (não viole)

1. **Não edite o brief** (`VALIDATED`) — divergência vira ADR ou entrada no `wiki/log/`.
2. **Não edite nada em `.keelson/`** — é o método vendorizado, pinado.
3. **Não invente** fato, dado ou contexto. Se faltar, **pare e pergunte** ao usuário (antídoto ao
   "confiante-mas-errado").
4. **O gate humano do Passo 2 é obrigatório** — nunca pule para o Passo 3 sem confirmação.

## Nota de maturidade

Skill em `draft-para-testar`. A **aterrissagem** em si é padrão validado no `flow.md`; a **automação** dela
(descoberta + gate de confirmação) é pré-validação. Promova ou pode conforme o que o **rastro do projeto**
(`wiki/log/`, o `tasks-fase<N>`, ou um testemunho de fase) registrar sobre o que a skill pegou/deixou passar —
*instrumentar antes de formalizar*.
