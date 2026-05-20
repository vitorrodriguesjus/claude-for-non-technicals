# Sessão 2 — GitHub como base de conhecimento + investigação

> Trilha pública · Sessão 2 de 3 · ~25 min de leitura
>
> ⏮️ **Anterior:** [Sessão 1 — Os 3 jeitos de usar o Claude](./01-getting-started.md) · ⏭️ **Próxima:** [Sessão 3 — BigQuery, /plan e sub-agents](./03-data-and-bigquery.md)

## TL;DR

- **GitHub é o filesystem da empresa** — quem sabe ler código sabe ler a empresa
- Pra investigação profunda: **clonar o repo localmente** e abrir Claude Code naquela pasta. Pra exploração rápida: MCP do GitHub
- **Investigação técnica via Claude** muda a postura: você **dirige perguntando**, não executando — e o Claude descobre coisas que você não tinha pensado em perguntar
- Memória entre sessões tem 3 camadas: **conversa atual** (RAM), **memória local** (no seu PC), **memória na nuvem** (repo). As três têm papéis distintos

A mensagem central: o que você descobre investigando vira `.md` num repo. Próxima vez que você ou alguém do time precisa, **a skill lê do GitHub**, não da sua memória.

---

## 1. GitHub em 4 minutos

### O básico sem mistério

| Conceito | Em uma frase |
|---|---|
| **Repositório (repo)** | Pasta com histórico de mudanças. Toda alteração fica registrada — quem fez, quando, por quê |
| **Branch** | Rascunho. Você muda coisas num "ramo" sem quebrar a versão oficial |
| **Pull Request (PR)** | Pedido de revisão antes de virar oficial. É onde o time discute o código |
| **Issue** | Ticket. Bug, ideia, pedido de feature |
| **Commit** | Uma mudança específica, com mensagem explicando |

### Pra que serve pra você (que não codifica)

- **Ler código** pra entender como uma feature funciona, sem depender de um dev te explicar
- **Acompanhar PRs** dos times — ver o que está mudando, não só o resultado final
- **Ler RFCs e docs técnicas** que vivem junto do código (cada vez mais o padrão)
- **Subir docs próprios** num repo seu — investigações, decisões, glossário, OKRs

### Como o Claude acessa o GitHub

| Modo | Como funciona | Quando usar |
|---|---|---|
| **CLI local (clone)** | `git clone <repo>` no terminal → abre Claude Code naquela pasta | Investigação profunda — Claude lê filesystem, vários arquivos, resultado rápido e barato |
| **MCP do GitHub** | Claude lê o repo remoto, sem clonar | Exploração rápida, ler 1 PR específico, abrir issue |

#### Heurística: quando clonar e quando não

| Pergunta | Sugestão |
|---|---|
| Investigação que vai durar ≥1 hora | Clone |
| Vai cruzar 5+ arquivos | Clone |
| Quer ler `git log`, `git blame`, histórico de mudanças | Clone |
| Pergunta pontual sobre 1 PR ou 1 arquivo | MCP |
| Quer abrir issue/PR | MCP |
| Não quer poluir disco | MCP |

**Por que clone é melhor pra investigação:** Claude lendo do filesystem local **gasta muito menos token** que via API do GitHub. E faz queries cruzadas instantaneamente. Manter atualizado é `git pull` antes da sessão.

> ***GitHub é o filesystem da empresa. Quem sabe ler código sabe ler a empresa.***

---

## 2. Caso prático — do ticket ao diagnóstico final

Vou contar uma história fictícia que **representa um padrão real**: investigação de produto que parece simples no começo e revela uma causa raiz mais profunda no meio.

> **Contexto:** você é PM de um produto que indexa dados de fornecedores externos via APIs deles. Um cliente abre um ticket dizendo que **dois fornecedores diferentes** estão aparecendo combinados num único registro no painel dele. Você não tem ideia de onde está o problema.

### Os 5 momentos da investigação

#### 1. O ticket chega

Primeira mensagem que você manda pro Claude:

> *"me ajude nessa thread, pode ler o ticket [link]"*

Claude lê o ticket, lê a discussão lá, **e propõe um rascunho de resposta**. Você olha o rascunho e pensa: *"falta entender o problema antes de responder"*. Daí vem a próxima pergunta:

> *"o que é o `external_id` no nosso sistema? Pode ver no repo do data-ingestion?"*

#### 2. O Claude entende o mecanismo

Sem você pedir pra ler arquivo nenhum específico, o Claude vai até o repo, lê o que precisa, e explica:

- A chave de identificação única no banco é uma tupla: `(provider_id, region, external_id)`
- Se `external_id` vier como `NULL` em duas coletas diferentes, o banco trata `NULL = NULL` como igualdade — colapsa os dois registros num só
- **Isso explica o ticket**: dois fornecedores reais com região coincidente estão sendo sobrescritos um pelo outro a cada nova coleta

A essa altura você ainda não entendeu direito. Pergunta:

> *"(provider_id, region, external_id). Não entendi como virou 1 registro só"*

