# Sessão 3 — Dados na prática: BigQuery, /plan e sub-agents

> Trilha pública · Sessão 3 de 3 · ~30 min de leitura
>
> ⏮️ **Anterior:** [Sessão 2 — GitHub como base de conhecimento](./02-github-and-knowledge.md)

## TL;DR

- **BigQuery via Claude Code** elimina o "pode rodar essa query rápida?" que custava 2 dias com o time de dados
- **6 práticas** pra fazer isso sem queimar dinheiro nem tirar conclusão errada: schema antes de query, dry-run sempre, partição como reflexo, memória estruturada, sanity-check com 10 linhas, output cita fonte
- **`/plan` é o `--dry_run` do Claude** — antes de executar, ele te mostra o que vai fazer. Use quando o custo (em $, tempo, ou efeito colateral) for relevante
- **Sub-agents** = Claudes pequenos rodando em paralelo, com contexto próprio. Use quando souber **em 1 frase** o que cada um deve devolver

A mensagem central: **pense antes de executar.** Antes de query cara: `--dry_run`. Antes de skill que mexe em arquivo: `/plan`. Antes de 3 sub-agents em paralelo: plano consolidado. Não é cerimônia — é o reflexo que separa quem queima fichas de quem entrega.

---

## 1. BigQuery + Claude — o mínimo

### O que muda quando o Claude tem acesso ao seu data warehouse

| Antes | Depois |
|---|---|
| Você pede pro time de dados rodar query, espera 2 dias | Pergunta direta, query roda no momento |
| Resultado vem como tabela estática, sem iteração | Você itera no mesmo prompt: *"agora quebra por dia"*, *"adiciona baseline"*, *"monta painel"* |
| Sem rastro do raciocínio, só o output final | Cada passo fica no histórico, dá pra revisitar |

### Como o Claude acessa o BQ

| Modo | Como funciona | Quando usar |
|---|---|---|
| **`bq` CLI local** | Claude roda `bq query` no seu terminal, com as suas credenciais | Default, é o que recomendo |
| **MCP de BigQuery** | Plug oficial via protocolo | Quando o time disponibilizar — varia por empresa |

> ***"Claude não substitui o time de dados. Substitui o ping de 'pode rodar essa query rápida pra mim?'"***

### Conectando o Claude no BQ (5 passos)

Setup custa ~5 min se tudo der certo:

1. **Acesso ao BigQuery** — peça via o canal do seu time de plataforma se ainda não tem. Acesso de leitura nos datasets que você usa basta
2. **Instalar `gcloud` SDK** — traz o `bq` CLI junto:
    - macOS: `brew install --cask google-cloud-sdk`
    - outros: [cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)
3. **Autenticar** — dois logins, um pra você e um pras aplicações (Claude usa o segundo):
    ```bash
    gcloud auth login
    gcloud auth application-default login
    ```
4. **Setar projeto default:**
    ```bash
    gcloud config set project <seu-projeto-bq>
    ```
5. **Validar:**
    ```bash
    bq query --use_legacy_sql=false --dry_run "SELECT 1"
    ```
    Se voltar `Query successfully validated. Assuming the tables are not modified, running this query will process 0 B`, tá pronto.

### Permitir o Claude rodar `bq` sem perguntar toda vez

Por default, o Claude Code pede permissão a cada comando shell. Pra `bq`, vale colocar na whitelist do `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Bash(bq:*)", "Bash(gcloud:*)"]
  }
}
```

**Edição via Claude:** *"abre `~/.claude/settings.json` e adiciona `bq:*` e `gcloud:*` na lista de allow."* Ele faz pra você.

### Travou?

- *"command not found: bq"* → `gcloud` não está no PATH. Reabra o terminal ou rode `source ~/.zshrc`
- *"Access Denied"* → falta permissão no projeto. Ticket no time de plataforma
- *"Project not specified"* → faltou o passo 4. Roda `gcloud config set project ...`

Pra qualquer outro erro, **manda direto pro Claude no Code** — ele resolve a maior parte sozinho lendo a mensagem.

---

## 2. Demo 1 · Análise de pipeline anômalo

> **Cenário:** você acompanha um pipeline de processamento (`example_pipeline_v2`) num dashboard do Metabase. Anomalia num dia específico — o tempo de processamento p90 saiu do baseline. Você quer descobrir o que aconteceu, e como detectar isso amanhã sem precisar entrar no Metabase todo dia.

### Setup

1. Metabase com a query original — você copia só o SQL
2. Cola no Claude Code, sem repo, sem contexto adicional

