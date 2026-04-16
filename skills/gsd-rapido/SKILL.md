---
name: gsd-rapido
description: "Executa uma tarefa trivial diretamente — sem subagentes, sem overhead de planejamento"
argument-hint: "[descrição da tarefa]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---


<objective>
Executa uma tarefa trivial diretamente no contexto atual sem criar subagentes
ou gerar arquivos PLAN.md. Para tarefas pequenas demais para justificar overhead de planejamento:
correções de typo, mudanças de configuração, pequenos refactors, commits esquecidos, adições simples.

Este NÃO é um substituto para /gsd-quick — use /gsd-quick para qualquer coisa que
precise de pesquisa, planejamento em múltiplos passos ou verificação. /gsd-fast é para tarefas
que você poderia descrever em uma frase e executar em menos de 2 minutos.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/fast.md
</execution_context>

<process>
Execute o fluxo fast a partir de @$HOME/.claude/get-shit-done/workflows/fast.md do início ao fim.
</process>
