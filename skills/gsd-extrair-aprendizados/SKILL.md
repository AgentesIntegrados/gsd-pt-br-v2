---
name: gsd-extrair-aprendizados
description: "Extrai decisões, lições, padrões e surpresas dos artefatos de uma fase concluída"
argument-hint: "<número-da-fase>"
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
  - Agent
---

<objective>
Extrai aprendizados estruturados dos artefatos de uma fase concluída (PLAN.md, SUMMARY.md, VERIFICATION.md, UAT.md, STATE.md) em um arquivo LEARNINGS.md que registra decisões tomadas, lições aprendidas, padrões descobertos e surpresas encontradas.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/extract_learnings.md
</execution_context>

Execute o fluxo extract-learnings a partir de @$HOME/.claude/get-shit-done/workflows/extract_learnings.md do início ao fim.
