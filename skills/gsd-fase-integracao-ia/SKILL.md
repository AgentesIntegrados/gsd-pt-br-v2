---
name: gsd-fase-integracao-ia
description: "Gerar contrato de design de IA (AI-SPEC.md) para fases que envolvem a construção de sistemas de IA — seleção de framework, orientação de implementação a partir da documentação oficial e estratégia de avaliação"
argument-hint: "[phase number]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - WebSearch
  - AskUserQuestion
  - mcp__context7__*
---

<objective>
Criar um contrato de design de IA (AI-SPEC.md) para uma fase que envolve desenvolvimento de sistema de IA.
Orquestra gsd-framework-selector → gsd-ai-researcher → gsd-domain-researcher → gsd-eval-planner.
Fluxo: Selecionar Framework → Pesquisar Documentação → Pesquisar Domínio → Desenhar Estratégia de Avaliação → Concluído
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/ai-integration-phase.md
@$HOME/.claude/get-shit-done/references/ai-frameworks.md
@$HOME/.claude/get-shit-done/references/ai-evals.md
</execution_context>

<context>
Número da fase: $ARGUMENTS — opcional, detecta automaticamente a próxima fase não planejada se omitido.
</context>

<process>
Executar @$HOME/.claude/get-shit-done/workflows/ai-integration-phase.md do início ao fim.
Preservar todos os gates do fluxo.
</process>
