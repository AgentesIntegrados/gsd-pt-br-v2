---
name: gsd-validar-fase
description: "Auditar retroativamente e preencher lacunas de validação Nyquist para uma fase concluída"
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
Auditar a cobertura de validação Nyquist para uma fase concluída. Três estados:
- (A) VALIDATION.md existe — auditar e preencher lacunas
- (B) Sem VALIDATION.md, SUMMARY.md existe — reconstruir a partir dos artefatos
- (C) Fase não executada — encerrar com orientações

Resultado: VALIDATION.md atualizado + arquivos de teste gerados.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/validate-phase.md
</execution_context>

<context>
Fase: $ARGUMENTS — opcional, padrão é a última fase concluída.
</context>

<process>
Executar @$HOME/.claude/get-shit-done/workflows/validate-phase.md.
Preservar todos os portões do fluxo de trabalho.
</process>
