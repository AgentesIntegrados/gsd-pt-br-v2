---
name: gsd-adicionar-fase
description: "Adicionar fase ao final do milestone atual no roadmap"
argument-hint: "<description>"
allowed-tools:
  - Read
  - Write
  - Bash
---


<objective>
Adicionar uma nova fase inteira ao final do milestone atual no roadmap.

Encaminha para o fluxo add-phase que cuida de:
- Cálculo do número da fase (próximo inteiro sequencial)
- Criação de diretório com geração de slug
- Atualizações na estrutura do roadmap
- Rastreamento da evolução do roadmap em STATE.md
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/add-phase.md
</execution_context>

<context>
Arguments: $ARGUMENTS (descrição da fase)

O roadmap e o estado são resolvidos no fluxo via `init phase-op` e chamadas de ferramentas direcionadas.
</context>

<process>
**Seguir o fluxo add-phase** de `@$HOME/.claude/get-shit-done/workflows/add-phase.md`.

O fluxo cuida de toda a lógica, incluindo:
1. Análise e validação de argumentos
2. Verificação da existência do roadmap
3. Identificação do milestone atual
4. Cálculo do próximo número de fase (ignorando decimais)
5. Geração de slug a partir da descrição
6. Criação do diretório de fase
7. Inserção da entrada no roadmap
8. Atualizações em STATE.md
</process>
