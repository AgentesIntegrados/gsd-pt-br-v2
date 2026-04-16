---
component: plan-checker
version: 1
last_calibrated: 2026-03-24
---

# Exemplos Few-Shot do Plan-Checker

## Exemplos Positivos

### Exemplo 1: BLOCKER sinalizado por ação de tarefa vaga sem nome de arquivo e função

**Entrada:** Tarefa do plano com ação: "Implement the authentication feature for the API routes."

**Saída:**

> ```yaml
> issues:
>   - dimension: task_completeness
>     severity: BLOCKER
>     finding: "Task T1 action says 'implement the authentication feature' without naming target files, functions to create, or middleware to apply. Executor cannot determine what to build."
>     affected_field: "<action>"
>     suggested_fix: "Specify: create authMiddleware in src/middleware/auth.js, apply to routes in src/routes/api.js lines 12-45, verify with integration test"
> ```

**Por que é bom:** O verificador citou a dimensão específica (task_completeness), citou o texto problemático, explicou por que é um blocker (o executor não consegue determinar o que construir) e forneceu uma correção concreta com caminhos de arquivo e nomes de função. O resultado é acionável — o planejador sabe exatamente o que adicionar.

### Exemplo 2: BLOCKER por conflito de arquivo na mesma wave entre dois planos

**Entrada:** Dois planos na wave 1 — Plano 01 modifica `get-shit-done/workflows/execute-phase.md`, Plano 02 também modifica `get-shit-done/workflows/execute-phase.md`. Nenhum `depends_on` declarado.

**Saída:**

> ```yaml
> issues:
>   - dimension: dependency_correctness
>     severity: BLOCKER
>     finding: "Plans 01 and 02 both modify get-shit-done/workflows/execute-phase.md in wave 1 with no depends_on relationship. Concurrent execution will cause merge conflicts or lost changes."
>     affected_field: "files_modified"
>     suggested_fix: "Either move Plan 02 to wave 2 with depends_on: ['01'] or consolidate the file changes into a single plan"
> ```

**Por que é bom:** O verificador identificou um problema estrutural real — dois planos modificando o mesmo arquivo na mesma wave sem uma relação de dependência. Citou dependency_correctness, nomeou ambos os planos, o arquivo conflitante e forneceu duas correções alternativas.

## Exemplos Negativos

### Exemplo 1: Todas as dimensões passam quando comandos de verificação estão ausentes

**Entrada:** Plano com 3 tarefas. Tarefa T2 tem `<verify></verify>` (vazio). Tarefa T3 tem `<verify>echo "done"</verify>` (trivial, não consegue distinguir aprovação de reprovação).

**Saída:**

> ## CHECK PASSED
> Plans: 1 | Tasks: 3 | Dimensions: 10 | Issues: 0

**Por que é ruim:** O verificador perdeu duas violações. verification_derivation requer que os comandos de verificação sejam executáveis e testem a saída real — um bloco de verificação vazio é um BLOCKER. Os comandos de verificação devem distinguir aprovação de reprovação — `echo "done"` sempre retorna 0 independentemente do estado da implementação. Ambos deveriam ter sido sinalizados.

### Exemplo 2: Falso positivo INFO para um plano de tamanho correto

**Entrada:** Plano com 3 tarefas, 6 arquivos únicos, 1 dependência. Tudo dentro dos limites.

**Saída:**

> ```yaml
> issues:
>   - dimension: scope_sanity
>     severity: INFO
>     finding: "Plan has 3 tasks -- consider splitting into smaller plans for faster iteration"
>     affected_field: "task count"
>     suggested_fix: "Split tasks into separate plans"
> ```

**Por que é ruim:** O verificador sinalizou uma não-questão. scope_sanity permite 2-3 tarefas por plano — 3 tarefas está dentro dos limites. O verificador aplicou uma preferência pessoal ("menor é melhor") em vez do limiar documentado. Isso desperdiça o tempo do planejador com falsos positivos e corrói a confiança no julgamento do verificador. Uma verificação correta não produziria nenhum problema para este plano.
