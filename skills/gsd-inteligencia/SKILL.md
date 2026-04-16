---
name: gsd-inteligencia
description: "Consulta, inspeciona ou atualiza arquivos de inteligência da base de código em .planning/intel/"
argument-hint: "[query <termo>|status|diff|refresh]"
allowed-tools:
  - Read
  - Bash
  - Task
---


**PARE -- NÃO LEIA ESTE ARQUIVO. Você já está lendo-o. Este prompt foi injetado no seu contexto pelo sistema de comandos do Claude Code. Usar a ferramenta Read neste arquivo desperdiça tokens. Comece a executar o Passo 0 imediatamente.**

## Passo 0 -- Banner

**Antes de QUALQUER chamada de ferramenta**, exiba este banner:

```
GSD > INTEL
```

Então prossiga para o Passo 1.

## Passo 1 -- Verificação de Configuração

Verifique se o intel está habilitado lendo `.planning/config.json` diretamente com a ferramenta Read.

**NÃO use o comando gsd-tools config get-value** -- ele encerra com erro em chaves ausentes.

1. Leia `.planning/config.json` usando a ferramenta Read
2. Se o arquivo não existir: exiba a mensagem de desabilitado abaixo e **PARE**
3. Analise o conteúdo JSON. Verifique se `config.intel && config.intel.enabled === true`
4. Se `intel.enabled` NÃO for explicitamente `true`: exiba a mensagem de desabilitado abaixo e **PARE**
5. Se `intel.enabled` for `true`: prossiga para o Passo 2

**Mensagem de desabilitado:**

```
GSD > INTEL

Sistema Intel está desabilitado. Para ativar:

  node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs config-set intel.enabled true

Depois execute /gsd-intel refresh para construir o índice inicial.
```

---

## Passo 2 -- Analisar Argumento

Analise `$ARGUMENTS` para determinar o modo de operação:

| Argumento | Ação |
|-----------|------|
| `query <termo>` | Executar consulta inline (Passo 2a) |
| `status` | Executar verificação de status inline (Passo 2b) |
| `diff` | Executar verificação de diff inline (Passo 2c) |
| `refresh` | Criar agente intel-updater (Passo 3) |
| Sem argumento ou desconhecido | Exibir mensagem de uso |

**Mensagem de uso** (exibida quando sem argumento ou argumento não reconhecido):

```
GSD > INTEL

Uso: /gsd-intel <modo>

Modos:
  query <termo>  Pesquisar arquivos intel por um termo
  status         Exibir atualidade e desatualização dos arquivos intel
  diff           Exibir alterações desde o último snapshot
  refresh        Reconstruir todos os arquivos intel a partir da análise da base de código
```

### Passo 2a -- Consulta

Execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs intel query <termo>
```

Analise a saída JSON e exiba os resultados:
- Se a saída contiver `"disabled": true`, exiba a mensagem de desabilitado do Passo 1 e **PARE**
- Se nenhum resultado for encontrado, exiba: `Nenhum resultado intel para '<termo>'. Tente /gsd-intel refresh para construir o índice.`
- Caso contrário, exiba as entradas correspondentes agrupadas por arquivo intel

**PARE** após exibir os resultados. Não crie um agente.

### Passo 2b -- Status

Execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs intel status
```

Analise a saída JSON e exiba cada arquivo intel com:
- Nome do arquivo
- Timestamp de `updated_at` mais recente
- Status DESATUALIZADO ou ATUALIZADO (desatualizado se tiver mais de 24 horas ou estiver ausente)

**PARE** após exibir o status. Não crie um agente.

### Passo 2c -- Diff

Execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs intel diff
```

Analise a saída JSON e exiba:
- Entradas adicionadas desde o último snapshot
- Entradas removidas desde o último snapshot
- Entradas alteradas desde o último snapshot

Se não existir snapshot, sugira executar `refresh` primeiro.

**PARE** após exibir o diff. Não crie um agente.

---

## Passo 3 -- Atualização (Criação de Agente)

Exiba antes de criar o agente:

```
GSD > Iniciando agente intel-updater para analisar a base de código...
```

Crie uma Task:

```
Task(
  description="Atualizar arquivos de inteligência da base de código",
  prompt="You are the gsd-intel-updater agent. Your job is to analyze this codebase and write/update intelligence files in .planning/intel/.

Project root: ${CWD}
gsd-tools path: $HOME/.claude/get-shit-done/bin/gsd-tools.cjs

Instructions:
1. Analyze the codebase structure, dependencies, APIs, and architecture
2. Write JSON intel files to .planning/intel/ (stack.json, api-map.json, dependency-graph.json, file-roles.json, arch-decisions.json)
3. Each file must have a _meta object with updated_at timestamp
4. Use gsd-tools intel extract-exports <file> to analyze source files
5. Use gsd-tools intel patch-meta <file> to update timestamps after writing
6. Use gsd-tools intel validate to check your output

When complete, output: ## INTEL UPDATE COMPLETE
If something fails, output: ## INTEL UPDATE FAILED with details."
)
```

Aguarde o agente concluir.

---

## Passo 4 -- Resumo Pós-Atualização

Após o agente concluir, execute:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs intel status
```

Exiba um resumo mostrando:
- Quais arquivos intel foram escritos ou atualizados
- Timestamps da última atualização
- Saúde geral do índice intel

---

## Anti-Padrões

1. NÃO crie um agente para operações de query/status/diff -- estas são chamadas CLI inline
2. NÃO modifique arquivos intel diretamente -- o agente gerencia as escritas durante o refresh
3. NÃO pule a verificação de configuração
4. NÃO use o CLI gsd-tools config get-value para a verificação de configuração -- ele encerra em chaves ausentes
