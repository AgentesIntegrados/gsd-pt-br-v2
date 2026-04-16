---
name: gsd-fluxos-trabalho
description: "Gerenciar workstreams paralelos — listar, criar, alternar, status, progresso, concluir e retomar"
allowed-tools:
  - Read
  - Bash
---


# /gsd-workstreams

Gerenciar workstreams paralelos para trabalho simultâneo em milestones.

## Uso

`/gsd-workstreams [subcommand] [args]`

### Subcomandos

| Comando | Descrição |
|---------|-----------|
| `list` | Listar todos os workstreams com status |
| `create <name>` | Criar um novo workstream |
| `status <name>` | Status detalhado de um workstream |
| `switch <name>` | Definir workstream ativo |
| `progress` | Resumo de progresso em todos os workstreams |
| `complete <name>` | Arquivar um workstream concluído |
| `resume <name>` | Retomar trabalho em um workstream |

## Etapa 1: Analisar Subcomando

Analisar a entrada do usuário para determinar qual operação de workstream executar.
Se nenhum subcomando for fornecido, usar `list` como padrão.

## Etapa 2: Executar Operação

### list
Executar: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workstream list --raw --cwd "$CWD"`
Exibir os workstreams em formato de tabela mostrando nome, status, fase atual e progresso.

### create
Executar: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workstream create <name> --raw --cwd "$CWD"`
Após a criação, exibir o novo caminho do workstream e sugerir próximos passos:
- `/gsd-new-milestone --ws <name>` para configurar o milestone

### status
Executar: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workstream status <name> --raw --cwd "$CWD"`
Exibir detalhamento detalhado de fases e informações de estado.

### switch
Executar: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workstream set <name> --raw --cwd "$CWD"`
Também definir `GSD_WORKSTREAM` para a sessão atual quando o runtime suportar.
Se o runtime expuser um identificador de sessão, o GSD também armazena o workstream ativo
localmente na sessão para que sessões concorrentes não se sobrescrevam.

### progress
Executar: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workstream progress --raw --cwd "$CWD"`
Exibir uma visão geral do progresso em todos os workstreams.

### complete
Executar: `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workstream complete <name> --raw --cwd "$CWD"`
Arquivar o workstream em milestones/.

### resume
Definir o workstream como ativo e sugerir `/gsd-resume-work --ws <name>`.

## Etapa 3: Exibir Resultados

Formatar a saída JSON do gsd-tools em uma exibição legível para humanos.
Incluir o flag `${GSD_WS}` em quaisquer sugestões de roteamento.
