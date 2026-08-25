---
class: skill
name: keelson-deploy
status: draft-para-testar
description: Implanta em ambiente real (homologação e/ou produção) um artefato do Método Keelson que já chegou em BUILT — tasks-fase<N>.md, fix-<slug>.md ou tweak-<slug>.md, genérica sobre os três. Publica (git push) o commit de BUILT no branch/mecanismo que o deploy do projeto lê — commit local não é suficiente, e publicar é responsabilidade desta skill, não de quem codou/revisou. Executa o guardrail "deploy + health check" da transição →PILOT/PROD, com sondagem obrigatória antes de qualquer comando de raio não-trivial. Produz BUILT_DEPLOYED_PILOT ou BUILT_DEPLOYED_PROD. Use quando o usuário mandar implantar/fazer deploy de uma fase ou fix já revisado (ou com override registrado). NÃO promove PILOT/PROD sozinha (isso é keelson-field-validation) e NÃO revisa.
---

# Deploy — implantação em ambiente real

Executa a transição `BUILT` → `BUILT_DEPLOYED_PILOT`/`BUILT_DEPLOYED_PROD`, descrita em
`.keelson/llm-dev-flow.md` ("Sub-degraus" e a linha `→PILOT/PROD` da tabela de guardrails: "deploy + health
check"). Genérica sobre o artefato — `tasks-fase<N>.md` de uma feature, `fix-<slug>.md` de um bug ou
`tweak-<slug>.md` de um ajuste sub-especificado, a mecânica é a mesma. Você implanta e confirma saúde; **não** promove `PILOT`/`PROD` (isso exige evidência de campo,
`keelson-field-validation`) e **não** revisa código.

**Publicar é seu trabalho, não de quem codou/revisou.** "Não commite até `BUILT`" (`llm-dev-flow.md`) para no
**commit local** — é sobre a integridade do ciclo coding→revisão, escopo de `keelson-coding`/
`keelson-review-session`, que nunca tocam remoto. Tornar esse commit **alcançável pelo mecanismo de deploy do
projeto** (`git push` pro branch que ele lê, ou o que for equivalente) é parte de implantar, então é sua
fronteira — não presuma que "commitado" já significa "publicado".

## Escopo — o que esta skill faz e onde para

- Cobre **uma** fronteira: `BUILT` → implantado, aguardando validação de campo.
- **Precondição:** o artefato-alvo está em `BUILT` (revisado) — ou em `PILOT` já, se o alvo agora é produção
  depois de já ter passado por homologação.
- **Não promove** `PILOT`/`PROD` — isso é sempre `keelson-field-validation`, depois de evidência observada ao
  vivo.
- **Não codifica, não revisa.**

## Passo 1 — Auto-descoberta do alvo e do ambiente

- **Artefato:** `tasks-fase<N>-<slug>.md`, `fix-<slug>.md` ou `tweak-<slug>.md` em `BUILT` (ou `PILOT`, pro caso de ir a
  produção depois de homologação). Cruze com `wiki/now/<branch>.md` e `wiki/known-issues.md`.
- **Ambiente-alvo:** homologação (se existir) ou produção. Se **homologação existe** e o pedido é ir direto pra
  produção mesmo assim — **isso é a exceção**, não o caminho normal; sinalize no Passo 2 com aviso reforçado
  (abaixo). Se **não existe** homologação neste projeto, produção é o caminho normal, sem aviso — variação
  legítima de operador único sem staging (ver `wiki/known-issues.md`/histórico).
- **Modo de acesso:** você opera direto, ou o operador executa e cola o resultado?
- **O commit de `BUILT` está publicado?** Confira (`git log`/`git status` local vs. remoto) se o commit já
  chegou no branch/mecanismo que o deploy do projeto de fato lê — commit local, sozinho, não é suficiente. Se
  não estiver, publicar é parte desta implantação (Passo 3), não algo que já deveria ter acontecido antes.
- **O frontmatter do artefato diz `Feature state: BUILT`, de fato?** Não presuma — abra e leia. "Virar a
  chave" (`llm-dev-flow.md`) é um checklist de dois atos humanos, commit **e** gravação do estado; um sem o
  outro é um artefato mentindo sobre si mesmo. Se o commit existe mas o frontmatter ainda não diz `BUILT` (ou
  vice-versa), **pare e sinalize** — não complete a metade que falta por conta própria, isso é decisão do
  humano, não sua.

## Passo 2 — Confirmação (gate humano; PARE — raio alto)

Pare e apresente:

> 🚀 **Deploy — alvo**
> - **Artefato:** {tasks-fase{N} ou fix-{slug} ou tweak-{slug}} — **estado atual:** {BUILT / PILOT}
> - **Ambiente:** {homologação / **PRODUÇÃO**}
> - **Homologação existe neste projeto?** {sim / não}
> {se sim e o alvo é produção direto:} ⚠ **Pulando homologação existente** — este é o caminho de exceção, não
>   o padrão. Confirma mesmo assim?
> - **Acesso:** {agente direto / operador executa e cola}
>
> Confirma? Comandos que tocam ambiente compartilhado eu **prescrevo**; você executa e cola o resultado, se o
> acesso for indireto.

**Regra dura:** não presuma acesso nem autorização. O aviso reforçado acima não é opcional quando homologação
existe e é pulada — não silencie.

## Passo 3 — Implantar, do mais seguro ao mais arriscado

1. **Sondagem obrigatória (antes de qualquer comando de raio não-trivial):** reverifique **por ferramenta** o
   estado real do ambiente-alvo — `git status`/`git log` no host, configuração em disco (ex.: um script de
   geração de config pode estar desatualizado e sobrescrever algo já corrigido na mão), containers/processos
   ativos — **mesmo que já tenha lido isso antes nesta sessão**. "Já li" ≠ "está verdadeiro agora". *(Precedente
   de campo: um deploy real encontrou um script de geração de Caddyfile desatualizado que teria revertido um
   fix já aplicado manualmente — só a sondagem antes do restart pegou isso.)*
2. **Publique o commit, se a sondagem achou que ainda não está** (`git push` pro branch/mecanismo que o deploy
   lê). É a primeira ação de verdade, antes de tocar o ambiente-alvo — commit local não implanta nada sozinho.
3. **Se houver estado real em risco** (ex.: estratégias/processos ativos no ambiente-alvo), reporte ao operador
   e confirme explicitamente antes de agir — não presuma que é seguro só porque o código está pronto.
4. **Execute o deploy** (o comando concreto depende do projeto — runbook, se existir) + **health check**:
   confirme não só que o processo reiniciou, mas que o **código certo está rodando** (ex.: inspecionar o
   bundle/artefato dentro do container, não só "subiu sem erro").
5. **Registre o que foi (e não foi) verificado** — health check confirma saúde do processo; não é golden path
   nem validação funcional. Isso é `keelson-field-validation`, próximo passo.

## Passo 4 — Handoff

- Estado final: `BUILT_DEPLOYED_PILOT` ou `BUILT_DEPLOYED_PROD`.
- **Não promova** `PILOT`/`PROD` — aponte explicitamente pra `keelson-field-validation` como o próximo passo
  obrigatório antes dessa promoção.
- Feche com a Superfície de incerteza: o que o health check cobriu vs. o que só a validação de campo vai cobrir.

## Regras duras (não viole)

1. **Sondagem obrigatória antes de todo comando de raio não-trivial**, mesmo já lido nesta sessão.
2. **Não promova `PILOT`/`PROD`** — só `keelson-field-validation` faz isso, com evidência observada.
3. **Homologação existente pulada = aviso reforçado, sempre**, nunca silencioso.
4. **Não presuma acesso/autorização** a ambiente compartilhado ou produção.
5. **Não edite `.keelson/`**; não invente — se faltar contexto (runbook, credenciais, topologia), pare e
   pergunte.
6. **Publicar (`push`) é sua responsabilidade, não uma precondição que outra skill já deveria ter cumprido** —
   não presuma que commit local implica alcançável pelo mecanismo de deploy.
7. **Não complete sozinha um "virar a chave" pela metade** — commit sem `Feature state: BUILT` no frontmatter,
   ou o inverso, é sinalizado e parado, nunca corrigido por conta própria; a gravação do estado é ato humano
   (`llm-dev-flow.md`, "Virar a chave é um checklist"), não seu.

## Nota de maturidade

Skill em `draft-para-testar`. A mecânica de deploy que ela empacota (sondagem, health check que confirma o
código certo, não só "reiniciou") já rodou de fato nos 3 fixes reais do OptiFlux, sem uma skill dedicada até
então. **1º uso real (Fase 4, OptiFlux):** bloqueou corretamente um deploy tentado antes de `BUILT` (precondição
funcionou de primeira), e na tentativa seguinte já identificou sozinha, antes mesmo da sondagem formal, que o
commit de `BUILT` não estava publicado — o que motivou nomear `push` como responsabilidade explícita desta
skill, não uma precondição presumida de quem codou/revisou. *Instrumentar antes de formalizar.* A checagem de
`Feature state: BUILT` na sondagem (leva 2026-08-13) ainda não rodou em campo — desenhada a partir do mesmo
gap que motivou o `push`, ainda sem 1º uso real observado.