### Os 4 saltos da conversa

#### Salto 1 — comparação simples

> *"vc consegue rodar essa query e ver a situação de hoje vs ontem?"*

Claude pega a SQL, roda no BQ via `bq` CLI, volta com **p90 por etapa pro dia anômalo vs dia anterior**:

- Etapa C: **37 min vs 0,01 min** no dia anterior — anomalia confirmada
- Etapa D: **0,38 min vs 0,02 min** — anomalia secundária correlacionada

#### Salto 2 — expansão temporal

> *"compare desde o começo do mês passado. dia a dia. Quais dias tivemos problemas?"*

Claude estende pra **42 dias**, sem você precisar reescrever a query. Identifica **3 clusters de incidente + 1 recidiva**:

| Janela | Etapas afetadas | Duração |
|---|---|---|
| Cluster 1 | colapso D + E | 2 dias |
| Cluster 2 | degradação progressiva de C | 14 dias |
| Cluster 3 | E + spikes de F | 10 dias |
| **Recidiva isolada** | C explodindo de novo | 1 dia |

#### Salto 3 — regra de alerta

> *"que dias você geraria alerta? Faça antes uma análise simples. Se alguma etapa fugir muito do padrão, ou se o tempo total subir muito do baseline, gera o alerta."*

Claude simplifica pra **uma regra só**: `total > 5 min OU etapa > 10× baseline`. Baseline total ≈ 1,8 min. Lista **15 dias com alerta + 4 borderline**.

#### Salto 4 — quebra horária do incidente

> *"Testa se o dia da recidiva tem algum ponto de atenção dentro do dia"*

Claude quebra o dia por hora, encontra **dois eventos distintos**:

- **17h**: spike na etapa D (45 min p90)
- **20h–23h**: spike na etapa C (137 min p90 às 21h)

Conclui: *"D estourou antes de C — assinatura de cascata reversa. Backpressure do consumidor downstream batendo no intermediário primeiro, depois empurrando no produtor."* Sugere investigação: deploys naquela janela, logs do consumidor.

### A mensagem por trás do caso

Esse arco — `query → comparativo → quebra → regra` — vira **uma skill com routine**. Roda diariamente, alerta no Slack quando algum dia bate a regra. **Você passa de descobrir problema quando alguém posta thread, pra ser avisado antes da thread existir.**

> ***"O salto não é descobrir o problema. É ser avisado antes dele virar incidente compartilhado."***

---

## 3. Demo 2 · BigQuery + produto real

> **Cenário diferente da Demo 1.** Lá tínhamos uma query existente e construímos painel + alerta. Aqui partimos de um **problema concreto de produto**: você quer **monitorar menções a tópicos específicos em fontes externas** (notícias, comunicados, blogs setoriais) que o seu sistema já coleta. Histórico mostra que casos importantes chegaram tarde — você descobriu *via cliente reclamando*, não dentro de casa.

### A motivação

Não é problema novo: empresas vivem descobrindo mudanças relevantes (de fornecedores, de regulação, de concorrentes) pelo canal mais lento — o cliente. **A v0 vai pelo caminho oposto:** usa dados que já estão sendo coletados em casa e ensina o Claude a ler.

### Demo da v0

#### Passo 1 — schema antes de query

> *"antes de me ajudar a montar a busca, lê o schema da tabela de notícias coletadas e me explica os campos."*

Claude lê o schema, lista as colunas relevantes (`data_publicacao`, `texto`, `fonte`, `categoria`, `url`), estima volume.

#### Passo 2 — `/plan` antes de executar

> *"agora monta um plano de busca. Quero pegar publicações dos últimos 90 dias que mencionem mudança de fornecedor/sistema."*

Claude entra em **`/plan` mode**, mostra a query que vai rodar — filtros, partição por `data_publicacao`, termos de busca. Estima custo via `--dry_run` antes de executar. **Você aprova, ele roda.**

#### Passo 3 — sanity-check com amostra

> *"antes de tirar conclusão, me mostra 10 linhas brutas das matches. Quero ler o texto."*

Claude retorna 10 linhas. Você olha — **7 são relevantes, 3 são falso positivo** (menções em contextos não relacionados). **Refinam os termos juntos.** Esse momento — pegar 3 falsos positivos em 10 linhas antes de generalizar — é a **maior economia de tempo da demo**.

#### Passo 4 — output cita fonte

> *"agora gera o painel final. Lista os tópicos com sinais, número de matches, datas, e cita a SQL final + tabela + data de execução."*

