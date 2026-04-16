---
name: gsd-grafificar
description: "Constrói, consulta e inspeciona o grafo de conhecimento do projeto em .planning/graphs/"
argument-hint: "[build|query <termo>|status|diff]"
allowed-tools:
  - Read
  - Bash
  - Task
---


**PARE -- NÃO LEIA ESTE ARQUIVO. Você já está lendo-o. Este prompt foi injetado no seu contexto pelo sistema de comandos do Claude Code. Usar a ferramenta Read neste arquivo desperdiça tokens. Comece a executar o Passo 0 imediatamente.**

## Passo 0 -- Banner

**Antes de QUALQUER chamada de ferramenta**, exiba este banner:

```
GSD > GRAPHIFY
```

Então prossiga para o Passo 1.

## Passo 1 -- Verificação de Configuração

Verifique se o graphify está habilitado lendo `.planning/config.json` diretamente com a ferramenta Read.

**NÃO use o comando gsd-tools config get-value** -- ele encerra com erro em chaves ausentes.

1. Leia `.planning/config.json` usando a ferramenta Read
2. Se o arquivo não existir: exiba a mensagem de desabilitado abaixo e **PARE**
3. Analise o conteúdo JSON. Verifique se `config.graphify && config.graphify.enabled === true`
4. Se `graphify.enabled` NÃO for explicitamente `true`: exiba a mensagem de desabilitado abaixo e **PARE**
5. Se `graphify.enabled` for `true`: prossiga para o Passo 2

**Mensagem de desabilitado:**

```
GSD > GRAPHIFY

Grafo de conhecimento está desabilitado. Para ativar:

  node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs config-set graphify.enabled true

Depois execute /gsd-graphify build para criar o grafo inicial.
```

---

## Passo 2 -- Analisar Argumento

Analise `$ARGUMENTS` para determinar o modo de operação:

| Argumento | Ação |
|-----------|------|
| `build` | Criar agente graphify-builder (Passo 3) |
| `query <termo>` | Executar consulta inline (Passo 2a) |
| `status` | Executar verificação de status inline (Passo 2b) |
| `diff` | Executar verificação de diff inline (Passo 2c) |
| Sem argumento ou desconhecido | Exibir mensagem de uso |

**Mensagem de uso** (exibida quando sem argumento ou argumento não reconhecido):

```
GSD > GRAPHIFY

Uso: /gsd-graphify <modo>

Modos:
  build           Construir ou reconstruir o grafo de conhecimento
  query <termo>   Pesquisar o grafo por um termo
  status          Exibir atualidade e estatísticas do grafo
  diff            Exibir alterações desde a última construção
```

### Passo 2a -- Consulta

Execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs graphify query <termo>
```

Analise a saída JSON e exiba os resultados:
- Se a saída contiver `"disabled": true`, exiba a mensagem de desabilitado do Passo 1 e **PARE**
- Se a saída contiver campo `"error"`, exiba a mensagem de erro e **PARE**
- Se nenhum nó for encontrado, exiba: `Nenhum resultado no grafo para '<termo>'. Tente /gsd-graphify build para criar ou reconstruir o grafo.`
- Caso contrário, exiba os nós correspondentes agrupados por tipo, com relacionamentos de arestas e níveis de confiança (EXTRACTED/INFERRED/AMBIGUOUS)

**PARE** após exibir os resultados. Não crie um agente.

### Passo 2b -- Status

Execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs graphify status
```

Analise a saída JSON e exiba:
- Se `exists: false`, exiba o campo message
- Caso contrário, exiba o horário da última construção, contagens de nós/arestas/hiperarestas e indicador DESATUALIZADO ou ATUALIZADO

**PARE** após exibir o status. Não crie um agente.

### Passo 2c -- Diff

Execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs graphify diff
```

Analise a saída JSON e exiba:
- Se `no_baseline: true`, exiba o campo message
- Caso contrário, exiba contagens de alterações em nós e arestas (adicionados/removidos/alterados)

Se não existir snapshot, sugira executar `build` duas vezes (primeiro para criar, segundo para gerar uma baseline de diff).

**PARE** após exibir o diff. Não crie um agente.

---

## Passo 3 -- Construção (Criação de Agente)

Execute a verificação de pré-voo primeiro:

```
PREFLIGHT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" graphify build)
```

Se o pré-voo retornar `disabled: true` ou `error`, exiba a mensagem e **PARE**.

Se o pré-voo retornar `action: "spawn_agent"`, exiba:

```
GSD > Iniciando agente graphify-builder...
```

Crie uma Task:

```
Task(
  description="Construir ou reconstruir o grafo de conhecimento do projeto",
  prompt="You are the graphify-builder agent. Your job is to build or rebuild the project knowledge graph using the graphify CLI.

Project root: ${CWD}
gsd-tools path: $HOME/.claude/get-shit-done/bin/gsd-tools.cjs

## Instructions

1. **Invoke graphify:**
   Run from the project root:
   ```
   graphify . --update
   ```
   This builds the knowledge graph with SHA256 incremental caching.
   Timeout: up to 5 minutes (or as configured via graphify.build_timeout).

2. **Validate output:**
   Check that graphify-out/graph.json exists and is valid JSON with nodes[] and edges[] arrays.
   If graphify exited non-zero or graph.json is not parseable, output:
   ## GRAPHIFY BUILD FAILED
   Include the stderr output for debugging. Do NOT delete .planning/graphs/ -- prior valid graph remains available.

3. **Copy artifacts to .planning/graphs/:**
   ```
   cp graphify-out/graph.json .planning/graphs/graph.json
   cp graphify-out/graph.html .planning/graphs/graph.html
   cp graphify-out/GRAPH_REPORT.md .planning/graphs/GRAPH_REPORT.md
   ```
   These three files are the build output consumed by query, status, and diff commands.

4. **Write diff snapshot:**
   ```
   node \"$HOME/.claude/get-shit-done/bin/gsd-tools.cjs\" graphify build snapshot
   ```
   This creates .planning/graphs/.last-build-snapshot.json for future diff comparisons.

5. **Report build summary:**
   ```
   node \"$HOME/.claude/get-shit-done/bin/gsd-tools.cjs\" graphify status
   ```
   Display the node count, edge count, and hyperedge count from the status output.

When complete, output: ## GRAPHIFY BUILD COMPLETE with the summary counts.
If something fails at any step, output: ## GRAPHIFY BUILD FAILED with details."
)
```

Aguarde o agente concluir.

---

## Anti-Padrões

1. NÃO crie um agente para operações de query/status/diff -- estas são chamadas CLI inline
2. NÃO modifique arquivos de grafo diretamente -- o agente de construção gerencia as escritas
3. NÃO pule a verificação de configuração
4. NÃO use gsd-tools config get-value para a verificação de configuração -- ele encerra em chaves ausentes
