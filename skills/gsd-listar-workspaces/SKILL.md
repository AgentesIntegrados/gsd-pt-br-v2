---
name: gsd-listar-workspaces
description: "Listar workspaces GSD ativos e seus status"
allowed-tools:
  - Bash
  - Read
---

<objective>
Escanear `~/gsd-workspaces/` em busca de diretórios de workspace que contenham manifestos `WORKSPACE.md`. Exibir uma tabela resumida com nome, caminho, quantidade de repositórios, estratégia e status do projeto GSD.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/list-workspaces.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<process>
Executar o workflow list-workspaces de @$HOME/.claude/get-shit-done/workflows/list-workspaces.md do início ao fim.
</process>
</content>
