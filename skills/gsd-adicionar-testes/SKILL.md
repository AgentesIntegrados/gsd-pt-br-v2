---
name: gsd-adicionar-testes
description: "Gerar testes para uma fase concluída com base nos critérios de UAT e implementação"
argument-hint: "<phase> [additional instructions]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

<objective>
Gerar testes unitários e E2E para uma fase concluída, usando seu SUMMARY.md, CONTEXT.md e VERIFICATION.md como especificações.

Analisa os arquivos de implementação, classifica-os em categorias TDD (unitário), E2E (browser) ou Skip, apresenta um plano de testes para aprovação do usuário e, em seguida, gera os testes seguindo as convenções RED-GREEN.

Resultado: Arquivos de teste commitados com a mensagem `test(phase-{N}): add unit and E2E tests from add-tests command`
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/add-tests.md
</execution_context>

<context>
Phase: $ARGUMENTS

@.planning/STATE.md
@.planning/ROADMAP.md
</context>

<process>
Executar o fluxo add-tests de @$HOME/.claude/get-shit-done/workflows/add-tests.md do início ao fim.
Preservar todos os gates do fluxo (aprovação de classificação, aprovação do plano de testes, verificação RED-GREEN, relatório de lacunas).
</process>