Resultado: tabela com tópicos que aparecem repetidamente nas notícias, ordenados por intensidade do sinal. **Casos conhecidos do passado aparecem em retrospectiva** — validação de que o método funciona pra detectar o que importaria.

### Por que esse caso é estruturalmente diferente do anterior

| Demo 1 — análise de pipeline | Demo 2 — monitoramento de fontes |
|---|---|
| Query existia, precisava interpretar | Não existia query, precisava construir |
| Resultado = painel + alerta | Resultado = **produto v0** que entra no roadmap |
| 1 fonte (Metabase já tinha SQL) | 1 fonte hoje, mas escala pra 3–4 (sub-agents) |
| Iteração linear | Iteração com sanity-check no meio |

> ***"Aqui é onde o Claude vira diagnóstico estrutural, não 'me ajuda com essa query'. Você dá o problema, ele te entrega a v0 do produto."***

---

## 4. Boas práticas no BQ com Claude

Não são comandos. **São posturas.** Você vai esquecer flags em uma semana, mas se internalizar isso, sai daqui usando bem.

### As 6 práticas

#### 1. Schema antes de query

> *"Antes de me ajudar com a query, lê o schema da tabela X e me explica os campos."*

Evita query no chute. Diferença entre uma sessão produtiva e uma sessão batendo cabeça em coluna que não existe.

#### 2. Dry-run sempre antes do execute

> *"estima o custo dessa query antes de rodar"*

Claude roda `--dry_run`, reporta bytes/US$, você decide. **Sem isso, fácil rodar query de US$ 20 sem perceber.**

#### 3. Partição/cluster como reflexo

> *"essa tabela é particionada por dia. Filtra por `_PARTITIONDATE` antes do WHERE de negócio"*

Item de **maior impacto financeiro** da lista. Em tabelas grandes, reduz custo em **10–100×**.

Exemplo concreto: query da Demo 1 tem `WHERE day >= '2026-04-01'`. Sem isso, custaria muitas ordens de magnitude a mais.

#### 4. Memória estruturada do que funcionou

Schema raro que você descobriu, projeto/dataset que usa toda hora, query template que voltou um insight — vira `.md`:

- **Local:** `~/.claude/projects/.../memory/queries.md`
- **Nuvem:** `<seu-repo>/queries/`

Próxima conversa, Claude lembra o caminho. **Fecha loop com a Sessão 2:** memória local pra continuidade pessoal, memória nuvem pra continuidade do time.

#### 5. Sanity-check com 10 linhas antes de generalizar

Antes de tirar conclusão de uma agregação, pedir `LIMIT 10` da query bruta e ler com olho humano. **Foi o que pegou os 3 falso positivos na Demo 2 — antes que virassem "TJX está em migração".**

#### 6. Output cita fonte

Pedir pro Claude sempre incluir no relatório: **SQL final + dataset/tabela + data de execução + custo estimado**. Sem isso, painel é "ipse dixit" — não auditável, não reprodutível 3 semanas depois.

### Guarda-corpos (não-negociáveis)

- **Dado sensível** (PII, valores, partes nominais) — não jogar em prompt aberto sem necessidade. Se precisar amostra, pede pro Claude mascarar
- **Query > 100 GB no dry-run** — segura, refaz com partição
- **DDL/DML** (`DELETE`, `INSERT`, `CREATE TABLE`) — Claude pede confirmação visual; **nunca aprovar no automático**

### O princípio que organiza tudo — `/plan`

> ***"`/plan` é o `--dry_run` do Claude. Antes de executar, ele te mostra o que vai fazer."***

`/plan` aparece duas vezes nas demos de hoje porque é **a postura situacional** que conecta todas as boas práticas:

- Antes de query cara → `--dry_run`
- Antes de skill que mexe em arquivo → `/plan`
- Antes de 3 sub-agents → plano consolidado

Não use sempre. Use quando **o custo (em $, tempo, ou efeito colateral) for relevante**. Cargo cult em `/plan` é tão ruim quanto ausência dele.

---

## 5. Sub-agents — paralelismo com critério

### O que é, em uma frase

> Sub-agent é um Claude pequeno, dentro do seu Claude grande, com **contexto próprio**, **resultado próprio**, e que **roda em paralelo com outros**.

### Os 3 built-in que valem conhecer

