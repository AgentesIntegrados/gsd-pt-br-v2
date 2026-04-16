---
name: gsd-auditoria-correcao
description: "Pipeline autônomo de auditoria e correção — encontrar problemas, classificar, corrigir, testar, commitar"
argument-hint: "--source <audit-uat> [--severity <medium|high|all>] [--max N] [--dry-run]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Agent
  - AskUserQuestion
---

<objective>
Executar uma auditoria, classificar os achados como corrigíveis automaticamente ou somente manualmente, e então corrigir
autonomamente os problemas auto-corrigíveis com verificação de testes e commits atômicos.

Flags:
- `--max N` — número máximo de achados a corrigir (padrão: 5)
- `--severity high|medium|all` — severidade mínima a processar (padrão: medium)
- `--dry-run` — classificar achados sem corrigir (exibe tabela de classificação)
- `--source <audit>` — qual auditoria executar (padrão: audit-uat)
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/audit-fix.md
</execution_context>

<process>
Executar o fluxo audit-fix de @$HOME/.claude/get-shit-done/workflows/audit-fix.md do início ao fim.
</process>
