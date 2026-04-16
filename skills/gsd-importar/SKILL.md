---
name: gsd-importar
description: "Importa planos externos com detecção de conflitos contra decisões do projeto antes de escrever qualquer coisa."
argument-hint: "--from <filepath>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
---


<objective>
Importa arquivos de plano externos para o sistema de planejamento GSD com detecção de conflitos contra as decisões do PROJECT.md.

- **--from**: Importa um arquivo de plano externo, detecta conflitos, escreve como PLAN.md do GSD, valida via gsd-plan-checker.

Futuro: o modo `--prd` para extração de PRD está planejado para um PR de acompanhamento.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/import.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
@$HOME/.claude/get-shit-done/references/gate-prompts.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
Execute o fluxo de importação do início ao fim.
</process>
