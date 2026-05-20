# Sessão 1 — Os 3 jeitos de usar o Claude

> Trilha pública · Sessão 1 de 3 · ~25 min de leitura
>
> ⏭️ **Próxima:** [Sessão 2 — GitHub como base de conhecimento](./02-github-and-knowledge.md)

## TL;DR

- O Claude tem **3 superfícies** (Chat, Cowork e Code) — mesma conta e modelo, mudam a interface e o que dá pra automatizar
- E **3 camadas de extensão** (CLI tools, MCP servers e Skills) que aumentam o que ele consegue fazer
- A grande sacada não é "Claude faz X" — é **você desenhar fluxos** que tiram fricção do que você já fazia
- Memória persistente faz o Claude lembrar de você entre conversas — sem cola mágica, é arquivo de texto

Esta sessão é mais **inspiração + conceito** que mão na massa. Mão na massa vem nas próximas duas.

---

## 1. Como uma prática pessoal com Claude se desenvolve

Vale começar com uma observação meta: ninguém domina Claude num dia. Ninguém domina Claude lendo a documentação inteira primeiro. **A prática se constrói em ciclos curtos** — você tenta, descobre uma fricção, ajusta, e amanhã faz melhor.

Um padrão típico de "do zero ao prático" em ~30 dias:

| Semana | O que costuma rolar |
|---|---|
| **1** | Conversas no Chat. Perguntas pontuais, brainstorm, "me ajuda a pensar" |
| **2** | Migra pra Cowork (Projects) buscando persistência. Sobe arquivos, organiza por projeto |
| **3** | Instala o Code. Começa a conectar Slack/Drive via MCP. Faz a primeira skill útil |
| **4** | Skill chama skill, agenda execução automática (loop ou cron), começa a versionar tudo num repo pessoal |

Esse arco não é prescrição — é o que **a maioria das pessoas descobre por conta**. A trilha aqui acelera esse aprendizado, mas o motor é seu uso real.

> **Frase-cola:** *não tente dominar tudo de uma vez. Use Chat hoje, descubra a fricção dele, e o próximo passo aparece.*

---

## 2. As 3 superfícies

