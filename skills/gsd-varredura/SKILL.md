---
name: gsd-varredura
description: "Avaliação rápida da base de código — alternativa leve ao /gsd-map-codebase"
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
  - Agent
  - AskUserQuestion
---

<objective>
Executar uma varredura focada da base de código para uma área específica, produzindo documentos direcionados em `.planning/codebase/`.
Aceita um flag opcional `--focus`: `tech`, `arch`, `quality`, `concerns` ou `tech+arch` (padrão).

Alternativa leve ao `/gsd-map-codebase` — cria um agente mapper em vez de quatro em paralelo.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/scan.md
</execution_context>

<process>
Executar o fluxo de trabalho scan de @$HOME/.claude/get-shit-done/workflows/scan.md do início ao fim.
</process>
