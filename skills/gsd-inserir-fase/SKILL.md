---
name: gsd-inserir-fase
description: "Insere trabalho urgente como fase decimal (ex: 72.1) entre fases existentes"
argument-hint: "<depois-de> <descrição>"
allowed-tools:
  - Read
  - Write
  - Bash
---


<objective>
Insere uma fase decimal para trabalho urgente descoberto no meio de um marco que precisa ser concluído entre fases inteiras existentes.

Usa numeração decimal (72.1, 72.2, etc.) para preservar a sequência lógica das fases planejadas enquanto acomoda inserções urgentes.

Propósito: Lidar com trabalho urgente descoberto durante a execução sem renumerar o roadmap inteiro.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/insert-phase.md
</execution_context>

<context>
Argumentos: $ARGUMENTS (formato: <número-da-fase-após> <descrição>)

O roadmap e o estado são resolvidos dentro do fluxo via `init phase-op` e chamadas de ferramentas específicas.
</context>

<process>
Execute o fluxo insert-phase a partir de @$HOME/.claude/get-shit-done/workflows/insert-phase.md do início ao fim.
Preserve todos os portões de validação (análise de argumentos, verificação de fase, cálculo decimal, atualizações do roadmap).
</process>
