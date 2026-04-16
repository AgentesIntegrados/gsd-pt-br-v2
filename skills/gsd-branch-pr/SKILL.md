---
name: gsd-branch-pr
description: "Criar um branch limpo para PR filtrando commits do .planning/ — pronto para revisão de código"
argument-hint: "[branch de destino, padrão: main]"
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---


<objective>
Criar um branch limpo adequado para pull requests, filtrando commits do .planning/
do branch atual. Revisores veem apenas alterações de código, não artefatos de planejamento GSD.

Isso resolve o problema de diffs de PR poluídos com mudanças em PLAN.md, SUMMARY.md e STATE.md
que são irrelevantes para revisão de código.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/pr-branch.md
</execution_context>

<process>
Executar o workflow pr-branch de @$HOME/.claude/get-shit-done/workflows/pr-branch.md do início ao fim.
</process>
</content>
