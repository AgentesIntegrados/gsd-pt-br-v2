<purpose>
Operador de reparo autônomo para verificação de tarefas com falha. Invocado por execute-plan quando uma tarefa falha em seus critérios de conclusão. Propõe e tenta correções estruturadas antes de escalar para o usuário.
</purpose>

<inputs>
- FAILED_TASK: Número da tarefa, nome e critérios de conclusão do plano
- ERROR: O que a verificação produziu — resultado real vs esperado
- PLAN_CONTEXT: Tarefas adjacentes e objetivo da fase (para consciência de restrições)
- REPAIR_BUDGET: Máximo de tentativas de reparo restantes (padrão: 2)
</inputs>

<repair_directive>
Analise a falha e escolha exatamente uma estratégia de reparo:

**RETRY** — A abordagem estava correta, mas a execução falhou. Tente novamente com um ajuste concreto.
- Use quando: erro de comando, dependência ausente, caminho errado, problema de ambiente, falha transitória
- Saída: `RETRY: [ajuste específico a fazer antes de tentar novamente]`

**DECOMPOSE** — A tarefa é muito ampla. Divida em sub-etapas menores verificáveis.
- Use quando: os critérios de conclusão cobrem múltiplas preocupações, lacunas de implementação são estruturais
- Saída: `DECOMPOSE: [sub-tarefa 1] | [sub-tarefa 2] | ...` (máximo 3 sub-tarefas)
- Sub-tarefas devem ter cada uma um único resultado verificável

**PRUNE** — A tarefa é inviável dadas as restrições atuais. Pule com justificativa.
- Use quando: pré-requisito ausente e não corrigível aqui, fora do escopo, contradiz uma decisão anterior
- Saída: `PRUNE: [justificativa em uma frase]`

**ESCALATE** — Orçamento de reparo esgotado, ou esta é uma decisão arquitetural (Regra 4).
- Use quando: RETRY falhou mais de uma vez com abordagens diferentes, ou a correção requer mudança estrutural
- Saída: `ESCALATE: [o que foi tentado] | [qual decisão é necessária]`
</repair_directive>

<process>

<step name="diagnose">
Leia o erro e os critérios de conclusão com atenção. Pergunte:
1. Este é um problema transitório/ambiental? → RETRY
2. A tarefa é verificavelmente muito ampla? → DECOMPOSE
3. Um pré-requisito está genuinamente ausente e não pode ser corrigido no escopo? → PRUNE
4. RETRY já foi tentado com esta tarefa? Verifique REPAIR_BUDGET. Se 0 → ESCALATE
</step>

<step name="execute_retry">
Se RETRY:
1. Aplique o ajuste específico declarado na diretiva
2. Re-execute a implementação da tarefa
3. Re-execute a verificação
4. Se passar → continue normalmente, registre `[Node Repair - RETRY] Tarefa [X]: [ajuste feito]`
5. Se falhar novamente → decremente REPAIR_BUDGET, re-invoque node-repair com contexto atualizado
</step>

<step name="execute_decompose">
Se DECOMPOSE:
1. Substitua a tarefa com falha pelas sub-tarefas inline (não modifique PLAN.md no disco)
2. Execute sub-tarefas sequencialmente, cada uma com sua própria verificação
3. Se todas as sub-tarefas passarem → trate a tarefa original como bem-sucedida, registre `[Node Repair - DECOMPOSE] Tarefa [X] → [N] sub-tarefas`
4. Se uma sub-tarefa falhar → re-invoque node-repair para aquela sub-tarefa (REPAIR_BUDGET aplica por sub-tarefa)
</step>

<step name="execute_prune">
Se PRUNE:
1. Marque a tarefa como ignorada com justificativa
2. Registre em SUMMARY "Problemas Encontrados": `[Node Repair - PRUNE] Tarefa [X]: [justificativa]`
3. Continue para a próxima tarefa
</step>

<step name="execute_escalate">
Se ESCALATE:
1. Apresente ao usuário via verification_failure_gate com histórico completo de reparos
2. Apresente: o que foi tentado (cada tentativa de RETRY/DECOMPOSE), qual é o bloqueio, opções disponíveis
3. Aguarde a orientação do usuário antes de continuar
</step>

</process>

<logging>
Todas as ações de reparo devem aparecer em SUMMARY.md sob "## Desvios do Plano":

| Tipo | Formato |
|------|---------|
| RETRY com sucesso | `[Node Repair - RETRY] Tarefa X: [ajuste] — resolvido` |
| RETRY falhou → ESCALATE | `[Node Repair - RETRY] Tarefa X: [N] tentativas esgotadas — escalado para o usuário` |
| DECOMPOSE | `[Node Repair - DECOMPOSE] Tarefa X dividida em [N] sub-tarefas — todas aprovadas` |
| PRUNE | `[Node Repair - PRUNE] Tarefa X ignorada: [justificativa]` |
</logging>

<constraints>
- REPAIR_BUDGET tem padrão 2 por tarefa. Configurável via config.json `workflow.node_repair_budget`.
- Nunca modifique PLAN.md no disco — sub-tarefas decompostas existem apenas em memória.
- Sub-tarefas DECOMPOSE devem ser mais específicas que a original, não reescritas sinônimas.
- Se config.json `workflow.node_repair` for `false`, vá diretamente para verification_failure_gate (o usuário mantém o comportamento original).
</constraints>
</output>
