---
name: gsd-remover-fase
description: "Remover uma fase futura do roadmap e renumerar as fases subsequentes"
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
---

<objective>
Remover uma fase futura não iniciada do roadmap e renumerar todas as fases subsequentes para manter uma sequência linear e organizada.

Propósito: Remoção limpa de trabalho que você decidiu não realizar, sem poluir o contexto com marcadores de cancelado/adiado.
Resultado: Fase excluída, todas as fases subsequentes renumeradas, commit git como registro histórico.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/remove-phase.md
</execution_context>

<context>
Fase: $ARGUMENTS

O roadmap e o estado são resolvidos no fluxo de trabalho via `init phase-op` e leituras direcionadas.
</context>

<process>
Executar o fluxo de trabalho remove-phase de @$HOME/.claude/get-shit-done/workflows/remove-phase.md do início ao fim.
Preservar todos os portões de validação (verificação de fase futura, verificação de trabalho), a lógica de renumeração e o commit.
</process>