| Superfície | O que é | Quando usar |
|---|---|---|
| **Chat** ([claude.ai](https://claude.ai)) | Conversa pontual, no navegador | Pergunta rápida, brainstorm, "me ajuda a pensar" |
| **Cowork** (Projects) | Espaço persistente com arquivos | Trabalhar 1 doc/projeto ao longo de dias, async |
| **Code** ([Claude Code](https://docs.claude.com/en/docs/claude-code)) | Controle total + automação local | Skills, ler repos, conectar ferramentas, rodar coisas |

> **Frase-cola:** *mesma conta, mesmo modelo — muda interface e o que dá pra fazer.*

### Diferenças que importam na prática

**Chat:** custa pouco token, é instantâneo, mas zera contexto a cada conversa. Use pra perguntas isoladas.

**Cowork (Projects):** sobe arquivos, persiste contexto. **Custo de token sobe muito** — porque ele relê os arquivos a cada interação. Bom pra "trabalhar nesse doc ao longo de uma semana", mas pra fluxos que repetem, fica caro.

**Code:** roda na sua máquina (CLI ou dentro do VS Code). Lê arquivos com inteligência (não joga tudo no contexto à toa). Executa comandos do terminal. Conecta serviços via MCP. **É onde a automação acontece.**

### Recomendação prática

- **Hoje** uso o Code dentro do VS Code (extensão Claude Code). Mesma experiência do terminal, integrada ao editor.
- **Chat** entra pra perguntas isoladas ou quando estou fora da máquina principal.
- **Cowork** entrou menos com o tempo — quase tudo que eu fazia lá, hoje faço melhor no Code com repo pessoal.

---

## 3. As 3 camadas de extensão

Em qualquer das 3 superfícies (mais o Code, na real), o Claude pode ser estendido por 3 mecanismos:

| Camada | O que é | Exemplo |
|---|---|---|
| **CLI tools** | Programa que já existe na sua máquina, o Claude **chama via terminal** | `git`, `gh`, `kubectl`, `bq`, scripts seus |
| **MCP servers** | Plug pra serviço externo via protocolo padrão | Slack, Jira, Notion, Drive, Calendar, GitHub |
| **Skills / slash commands** | Receitas suas que combinam tudo acima | `/weekly-recap`, `/promise-tracker`, `/dream` |

> **Frase-cola:** *MCP é nuvem. CLI é terminal. Skill junta os dois.*

### O que vale conectar primeiro

Pra perfil de PM/analista/líder, sugiro essa ordem:

1. **Slack** — onde decisões acontecem em texto
2. **Drive ou Notion** — onde docs vivem
3. **GitHub** — quando você quer começar a ler código
4. **Calendar** — pra agendamento e contexto de reuniões
5. **Jira/Linear** — pra rastreamento de trabalho do time

Cuidado com **Gmail**: ruído altíssimo. Só conecta se você for tratar email como dado de trabalho. Senão, ignora.

### Como conectar (5 segundos cada)

No Claude Code, digite `/mcp` e siga as instruções pra adicionar um MCP server. A maioria dos serviços tem autenticação OAuth — você clica num link, autoriza no navegador, pronto.

---

## 4. O conceito-chave — fluxo, não funcionalidade

A pergunta errada é: *"o que o Claude faz?"*

A pergunta certa é: *"que fluxo do meu dia tem fricção, e o Claude pode tirar?"*

### Exemplos do que vira fluxo

Pra um PM típico:

- **Resumo semanal** — varrer Slack, Jira, Grafana, compilar doc em formato padrão pra publicar no Notion
- **Caçador de promessas** — varrer DMs e canais atrás de coisas que você prometeu e ainda não cumpriu; atualizar um checklist seu
- **Onboarding em repo novo** — clonar repo, ler estrutura, pedir resumo dos sistemas + dos PRs recentes
- **Status de OKR** — atualizar progresso dos KRs cruzando dados de várias fontes

Pra um analista:

- **Investigação de bug** — partir do ticket, ler código relevante, propor hipóteses e queries
- **Painel pontual** — pegar uma query do Metabase, expandir histórico, virar alerta
- **Glossário do domínio** — varrer docs/código e produzir uma taxonomia que dura

### O que **não** é caso de fluxo

- Tarefas únicas que você faz uma vez por mês ou menos (custo de criar a skill > ganho)
- Coisas que demandam julgamento humano em cada execução (anti-padrão automatizar)
- Tarefas que dependem de informação que não está acessível ao Claude (preso em outro sistema, oral, etc.)

> **Frase-cola:** *Claude boa não faz coisa difícil. Tira fricção de coisa que você já faria — e se encaixa onde a decisão acontece.*

---

## 5. Skills — receitas que compõem tudo

Uma "skill" é só um arquivo `.md` com:

- **Frontmatter** (cabeçalho YAML) com nome, descrição e gatilhos
- **Corpo** com instruções pro Claude executar

### Exemplo mínimo

```markdown
---
name: weekly-recap
description: Use quando o usuário pedir "resumo semanal", "weekly recap",
  "o que rolou essa semana", ou rodar a rotina semanal de sexta.
---

Gere um resumo do que aconteceu essa semana:

1. Liste compromissos confirmados na agenda (segunda a sexta)
2. Liste threads do Slack em que eu participei nos últimos 5 dias
3. Liste PRs que eu abri ou revisei no GitHub
4. Compile um doc com:
   - **Highlights** (3-5 bullets do que mais importa)
   - **Pendências** (o que ficou em aberto)
   - **Próxima semana** (compromissos já agendados)
```

Salva isso em `~/.claude/skills/weekly-recap.md` e na próxima vez você digita `/weekly-recap` no Code, ela roda.

### 4 práticas que diferenciam skill amadora de skill útil

#### 1. Triggers redundantes em PT-BR

```markdown
description: Use quando o usuário pedir "weekly recap", "resumo semanal",
  "o que rolou essa semana", "review da semana", ou ao rodar como
  rotina automática de sexta-feira.
```

Vários jeitos de chamar a mesma skill. O Claude precisa **reconhecer a intenção** mesmo que você não lembre o nome exato.

#### 2. Argumento `dry-run` — padrão de ouro

```
- Se vazio: roda em modo normal
- Se contém "dry-run": executa tudo MAS não escreve no Slack/Notion/Jira — só imprime no terminal
```

Toda skill que **escreve** em algum lugar deveria ter um modo `dry-run`. Permite testar mudanças sem efeito colateral. Inestimável quando você está iterando no design da skill.

#### 3. Configuração externa — não hardcode

Em vez de escrever `meu_slack_id = "U03H8...".` direto na skill, a skill lê de um arquivo `config.md` próprio:

```markdown
## Fase 0 — Carregar config

Leia `config.md` e extraia:
- `owner.name`
- `slack.user_id`
- `notion.workspace_id`
```

Por quê: **outra pessoa pode adotar sua skill mudando 1 arquivo.** Skills viram portáveis. Se você publica suas skills num repo, gente pode forkar e usar imediatamente.

#### 4. Loop automático + interface natural

Skill que escreve direto num **Canvas no Slack** (ou doc no Notion, ou issue no Jira) — não cria nova mensagem cada vez, **sobrescreve** o canvas existente.

- O **dismissal acontece pelo canal natural**: marca como done no Slack/Notion, na próxima execução a skill ignora.
- **Você fecha o loop pelo canal natural, não volta no Claude.** Esse é o jeito limpo.

---

## 6. Memória persistente

O Claude Code mantém **memória local** entre sessões — arquivos `.md` que ele relê automaticamente quando você abre uma conversa.

### A pilha

| Onde mora | Pra quê |
|---|---|
| `~/.claude/CLAUDE.md` | Instruções globais — idioma, postura, regras gerais. Lido em **toda** conversa |
| `~/.claude/projects/<projeto>/memory/MEMORY.md` | Índice de memórias específicas daquele projeto |
| `~/.claude/projects/<projeto>/memory/<tema>.md` | Uma memória — fato, preferência, contexto |

> **Não é mágica.** É arquivo de texto. Você lê, edita, apaga.

### Tipos de memória que valem persistir

| Tipo | Exemplo |
|---|---|
| **user** | "Sou PM em e-commerce, foco em retenção" |
| **feedback** | "Não usar tabelas grandes em drafts no Slack" |
| **project** | "OKRs do quarter atual, status semanal" |
| **reference** | "IDs de canais Slack, dashboards Grafana que uso toda hora" |

### Skill que cuida da memória

Você pode escrever uma skill que **consolida memórias automaticamente** — varre o que está salvo, detecta duplicatas, marca o que ficou obsoleto, propõe atualizações.

Padrão: roda em modo `dry-run` primeiro (declarativo, só lista o que mudaria), você aprova, ela aplica.

```
## Dream — Consolidação de memórias

### Atualizadas (3)
- user_role.md — papel mudou de Senior pra Staff
- feedback_drafts.md — adiciona regra sobre tabelas grandes

### Criadas (1)
- project_q3_okrs.md — OKRs do quarter novo

### Removidas (2)
- project_q1_okrs.md — quarter encerrado
- feedback_old.md — superada por feedback_drafts.md
```

> **Frase-cola:** *memória é arquivo. Arquivo tem skill que cuida pra você.*

---

## 7. O que tem por baixo (e o que **não** tem)

Pra desmistificar:

| O que tem | O que **não** tem |
|---|---|
| Modelos de linguagem (Claude) que predizem texto bem | Capacidade de "querer" ou "ter intenção" |
| Acesso a ferramentas (terminal, MCP, web) que você configura | Acesso espontâneo a coisas que você não autorizou |
| Memória estruturada em arquivos `.md` que você controla | Memória secreta que aprende de você sem você saber |
| Capacidade de seguir instruções complexas e iterar | Capacidade de produzir trabalho que você não saberia avaliar |

Isso importa porque define **o que você delega com tranquilidade e o que você revisa**.

> **Frase-cola:** *delega o tedioso (varrer, compilar, formatar). Revisa o julgamento (priorizar, decidir, comunicar).*

---

## Pós-sessão — exercícios

Antes de seguir pra sessão 2:

1. **Instalar Claude Code** (CLI ou extensão VS Code)
2. **Criar `~/.claude/CLAUDE.md`** com 3 linhas sobre você:
    - Quem você é (papel + área)
    - Idioma preferido pra respostas
    - Uma regra que você quer que ele sempre siga (ex: "responda direto, sem 'ótima pergunta'")
3. **Conectar 1 MCP** que faça sentido pro seu dia (Slack, Drive ou Notion são bons pontos de partida)
4. **Fazer 1 pergunta real** ao Claude Code — algo que você normalmente faria no Chat. Veja a diferença

---

## Recursos

- **Claude Code (docs oficiais):** [docs.claude.com/en/docs/claude-code](https://docs.claude.com/en/docs/claude-code)
- **MCP — protocolo:** [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **Exemplo vivo de PM com Claude em produção:** [github.com/vitorrodriguesjus/testador_pessoal](https://github.com/vitorrodriguesjus/testador_pessoal) — skills, ciclo semanal, OKRs, memórias

---

Próxima sessão: **[GitHub como base de conhecimento](./02-github-and-knowledge.md)** — como usar repos pra investigação técnica sem ser dev, e como organizar memória entre sessões pro time inteiro.
