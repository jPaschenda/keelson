# Método Keelson — User Guide

*Seu colega brilhante tem amnésia e mente com convicção — eis como construir software com ele mesmo assim.*

> **Rascunho (v0.1).** **O guia de uso, passo a passo.** Se o livro *Confiante e Errado* explica **por que** o método é assim, este guia explica **como** colocá-lo para rodar no seu projeto. O Guia é para **consultar**, não para ler de cabo a rabo: vá direto ao caminho que é o seu.

# Sumário

Este guia é para **consultar**, não para ler de ponta a ponta. Use este mapa para ir direto ao ponto — e volte a ele quando precisar. O coração é a bifurcação em **§2.3**: projeto novo segue o **§3**; projeto que já roda segue o **§4**. O resto (**§5** a **§9**) vale para os dois.

**Introdução.** Para que serve o guia e por que ele anda junto com o livro *Confiante e Errado*: o propósito, o ecossistema de cinco pilares do Método Keelson, a tese e a história do livro, e por que separar o *porquê* (livro) do *como* (guia).

**§1 — O que é este guia (e como usá-lo).** As peças do método em uma respiração — os três padrões do núcleo, a camada por ferramenta, os instaladores e a skill — e a regra que rege o próprio guia: *aponta, não re-explica*.

**§2 — Antes de começar.** As coisas no lugar antes do primeiro comando: instalar o método em `.keelson/` (2.1), como a evidência sobre a ferramenta é gerada (2.2), descobrir qual é o seu caminho (2.3), instalar as skills (2.4), o teste honesto — *este projeto merece o método?* (2.5) — e navegar tudo isto no Obsidian (2.6).

**§3 — Caminho A · Projeto novo (greenfield).** O passo a passo do bootstrap, o esqueleto "Dia 1" que você deve obter, e os próximos degraus por demanda — não por mutirão. *Vá aqui se está começando do zero.*

**§4 — Caminho B · Projeto que já roda (migração).** Como classificar o que já existe sem quebrar o que funciona — e o que **não** fazer. *Vá aqui se já há código e documentação para reorganizar.*

**§5 — O ciclo de uma feature.** Como uma feature viaja do brief ao campo, e as cinco skills que a conduzem por cada fronteira: lançar o plano, aterrissar a fase, codificar, revisar em sessão independente e validar em campo. O motor operacional do método — vale para os dois caminhos.

**§6 — Exemplos de campo.** O método num sistema real, em produção: a aterrissagem que se paga antes do código, a revisão independente que pega o que 275 testes verdes não pegam, o bug que só o campo achou, e a chave que só o humano vira. Ilustra o §5 sem entrar no detalhe do sistema.

**§7 — A ferramenta.** O guia quente, o log frio de capturas e a skill de update: manter vivo o encaixe do método na sua ferramenta, sem editar o guia à mão. Vale para os dois caminhos.

**§8 — A disciplina que faz o método grudar.** As rotinas que sustentam a memória viva, o dia de jogo (o ritmo da prática) e o painel de saúde do praticante. É o que separa o método de um teatro de arquivos.

**§9 — Referência rápida.** A tabela de consulta para o dia a dia — o que abrir para cada tarefa — e o checklist de instalação.

**Apêndices — os três padrões do núcleo.** Ponteiros de leitura para os contratos que este guia opera mas não copia: a **memória** (A → `llm-dev-memory.md`), o **fluxo** (B → `llm-dev-flow.md`) e o **jogador** (C → `llm-dev-player.md`).

---

# Sobre a obra

*"Este Guia foi desenvolvido de forma ágil, utilizando o Claude Code (Anthropic) como assistente de cocriação de conteúdo e revisão. Ao longo da leitura, você descobrirá que a própria escrita desta obra ajudou a moldar a construção do método. Muitos de seus princípios do método foram aplicados em tempo real durante a produção das páginas, criando uma relação simbiótica e de indução mútua entre autor/desenvolvedor, Claude Code, método e livro."*

---

# Introdução

## Propósito

Este User Guide (Guia do Usuário) é o manual prático para a aplicação do método apresentado no livro *Confiante e Errado*. Juntos, os dois textos apresentam um método de desenvolvimento assistido por LLMs que batizei de Método Keelson (no livro, você entenderá o porquê deste nome).

Não faz sentido utilizar este guia sem a leitura prévia da obra. Enquanto o livro explica o porquê por trás do método, este manual detalha o como colocá-lo em prática no seu projeto. Por ser um documento estritamente prático, o guia não apresentará conceitos teóricos, nem explicará a natureza ou a importância das ações orientadas.

Ler este material sem o embasamento do livro tornará a compreensão das propostas muito difícil. O inverso também é verdadeiro: ler apenas o livro trará grandes desafios na hora de executar as ideias discutidas. Os dois materiais completam-se mutuamente.

## O Ecossistema do Método Keelson

Gosto de pensar que a estrutura completa funciona como um ecossistema integrado, composto por cinco pilares:
1. O livro *Confiante e Errado*: A fundação conceitual — a tese, a história e o principios por trás do método.

2. Este **User Guide**: O manual prático de execução e consulta para o seu dia a dia.

3. Os **Padrões do Método**: Um conjunto de instruções escritas especificamente para orientar o LLM na aplicação correta do método.

4. As **Skills do Método**: instruções executáveis que conduzem o desenvolvedor humano pelas etapas do fluxo — do plano à validação de campo — e mantêm o encaixe na ferramenta sempre em dia.

5. **Ferramentas e Prompts**: Uma camada de utilitários e alguns exemplos práticos de prompts para acelerar a ativação do método no seu projeto.

## A Tese e a História do Livro

O livro **Confiante e Errado** assenta-se sobre uma fundação clara:

- A Tese: Quando o programador mais frequente de um projeto passa a ser uma Inteligência Artificial, o artefato mais valioso deixa de ser o código ou a especificação. Ele passa a ser a memória compartilhada viva que mantém ambos atualizados. A disciplina que congela essa memória por evidência de funcionamento é o que impede o código de derivar.

- A História: Nada disso foi projetado teoricamente em uma mesa de escritório. O método emergiu de um sistema real, em produção, através do uso prático e iterativo no campo. O próprio método se refinou seguindo sua própria regra de aprendizado.

## Por que separar o Livro do Guia?

Explicar os conceitos teóricos e demonstrar o uso prático simultaneamente tornaria o livro excessivamente extenso.

A divisão em dois textos independentes garante missões claras e vantagens estratégicas:

- Consulta Dinâmica: O User Guide tornou-se um texto de cabeceira para o dia a dia. Ele foi desenhado para ser aberto e consultado durante o uso cotidiano do método.

- Ciclo de Atualização Rápido: No desenvolvimento de software com LLMs, as ferramentas evoluem em ritmo acelerado. Esta divisão permite manter os conceitos estáveis e perenes no livro, concentrando as atualizações rápidas, evoluções de ferramentas e refinamentos práticos exclusivamente neste Guia.

---

# 1. O que é este guia (e como usá-lo)

A utilização do Método Keelson pelo Humano e pelo LLM é suportada por conjunto de arquivos. Em uma respiração:

- **Três padrões do núcleo** — [`llm-dev-memory.md`](llm-dev-memory.md) (onde o conhecimento mora),
  [`llm-dev-flow.md`](llm-dev-flow.md) (como o trabalho anda), [`llm-dev-player.md`](llm-dev-player.md) (o
  papel do humano). Agnósticos a ferramenta.
- **Uma camada fina por ferramenta** — o *application guide* (hoje [`llm-dev-claude.md`](llm-dev-claude.md),
  para o Claude Code) e o seu **log frio** de capturas ([`llm-dev-claude.log.md`](llm-dev-claude.log.md)),
  que ligam os papéis do método aos mecanismos concretos da sua ferramenta.
- **Os instaladores** — dois prompts de exemplo ([bootstrap](llm-dev-prompt-bootstrap.md) para projeto novo,
  [migração](llm-dev-prompt-migration.md) para projeto existente) e um playbook  ([`llm-dev-migration.md`](llm-dev-migration.md)).
- **As skills** — as **cinco do fluxo** ([`keelson-plan-init`](skills/keelson-plan-init/),
  [`keelson-phase-landing`](skills/keelson-phase-landing/), [`keelson-phase-coding`](skills/keelson-phase-coding/),
  [`keelson-phase-review`](skills/keelson-phase-review/), [`keelson-field-validation`](skills/keelson-field-validation/)),
  que conduzem uma feature pelas fronteiras do fluxo, e [`keelson-application-guides-update`](skills/keelson-application-guides-update/),
  que mantém o guia da ferramenta em dia quando a **ferramenta** muda.

A lista completa da caixa está no [`llm-dev-package.md`](llm-dev-package.md); o mapa conceitual e o glossário,
no [`llm-dev-README.md`](llm-dev-README.md). **Comece por esses dois se ainda não os leu** — este guia
pressupõe que você sabe o que cada peça é. A partir daí, o trabalho deste documento é um só: **te levar do
zero a um método instalado e se sustentando sozinho no seu projeto.**

## A regra que rege este guia também

