---
schema_version: "0.1"
class: application-guide-log
tool: Claude Code
clouple: llm-dev-claude.md
status: draft-pre-validacao
data: 2026-07-23
---

# Claude Code — log de capturas (frio, append-only)

> **O que é:** o lado **frio** da memória viva do application guide. O guia
> [`llm-dev-claude.md`](llm-dev-claude.md) é o roteador **quente** (a tabela de mapeamento por papel,
> pequena e atual). Este arquivo guarda as **capturas datadas** que embasam as células daquele guia —
> consultado só **sob escalada** (quando uma falha ou drift exige localizar *a ferramenta mudou* vs *a
> descrição nasceu errada*). **Append-only:** nunca se reescreve uma captura, só se adiciona a próxima.
>
> **Por que separado:** empilhar captura dentro do guia recria o **monólito que enterra** — o roteador
> quente ficaria ilegível. É o par índice-quente / log-frio da memória viva (ver
> [`llm-dev-memory.md`](llm-dev-memory.md)), aplicado ao guia.

## O que é uma captura

Cada entrada registra, com data, **duas metades**:

1. **Observável** (âncora dura, de agora): a saída de um comando determinístico — `claude --help` e
   subcomandos, o schema de `settings.json`, o changelog do CLI. Adjudica só a **superfície sintática** que
   toca (nomes de flag, de comando, de campo). É a metade **atual** — não envelhece com o training-cutoff.
2. **Auto-descrição** (a metade semântica): a ferramenta descrevendo os próprios mecanismos contra o molde
   das *linhas* do guia (os papéis do método) + a linha aberta "o que mudou / o que não perguntei?". Cobre o
   que o `--help` **não** alcança (ex.: *que* `@import` puxa conteúdo pro contexto; *que* hooks disparam a
   cada turno). **Nasce `desk`** até o campo confirmar.

Em conflito entre as duas, **o observável ganha — na superfície dele**. A metade semântica não observada
permanece rotulada `desk` na célula correspondente do guia.

## Capturas

_(Ainda sem captura registrada. A primeira entra quando a skill `keelson-application-guides-update` rodar contra o
Claude Code corrente — ver o loop de atualização no guia quente. Formato de cada entrada abaixo.)_

<!--
### [AAAA-MM-DD] captura | Claude Code <versão>
**Observável** (comando → saída relevante):
- `claude --help` → …
- schema de settings → …

**Auto-descrição** (papel → como a ferramenta diz que entrega; + "o que mudou?"):
- Tier 0 / carga incondicional → …
- …

**Reconciliação / conflitos** (onde observável corrigiu a auto-descrição):
- …

**Confiança resultante por linha:** Tier 0 = campo; hooks = desk; …
-->
