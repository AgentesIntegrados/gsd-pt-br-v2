---
name: gsd-planejar-lacunas
description: "Criar fases para fechar todas as lacunas identificadas pela auditoria de milestone"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Criar todas as fases necessárias para fechar as lacunas identificadas por `/gsd-audit-milestone`.

Lê MILESTONE-AUDIT.md, agrupa lacunas em fases lógicas, cria entradas de fase no ROADMAP.md e oferece para planejar cada fase.

Um único comando cria todas as fases de correção — sem `/gsd-add-phase` manual por lacuna.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/plan-milestone-gaps.md
</execution_context>

<context>
**Resultados da auditoria:**
Glob: .planning/v*-MILESTONE-AUDIT.md (usar o mais recente)

A intenção original e o estado atual do planejamento são carregados sob demanda dentro do workflow.
</context>

<process>
Executar o workflow plan-milestone-gaps de @$HOME/.claude/get-shit-done/workflows/plan-milestone-gaps.md do início ao fim.
Preservar todos os pontos de controle do workflow (carregamento da auditoria, priorização, agrupamento de fases, confirmação do usuário, atualizações do roadmap).
</process>
</content>
