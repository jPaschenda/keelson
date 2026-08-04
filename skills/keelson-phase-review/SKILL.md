---
class: skill
name: keelson-phase-review
status: draft-para-testar
description: Revisão INDEPENDENTE do código de uma fase do Método Keelson (guardrail tasks-faseN → BUILT). Roda em sessão separada da que escreveu o código, com contexto fresco; confere o diff da fase contra a camada congelada (brief §, ADR/invariants, glossary), faz mecânica antes de semântica, checa a QUALIDADE dos testes (não só "passam"), e emite veredito com poder de mandar de volta. Use quando o usuário pedir para revisar/validar o código de uma fase antes de promover a BUILT. NÃO use para escrever/aterrissar/codificar, NÃO promove a BUILT (o humano vira a chave) e NÃO é validação de campo/deploy.
---

# Revisão de fase (independente)

Implementa o guardrail que o `.keelson/llm-dev-flow.md` prescreve para `tasks-fase<N>` `NOT_BUILT` → **soleira**
de `BUILT`: a **revisão independente em sessão separada**. Por que existe: o modo de falha do agente não é
preguiça, é **confiante-mas-errado** — testes verdes provam só o que o autor pensou em testar, e ele escreveu o
código **e** os testes, que podem compartilhar o mesmo mal-entendido (*"um mock nunca discorda do autor"*). O
guardrail que importa é **check externo objetivo que o autor não consegue racionalizar** — por isso esta skill
roda com **contexto fresco**, ancorada na camada congelada, e **quem revisa ≠ quem escreveu**.

## Escopo — o que esta skill faz e onde para

- Confere o **diff da fase** contra a camada congelada, na transição `tasks-fase<N>` → **soleira** de `BUILT`.
- **Roda em sessão separada** da que codou (a independência vem do contexto fresco + árbitro objetivo, não da
  mesma cabeça relendo o próprio trabalho).
- **Não promove a `BUILT`** — o humano vira a chave; a skill pode mandar a fase **de volta** para coding.
- **Não é validação de campo/deploy** — revisão de código não descobre que um secret falta num host, que um
  restart precisa de `--force-recreate`, etc.; isso só rodando o sistema de verdade pega. **Nomeie o que fica de fora.**
- Não escreve/edita `brief` nem `plan`; não codifica; não edita `.keelson/`.

## Passo 1 — Descoberta do alvo + precondição de independência

- **Ache a fase a revisar:** o `tasks-fase<N>-<slug>.md` na **soleira** (checkboxes fechados, testes verdes)
  ainda `NOT_BUILT`. Cruze com `wiki/now/<branch>.md` e o `git status`/`git log` (o **diff da fase** é o objeto
  da revisão).
- **Precondição de independência (dura):** esta sessão deve ter **contexto fresco** — **não** ser a que escreveu
  o código. Se você percebe que acabou de aterrissar/codificar esta fase **nesta mesma sessão**, a revisão perde
  o valor (autor abençoando a própria prova) → **avise no Passo 2** e recomende sessão nova.
- **Colha as âncoras da camada congelada:** os §§ do brief da fase (via a **tabela de aterrissagem** do
  `tasks`), os **ADRs/invariants** (índice fino), o **glossary**. O revisor **não inventa critério** — confere
  contra o que a wiki já mantém vivo.

## Passo 2 — Confirmação (gate humano; PARE aqui)

Pare e apresente:

> 🔍 **Phase Review — alvo**
> - **Feature / Fase:** {feature} — Fase {N}
> - **Tasks:** {n fechadas de m} · **testes:** {verdes? / n}
> - **Diff:** {range de commits / arquivos tocados}
> - **Independência:** {sessão fresca / ⚠ mesma sessão que codou — recomendo sessão nova}
>
> Confirma que reviso? (a revisão pode mandar a fase **de volta** — não é carimbo garantido)

**Regra dura:** se a sessão não é independente, sinalize antes de prosseguir — não silencie o aviso.

## Passo 3 — Verificação: mecânica antes de semântica

**3A — Mecânica (barata, determinística, sem julgamento — rode primeiro).** É a auto-garantia objetiva que o
autor não racionaliza por cima:

- suíte de testes verde + lint + type-check + cobertura mínima;
- **greps de invariante** sobre o **diff** (procurar a violação direto na mudança).

**3B — Semântica (só o que a mecânica não pega), ancorada na camada congelada:**

- **Fidelidade ao brief §:** cada task fecha o requisito que a tabela de aterrissagem mapeia? Algum item do §
  ficou sem cobertura? A rastreabilidade (task → §) se sustenta?