> **Aponta, não re-explica.** Quando você precisar do contrato completo de um padrão — o formato exato, as
> disciplinas de edição, as convenções —, este guia **manda você ao arquivo do padrão**; ele não copia o
> conteúdo. O padrão é a autoridade; este guia é o procedimento. É de propósito: assim o padrão pode evoluir
> sem obrigar uma revisão do guia.

## Como navegar

Escolha o seu caminho — não leia os dois:

- **Projeto novo, do zero** → **§3 (Greenfield)**.

- **Projeto que já roda**, com documentação espalhada ou um arquivo-raiz que já virou monólito → **§4  (Migração)**.

Depois, **§5** (o ciclo de uma feature, conduzido pelas skills) é o motor do trabalho, ilustrado em **§6** (exemplos de campo de um sistema real); **§7** (manter o guia da ferramenta vivo) e **§8** (a disciplina que faz o método grudar) valem para os dois. A **§9** é a referência rápida para consultar depois. Os **apêndices** apontam para os três padrões do núcleo, com uma orientação curta de leitura de cada.

> **Uma convenção de vocabulário deste guia.** Chamamos de **Tier 0** o arquivo-raiz que a ferramenta carrega em *toda* sessão (no Claude Code, o `CLAUDE.md`). O nome concreto de cada ferramenta vive no *application guide* dela; aqui usamos o papel. Os demais termos do conjunto (application guide, referência do fabricante, Tier 0 seed) estão no glossário do [`llm-dev-README.md`](llm-dev-README.md).

---

# 2. Antes de começar

Três coisas no lugar antes do primeiro comando.

## 2.1 Os arquivos do método, dentro do projeto (a pasta `.keelson/`)

Você recebeu um conjunto de arquivos — o **pacote do método** (a lista completa está no
[`llm-dev-package.md`](llm-dev-package.md)). Antes de qualquer coisa, esses arquivos precisam **morar dentro do seu projeto**. Dois passos simples:

1. **Crie uma pasta chamada `.keelson/`** na **raiz** do seu projeto (o mesmo nível do `.claude/` e do  `.git/`). O ponto no início do nome é de propósito — sinaliza "isto é ferramenta/método, não código do  projeto".
2. **Copie para dentro dela** todos os arquivos do pacote que você recebeu (os `llm-dev-*.md` e a pasta  `skills/`). Ao final, você terá algo como `.keelson/llm-dev-flow.md`, `.keelson/llm-dev-memory.md`, e assim  por diante.

É só isso. A partir daqui, é **de dentro de `.keelson/`** que o agente lê o método, e é para lá que o Tier 0 (o `CLAUDE.md`) vai apontar — por caminho **relativo** (`.keelson/llm-dev-flow.md`), que funciona em qualquer máquina. 

> **Por que uma pasta separada, e não solta na raiz nem dentro de `wiki/`/`docs/`?** Porque `wiki/` e `docs/` são a memória do **seu projeto**; o método é outra coisa — é a *ferramenta*. Mantê-lo em `.keelson/` deixa essa fronteira óbvia e evita misturar as duas coisas. Pense na cópia como **fixa (pinada)**: ela fica na versão que você copiou até você decidir, deliberadamente, trocá-la por uma versão nova do método.

Um lembrete prático: os prompts de instalação (§3 e §4) já assumem que o método está em `.keelson/`. Se o agente não conseguir enxergar esses arquivos, a instalação não acontece — então confira este passo antes de seguir.

## 2.2 A evidência sobre a ferramenta de codificação (o agente de IA)

O *application guide* (ex.: `llm-dev-claude.md`) **não** resume a documentação da sua ferramenta de codificação (Claude Code, Codex, Kimi Code…) — ele **aponta** para a evidência sobre ela. Ferramentas de linha de comando não têm um "manual congelável".

Em vez disso, a evidência é **gerada sob demanda** — pela skill de update (§7), quando você souber que a ferramenta mudou (por exemplo, após um update/upgrade de versão, **você roda a skill de update**). Ela captura o **observável** da ferramenta (`--help`, schema de configuração) somado à **auto-descrição** dela, e grava tudo, datado, no **log frio** `llm-dev-<ferramenta>.log.md` (para o Claude Code, [`llm-dev-claude.log.md`](llm-dev-claude.log.md)). Como isso funciona por dentro fica na §7.

> **Por que isso é bom:** o método não fica preso a uma ferramenta. Para cada uma, o guia é gerado *perguntando à própria ferramenta* o que ela oferece e adaptando-se a ela — então o mesmo método serve Claude Code, Codex, Kimi Code e os que vierem.
>  
> Neste momento, antes de começar, **você não pré-carrega nada** — só garante a skill instalada (§2.4) e segue.

## 2.3 Qual é o seu caminho

A bifurcação decide tudo o que vem depois:

| A sua situação | O caminho | A natureza do trabalho |
|---|---|---|
| Projeto **novo**, sem conhecimento a sintetizar ainda | **Greenfield — §3** | **criar** o esqueleto certo, para que todo conteúdo novo já nasça no lugar |
| Projeto que **já roda** — código, docs espalhados, um Tier 0 que provavelmente já inchou | **Migração — §4** | **classificar** o que já existe, sem quebrar o que funciona |

O critério de sucesso também muda: no greenfield é *criar bem*; na migração é **não quebrar**. Se você tem dúvida sobre em qual caso está, a pergunta decisiva é: *já existe documentação gerada que eu teria de reorganizar?* Se sim, é migração.

## 2.4 Instale as skills do método (Claude Code)

O método vem com **skills** — instruções executáveis que conduzem você pelas etapas do fluxo e pela manutenção do guia da ferramenta. Elas são o único artefato do pacote que **não** roda de `.keelson/`: o Claude Code só descobre skills em `.claude/skills/`. Então **copie cada pasta `skills/<nome>/` para `.claude/skills/<nome>/`** no seu projeto (ou no seu diretório de usuário, se quiser tê-las em todos os projetos).

São seis, em duas famílias:

- **As cinco skills do fluxo** — `keelson-plan-init`, `keelson-phase-landing`, `keelson-phase-coding`, `keelson-phase-review`, `keelson-field-validation` — conduzem uma feature do brief ao campo, uma fronteira de cada vez. São o motor do **§5**.
- **A skill de atualização do "application-guide" da sua ferramento** — `keelson-application-guides-update` — mantém o guia da ferramenta em dia quando a *ferramenta* muda. Entra em ação no **§7**.

Deixe todas instaladas agora; cada uma entra em cena na sua seção. (Por que elas moram em `.claude/skills/` e não em `.keelson/` como o resto do pacote: o contrato está no [`llm-dev-package.md`](llm-dev-package.md), "Exceção: skills rodam de `.claude/skills/`".)

## 2.5 Um teste honesto antes de instalar qualquer coisa: **este projeto merece o método?**

O método cobra uma moeda — **curadoria contínua** — e há projetos onde ela não se paga. Antes de seguir, responda duas perguntas:

1. **Este código vai viver mais do que umas poucas sessões, e alguém responde por ele?**
2. **Vai existir um guardião** — uma pessoa que sustenta a curadoria (audita, poda, vira as chaves)?

Se qualquer resposta for "não", **pare aqui**: um protótipo descartável precisa de velocidade e de uma lixeira, não de memória viva; e uma wiki sem guardião é pior que nenhuma — é uma fonte da verdade mentindo com autoridade. O *porquê* completo desse teste está no livro (Cap. 23, "Quando NÃO usar"). Passou nas duas? Siga em frente.

## 2.6 Navegar a documentação: o Obsidian (opcional, muito recomendado)+

O método produz uma **teia**, não uma pilha de arquivos: o `index.md` aponta para `now/` e `log/`; um `brief` se liga ao seu `plan` e aos seus `tasks`; um ADR referencia um invariante; o glossário é citado de toda parte. Num explorador de arquivos comum você vê a *árvore*, mas não as *ligações* — e são as ligações que fazem a memória navegável.

