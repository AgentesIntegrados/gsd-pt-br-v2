---
name: gsd-auditoria-uat
description: "Auditoria entre fases de todos os itens de UAT e verificação pendentes"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

<objective>
Varrer todas as fases em busca de itens de UAT pendentes, ignorados, bloqueados e que precisam de intervenção humana. Cruzar referências com a base de código para detectar documentação desatualizada. Produzir plano de testes humano priorizado.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/audit-uat.md
</execution_context>

<context>
Os arquivos centrais de planejamento são carregados no fluxo via CLI.

**Escopo:**
Glob: .planning/phases/*/*-UAT.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>