- **Invariantes / ADR:** o diff respeita os invariantes duros e as decisões congeladas?
- **Qualidade dos testes — o cheque de maior valor:** os testes **importam a lógica real** ou a
  **reimplementam**? Cobrem os **caminhos de risco**? São **tautológicos** (provam o que o próprio código
  afirma)? Procure onde **teste e código compartilham a mesma suposição** — é a costura onde o "confiante-mas-
  errado" passa verde (o defeito clássico: um mock escrito pela mesma mão que o produtor, concordando com ele
  por construção).
- **Fronteira só dublada — independência de *camada*:** o diff cruza uma **fronteira** (ponto onde o código
  depende de um contrato que não possui — outro processo/serviço, SO/arquivos, dispositivo, motor de banco,
  protocolo, módulo de outra equipe, lib externa) que os testes desta fase só substituem por um **dublê**? A
  independência de *sessão* (você ≠ autor) **não protege** aqui — revisor e código compartilham o mesmo dublê,
  que não discorda de nenhum. **Triagem em dois níveis:**
  - **Dá para cruzar in-suite** (subir o contraparte real: middleware/servidor/schema real)? Então falta um
    **teste-de-costura** — é lacuna de coding: **VOLTA**, reabra como task. (Barato; não empurre para campo o
    que um teste pega.)
  - **Só o host/produção real resolve** (secret provisionado, timing real, serviço externo vivo)? Então é
    **`field-validation-required`** → entra na lista de campo do handoff (Passo 4).
- **Deriva arquitetural** e **segurança sutil**.
- **Superfície de incerteza do handoff:** o coding declarou "o que assumi / onde posso estar errado / o que não
  verifiquei"? Comece a cutucar exatamente por aí.

Proporcional ao **raio de explosão** — não uniformize rigor; energia onde produz solidez.
*(No Claude Code, a passada semântica pode apoiar-se no `/code-review` apontado para os §§/ADRs — mas a âncora
na camada congelada e o cheque de qualidade-dos-testes são o que esta skill acrescenta.)*

## Passo 4 — Veredito e handoff

- **Lista de achados**, rankeada (mais severo primeiro). Cada achado: *o que está errado* · *cenário de falha
  concreto* (inputs/estado → saída errada) · *o § / invariante / task que ele fere*.
- **Veredito:**
  - **VOLTA para coding** — há achado bloqueante; liste-os como tasks a reabrir no `tasks-fase<N>`. (Revisão sem
    poder de mandar de volta é *review theater* — custo sem valor.)
  - **PRONTO para o humano virar a chave** — nenhum achado bloqueante. Mesmo assim **você não promove**: a chave
    de `BUILT` é do humano.
- **Registre o evento** no `wiki/log/` (revisão da Fase N: veredito + achados).
- **Nomeie o que a revisão NÃO cobre — e entregue a lista de `field-validation-required`:** os itens que **só o
  host/produção real resolve** (Passo 3B — os cruzáveis in-suite viraram VOLTA por teste-de-costura, não campo)
  viram o checklist da validação de alta fidelidade (o modo `keelson-field-validation`). Some a validação de
  **deploy** (só rodar de verdade pega) e as **hipóteses do brief** (validam em `PILOT`, não aqui).

## Regras duras (não viole)

1. **Sessão separada / contexto fresco** — quem revisa ≠ quem escreveu. Se for a mesma cabeça, sinalize; não finja independência.
2. **Não promova a `BUILT`** — o humano vira a chave.
3. **Mecânica antes de semântica** — o check objetivo primeiro; é o que o autor não racionaliza.
4. **Ancore na camada congelada** — não invente critério; confira contra brief §, ADR/invariants, glossary.
5. **Poder de devolver** — achado bloqueante manda a fase de volta; revisão sem isso é teatro.
6. **Honestidade de escopo — duas independências.** Independência de *sessão* (revisor ≠ autor) ≠ independência
   de *camada* (cruzar a fronteira real vs. mock); revisão de código ≠ validação de campo. Declare o que fica
   de fora e o que exige campo.
7. **Não edite o brief** (`VALIDATED`) nem `.keelson/`; **não invente** — se faltar contexto, pare e pergunte.

## Nota de maturidade

Skill em `draft-para-testar`. A revisão independente é o **guardrail nº 1** do método para `→BUILT` (o antídoto
ao confiante-mas-errado). **Medição de campo (OptiFlux, Fase 2):** pegou **2 bugs reais que 275 testes verdes
não pegaram** (um deles derrubava o processo inteiro), reproduzidos antes de virar task; e teve o **limite
medido** de não pegar um bug de **fronteira HTTP só mockada** — o que motivou o check de *independência de
camada* (Passo 3B). Continue registrando **no rastro do projeto** (`wiki/log/` ou um testemunho de fase) o que
ela pegou que os testes verdes não pegaram — é isso que decide se ela endurece. *Instrumentar antes de
formalizar.* Origem de campo: testemunhos Fase 0/1 (a revisão que faltou **por ritmo**) e Fase 2 (a revisão com
dentes + o limite de camada).
