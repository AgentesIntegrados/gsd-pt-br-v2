---
name: gsd-saude
description: "Diagnostica a integridade do diretório de planejamento e opcionalmente repara problemas"
argument-hint: "[--repair]"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---

<objective>
Valida a integridade do diretório `.planning/` e reporta problemas acionáveis. Verifica arquivos ausentes, configurações inválidas, estado inconsistente e planos órfãos.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/health.md
</execution_context>

<process>
Execute o fluxo health a partir de @$HOME/.claude/get-shit-done/workflows/health.md do início ao fim.
Extraia a flag --repair dos argumentos e passe para o fluxo.
</process>