| Agent | Pra que serve | Quando usar |
|---|---|---|
| **`Explore`** | Busca cross-repo / cross-arquivo. Lê excerpts | "Onde isso é usado? Onde está definido X?" |
| **`Plan`** | Arquitetura de implementação | Antes de tarefa não trivial — propõe plano, pede aprovação |
| **`general-purpose`** | Tarefas multi-step abertas | Pesquisa, busca complexa, análises que não cabem nos 2 acima |

### Sub-agent custom — `~/.claude/agents/`

Você cria um agente especializado pro seu domínio. Frontmatter + descrição + lista de tools. Exemplo:

```yaml
---
name: signal-detector
description: Use proativamente quando precisar identificar sinais
  de mudança em fontes textuais (notícias, comunicados, blogs).
  Cruza com dados internos pra dar confiança.
tools: Bash, Read, Grep, mcp__bigquery__query
---

Você é um detector de sinais de mudança em fontes externas.
Procure por: "mudança", "migração", "novo sistema", "transição".
Sempre cite fonte (URL, data, trecho).
```

Salva no diretório, escolhe no menu de agents do Claude Code. Pronto. **Custo: 30 segundos.**

### Demo de paralelismo aplicada ao caso da Demo 2

A v0 funciona com 1 busca. Pra capturar o sinal antes do cliente, você precisa varrer **3 fontes em paralelo**:

1. **Notícias internas** já coletadas (a Demo 2)
2. **Documentos oficiais** publicados por reguladores ou parceiros
3. **Métricas internas** que poderiam indicar a mudança antes (queda em coleta, mudança de schema da API)

**Sequencial = lenta. Paralelo, com sub-agents, escala sem virar pipeline de engenharia.**

Antes de disparar:

> *"Antes de eu mandar os 3 em paralelo, faço `/plan`. Quero saber o que cada um vai fazer. Sem isso é gastar fichas sem entender."*

Claude entra em plan mode. Mostra o que cada sub-agent vai rodar, qual tabela cada um vai ler, custo estimado. **Você aprova.**

3 agents rodam em paralelo. Cada um volta com sua tabela. Claude principal **junta os resultados**, identifica overlap (tópico que aparece nas 3 fontes = sinal forte), e produz painel consolidado.

### Adversarial breve — o uso mais alto leverage

> *"agora me dá uma crítica desse plano. O que pode dar errado? Que viés tem nos meus termos de busca?"*

Claude vira o jogo: aponta que palavra-X capitaliza mal em algumas fontes, que dados internos podem mudar também por outros motivos, que falta um sinal de **negativo** (notícia desmentindo mudança). **Você refina.**

**Pedir crítica antes de executar, não depois** é onde sub-agent rende mais pra PM/analista.

### Quando NÃO usar sub-agent

- Task simples (1 arquivo, 1 query, 1 leitura) — sub-agent vira overhead
- Quando você não consegue **escrever em 1 frase o que ele deve devolver** — significa que você ainda não pensou o suficiente
- Em loops sem critério de parada — sub-agent não para sozinho se você não definir condição

> ***"Sub-agent é amplificador. Amplifica clareza e amplifica confusão. Use quando souber o que pediu."***

---

## Pós-sessão — exercícios

1. **Rodar 1 query no BQ via Claude Code** — preferencialmente algo do seu próprio dia a dia (métrica de produto, exploração de tabela nova)
2. **Usar `/plan` pelo menos 2x** essa semana antes de query custosa ou ação que mexe em arquivo
3. **Criar 1 sub-agent custom** em `~/.claude/agents/` — pode ser simples (ex: revisor de draft, sanity-checker de query)
4. **Salvar 1 memória estruturada** — o schema que descobriu, o projeto/dataset que usa toda hora, ou a query template que funcionou. Local ou no repo do time

---

## Recursos

- **Sub-agents (oficial):** [docs.claude.com/en/docs/claude-code/sub-agents](https://docs.claude.com/en/docs/claude-code/sub-agents)
- **Plan mode:** [docs.claude.com/en/docs/claude-code/plan-mode](https://docs.claude.com/en/docs/claude-code/plan-mode)
- **BigQuery — controle de custo:** [cloud.google.com/bigquery/docs/best-practices-costs](https://cloud.google.com/bigquery/docs/best-practices-costs)
- **Exemplo de skill que orquestra BQ + Slack + GitHub:** [testador_pessoal/toolbox/skills](https://github.com/vitorrodriguesjus/testador_pessoal/tree/main/toolbox/skills)

---

> *Sessão 1 — "olha o que dá pra fazer."*
> *Sessão 2 — "olha como faz."*
> *Sessão 3 — "faça em paralelo, e pense antes de executar."*

Trilha completa. Bom uso.
