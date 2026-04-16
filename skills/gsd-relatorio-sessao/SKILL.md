---
name: gsd-relatorio-sessao
description: "Gerar relatório de sessão com estimativas de uso de tokens, resumo do trabalho e resultados"
allowed-tools:
  - Read
  - Bash
  - Write
---

<objective>
Gerar um documento SESSION_REPORT.md estruturado capturando resultados da sessão, trabalho realizado e estimativa de recursos utilizados. Fornece um artefato compartilhável para revisão pós-sessão.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/session-report.md
</execution_context>

<process>
Executar o fluxo de trabalho session-report de @$HOME/.claude/get-shit-done/workflows/session-report.md do início ao fim.
</process>