E o Claude explica de novo com paciência. **Esse é o ponto:** ele não te cobra conhecimento. Vai reduzindo até a resposta fazer sentido.

#### 3. A primeira hipótese, que parecia óbvia

Você pede pro Claude propor próximos passos:

> *"mas como resolveríamos isso? Que query eu preciso rodar pra saber desde quando isso acontece?"*

Claude propõe queries no BigQuery (ou no seu data warehouse), roda, volta com:

- **~3 milhões de registros** com `external_id = NULL` afetados
- **Dia-zero**: Janeiro/2024 — saltou de 0% pra 94% de NULL num único mês

Você revisa os outputs do Claude na medida em que saem: pede pra ajustar nome de colunas (*"só `null` não fala o que é o null. Coloca tipo `nome do campo = null`"*), incluir intervalo histórico pra baseline, gerar amostras de IDs específicos pra você validar manualmente.

Da leitura do código nasce a primeira hipótese:

> O fornecedor trocou o nome do campo na API deles. Antes era `code_a`, agora é `code_b`. **Solução**: adicionar `code_b` como fallback no nosso código e tá resolvido.

Você sobe uma doc inicial pro seu repo, com plano em fases (0–5). Vida boa.

#### 4. A virada — onde o Claude descobre o que ninguém procurou

Duas semanas depois você retoma o tema pra fechar o RFC. Pede pro Claude **aprofundar a análise**.

**Aqui é onde o Claude vai além do que você pediu.**

Sem você perguntar sobre categorias, o Claude **observa nos próprios dados** que 94% dos casos com `external_id` preenchido pós-Janeiro/2024 são de **uma categoria só** (a que não estava no escopo do bug). Esse achado sozinho já é estranho — então ele volta ao código pra entender por quê, e descobre o que a leitura humana corrida não tinha pego:

> O fornecedor tem **duas APIs paralelas**, e ele escolhe qual usar pra cada categoria.

| URL retornada pelo fornecedor | Endpoint usado | Campo lido | Status pós-Jan/2024 |
|---|---|---|---|
| `…/legacy/…` | API XML legada | `code_b` | ✅ Funciona |
| `…/v2/…` | API JSON nova | `code_a` / `code_a_link` | ❌ Vem vazio |

A conclusão recoloca o problema:

> **Não é troca de nome de campo. É mudança de roteamento upstream do fornecedor.**
>
> Em Janeiro/2024 o fornecedor passou a apontar URLs de uma categoria pra API JSON nova — e essa API parou de mandar o `code_a`. As outras categorias ficaram no caminho legacy e por isso escaparam. **A causa raiz é externa, não nossa.**

E o tamanho real do problema vem junto: o `external_id` desses 3 milhões de registros **já era** — o fornecedor não devolve mais. Sem recoleta, não há como recuperar.

**A "solução simples" do plano inicial não atacaria o passivo.**

#### 5. O RFC final

A doc executiva fecha em **3 semanas** desde o ticket inicial:

- Duas opções principais — **A (recoletar histórica)** e **B (corrigir só pra frente)** — mais alternativas avaliadas e descartadas
- **Recomendação preliminar: Opção A** — recoleta completa, ainda que cara, porque o passivo afeta clientes existentes
- Decisão depende de: validação operacional (volume de chamadas extras à API do fornecedor), alinhamento comercial (risco de quebra de SLA), investigação de impacto em segmentos premium
- Pacote completo no repo: RFC, 6 queries documentadas, amostras com IDs reais validados, threads de comunicação prontas pra postar
- Sistema **não foi corrigido em produção** — segurando até a decisão fechar

Custo total das queries pra reproduzir o RFC: **alguns dólares**.

### O que essa história ensina

A primeira hipótese (trocar nome do campo) **parecia óbvia e simples**. Era cosmética. Não resolveria — e mais grave, escondia o real tamanho do passivo.

Foi quando o Claude **cruzou os dados com o código sem comando direto** — observando o padrão de categoria nos dados, depois voltando ao código pra explicar — que o diagnóstico real apareceu, e com ele a percepção de que o caminho de correção era mais caro do que parecia.

> ***O salto não é o Claude resolver tudo. É o Claude virar o diagnóstico — perguntando coisas que você não tinha pensado em perguntar — e mostrar quando o problema é maior do que parecia.***

### Princípios que valem destacar

1. **Pergunte como dev curioso, não como gerente cobrando** — *"o que é X?"* > *"resolve esse bug"*
2. **Deixe Claude propor a próxima pergunta** — *"o que mais você acha que vale investigar?"* costuma render
3. **Salve cada descoberta como `.md`** — a investigação vira artefato auditável, não conhecimento na sua cabeça
4. **Refine o output dele com olho crítico** — *"adiciona baseline"*, *"gera amostras"*, *"prova com 10 IDs"* — você dirige, ele executa

---

## 3. Memória — local vs nuvem

### As 3 camadas

