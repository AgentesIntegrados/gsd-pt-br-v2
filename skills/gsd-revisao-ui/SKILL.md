---
name: gsd-revisao-ui
description: "Auditoria visual retroativa em 6 pilares do código frontend implementado"
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

<objective>
Conduzir uma auditoria visual retroativa em 6 pilares. Produz UI-REVIEW.md com
avaliação graduada (1-4 por pilar). Funciona em qualquer projeto.
Resultado: {phase_num}-UI-REVIEW.md
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/ui-review.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Fase: $ARGUMENTS — opcional, padrão é a última fase concluída.
</context>

<process>
Executar @$HOME/.claude/get-shit-done/workflows/ui-review.md do início ao fim.
Preservar todos os portões do fluxo de trabalho.
</process>
