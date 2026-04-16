---
name: gsd-fase-ui
description: "Gerar contrato de design de UI (UI-SPEC.md) para fases de frontend"
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - AskUserQuestion
  - mcp__context7__*
---

<objective>
Criar um contrato de design de UI (UI-SPEC.md) para uma fase de frontend.
Orquestra gsd-ui-researcher e gsd-ui-checker.
Fluxo: Validar → Pesquisar UI → Verificar UI-SPEC → Concluído
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/ui-phase.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Número da fase: $ARGUMENTS — opcional, detecta automaticamente a próxima fase não planejada se omitido.
</context>

<process>
Executar @$HOME/.claude/get-shit-done/workflows/ui-phase.md do início ao fim.
Preservar todos os portões do fluxo de trabalho.
</process>