| Camada | Onde mora | Quem acessa | Exemplo |
|---|---|---|---|
| **1. Conversa atual** | RAM da sessão | Só essa sessão | "Lembra do que falei há 5 mensagens" |
| **2. Memória local persistente** | `~/.claude/projects/.../memory/` no seu PC | Só você | *"Sou PM de fintech. Não usar tabelas grandes em drafts."* |
| **3. Memória na nuvem (repo)** | Repositório GitHub | **Você + suas skills + outras pessoas** | OKRs sincronizados, glossário, RFCs, estado de processos |

### Quatro arquivos típicos pra abrir

1. **`~/.claude/CLAUDE.md`** — instruções globais (idioma, postura, regras)
2. **`~/.claude/projects/<projeto>/memory/MEMORY.md`** — índice das memórias do projeto
3. **`<seu-repo>/state.json`** — estado entre execuções de skills (ex: snapshot da semana passada pra comparar com a atual)
4. **`<seu-repo>/config.md`** — configuração da skill que outras pessoas podem adotar mudando 1 arquivo

> ***Memória local é sua. Memória na nuvem é da equipe. As duas convivem.***

### Skill que cuida da memória

(Já mencionada na Sessão 1, vale relembrar com mais detalhe.)

Skill que **lê todas as memórias, compara entre si, com a conversa atual** — detecta duplicatas, sobreposições, memórias desatualizadas, novas que valem salvar.

```
## Consolidação de memórias

### Atualizadas (2)
- user_role.md — papel mudou de Senior pra Staff
- project_q3_okrs.md — KR2 saiu de 30% pra 65%

### Criadas (1)
- reference_dashboards.md — IDs de 3 dashboards Grafana usados toda hora

### Removidas (1)
- project_q1_okrs.md — quarter encerrado

### Fundidas (0)
```

**Roda em `dry-run` primeiro** — você aprova, depois aplica.

> ***Memória é arquivo. Arquivo tem skill que cuida pra você.***

---

## 4. O repo como base de conhecimento

A grande virada conceitual que essa trilha quer passar:

> O que você descobre investigando vira `.md` no seu repo. Próxima vez que alguém — você ou outra pessoa — precisa, **a skill lê do GitHub**, não da memória local.

### Por que repo > memória local pra conhecimento de time

| Memória local | Repo |
|---|---|
| Só você | Time todo |
| Some quando você troca de máquina | Versionado, eterno |
| Não tem histórico | `git log` mostra a evolução |
| Sem revisão | PR vira discussão |
| Não dá pra reusar em automação remota | Cron/CI pode rodar contra ele |

### O padrão "investigação → artefato"

A investigação do caso fictício acima virou:

```
seu-repo/
└── investigacoes/
    └── 2026-04-bug-external-id/
        ├── README.md          # Estado atual + decisões abertas
        ├── problema-null.md   # RFC v3 (Markdown fonte da doc executiva)
        ├── queries/           # 6 queries SQL documentadas
        ├── amostras/          # IDs validados manualmente
        └── threads/           # Comunicações prontas pra postar
```

Quando o time precisa do contexto daquele bug 3 meses depois — pega tudo lá. Quando alguém quer propor mudança — abre PR. Quando outro time precisa da query pra investigação parecida — copia daquela pasta.

### Princípios práticos

1. **Toda investigação → branch + commit + push** (ainda que solo). Não confie em "tá no meu desktop".
2. **README.md em cada investigação** com: contexto, estado atual, decisões abertas, próximos passos. Atualize a cada sessão.
3. **Skills lendo do repo** — `state.json` é o padrão; armazena o que precisa lembrar.
4. **Glossário viva no repo** — quando descobrir um termo novo do domínio que importa, anota num arquivo `GLOSSARY.md`. Skills consultam.

> ***Memória local é continuidade pessoal. Memória na nuvem é alavanca do time. A primeira faz você não repetir esforço. A segunda faz o time não repetir.***

---

## Pós-sessão — exercícios

Antes de seguir pra sessão 3:

1. **Clonar um repo** que você queira entender — pode ser do seu time, da sua empresa, ou um projeto open-source que te interessa
2. **Rodar 1 investigação real:** abra o Claude Code nessa pasta e faça uma pergunta específica (*"como X trata o caso Y, e o que acontece se Z?"* > *"como funciona X"*)
3. **Abrir seu `MEMORY.md`** e ler 2-3 entradas. Apagar/editar o que estiver desatualizado
4. **Rodar consolidação de memória** (`dry-run`) — só pra ver o output

---

## Recursos

- **GitHub para iniciantes (oficial):** [docs.github.com/get-started](https://docs.github.com/en/get-started)
- **Claude Code — memória persistente:** [docs.claude.com/en/docs/claude-code/memory](https://docs.claude.com/en/docs/claude-code/memory)
- **MCP GitHub server:** [github.com/github/github-mcp-server](https://github.com/github/github-mcp-server)
- **Exemplo prático de PM organizando investigações em repo:** [github.com/vitorrodriguesjus/testador_pessoal](https://github.com/vitorrodriguesjus/testador_pessoal)

---

Próxima sessão: **[Dados na prática — BigQuery, /plan e sub-agents](./03-data-and-bigquery.md)** — como rodar queries com segurança, postura "pense antes de executar", e quando vale paralelizar com sub-agents.
