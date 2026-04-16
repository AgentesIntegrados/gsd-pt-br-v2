# Padrão de Loop de Revisão

Padrão padrão para revisão iterativa de agentes com feedback. Usado quando um verificador/validador encontra problemas e o agente produtor precisa revisar sua saída.

---

## Padrão: Verificar-Revisar-Escalar (máximo 3 iterações)

Este padrão se aplica sempre que:
1. Um agente produz saída (planos, importações, planos de fechamento de lacunas)
2. Um verificador/validador avalia essa saída
3. Problemas são encontrados que precisam de revisão

### Fluxo

```
prev_issue_count = Infinity
iteration = 0

LOOP:
  1. Executar verificador/validador na saída atual
  2. Ler resultados do verificador
  3. Se PASSOU ou apenas problemas de nível INFO:
     -> Aceitar saída, sair do loop
  4. Se problemas BLOCKER ou WARNING encontrados:
     a. iteration += 1
     b. Se iteration > 3:
        -> Escalar ao usuário (veja "Após 3 Iterações" abaixo)
     c. Analisar contagem de problemas da saída do verificador
     d. Se issue_count >= prev_issue_count:
        -> Escalar ao usuário: "Loop de revisão estagnado (contagem de problemas não diminuiu)"
     e. prev_issue_count = issue_count
     f. Reiniciar o agente produtor com o feedback do verificador anexado
     g. Após a revisão completar, ir para LOOP
```

### Rastreamento de Contagem de Problemas

Rastreie o número de problemas BLOCKER + WARNING retornados pelo verificador em cada iteração. Se a contagem não diminuir entre iterações consecutivas, o agente produtor está estagnado e novas iterações não ajudarão. Pare cedo e escale ao usuário.

Exiba o progresso da iteração antes de cada spawn de revisão:
`Revision iteration {N}/3 -- {blocker_count} blockers, {warning_count} warnings`

### Estrutura do Prompt de Reinicialização

Ao reiniciar o agente produtor para revisão, passe os problemas formatados em YAML do verificador. A saída do verificador contém um cabeçalho `## Issues` seguido de um bloco YAML. Analise este bloco e passe-o verbatim ao agente de revisão.

```
<checker_issues>
The issues below are in YAML format. Each has: dimension, severity, finding,
affected_field, suggested_fix. Address ALL BLOCKER issues. Address WARNING
issues where feasible.

{YAML issues block from checker output -- passed verbatim}
</checker_issues>

<revision_instructions>
Address ALL BLOCKER and WARNING issues identified above.
- For each BLOCKER: make the required change
- For each WARNING: address or explain why it's acceptable
- Do NOT introduce new issues while fixing existing ones
- Preserve all content not flagged by the checker
This is revision iteration {N} of max 3. Previous iteration had {prev_count}
issues. You must reduce the count or the loop will terminate.
</revision_instructions>
```

### Após 3 Iterações

Se os problemas persistirem após 3 ciclos de revisão:

1. Apresente os problemas restantes ao usuário
2. Use o prompt de gate (padrão: yes-no de `references/gate-prompts.md`):
   question: "Problemas persistem após 3 tentativas de revisão. Prosseguir com a saída atual?"
   header: "Prosseguir?"
   options:
     - label: "Prosseguir assim mesmo"   description: "Aceitar saída com problemas restantes"
     - label: "Ajustar abordagem"        description: "Discutir uma abordagem diferente"
3. Se "Prosseguir assim mesmo": aceitar a saída atual e continuar
4. Se "Ajustar abordagem" ou "Outro": discutir com o usuário, depois reingressar na etapa de produção com contexto atualizado

### Variações Específicas por Fluxo de Trabalho

| Fluxo de Trabalho | Agente Produtor | Agente Verificador | Notas |
|-------------------|----------------|-------------------|-------|
| plan-phase | gsd-planner | gsd-plan-checker | Prompt de revisão via planner-revision.md |
| execute-phase | gsd-executor | gsd-verifier | Verificação pós-execução |
| discuss-phase | orchestrator | gsd-plan-checker | Revisão inline pelo orquestrador |

---

## Notas Importantes

- **Problemas de nível INFO são sempre aceitáveis** -- não disparam revisão
- **Cada iteração recebe um novo spawn de agente** -- não tente continuar no mesmo contexto
- **O feedback do verificador deve ser embutido** -- o agente de revisão precisa ver exatamente o que falhou
- **Não engula problemas silenciosamente** -- sempre apresente o estado final ao usuário após sair do loop
