---
name: gsd-analisar-dependencias
description: "Analisar dependências entre fases e sugerir entradas 'Depends on' para o ROADMAP.md"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Analisar o grafo de dependências das fases do milestone atual. Para cada par de fases, determinar se existe uma relação de dependência com base em:
- Sobreposição de arquivos (fases que modificam os mesmos arquivos precisam ser ordenadas)
- Dependências semânticas (uma fase que usa uma API construída por outra fase)
- Fluxo de dados (uma fase que consome a saída de outra fase)

Em seguida, sugerir atualizações de `Depends on` no ROADMAP.md.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/analyze-dependencies.md
</execution_context>

<context>
Nenhum argumento necessário. Requer um milestone ativo com ROADMAP.md.

Execute este comando ANTES do `/gsd-manager` para preencher campos `Depends on` ausentes e evitar conflitos de merge por execução paralela desordenada.
</context>

<process>
Executar o fluxo analyze-dependencies de @$HOME/.claude/get-shit-done/workflows/analyze-dependencies.md do início ao fim.
Apresentar as sugestões de dependência de forma clara e aplicar as atualizações confirmadas no ROADMAP.md.
</process>
