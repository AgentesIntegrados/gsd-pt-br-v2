---
name: gsd-limpeza
description: "Arquivar diretórios de fases acumulados de milestones concluídos"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

<objective>
Arquivar diretórios de fases de milestones concluídos em `.planning/milestones/v{X.Y}-phases/`.

Usar quando `.planning/phases/` acumulou diretórios de milestones anteriores.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/cleanup.md
</execution_context>

<process>
Seguir o fluxo de limpeza em @$HOME/.claude/get-shit-done/workflows/cleanup.md.
Identificar milestones concluídos, exibir um resumo em modo simulado e arquivar após confirmação.
</process>
