---
name: gsd-desfazer
description: "Reversão git segura. Desfaz commits de fase ou plano usando o manifesto de fase com verificações de dependência."
argument-hint: "--last N | --phase NN | --plan NN-MM"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---


<objective>
Reversão git segura — desfaz commits de fase ou plano do GSD usando o manifesto de fase, com verificações de dependência e um portão de confirmação antes da execução.

Três modos:
- **--last N**: Mostrar commits GSD recentes para seleção interativa
- **--phase NN**: Reverter todos os commits de uma fase (manifesto + fallback do git log)
- **--plan NN-MM**: Reverter todos os commits de um plano específico
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/undo.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
@$HOME/.claude/get-shit-done/references/gate-prompts.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
Executar o fluxo de trabalho undo de @$HOME/.claude/get-shit-done/workflows/undo.md do início ao fim.
</process>
