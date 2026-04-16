---
name: gsd-entregar
description: "Criar PR, executar revisão e preparar para merge após a verificação ser aprovada"
argument-hint: "[phase number or milestone, e.g., '4' or 'v1.0']"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - Write
  - AskUserQuestion
---

<objective>
Fazer a ponte entre conclusão local → PR mesclado. Após /gsd-verify-work ser aprovado, publicar o trabalho: fazer push da branch, criar PR com corpo gerado automaticamente, opcionalmente acionar revisão e acompanhar o merge.

Fecha o ciclo plan → execute → verify → ship.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/ship.md
</execution_context>

Executar o fluxo de trabalho ship de @$HOME/.claude/get-shit-done/workflows/ship.md do início ao fim.
