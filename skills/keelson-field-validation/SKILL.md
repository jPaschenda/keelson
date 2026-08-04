---
class: skill
name: keelson-field-validation
status: draft-para-testar
description: Orienta a VALIDAÇÃO DE ALTA FIDELIDADE de uma fase/feature do Método Keelson contra o ambiente real (stage/produção) — o "quarto modo". Consome a lista field-validation-required da revisão + os gates que só o ambiente real fecha, prescreve as checagens da mais segura à mais arriscada, registra evidência OBSERVADA ao vivo (não lida em log), e recomenda promoção a PILOT/PROD sem virar a chave. Use quando houver itens field-validation-required ou um gate que só o ambiente real fecha. NÃO executa deploy sozinha, NÃO promove (o humano vira a chave), e é honesta sobre o que não conseguiu reproduzir.
---

# Validação de campo (alta fidelidade)

Orienta o **quarto modo** — exercitar a **coisa real** (stage/produção), o eixo de **fidelidade** que nem o
coding nem a revisão independente cobrem (ambos operam sobre dublês; ver `.keelson/llm-dev-flow.md`, "As duas
independências"). **Honestidade de partida:** validação de alta fidelidade é **irredutivelmente cara e
parcialmente fora do alcance do método** — precisa de infra real, acesso ao ambiente, e às vezes não é
reproduzível. Esta skill **não executa** a validação; **disciplina e orienta** o que já é, por natureza, manual
e arriscado. O melhor que faz: ordenar as checagens, exigir evidência observada ao vivo, e nomear com franqueza
o que ficou de fora.

## Escopo — o que esta skill faz e onde para

- Consome a lista **`field-validation-required`** (produzida pela revisão) + os **gates** do plan que só o
  ambiente real fecha + as **hipóteses do brief** cujo nível-dono é `PILOT`/`PROD`.
- Prescreve as checagens contra o ambiente real e **registra a evidência observada ao vivo**.
- **Não** faz deploy sozinha; **não** promove a `PILOT`/`PROD` (o humano vira a chave); **não** escreve/edita
  brief/plan; **não** edita `.keelson/`.
- Não substitui revisão nem coding — é o eixo ortogonal (fidelidade), não "mais quem".

## Passo 1 — Reunir o alvo de campo

- A lista **`field-validation-required`** do handoff da revisão (o que só o host/prod real resolve).
- Os **gates** que exigem ambiente real (ex.: "alerta dispara ao derrubar o serviço"; "o gate de segurança
  dispara organicamente").
- As **suposições-sobre-o-mundo** da Superfície de incerteza (formatos/contratos inferidos de código, nunca
  confirmados contra o real).
- O **ambiente-alvo** (stage/produção) e o **modo de acesso**: você opera direto, ou o operador executa e cola
  o resultado (o modo "consulta à distância")?

## Passo 2 — Confirmação (gate humano; PARE — alto raio de explosão)

> 🌐 **Field Validation — alvo**
> - **Feature / Fase:** {…} — **ambiente:** {stage / PRODUÇÃO}
> - **Itens a validar:** {n de `field-validation-required` + gates + hipóteses}
> - **Acesso:** {agente direto / operador executa e cola}
> - **Risco:** {toca produção compartilhada? destrutivo?}
>
> Confirma? Comandos que tocam produção/infra compartilhada eu **prescrevo**; você executa e cola o resultado.

**Regra dura:** tocar ambiente real = alto raio. Não presuma acesso nem autorização; confirme o ambiente e o
modo antes.

## Passo 3 — Validar, do mais seguro ao mais arriscado

Para cada item, na ordem (**leitura → não-destrutivo → destrutivo/restart**; stage antes de prod quando existir):

1. **Checagem de pré-voo (obrigatória antes de qualquer comando de raio não-trivial):** reverifique **por
   ferramenta** a premissa de que o comando depende (a config / o estado real — grep/read/comando de leitura),
   **mesmo que já tenha lido antes nesta sessão**. "Já li" ≠ "está verdadeiro agora" (o runbook / o arquivo de
   config / o compose dizem o que você acha que dizem? confirme).
2. **Prescreva o comando concreto + o observável esperado** (o que provaria o item). Acesso indireto: entregue
   o comando para o operador rodar e colar.
3. **Registre a evidência OBSERVADA AO VIVO** — o que rodou, o que voltou de verdade. Distinga sempre
   **observado agora** de **lido em log antigo** ou **simulado**. Um item só "passa" se a coisa real foi vista
   fazendo a coisa certa.
4. **Se algo falha:** diagnostique contra o real, não contra a suposição — um sintoma pode não ter relação com
   a primeira hipótese razoável (um `401` pode ser um secret virando diretório, não auth).

**Limite conhecido — não bata a cabeça:** timing sub-segundo (janelas de corrida/crash) **não** é controlável
de forma confiável por orquestração remota (SSH / `docker exec`). Se um gate depende disso, **não** re-tente
"com mais cuidado" — **recomende instrumentação no próprio código** (delay/breakpoint gated por env) e registre
como **lacuna**, não como falha da validação.

## Passo 4 — Veredito e handoff

- **Evidência de campo por item:** confirmado ao vivo / não reproduzido / refutado. Hipótese **refutada** →
  seta de volta (revisão versionada do brief + `wiki/log/`), nunca edição silenciosa.
- **Recomendação de promoção** a `PILOT`/`PROD` **com base na evidência de campo** — mas **você não vira a
  chave**: a promoção é do humano.
- **Nomeie a lacuna residual sem disfarce** — o que ficou coberto só pela suíte, nunca reproduzido em campo (é
  a honestidade que a promoção carrega: "a única lacuna desta promoção é …").
- **Registre** no `wiki/log/` (validação de campo: itens, evidência ao vivo, lacuna residual) e no
  `tasks-fase<N>` / testemunho.

## Regras duras (não viole)

1. **Evidência = observada ao vivo**, não lida em log nem simulada. Se não viu a coisa real fazer, não passou.
2. **Não promova** a `PILOT`/`PROD`; **não** faça deploy sozinha — o humano vira a chave e autoriza cada comando
   de alto raio.
3. **Pré-voo obrigatório** antes de comando de raio não-trivial: reverifique a premissa por ferramenta, mesmo já
   lida na sessão.
4. **Timing sub-segundo não se valida por SSH remoto** — recomende instrumentação in-code, não re-tentativa.
5. **Honestidade da lacuna** — nomeie explicitamente o que não foi reproduzido; a promoção carrega isso à vista.
6. **Não edite o brief** (hipótese refutada = seta de volta) nem `.keelson/`; **não invente** — se faltar
   contexto, pare e pergunte.

## Nota de maturidade

Skill em `draft-para-testar`. É a **fronteira atual do método**: forte no pré-código (validação da spec) e na
verificação (camada congelada + revisão fresca), **fino na validação de alta fidelidade**. Esta skill **não
baratea** o que é caro — orienta e disciplina o irredutível. Nasce de campo: os testemunhos OptiFlux nomearam o
"quarto modo" (validação de campo assistida) duas vezes, incluindo a **parede de timing SSH**. Registre no
rastro do projeto (`wiki/log/` ou um testemunho de fase) o que ela ajudou e o que ficou fora — *instrumentar
antes de formalizar*.