O [**Obsidian**](https://obsidian.md) (gratuito) lê esses mesmos arquivos `.md` no lugar e revela a teia: **wikilinks** clicáveis, **backlinks** ("o que aponta para esta página?"), o **outline** de cada documento e o **grafo** — o mapa visual de como tudo se conecta. Não é obrigatório e o método não depende dele (tudo é Markdown puro); mas para *consultar* e *fazer a curatoria* da documentação, ele muda o jogo.

![O cofre de um projeto real no Obsidian: a árvore de docs à esquerda, o grafo da documentação no topo, um brief aberto com seu outline à direita e a contagem de backlinks ("9 links inversos") no rodapé — a teia do método, navegável.](images/obsidian-overview-documentation.png)

Quatro recomendações — e paramos por aqui, porque isto não é um manual do Obsidian:

1. **Baixe em [obsidian.md](https://obsidian.md)** — grátis, multiplataforma. Ele não converte nem move nada: abre a pasta como ela está.

2. **Um cofre por projeto.** Aponte o cofre para a **raiz do projeto** (o repo), para ele indexar `wiki/`, `docs/` e os arquivos do método em `.keelson/` juntos. Um cofre = a teia de conhecimento de **um** projeto; não misture projetos no mesmo cofre.

3. **Configure o `userIgnoreFilters`** (Configurações → *Arquivos e links* → "Arquivos excluídos"). Exclua o ruído — `node_modules/`, `.git/`, artefatos de build e, se quiser, o próprio `.keelson/` (o método vendorizado) — para que o **grafo mostre o conhecimento do seu projeto**, não a biblioteca de vendor. Um grafo poluído não ajuda a farejar órfão nem aglomerado solto; um grafo limpo é o "passeio pelo grafo" da §8 virando ferramenta de verdade. Para configurar o userIgnoreFilters, no diretorio vendorizado do obsidian você vai encontrar um arquivo chamado app.json. ('.obsidian/app.json). 
Veja exemplo parcial do `app.json`:
```
   {
    "userIgnoreFilters": [
     ".claude/",
     ".git/",
     ".github/",
     ".keelson/",
```
4.  Opcionalmente, pode para remover a visão da estrutura diretório do "file explorer" você precisa do plugin **CSS snippets** e necessita criar um arquivo ccs que chamo `only-docs-wiki.ccs` que fica dentro de .obsidian/snippets/only-docs-wiki.ccs. Eu uso as vezes, mas não consegui remover um bug que o obsidian corta o final da lista do "file explorer"
Mas de qualquer forma, veja um exemplo parcial do `only-docs-wiki.ccs`:

```
/* userIgnoreFilters (app.json) esconde os ARQUIVOS de fora de docs/ e wiki/,
   mas a pasta-container continua na árvore (agora vazia). Este snippet
   esconde a linha da própria pasta. Mantenha esta lista em sincronia com
   as entradas de pasta ("nome/") de userIgnoreFilters em app.json. */

div.nav-folder-title[data-path=".claude"],
div.nav-folder-title[data-path=".claude"] + div.nav-folder-children,
div.nav-folder-title[data-path=".git"],
div.nav-folder-title[data-path=".git"] + div.nav-folder-children,
div.nav-folder-title[data-path=".github"],
div.nav-folder-title[data-path=".github"] + div.nav-folder-children,
div.nav-folder-title[data-path=".keelson"],
div.nav-folder-title[data-path=".keelson"] + div.nav-folder-children,
```

Dica: você pode copiar o **prompt** abaixo para o próprio LLM fazer a configuração: 

```
Configure o Obsidian deste vault deste projeto para exibir no File Explorer e no Graph apenas as pastas `docs/` e `wiki/`, escondendo tudo mais que estiver na raiz do vault. Faça assim:
1. Liste os itens atuais da raiz do vault (pastas e arquivos), excluindo `docs/` e `wiki/`.
2. Em `.obsidian/app.json`, adicione (ou atualize) `userIgnoreFilters` com uma entrada por item: pastas como `"Nome/"` (com barra), arquivos com o nome exato.
3. Em `.obsidian/snippets/only-docs-wiki.css`, para cada pasta da lista, adicione o par de seletores (sem usar `:has()`, que não funciona aqui):
   ```css
   div.nav-folder-title[data-path="NomeDaPasta"],
   div.nav-folder-title[data-path="NomeDaPasta"] + div.nav-folder-children {
     display: none;
   }
(Isso é necessário porque `userIgnoreFilters`só esconde arquivos — a pasta-container vazia continua aparecendo na árvore sem esse CSS.)
4. Em `.obsidian/appearance.json`, garanta `"enabledCssSnippets": ["only-docs-wiki"]`.
5. Me avise para reabrir Configurações → Aparência → CSS snippets e clicar em "reload" para o Obsidian pegar o arquivo.
```
### Não só conforto
> **O método, não só conforto:** a §8 pede, na cadência da semana, um *passeio pelo grafo* — caçar páginas órfãs (sem links de entrada) e aglomerados que se desconectaram. Isso é quase impossível numa árvore de arquivos e trivial num grafo. O Obsidian transforma uma disciplina abstrata ("vigie a saúde da memória") num gesto visual de poucos segundos. 



Agora sim, siga para o seu caminho — **§3** (greenfield) ou **§4** (migração).

---

# 3. Caminho A — Projeto novo (greenfield)

Você tem um projeto do zero. O objetivo aqui **não** é documentar nada ainda — é montar o esqueleto correto e instalar as convenções, para que **todo conteúdo novo já nasça no lugar certo**.

## Passo a passo

1. **Confira os pré-requisitos** (§2): arquivos do método acessíveis, idioma do projeto decidido.
2. **Abra uma sessão** da sua ferramenta com o projeto (vazio ou quase) aberto.
3. **Rode o prompt de bootstrap.** Abra o [`llm-dev-prompt-bootstrap.md`](llm-dev-prompt-bootstrap.md), copie o bloco de instruções e cole na sessão. **Ajuste os `<placeholders>`** — principalmente o idioma do projeto e o nome da sua ferramenta. Ele é um ponto de partida genérico; adapte ao seu contexto.
4. **Deixe o agente criar o esqueleto.** Ao terminar, ele deve te mostrar a árvore criada e o conteúdo do  Tier 0 — e, se a sua ferramenta tiver um *application guide*, disparar o **teste de fora**, que confirma que o guia bate com a ferramenta real que você está utilizando. Você começa com evidência já no **dia 1** (o detalhe está na §7).

## O que você deve obter — o esqueleto "Dia 1"

Um esqueleto mínimo, que cabe numa tela:

```
.keelson/           # o pacote do método, vendorizado (cópia pinada) — o Tier 0 aponta aqui por path relativo
wiki/
  index.md          # o mapa: o que existe, onde mora, onde NÃO olhar
  glossary.md       # nasce vazio; cresce dos mal-entendidos reais
  invariants.md     # nasce vazio; cresce do teste de admissão
  known-issues.md   # nasce vazio; ponteiro do que está quebrado/em tratamento AGORA
  now/main.md       # onde estamos, o que se estava fazendo, próximo passo
  log/2026-07.md    # o diário do mês, append-only
docs/
  architecture.md   # um stub curto — a preencher por uso
  decisions/        # os registros de decisão (ADR), quando vierem
  specs/            # o que ainda vai ser construído (brief/plan/tasks)
  fixes/            # o irmão corretivo de specs/ (conserta o presente) — nasce vazio
  reports/          # relatórios/auditorias externas, cruas
  runbooks/         # manobra operacional (deploy, rotação de chave) — nasce vazio
  domain/           # fontes externas (regulamento, norma) — quando houver
```

Uma nota sobre `runbooks/`, que é fácil confundir com "documentação de operação": ele guarda o **roteiro imperativo de uma manobra** — o passo-a-passo de um deploy, de uma rotação de chave, de um restart — **junto com os guardrails de cada passo** (o "não faça isto antes daquilo"). Nasce vazio, como `decisions/`, e só ganha o primeiro arquivo quando você de fato executar uma manobra que valha registrar. Não é o **estado** do
sistema (o que está no ar mora no `now/`); é **como se muda** o estado.

Atenção para mais dois nós fáceis de confundir. Para diferenciar o `docs/fixes/` é o **irmão corretivo** da `specs/`.  No caso a  `specs/` constrói o futuro, o `fixes/` conserta o presente — nasce vazio e só ganha o primeiro`fix-<slug>.md` quando um bug de porte aparecer. Importante que bug trivial não vira documento: é commit + linha no `log/` + teste de regressão. 

Já `known-issues.md` é o **ponteiro transiente** do que está quebrado ou em tratamento **agora** — o par do `invariants.md` (um diz o que o sistema *sempre/nunca* faz; o outro, o que está quebrado *hoje*). O `known-issues.md` é para o agente que começa frio não retrabalhar um bug já conhecido nem construir sobre ele achando que é comportamento correto. Dizemos que o `known-issues.md` é transiente porque a linha **sai** dele quando o problema é resolvido. O contrato dos dois vive no [`llm-dev-memory.md`](llm-dev-memory.md); o processo que os move em [`llm-dev-flow.md`](llm-dev-flow.md) e chama-se "Manutenção".

Uma observação interessante que o `docs/fixes/` e o  `known-issues.md` não existe para fazer o controle "administrativo" dos bugs do solução, há muitas soluções no mercado para isto, e se seu sistema é grande você provavelmente precisa delas.

E o **Tier 0** (o arquivo-raiz — por exemplo, no Claude Code é o `CLAUDE.md`) deve nascer **magro**: idioma, os comandos essenciais (build/test), as proibições, e **ponteiros** em **duas direções** — para o conhecimento do **projeto** (via `wiki/index.md`) e para a **trilogia do método** vendorizada em `.keelson/` (o fluxo — nova funcionalidade *e* correção de bug —, o jogador e o tabuleiro; por path relativo). Nada de conhecimento
profundo. Esse segundo roteamento é de propósito: o agente frio não sabe, por default, *como se trabalha aqui*; o Tier 0 lhe diz onde o método mora (o contrato está em [`llm-dev-memory.md`](llm-dev-memory.md), "O que faz um bom Tier 0"). Esses ponteiros — junto com o mapa dos contratos Tier-1 *always-on* (`glossary.md`, `invariants.md`, `known-issues.md`, `now/`) — formam um **preâmbulo de navegação** no topo/cabeçalho; do
preâmbulo para baixo, tudo é específico do projeto (abrindo pelas proibições turn-1). O `known-issues.md` está nesse mapa de propósito: o agente frio não pode construir sobre um bug conhecido.

> **O ponto que mais confunde: a wiki nasce quase vazia — de propósito.** Ela não é um formulário a preencher hoje; é um organismo que **se enche à medida que você trabalha**. Se você ceder à tentação de "documentar tudo agora", recriou na primeira hora o monólito que o método existe para evitar. O primeiro retorno aparece **amanhã de manhã**, na primeira sessão que abre lendo o `now/` e sabe em dez segundos onde parou. É esse retorno rápido que compra a disciplina para o resto.

## Depois do bootstrap: os próximos degraus (por demanda, não por mutirão)

Não construa o resto de uma vez. Cada peça entra quando o projeto **pede**:

- **Os dois contratos** (`glossary.md`, `invariants.md`) crescem dos primeiros mal-entendidos reais — o termo  que o agente já usou torto, a regra cuja violação silenciosa causaria incidente. Não invente verbetes por completude.
- **O primeiro ADR** (`docs/decisions/`) se escreve na primeira vez em que alguém — você ou o agente — quase  reabre uma decisão já fechada. A dor recente garante um porquê que morde.
- **O fluxo** (`brief → plan → tasks`) estreia na primeira funcionalidade nova de porte. Não o retrofite em trabalho já em andamento; espere a próxima que mereça uma especificação. O contrato completo do fluxo está em [`llm-dev-flow.md`](llm-dev-flow.md).

> **Dois gotchas do greenfield.**
> 1. **Não inche o Tier 0.** Quando algo importante nascer, o reflexo "documenta no arquivo-raiz pra não perder" é o **errado** — o certo é uma linha de ponteiro para onde a coisa mora. Duplicar no arquivo sempre-carregado é a pior duplicação de todas. O critério de o que entra no Tier 0 está em  [`llm-dev-memory.md`](llm-dev-memory.md), seção "O que faz um bom Tier 0".
> 2. **Não invente arquitetura nem decisões** que ainda não existem. Greenfield cria estrutura*, não *conteúdo fictício*. Se não há decisão tomada, `decisions/` fica vazia — e tudo bem.

---

# 4. Caminho B — Projeto que já roda (migração)

Você já tem um projeto vivo: código, documentação espalhada, provavelmente um Tier 0 que já virou um paredão. O objetivo é **trazê-lo para o método sem quebrar o que funciona**. Aqui o trabalho não é criar — é **classificar** o que já existe. Este é o resumo operacional; o playbook completo, com a razão de cada passo, está em [`llm-dev-migration.md`](llm-dev-migration.md).

> **A regra-mãe: incremental, não big-bang.** O modo de falha nº 1 é abrir o `architecture.md` de mil linhas e tentar reclassificar tudo numa sessão — isso **trava a migração antes de começar**. Migração é um **gradiente, não um evento**: extraia agora só os ganhos óbvios, deixe o resto onde está, e garanta que todo conteúdo novo já nasça no lugar certo.

## Passo a passo

1. **Rede de segurança — ANTES de tocar em qualquer coisa.**
   - **Trabalhe numa branch.** Mover arquivo é reversível via git; a migração inteira fica *candidata* até o  merge.
   - **Capture o baseline de métricas ANTES de migrar** — um snapshot (`wiki/metrics/AAAA-MM-DD-metric.md`) que reflita o estado **pré-método**. É contra ele que você vai medir se a migração melhorou custo/continuidade.
   > **Atenção:** baseline coletado *depois* de já ter mexido não mede nada. Este passo é irreversível no tempo — se você pular, perdeu a chance de saber se o método ajudou.

2. **Rode o prompt de migração.** Abra o [`llm-dev-prompt-migration.md`](llm-dev-prompt-migration.md), ajuste os `<placeholders>` e cole na sessão. Ele apenas *dispara e guarda* o playbook — quem manda é o`llm-dev-migration.md`, e ele foi feito para **parar nos pontos de checkpoint** e esperar você.

3. **CHECKPOINT de classificação — o passo que é seu.** O agente varre os docs existentes e te apresenta uma tabela **"artefato → balde do método"**. **Não deixe ele mover nada ainda.** Confira a tabela: a classificação exige julgamento humano (esta seção é decisão ou estado atual? este doc é fonte que o código carrega em runtime?). Guia de triagem:

   | Artefato existente | Sinal | Vai para |
   |---|---|---|
   | Monólito de arquitetura (estado + história misturados) | edita in-place **e** acumula narrativa datada | separar: estado atual → `docs/architecture.md`; decisões → ADR; incidentes → `wiki/log/` |
   | "Decidimos X porque Y", citável | um porquê que o time **possui** | `docs/decisions/NNNN-*.md` (ADR) |
   | Backlog / lista de tarefas | o que ainda vai ser feito | `docs/specs/` (+ `backlog.md`) |
   | Lista de bugs conhecidos / *known issues* / limitações **em tratamento** | o que está quebrado agora (a lição durável vai para ADR/invariants/log) | `wiki/known-issues.md` |
   | Postmortem / registro de incidente solto | o que aconteceu (o quê) **e** a lição (o porquê) | incidente → `wiki/log/`; lição → ADR |
   | Roteiro operacional (deploy, rotação de chave, restart) | procedimento imperativo ancorado numa **operação** | `docs/runbooks/` |
   | Relatório externo / auditoria não digerida | fonte primária ainda crua | `docs/reports/` |
   | Regulamento / norma / legislação externa | autoridade **de fora** do projeto | `docs/domain/` (fonte crua em `source/`) |
   | Doc que o **código carrega em runtime** (base RAG, template de prompt, schema) | há um `require`/path apontando pra ele | **NÃO mova** — indexe no `wiki/index.md` onde está |
   | Notas ad-hoc já existentes (ex.: memória do agente) | conhecimento solto | migra/referencia — **não recria do zero** |

4. **Depois de confirmar: crie o esqueleto e enxugue o Tier 0.** Monte as pastas `wiki/` e `docs/` (como no §3). Agora o enxugue do Tier 0 — e aqui vai o aviso que poupa o pior erro: um arquivo-raiz já-inchado **não se poda a olho, se migra**. Trate como uma mini-migração, **cobertura antes do corte**: para cada fato no Tier 0, grep o destino que já o cobre (o `index.md`, um `docs/` qualquer); **o que ninguém cobre migra primeiro**, e só depois disso você corta. Cortar antes de conferir a cobertura é perder justo o fato cuja falta ninguém percebe na hora. O passo-a-passo desse funil está na **Fase A** do [`llm-dev-migration.md`](llm-dev-migration.md). No fim, o Tier 0 fica uma semente: ponteiros + proibições.

5. **Colha só os ADRs óbvios.** Extraia do monólito as 2–3 decisões já claras para `docs/decisions/`. **Deixe o resto do arquivo como está** — não é hora de reescrevê-lo.

6. **Mova o barato e seguro — mas grep antes.** `git mv` dos arquivos claramente mal-colocados (backlog, reports). **Antes de cada move, grep pelo nome do arquivo:** nada aponta para ele por um path de código?
   `git mv` preserva a história. As **fontes-que-são-runtime** (o código as carrega) **não migram** — ganham uma entrada no `index.md` marcando o acoplamento, e ficam onde estão.

7. **Pare.** O resto do monólito seca **aos poucos, por uso**: cada vez que uma seção do arquivo velho for tocada por outro motivo, ela vai para o lugar certo. Nunca uma força-tarefa de reescrita.

## O que NÃO fazer

- Não reescrever o `architecture.md` inteiro de uma vez (trava tudo — a regra-mãe existe por isso).
- Não mover fonte-que-é-runtime "por estética de organização" — quebra o caminho que o código espera.
- Não renomear/mover sem **grepar as referências de entrada** primeiro.
- Não coletar o baseline **depois** de já ter migrado.
- Não tratar a migração como projeto com data de fim — ela termina quando o monólito secou, e isso é gradual.

---

# 5. O ciclo de uma feature — do brief ao campo, conduzido por skills

Instalado o esqueleto (§3 ou §4), começa o trabalho de verdade: **construir features**. Esta seção é o motor do dia a dia — como uma feature viaja da ideia à produção, e como as **skills do método** a carregam por cada fronteira do caminho. É a parte mais operacional do guia: é aqui que o método deixa de ser estrutura de pastas e vira **ritmo de trabalho**.

Como em todo o resto: **aponta, não re-explica.** O contrato completo do fluxo — os dois eixos de estado, a escada de evidência, os guardrails por transição — vive no [`llm-dev-flow.md`](llm-dev-flow.md) (o **Apêndice B** aponta para lá). Esta seção ensina a **operação**: qual skill, quando, o que ela faz por você e — o mais importante — **onde ela para**.

## O fluxo é uma cadeia de estados; cada skill carrega uma fronteira

Uma feature não salta da ideia ao ar. Ela **sobe uma escada de estados**, um degrau de cada vez:

```
brief → plan → tasks → código → BUILT → campo → PILOT/PROD
```

Cada uma das cinco skills do fluxo é responsável por **exatamente uma fronteira** dessa cadeia — e cada uma **para na soleira** da fronteira seguinte. Nenhuma vira a chave de estado sozinha: **quem promove é você.** A skill descobre o contexto, propõe, executa até o limite seguro e **te devolve a decisão**. Esse é o método em movimento — o agente é executor forte, mas o *veredito* de avançar é do humano.

> **Duas coisas que toda skill do fluxo tem em comum — e valem mais que qualquer detalhe individual:**
>
> 1. **Um portão humano (PARE).** Antes de escrever qualquer coisa, a skill faz uma *auto-descoberta* — varre o workspace para achar a feature, a fase-alvo, o estado — e então **para** e te mostra o que achou, esperando sua **confirmação**. É o antídoto do "confiante-mas-errado" embutido no primeiro passo: ela nunca age sobre um palpite silencioso, e nunca inventa um dado que falta (se falta, pergunta).
> 2. **A chave é sua.** Os estados que marcam progresso real — `VALIDATED`, `BUILT`, `PILOT`, `PROD` — **só o humano vira**. A skill leva até a *soleira* (o plano rascunhado, o código na trave, os testes verdes) e faz o *handoff* ali. Ela nunca se auto-promove — de propósito.

## A cadeia, skill a skill

| Fronteira do fluxo | Skill | O que ela faz por você | Onde para (você decide) |
|---|---|---|---|
| `brief` VALIDATED → `plan` | [`keelson-plan-init`](skills/keelson-plan-init/) | escreve o `plan-<slug>.md`: fases com **gates objetivos**, referenciando o brief por §, sem detalhar tasks | plano nasce `DRAFT`; **você valida** |
| `plan` → `tasks-fase<N>` | [`keelson-phase-landing`](skills/keelson-phase-landing/) | a **aterrissagem**: reconcilia requisito a requisito o que o brief pede contra o **código que já existe**, e abre o `tasks` por uma tabela de rastreabilidade | `tasks` pronto; **colisões esperam sua decisão** |
| `tasks` NOT_BUILT → soleira de `BUILT` | [`keelson-phase-coding`](skills/keelson-phase-coding/) | implementa **task a task**: red-first no que tem raio, mecânica antes de semântica, teste-de-costura nas fronteiras | soleira (testes verdes); **você não promove** |
| revisão antes de `→ BUILT` | [`keelson-phase-review`](skills/keelson-phase-review/) | **revisão independente, em sessão fresca**: confere o diff contra a camada congelada e a **qualidade dos testes**; pode mandar de volta | veredito; **você vira a chave de `BUILT`** |
| campo → `PILOT`/`PROD` | [`keelson-field-validation`](skills/keelson-field-validation/) | orienta a **validação de alta fidelidade** contra o ambiente real; registra evidência **observada ao vivo** | recomenda promoção; **você promove** |

As quatro primeiras linhas são a espinha do trabalho de fase; a quinta é o "quarto modo" (mais abaixo). Vamos por partes.

## Onde a cadeia começa — o brief (e o que as skills NÃO fazem)

A cadeia de *skills* começa obrigatoriamente em um **brief `VALIDATED`**. Esse ponto de partida é deliberado: **nenhuma \*skill\* escreverá ou validará o brief por você.**

Escrever a especificação e fechá-la (`DRAFT → VALIDATED`) é um trabalho estritamente seu em conjunto com o agente, sob o contrato do  [`llm-dev-flow.md`](llm-dev-flow.md). O motivo é simples: é exatamente aí que mora a definição do que a sua solução (ou uma nova *feature*) deve fazer.

**Isto não se automatiza:**
- O que a *feature* realmente é.
- O que ela promete entregar.
- Onde ela pode falhar.

Este é o trabalho principal do humano. É você quem conhece o usuário final, compreende as dores dele e consegue traduzi-las em requisitos de negócio. Esses requisitos são a fundação do sistema que está sendo construído. Em uma frase **A intenção da solução é sempre sua exclusiva responsabilidade!**

A escrita e revisão do brief é **humana** (o guardrail da linha `brief →VALIDATED` no flow). Só **depois** que o brief fecha é que a `keelson-plan-init` tem chão para pisar.

Guarde a regra: **as skills conduzem do plano para baixo; o brief é seu.**

## O passo a passo de uma fase

Com um brief `VALIDATED` na mão, uma feature caminha assim — e você invoca **uma skill por vez**, na sua própria sessão:

1. **Lance o plano** — invoque a `keelson-plan-init`. Ela acha o brief `VALIDATED` que ainda não tem plano, confirma com você e escreve o `plan-<slug>.md`: as **fases**, cada uma com um **gate de passagem objetivo** (critério de *evidência*, não de data — "o endpoint responde e passa nos testes de contrato"), e as dependências. Ela **não detalha tasks** — isso seria congelar decisão antes de ter chão de código. O plano nasce `DRAFT`; **você o revisa e valida** (cutuque os gates frágeis, as fases grandes demais).

2. **Aterrisse a fase** — invoque a `keelson-phase-landing` para a próxima fase cujo gate anterior fechou. Aqui está o passo que mais distingue o método de "cuspir código": **aterrissar não é transcrever o brief.** A skill lê o **código real** dos módulos que a fase toca e reconcilia, requisito a requisito, três desfechos possíveis — *já-existe* (a task vira *testar o que há*, não reconstruir), *lacuna-de-HOW* (o COMO se inventa no `tasks`, ancorado no § do brief) e **colisão** (o requisito bate contra algo que já existe — a skill **para e traz a decisão a você**; nunca edita o brief em silêncio). O resultado é o `tasks-fase<N>` aberto por uma **tabela de rastreabilidade** (cada task → § do brief).

3. **Codifique a fase** — invoque a `keelson-phase-coding`. Ela leva o `tasks` de `NOT_BUILT` até a **soleira** de `BUILT`, task a task, respeitando a classificação da aterrissagem. A verificação é **proporcional ao raio de explosão**: red-first nas tasks de alto raio (escreve o teste, confirma que ele **falha**, só então implementa — mata o teste tautológico); mecânica antes de semântica (testes + lint/types + greps de invariante antes de qualquer leitura); e **um teste-de-costura por fronteira que sustenta peso** (o contraparte real in-suite, não só o mock — porque "um dublê escrito pela mesma mão concorda consigo por construção"). Ela para na soleira: código escrito, testes verdes. **Não vira a chave de `BUILT`.**

4. **Revise — em outra sessão.** Este é o guardrail nº 1 do método, e ele tem uma regra que **não é opcional**: a `keelson-phase-review` roda numa **sessão separada da que codou**. Por quê? Porque o modo de falha do agente não é preguiça, é **confiante-mas-errado**: testes verdes provam só o que o autor pensou em testar, e ele escreveu o código *e* os testes — que podem partilhar o mesmo mal-entendido. A independência vem do **contexto fresco**: quem revisa ≠ quem escreveu. A skill confere o diff contra a **camada congelada** (os § do brief via a tabela de aterrissagem, os ADRs/invariants, o glossário — ela **não inventa critério**) e faz o cheque de maior valor: a **qualidade dos testes** (eles importam a lógica real ou a reimplementam? são tautológicos? onde teste e código partilham a mesma suposição?). E ela tem **dentes**: pode mandar a fase **de volta** para coding (revisão sem poder de devolver é *teatro*). Passou? **Você vira a chave de `BUILT`.**

> **Isto não é cerimônia — é evidência de campo.** Na Fase 2 do OptiFlux (um sistema real, medido), a revisão independente pegou **dois bugs reais que 275 testes verdes não pegaram** — um deles derrubava o processo inteiro —, reproduzidos num teste que falha e **devolvidos para correção** antes de a fase ser promovida. É esse retorno medido que compra o custo de uma sessão a mais. (A mesma medição também achou o **limite** da revisão de código: ela não pegou um bug de fronteira HTTP que só existia no mock — o que nos leva ao próximo ponto.)

## O quarto modo — validação de campo

Coding e revisão são poderosos, mas partilham um ponto cego: **ambos operam sobre dublês.** Um mock de um serviço externo, um stub do sistema de arquivos, um fixture do banco — nenhum *discorda* de você, porque foi você (ou o agente) que os escreveu. Nenhuma revisão de código, por mais independente que seja a *sessão*, descobre que **falta um secret no host**, que um restart precisa de `--force-recreate`, ou que o formato real que o serviço lá fora devolve não é o que o código assumiu. Isso é outra independência — a de **camada** (a coisa real vs. o dublê), não a de sessão.

A `keelson-field-validation` orienta esse **quarto modo**: exercitar a **coisa real** (stage/produção). Ela é honesta sobre a própria natureza — validação de alta fidelidade é **irredutivelmente cara e parcialmente fora do alcance do método**: precisa de infra real, acesso ao ambiente, e às vezes não é reproduzível. A skill **não executa** deploy nem promove nada; ela **disciplina o que já é manual e arriscado**: consome a lista `field-validation-required` que a revisão produziu, ordena as checagens **da mais segura à mais arriscada** (leitura → não-destrutivo → destrutivo/restart), exige um **pré-voo** (reverificar a premissa por ferramenta antes de cada comando de raio — "já li" ≠ "está verdadeiro agora"), e registra **evidência observada ao vivo** (não lida em log antigo, não simulada). No fim, recomenda a promoção a `PILOT`/`PROD` **com base na evidência de campo** — mas, como sempre, **você vira a chave** — e **nomeia a lacuna residual sem disfarce**: o que ficou coberto só pela suíte, nunca reproduzido em campo. É a honestidade que a promoção carrega à vista.

> **As duas independências, lado a lado.** A **revisão** (o passo 4 acima) te dá independência de *sessão* — outra cabeça, contexto fresco. A **validação de campo** te dá independência de *camada* — a coisa real, não o dublê. Uma não substitui a outra: a revisão pega o bug de lógica que o autor racionalizou; o campo pega o bug de costura que só aparece quando o contraparte real discorda. Um bug que dá para reproduzir **in-suite** (subindo o serviço real num teste) é **lacuna de coding — volta**, não campo; só o que **exige o host/produção real** é campo. O contrato dessa triagem está no [`llm-dev-flow.md`](llm-dev-flow.md) ("As duas independências").

## Duas regras que atravessam tudo

- **A chave de estado é sempre sua.** `VALIDATED`, `BUILT`, `PILOT`, `PROD` — nenhuma skill vira nenhuma delas. Elas param na soleira e fazem o handoff; **a promoção é um ato de julgamento humano**, lastreado no que a skill entregou. Se uma skill "passou de fase" sozinha, algo está errado.

- **As skills são `draft-para-testar` — instrumente antes de formalizar.** Elas automatizam uma disciplina que o [`llm-dev-flow.md`](llm-dev-flow.md) já validava à mão; a *automação* em si ainda é jovem. A regra do método vale para elas mesmas: registre no `wiki/log/` (ou num testemunho de fase) **o que cada skill pegou que os testes verdes não pegaram** — é esse rastro que decide se ela endurece ou é podada. Não confie na skill porque a teoria a acha elegante; confie no que o **campo** registrou.

O contrato completo — os guardrails por transição, a escada de evidência, o faseamento — está no **Apêndice B** ([`llm-dev-flow.md`](llm-dev-flow.md)). Esta seção foi só o **mapa de operação**: qual skill pega qual fronteira, e onde cada uma te devolve a chave.

---

# 6. Exemplos de campo — o método num sistema real

As skills do §5 não foram desenhadas numa mesa. Elas **emergiram de operar um sistema de verdade**, em produção — e cada uma ganhou a forma que tem porque o campo cobrou. Esta seção conta alguns momentos honestos desse uso real: não o que a teoria promete, mas o que aconteceu quando o método encontrou o mundo.

> **Uma nota antes de começar.** O sistema em questão é uma ferramenta de operações de trading — mas **os detalhes dele não importam aqui**, e de propósito você quase não vai encontrá-los. Troque-os mentalmente pelos do seu projeto. O que interessa é o *padrão*: onde o método pegou o que os testes não pegavam, onde parou para perguntar, e onde a chave só virou por decisão humana.

Os artefatos são reais. A cada feature, o fluxo deixa um rastro navegável — o `brief`, o `plan`, os `tasks` de cada fase — que cresce junto com o trabalho:

![O grafo dos artefatos de uma feature real: o index aponta o plan, que se liga ao brief e aos tasks de cada fase — a cadeia do §5, tornada visível.](images/grafo-brief-plan-tasks.png)

## A aterrissagem se paga antes da primeira linha de código

O primeiro instinto ao receber "escreva as tasks desta fase" é transcrever o plano em checkboxes. O método proíbe isso — e a `keelson-phase-landing` (§5) prova o porquê antes de qualquer código.

Numa das fases, ler o **código real** (não o brief) revelou três coisas que teriam virado bug se descobertas só na hora de codificar. A mais didática: o **critério de passagem da fase**, escrito pelo próprio operador, exigia um alerta visível na tela que a *lista de entregáveis da mesma fase* não mencionava — uma colisão entre dois pedaços do mesmo plano. O agente **não** suavizou o critério nem inventou o alerta por conta própria. Parou, levantou a tensão com três opções concretas, e o operador escolheu — com uma instrução de posicionamento que o agente jamais teria adivinhado. Uma task nasceu inteira dessa decisão, registrada como *"decisão do operador nesta sessão"*, não como escolha do agente disfarçada de requisito.

**O que isso mostra:** aterrissar é reconciliar o que se presumia contra o que é real. O ganho aparece numa passada de leitura — mais barato que descobrir como bug depois. E colisão vira **pergunta ao humano**, nunca escolha silenciosa.

## Coding: task a task, parando na soleira

A implementação segue o padrão da `keelson-phase-coding`: cada task ganha código e teste antes de a próxima começar, e a sessão **para na soleira de `BUILT`** — não vira a chave sozinha.

![Uma sessão de coding real marcando as tasks uma a uma; ao fechar, o registro foi "não deployado, não promovido a BUILT" — a regra dura da skill respeitada por construção.](images/coding-tasks-progresso.png)

Numa fase, o pacote de testes foi de zero a 275 verde. Isso prova que **o código** funciona — não que ele está pronto. A skill sabe da diferença: entrega a soleira e faz o handoff. Quem promove é o próximo guardrail — e, no fim, o humano.

## A revisão independente pega o que 275 testes verdes não pegam

Este é o exemplo central. A `keelson-phase-review` roda numa **sessão fresca** — outra aba, sem memória de quem escreveu o código:

![A sessão de revisão (comando /keelson-phase-review, em aba própria): mecânica primeiro, depois semântica, e o veredito VOLTA para coding com dois achados bloqueantes reabertos como tasks.](images/review-veredito-volta.png)

Naquela fase, os 275 testes estavam verdes e uma revisão independente ainda achou **dois bugs reais** — ambos *reproduzidos empiricamente antes de virarem task*, não hipóteses. O mais grave: duas rotas HTTP chamavam um serviço externo sem proteção; se ele piscasse no instante errado, uma única requisição **derrubava o processo inteiro** — levando junto o monitoramento de tudo o que estava rodando. O detalhe que crava o ponto: essa mesma proteção **já existia em três outros lugares** do código, escrita pelo mesmo autor — que a esqueceu em dois. Não era conceito novo escapando; era o ponto cego do próprio autor sobre o próprio trabalho.

O veredito foi **VOLTA** — devolução real, com os achados reabertos como novas tasks. Não "aprovado com ressalvas".

**O que isso mostra:** testes verdes provam só o que o autor pensou em testar. A sessão fresca é um árbitro que o autor não consegue racionalizar — e uma revisão que pode **mandar de volta** é a diferença entre guardrail e teatro. (Vale lembrar: na fase anterior, essa revisão **não** aconteceu — não por esquecimento, mas por *ritmo*, com o operador pedindo o próximo passo assim que o anterior fechava. Foi essa ausência, nomeada honestamente, que fez a skill nascer.)

## Sessão fresca ≠ camada real: o bug que só o campo pegou

Há um ponto cego que **nem a revisão independente** cobre — e a mesma feature o demonstrou. Depois de `BUILT`, a validação de campo (§5, o quarto modo) rodou a coisa real contra produção pela primeira vez. Uma tática real foi criada — e sumiu: uma consulta seguinte devolveu vazio.

A causa: um cabeçalho que **toda** chamada de mutação deveria enviar nunca era enviado — um dado nascia sempre nulo. E porque nascia nulo, **quatro** mecanismos de segurança distintos, construídos em quatro tasks diferentes, falhavam em silêncio ao mesmo tempo, todos dependendo — sem saber uns dos outros — do mesmo dado nunca preenchido.

O ponto que crava a lição: **nem os 275 testes verdes, nem os dois achados da revisão independente pegaram isso** — porque *ambos* operavam sobre o mesmo dublê (mock). Nenhum exercitava a fronteira real. Só rodar contra o serviço de verdade, em produção, achou.

**O que isso mostra:** são **duas independências diferentes**. A revisão te dá independência de *sessão* (outra cabeça); o campo te dá independência de *camada* (a coisa real, não o dublê). Uma pega o bug de lógica que o autor racionalizou; a outra, o bug de costura que só aparece quando o contraparte real discorda. Nenhuma substitui a outra.

## A chave é sempre do humano

Nas duas fases, o estado só mudou quando o operador disse. Depois da evidência de campo confirmada **ao vivo** (não relida em log), o agente *propôs* a promoção; a autorização explícita do operador — *"pode promover, com base nas evidências de campo"* — foi o evento que efetivamente virou `NOT_BUILT` em `BUILT`. O agente preparou a evidência; não tomou a decisão.

E a honestidade que a promoção carrega: registrou-se, sem disfarce, a **única lacuna** que ela levava junto — um cenário que só a suíte cobria, nunca reproduzido em campo. Promover é um ato de julgamento humano, com as lacunas à vista.

---

# 7. A ferramenta: o guia quente, o log frio de capturas e a skill de update

O núcleo do método é agnóstico; a sua ferramenta (Claude Code, Cursor, Codex…) tem mecanismos próprios. O *application guide* é a peça que liga os dois — e é a peça que muda quando (e só quando) a ferramenta muda.

Esta seção é sobre mantê-la em dia **com o mínimo de trabalho manual e de erro possível**.

## O que é o guia da ferramenta

Para o Claude Code, por exemplo, é o [`llm-dev-claude.md`](llm-dev-claude.md). Leia-o uma vez: para cada necessidade do método (o Tier 0, apontar-não-re-explicar, automação, subagentes, skills…), ele diz **como o Claude Code entrega** aquilo e **onde ele não consegue** (os "buracos"). Não é um tutorial da ferramenta nem um resumo das funcionalidades dela — é um **mapeamento por papel**: as linhas são as necessidades do Método Keelson (estáveis), as células são os mecanismos da ferramenta (mudam). Ele é o lado **quente** — pequeno e atual; as evidências datadas que sustentam cada célula moram no lado **frio**, o log de capturas (logo abaixo).

> **As duas coordenadas de versão** (no frontmatter do guia) dizem contra o que ele foi escrito: `adapta:` = quais versões dos padrões ele implementa; `verificado_em:` = contra qual estado da ferramenta de codificação foi conferido. É isso que torna auditável a promessa "só mexemos no guia, o núcleo não se tocou".
>
> **A confiança é por linha, não um selo único do guia.** Cada linha do mapeamento carrega o próprio rótulo na coluna **Confiança** — `campo` (a metodologia roda esse mecanismo no dia a dia), `observável` (lastreado em `--help`/schema) ou `desk` (só a auto-descrição que o próprio LLM da ferramenta faz de si, ainda não confirmada em uso). Assim uma linha sólida não empresta credibilidade a uma frágil na mesma tabela: você lê a coluna e sabe exatamente onde pisar firme e onde desconfiar.

## O log frio de capturas — e por que você nunca edita o guia à mão

A verdade do guia é congelada **pelo fabricante**, não por você: quando a Anthropic muda o Claude Code, o guia expira — e quem percebe isso é **você**. Olhando a versão da ferramenta, o humano constata que houve uma mudança; **as sessões e os agentes não têm como saber que passaram a rodar sobre uma versão nova.** Só o humano fecha esse laço — é por isso que o gatilho da atualização é seu (§8).

O plano, então: o guia **quente** anda leve — ele apenas aponta — e as evidências datadas ficam num arquivo separado, o **log frio** `llm-dev-<ferramenta>.log.md` (§2.2). É o mesmo par quente/frio da memória viva: o índice atual de um lado, o histórico *append-only* do outro. Empilhar captura dentro do próprio guia recriaria, ali mesmo, o monólito que o método existe para evitar.

Você **nunca edita à mão** — nem o guia (o *application guide*, p. ex. `llm-dev-claude.md`) nem o log (`llm-dev-claude.log.md`). É a skill (a seguir) que re-deriva as *células* e anexa a captura ao log. Editar qualquer um dos dois na caneta pula o registro da evidência: o *binding* passa a afirmar coisas sobre a ferramenta sem lastro, com a autoridade de um documento "atual". A regra é curta: **soube que a ferramenta mudou (update/upgrade) → rode a skill.** Nunca a mão.

## Como rodar a skill

No Claude Code, com a skill instalada (§2.4), invoque a [`keelson-application-guides-update`](skills/keelson-application-guides-update/) apontando a ferramenta que mudou. Você não prepara nada antes — ela mesma colhe a evidência. No **caminho barato** (a rotina), ela:

1. **Pergunta à ferramenta** como ela entrega cada papel do método (a auto-descrição), incluindo a pergunta aberta "o que mudou que eu não perguntei?" — para pegar também o que você não pensou em conferir.
2. **Ancora no observável:** captura `claude --help`, os subcomandos, o schema de `settings.json`, o changelog do CLI. É a metade *de agora* — não envelhece com o modelo.
3. **Reconcilia:** onde o observável contradiz a auto-descrição, **o observável ganha — na superfície dele** (nomes de flag/comando/campo); a metade semântica que ninguém observou fica marcada `desk`.
4. **Atualiza só as células** (nunca as *linhas*), grava a **confiança de cada uma**, **anexa a captura datada** ao log frio e dá bump de versão. O guia sai daí como **candidato** — ainda não abençoado.

Esse candidato ainda passa por dois momentos bem diferentes — e fáceis de confundir:

> **1. Teste de fora — "será que funciona mesmo?"** *(logo depois do update, para fechar)* A skill acabou de reescrever o guia, mas ninguém *experimentou* nada ainda. Então a própria skill abre uma **sessão nova e limpa** e põe o mecanismo pra rodar de verdade: o Tier 0 **carrega** mesmo? a checagem **dispara** mesmo?
>
> Por que uma sessão nova, e não a mesma que escreveu o guia? Porque a mesma sessão só releria o próprio texto e concordaria consigo — para ter certeza, é preciso ver a **ferramenta agir**.
>
> **Resultado:** funcionou → a skill carimba `verificado_em:` e o candidato vira oficial. Não funcionou → vai para a escalada (o caso 2).
>
> *Só vale a pena rodar o teste de fora quando o update mexeu em algo que importa; por exemplo, renome cosmético de flag dispensa.*

> **2. Escalada — "quebrou; por quê?"** *(a qualquer momento, quando algo falha)*
>
> O teste de fora pode *chamar* a escalada (quando falha). Mas a escalada também dispara sozinha se, mais tarde, em uso real, um comportamento quebrar.
>
> Aqui não é rotina. A escalada só entra se **algo deu errado de verdade**: o guia diz "faça X" e X não acontece,
> ou uma métrica sua mostrou que ele anda desalinhado. A skill então compara (**diff**) o comportamento de
> agora com a **última captura** no log frio, para responder uma pergunta só — *a ferramenta mudou* (a
> captura ficou velha) ou *a descrição já nasceu errada* (nunca bateu)? Achou a causa, corrige aquela célula.
> A skill te avisa quando é hora disso; você não precisa ficar de vigília.

**Em uma frase:** o **teste de fora** confere *antes de confiar* (planejado, ao fechar um update); a **escalada** investiga *depois que algo quebra* (reativo, só sob evidência) — inclusive quando o próprio teste de fora não fechou.

---

# 8. A disciplina que faz o método grudar

Aqui está a parte que nenhum arquivo faz por você. O agente é um **executor forte** — constrói a memória, rascunha o log, move os arquivos. Mas ele **não se vigia entre sessões**: só enxerga o contexto quando é invocado, e o mesmo passe que "documenta pra não perder" é o que incha o Tier 0. A vigilância é **sua**, e ela é irredutível a duas coisas: **intenção** (os briefs, o que entra) e **gatilho** (disparar as checagens). Se você não fizer o que segue, a memória apodrece — devagar, e com autoridade.

## As rotinas que sustentam a memória viva

- **Mantenha o Tier 0 magro.** Toda linha nova passa pelo teste de inclusão: *o agente erraria por default  sem isto, de um jeito que não recuperaria lendo o índice na hora?* Se não, é ponteiro, não conteúdo. Vigie o **churn**: se o arquivo-raiz muda toda semana, há conhecimento (que muda) vazando para dentro dele.  O contrato completo está em [`llm-dev-memory.md`](llm-dev-memory.md), "O que faz um bom Tier 0". Dois erros que aparecem toda hora, e são fáceis de cometer sem perceber:
    - *Nada de mini-índice.* Uma tabela de "arquivos-chave" dentro do Tier 0 **parece** roteamento — mas é   biblioteca disfarçada de rota: ela duplica o `index.md` e cresce junto com o código, exatamente o que o Tier 0 não pode fazer. O Tier 0 **aponta** para o índice; não reimplementa um índice dentro de si.
    - *Comando-do-dia fica; manobra sai.* O comando **atômico** que se roda no dia a dia (`npm test`, `npm run build`) mora no Tier 0. Já o **roteiro multi-passo** (o deploy, a rotação de chave) vira um arquivo em `runbooks/`, e o Tier 0 só aponta para ele — senão a manobra incha o arquivo-raiz por não ter um lar  melhor.
- **Pode.** O documento de presente (`architecture.md`, o `now/`) deve *encolher* tanto quanto cresce. Apagar um parágrafo obsoleto é **seguro** quando a história dele já vive no `log/` e a decisão que o originou já vive num ADR — é para isso que os baldes existem. Memória que só cresce é o monólito voltando. O `known-issues.md` é o caso extremo dessa regra: é **transiente por contrato** — cada linha sai no instante em que o bug conserta (ou vira limitação assumida, registrada num ADR). Um `known-issues.md` que só cresce é sinal de que a poda parou. A disciplina de correção em si — o teste-*red* como piso do bug — é do fluxo ([`llm-dev-flow.md`](llm-dev-flow.md), "Manutenção").
- **Promova por evidência de funcionamento, não por vontade.** Conteúdo sobe de camada (Tier 2 → índice → contrato) e congela  **quando a evidência empurra** — um contrato que o código provou, uma decisão que ninguém mais reabre. As regras de promoção estão em [`llm-dev-memory.md`](llm-dev-memory.md); a régua de evidência, em  [`llm-dev-flow.md`](llm-dev-flow.md).
- **Cure as fontes — inclusive as de fora.** Se o projeto depende de fonte externa (regulamento, norma), é  você que decide qual entra, qual versão vale e quando reconferir se a autoridade lá fora mudou. O agente não faz *sourcing*. Ver a vigília da Estrutura em [`llm-dev-player.md`](llm-dev-player.md).
- **Dispare as checagens.** O agente só é vigilante **se instrumentado**: um lint semântico, um painel de  saúde, o snapshot de métricas. Nada disso roda sozinho entre sessões — o gatilho é seu.

## O dia de jogo — o ritmo da prática

As rotinas acima são *o quê*; aqui está *quando*, no ritmo de um dia e de uma semana.

**Manhã.** O agente lê índice → `now/` e está produtivo numa leitura. Você faz duas coisas: (a) **confere o bilhete** — o próximo passo que anotou ontem ainda é o certo hoje? Só você sabe o que a sessão de ontem não sabia; (b) entrega o problema enquadrado, o contexto certo e o "pronto" definido. Minutos, não horas.

**Prompt a prompt.** Primeira regra: **apontar, não colar** — "siga o glossário", "respeite o invariante X", "a decisão NNNN já fechou isso"; ponteiro, nunca cópia. Segunda: o *como* é do agente — quem dita linha a linha paga para digitar com as mãos dos outros. O resto é olfato: **farejar deriva** pelos quatro cheiros —termo canônico usado torto; decisão congelada relitigada "como ideia nova"; confiança total onde devia haver
ressalva; superfície de incerteza suspeitamente vazia. Nenhum exige reler cada linha.

**Registro no ato** (mata a "famosa sexta-feira da documentação"): incidente → `wiki/log/`; decisão fechada → ADR; aprendizado que muda o objetivo → revisão versionada do brief; termo novo → `glossary.md`; bug conhecido e importante em tratamento → `known-issues.md`. Dez segundos por evento: *"isto merece registro? onde mora?"*.

**Fechamento — ritual de 5 minutos.** Atualize o `now/` (sobrescreva sem pena); deposite o dia no `log/`; confira a sobra (decisão de boca sem registro? aprendizado só no chat?). Fechar sem ritual é o pior hábito do iniciante: a sessão seguinte nasce órfã e o cold-start volta com juros — porque agora o `now/` mente com confiança.

**A semana.** Lint da memória; passeio pelo grafo (órfãos, aglomerados que se desconectaram); poda; olhada nos números do painel abaixo. Cadência proporcional ao raio de explosão do projeto. A postura é de **jardineiro, não de inspetor**: poda pouco, com frequência, e por isso nunca precisa de mutirão.

## O painel de saúde do praticante

Nenhum destes sinais precisa de ferramenta sofisticada; todos dizem se o método está vivo ou virando teatro:

| Sinal | O que observar | O que significa |
|---|---|---|
| **tempo de cold-start** | minutos até a primeira contribuição útil da manhã | o primeiro número a cair; é o que justifica tudo |
| **taxa de relitigação** | quantas vezes por semana uma decisão fechada foi reaberta | deve tender a zero conforme os ADRs cobrem o território |
| **idade do `now/`** | há quanto tempo não é atualizado | se descreve a semana passada, o ritual de fechamento quebrou |
| **emendas no `log/`** | raras, mas existentes | zero = ninguém confere o agente; muitas = qualidade de registro baixa |
| **onde as violações são pegas** | por grep, pelo revisor ou em produção | quanto mais à esquerda, melhor; o movimento para a esquerda é o método funcionando |

O papel do guardião que lê estes sinais — as três vigílias — está em [`llm-dev-player.md`](llm-dev-player.md).

> **O sinal honesto de fracasso — e o que fazer com ele.** Se manter a memória está consumindo mais do que devolve, você **subiu degraus demais para a sua escala**. A resposta não é abandonar o método; é **descer um degrau**, sem culpa. O método prevê o controle da própria dose — "a diferença entre o remédio e o veneno é, muitas vezes, a dose".

---

# 9. Referência rápida

**Qual arquivo para quê:**

| A sua situação | Rode / leia |
|---|---|
| Entender o conjunto | [`llm-dev-README.md`](llm-dev-README.md) · [`llm-dev-package.md`](llm-dev-package.md) |
| Instalar em projeto novo | [`llm-dev-prompt-bootstrap.md`](llm-dev-prompt-bootstrap.md) → **§3** |
| Migrar projeto existente | [`llm-dev-prompt-migration.md`](llm-dev-prompt-migration.md) + [`llm-dev-migration.md`](llm-dev-migration.md) → **§4** |
| Lançar o plano de uma feature (brief já validado) | [`keelson-plan-init`](skills/keelson-plan-init/) → **§5** |
| Aterrissar uma fase (reconciliar brief × código) | [`keelson-phase-landing`](skills/keelson-phase-landing/) → **§5** |
| Codificar uma fase já aterrissada | [`keelson-phase-coding`](skills/keelson-phase-coding/) → **§5** |
| Revisar uma fase antes de promover (sessão fresca) | [`keelson-phase-review`](skills/keelson-phase-review/) → **§5** |
| Validar em campo (stage/produção) | [`keelson-field-validation`](skills/keelson-field-validation/) → **§5** |
| Corrigir um bug de porte | `docs/fixes/` + [`llm-dev-flow.md`](llm-dev-flow.md) ("Manutenção") |
| Encaixar na sua ferramenta | [`llm-dev-claude.md`](llm-dev-claude.md) → **§7** |
| Atualizar o guia quando a ferramenta muda | [`keelson-application-guides-update`](skills/keelson-application-guides-update/) → **§7** |
| Manter tudo vivo | [`llm-dev-player.md`](llm-dev-player.md) → **§8** |

**Checklist de instalação:**

- [ ] Li o `README` e o `package`.
- [ ] Vendorizei o pacote do método em `.keelson/` na raiz do projeto (cópia pinada).
- [ ] *(Nada a pré-baixar.)* Sei que a evidência da ferramenta nasce sob demanda: a skill gera a captura no `.log.md` quando a ferramenta muda.
- [ ] Instalei **as skills** em `.claude/skills/` (as cinco do fluxo + a de manutenção do guia).
- [ ] Passei no teste "este projeto merece o método?" (§2.5).
- [ ] Rodei o prompt do meu caminho (bootstrap **ou** migração).
- [ ] *(migração)* Trabalhei numa branch e capturei o baseline **antes** de mover nada.
- [ ] O Tier 0 nasceu magro (ponteiros + proibições, não conhecimento).

**Glossário do conjunto:** ver a tabela no [`llm-dev-README.md`](llm-dev-README.md) (Keelson, application guide, referência do fabricante, Tier 0 seed).

---

# Apêndices — os três padrões do núcleo

> Estes apêndices **não repetem** aqui no Guia os padrões já escritos — eles apontam para o respectivo arquivo, com uma orientação curta de leitura. Assim o padrão pode evoluir sem que este guia precise ser revisado. Quando precisar do contrato completo de qualquer artefato, o endereço é o arquivo do padrão, não este guia.

## Apêndice A — O padrão da memória → [`llm-dev-memory.md`](llm-dev-memory.md)

**O tabuleiro: onde o conhecimento mora.** É a autoridade sobre: as três camadas ordenadas por custo de carregar (Tier 0/1/2); os dois contratos (`glossary.md` e `invariants.md`); os baldes de `docs/` (`architecture.md`, `decisions/` ADR, `specs/`, `fixes/` (correção), `reports/`, `domain/`, `runbooks/`, `general/`); a `wiki/` (`index.md`, `log/`, `now/`, `known-issues.md`); concorrência (candidato-na-branch/canônico-na-main); e a instrumentação (métricas).

- **Leia primeiro:** "O que faz um bom Tier 0" — é o que mais decide se a sua instalação vai prosperar ou  virar monólito.
- **Volte aqui quando:** precisar do formato exato de qualquer artefato, ou da regra de promoção/poda.

## Apêndice B — O padrão do fluxo → [`llm-dev-flow.md`](llm-dev-flow.md)

**O jogo: como o trabalho anda.** É a autoridade sobre: o trio `brief → plan → tasks`; os dois eixos de estado no frontmatter de cada spec; a escada de evidência (NOT_BUILT → BUILT → PILOT → produção) e o congelamento proporcional; a aterrissagem (pousar a spec sobre o código antes de gerar tarefas); os guardrails por transição de estado; e a **Manutenção** (a seta de volta operacional — o `fix-<slug>.md` no balde `docs/fixes/` e o teste-regressão-*red* como piso do bug).
- **Leia primeiro:** a tabela de guardrails por transição — o que é mecânico (grep, testes) e o que é  semântico (revisão) em cada portão.
- **Volte aqui quando:** for estrear o fluxo numa funcionalidade nova (o Degrau 4 do §3), conduzido pelas skills do **§5**.

## Apêndice C — O padrão do jogador → [`llm-dev-player.md`](llm-dev-player.md)

**O jogador: o papel do humano.** É a autoridade sobre: por que existe um padrão *para o humano* (os três fatos do agente + a assimetria de posição); as três vigílias (Intenção, Estrutura, Veredito); as três entregas de cada sessão + a regra de delegação; o dia de jogo; o painel de saúde; e a fronteira "curar as fontes inclui as de fora".
- **Leia primeiro:** as três vigílias — é o mapa do que você, como guardião, nunca delega.
- **Volte aqui quando:** estiver montando a sua rotina de curadoria, ou decidindo o que entregar ao agente a cada sessão.
