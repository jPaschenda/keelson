---
class: skill
name: keelson-phase-coding
status: draft-para-testar
description: Implementa (coding) uma fase já aterrissada do Método Keelson — leva o tasks-fase<N> de NOT_BUILT até a soleira de BUILT. Descobre a fase-alvo, confirma, executa task a task (red-first proporcional; verificação mecânica antes da semântica) e para no handoff sem virar a chave de BUILT. Use quando o usuário mandar codificar/implementar uma fase cujo tasks-fase<N> já existe. NÃO use para aterrissar (isso é keelson-phase-landing) nem para promover a BUILT/PILOT/PROD.
---

# Coding de fase

Implementa uma fase **já aterrissada** — a transição `tasks-fase<N>` `NOT_BUILT` → **soleira** de `BUILT`,
descrita em `.keelson/llm-dev-flow.md` (§"Sessões", §"Guardrails"). **Precondição:** a fase já foi aterrissada
(existe um `tasks-fase<N>-<slug>.md` com a tabela de aterrissagem preenchida). Se não existe, **isto não é
coding ainda** — rode `keelson-phase-landing` primeiro. Você escreve código; **não** vira a chave de `BUILT`
(revisão + deploy + ADR são do usuário).

## Escopo — o que esta skill faz e onde para

- Cobre **uma** fronteira: `tasks-fase<N>` `NOT_BUILT` → **soleira** de `BUILT` (código escrito, testes verdes).
- **Não aterrissa** (isso é `keelson-phase-landing`) e **não promove** a `BUILT`/`PILOT`/`PROD` (o usuário vira a chave).
- Não escreve nem edita `brief` nem `plan`.

## Passo 1 — Auto-descoberta da fase-alvo (antes de escrever código)

- **`<slug>` / `<feature>` / `<N>`:** ache o `tasks-fase<N>-<slug>.md` **aterrissado e ainda não `BUILT`** — o
  que tem checkboxes **abertos**. Cruze três sinais: os `tasks-fase*-<slug>.md` em `docs/specs/<slug>/` (e o
  `Feature state` no frontmatter), `wiki/now/<branch>.md`, e o `git status`.
- **Precondição de aterrissagem:** confirme que o `tasks` tem a **tabela de aterrissagem** preenchida. Se não
  tem (ou o arquivo não existe), **pare** e avise: a fase precisa ser aterrissada antes (`keelson-phase-landing`).
- **Colisões pendentes:** varra o `tasks` e a tabela por **colisões não resolvidas** (decisões que ficaram para
  o usuário). Se houver, elas **bloqueiam** — traga-as no Passo 2.

Se algum parâmetro não sair com confiança, deixe em branco para o Passo 2 — não invente.

## Passo 2 — Confirmação (gate humano; PARE aqui)

Pare e apresente:

> 🔨 **Phase Coding — alvo**
> - **Feature / Fase:** {feature} — Fase {N}
> - **Arquivo:** docs/specs/{slug}/tasks-fase{N}-{slug}.md
> - **Tasks abertas:** {n de m}
> - **Colisões pendentes:** {nenhuma / lista}
>
> Confirma que começo a implementar? Se houver colisão pendente, decida antes — não implemento por cima dela.

**Regra dura:** não comece a codificar sem confirmação. Colisão pendente **bloqueia**: resolva (a decisão vira
registro no `tasks` + `wiki/log/`) antes do Passo 3.

## Passo 3 — Implementação (após confirmação)

Trabalhe **task a task**, marcando os checkboxes conforme fecha. Respeite a **classificação da tabela de
aterrissagem**:

- task **já-existe** → *validar/testar o que há*; **não** reimplemente nem construa caminho paralelo que
  duplique o dono.
- task **lacuna-de-HOW** → implemente o COMO definido no `tasks`, ancorado no § do brief.

Verificação, na ordem (**proporcional ao raio de explosão** — não em toda task trivial):

1. **Red-first** para tasks de alto raio: escreva o teste a partir do critério de aceite, **confirme que
   falha**, e só então implemente. (Mata o teste tautológico/falso-positivo.)
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

**Se surgir um delta novo** (colisão ou suposição que a aterrissagem não previu): **PARE e traga ao usuário** —
pode virar ADR ou exigir revisar a aterrissagem. **Não** improvise nem edite o brief para acomodar.

## Handoff — a soleira de BUILT

- Leve o `tasks-fase<N>` até a **soleira**: código escrito, **testes verdes**, checkboxes fechados. **Pare aí.**
- **Não vire a chave de `BUILT`** — a promoção formal (revisão independente em sessão separada + deploy + ADR)
  é do usuário.
- Feche com a **Superfície de incerteza**: "o que assumi / onde posso estar errado / o que não verifiquei" —
  mais o que **não** foi coberto (tasks que ficaram abertas e por quê).

## Regras duras (não viole)

1. **Não edite o brief** (`VALIDATED`) — divergência vira ADR ou entrada no `wiki/log/`.
2. **Não edite nada em `.keelson/`** — método vendorizado, pinado.
3. **Não invente** fato/dado/contexto — se faltar, pare e pergunte.
4. **Não vire a chave de `BUILT`** sozinho; não promova a `PILOT`/`PROD`.
5. **Não re-aterrisse por conta própria** — delta novo é sinal para o usuário, não para improviso.

## Nota de maturidade

Skill em `draft-para-testar`. A disciplina de coding (task a task, red-first, mecânica-antes-de-semântica,
handoff na soleira) é padrão do `flow.md`; a **automação** (descoberta + gate) é pré-validação. Promova ou pode
conforme o que o **rastro do projeto** (`wiki/log/`, o `tasks-fase<N>`, ou um testemunho de fase) registrar
sobre o que a skill pegou/deixou passar — *instrumentar antes de formalizar*.
