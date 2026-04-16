---
name: gsd-proteger-fase
description: "Verificar retroativamente mitigações de ameaças para uma fase concluída"
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

<objective>
Verificar mitigações de ameaças para uma fase concluída. Três estados:
- (A) SECURITY.md existe — auditar e verificar mitigações
- (B) Sem SECURITY.md, PLAN.md com modelo de ameaças existe — executar a partir dos artefatos
- (C) Fase não executada — encerrar com orientações

Resultado: SECURITY.md atualizado.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/secure-phase.md
</execution_context>

<context>
Fase: $ARGUMENTS — opcional, padrão é a última fase concluída.
</context>

<process>
Executar @$HOME/.claude/get-shit-done/workflows/secure-phase.md.
Preservar todos os portões do fluxo de trabalho.
</process>
