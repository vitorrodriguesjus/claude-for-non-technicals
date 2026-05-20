# Claude pra quem não é técnico

Trilha de 3 sessões pra **PMs, analistas, líderes e curiosos não-técnicos** que querem **dominar Claude** no dia a dia — do Chat ao Code, de skill simples a investigação cruzando código e dados.

Sem hype, sem promessa de "IA vai resolver tudo". Foco em **resultado prático**: o que dá pra fazer hoje, com a ferramenta que existe hoje, em casos de uso reais de produto e operação.

## Pra quem é

- **PM/PO** que coordena times técnicos mas não codifica
- **Analista** (de produto, ops, dados) que vê código mas não escreve diariamente
- **Líder** que quer entender a alavanca de IA pro seu time
- **Curioso** que já usou ChatGPT/Claude no navegador e quer ir além

Não pressupõe que você sabe programar. Pressupõe que você **lê coisa técnica** quando precisa e quer aumentar essa autonomia.

## O que você sai sabendo

| Depois de... | Você vai conseguir |
|---|---|
| **Sessão 1** | Distinguir as 3 superfícies do Claude (Chat / Cowork / Code), entender o que são MCP servers e skills, e criar sua primeira instrução persistente |
| **Sessão 2** | Usar GitHub como base de conhecimento, conduzir uma investigação técnica em código sem ser dev, e organizar memória entre sessões |
| **Sessão 3** | Rodar queries em BigQuery via Claude com segurança, aplicar postura "pense antes de executar", e entender quando vale paralelizar com sub-agents |

## Sessões

| # | Tema | Tempo de leitura |
|---|------|------------------|
| 1 | [Os 3 jeitos de usar o Claude](./01-getting-started.md) | ~25 min |
| 2 | [GitHub como base de conhecimento + investigação](./02-github-and-knowledge.md) | ~25 min |
| 3 | [Dados na prática — BigQuery, /plan e sub-agents](./03-data-and-bigquery.md) | ~30 min |

## Como usar

**Leitura linear** — siga 1 → 2 → 3, cada uma assume contexto da anterior.

**Pré-requisitos antes de começar:**

1. Conta no [Claude](https://claude.ai) (Pro ou maior, pra acesso ao Code)
2. [Claude Code](https://docs.claude.com/en/docs/claude-code) instalado (CLI ou extensão VS Code)
3. Acesso a pelo menos 1 ferramenta que você quer integrar — Slack, Drive, GitHub, Notion (qualquer uma já vale)

A trilha é construída pra ser **autoaplicada**: você lê, tenta no seu contexto, e segue pra próxima. Não tem exercícios fechados de "certo/errado" — tem padrões pra você adaptar.

## Origem deste material

Este material foi originalmente desenvolvido como trilha interna pro time de Data Collection do [Jusbrasil](https://jusbrasil.com.br), apresentada pelo [Vitor Rodrigues](https://www.linkedin.com/in/vitorrodriguesjus/) (Staff PM) em Abril–Maio de 2026. A versão pública aqui presente **substitui casos internos por análogos genéricos** preservando o arco didático.

Material complementar (skills, exemplos de configuração, ciclo semanal) está disponível no repo público [`testador_pessoal`](https://github.com/vitorrodriguesjus/testador_pessoal) — referência viva de "PM com Claude em produção".

## Licença

Material publicado sob [Creative Commons BY 4.0](./LICENSE) — use, adapte, traduza, redistribua, citando a origem.

## Contribuindo

Encontrou erro? Tem sugestão de exemplo melhor? Quer adicionar uma 4ª sessão (skills auto-melhoráveis, agentes recorrentes etc.)? Abra uma issue ou PR.

Pull requests são especialmente bem-vindos pra:

- **Traduções** (EN, ES)
- **Casos práticos** de outros domínios (financeiro, jurídico, marketing, ops)
- **Capturas de tela** que ajudem leitores menos familiarizados com o terminal
