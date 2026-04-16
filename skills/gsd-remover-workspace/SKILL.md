---
name: gsd-remover-workspace
description: "Remover um workspace do GSD e limpar worktrees"
argument-hint: "<workspace-name>"
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

<context>
**Argumentos:**
- `<workspace-name>` (obrigatório) — Nome do workspace a remover
</context>

<objective>
Remover um diretório de workspace após confirmação. Para a estratégia de worktree, executa `git worktree remove` para cada repositório membro primeiro. Recusa a operação se algum repositório tiver alterações não confirmadas.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/remove-workspace.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<process>
Executar o fluxo de trabalho remove-workspace de @$HOME/.claude/get-shit-done/workflows/remove-workspace.md do início ao fim.
</process>
