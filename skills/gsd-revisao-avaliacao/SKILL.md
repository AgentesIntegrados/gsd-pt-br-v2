---
name: gsd-revisao-avaliacao
description: "Audita retroativamente a cobertura de avaliação de uma fase de IA executada — classifica cada dimensão de avaliação como COBERTA/PARCIAL/AUSENTE e produz um EVAL-REVIEW.md com plano de remediação"
argument-hint: "[número da fase]"
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
Realiza uma auditoria retroativa de cobertura de avaliação de uma fase de IA concluída.
Verifica se a estratégia de avaliação do AI-SPEC.md foi implementada.
Produz EVAL-REVIEW.md com pontuação, veredicto, lacunas e plano de remediação.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/eval-review.md
@$HOME/.claude/get-shit-done/references/ai-evals.md
</execution_context>

<context>
Fase: $ARGUMENTS — opcional, padrão é a última fase concluída.
</context>

<process>
Execute @$HOME/.claude/get-shit-done/workflows/eval-review.md do início ao fim.
Preserve todos os portões do fluxo.
</process>
</output>
