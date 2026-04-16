<purpose>
Executa uma tarefa trivial inline sem o overhead de subagente. Sem PLAN.md, sem spawn de Task,
sem pesquisa, sem verificação de plano. Apenas: entenda → faça → commit → registre.

Para tarefas como: corrigir um typo, atualizar um valor de configuração, adicionar um import ausente, renomear uma
variável, commitar trabalho não commitado, adicionar uma entrada no .gitignore, incrementar um número de versão.

Use /gsd-quick para qualquer coisa que precise de planejamento ou pesquisa em múltiplas etapas.
</purpose>

<process>

<step name="parse_task">
Analise `$ARGUMENTS` para a descrição da tarefa.

Se vazio, pergunte:
```
Qual é a correção rápida? (em uma frase)
```

Armazene como `$TASK`.
</step>

<step name="scope_check">
**Antes de fazer qualquer coisa, verifique se a tarefa é realmente trivial.**

Uma tarefa é trivial se puder ser concluída em:
- ≤ 3 edições de arquivo
- ≤ 1 minuto de trabalho
- Sem novas dependências ou mudanças arquiteturais
- Sem necessidade de pesquisa

Se a tarefa parecer não-trivial (refatoração multi-arquivo, nova funcionalidade, precisa de pesquisa),
diga:

```
Isso parece precisar de planejamento. Use /gsd-quick em vez disso:
  /gsd-quick "{task description}"
```

E pare.
</step>

<step name="execute_inline">
Faça o trabalho diretamente:

1. Leia o(s) arquivo(s) relevante(s)
2. Faça a(s) mudança(s)
3. Verifique se a mudança funciona (execute os testes existentes se aplicável, ou faça uma verificação rápida de sanidade)

**Sem PLAN.md.** Apenas faça.
</step>

<step name="commit">
Faça commit da mudança atomicamente:

```bash
git add -A
git commit -m "fix: {concise description of what changed}"
```

Use o formato de commit convencional: `fix:`, `feat:`, `docs:`, `chore:`, `refactor:` conforme apropriado.
</step>

<step name="log_to_state">
Se `.planning/STATE.md` existir, anexe à tabela "Quick Tasks Completed".
Se a tabela não existir, ignore este passo silenciosamente.

```bash
# Verifique se STATE.md tem a tabela de tarefas rápidas
if grep -q "Quick Tasks Completed" .planning/STATE.md 2>/dev/null; then
  # Anexe entrada — o fluxo gerencia o formato
  echo "| $(date +%Y-%m-%d) | fast | $TASK | ✅ |" >> .planning/STATE.md
fi
```
</step>

<step name="done">
Reporte a conclusão:

```
✅ Concluído: {o que foi mudado}
   Commit: {short hash}
   Arquivos: {lista de arquivos alterados}
```

Sem sugestões de próximos passos. Sem roteamento de fluxo. Apenas concluído.
</step>

</process>

<guardrails>
- NUNCA spawne uma Task/subagente — executa inline
- NUNCA crie arquivos PLAN.md ou SUMMARY.md
- NUNCA execute pesquisa ou verificação de plano
- Se a tarefa exigir mais de 3 edições de arquivo, PARE e redirecione para /gsd-quick
- Se você não tiver certeza de como implementá-la, PARE e redirecione para /gsd-quick
</guardrails>

<success_criteria>
- [ ] Tarefa concluída no contexto atual (sem subagentes)
- [ ] Commit atômico com mensagem convencional
- [ ] STATE.md atualizado se existir
- [ ] Operação total em menos de 2 minutos de tempo real
</success_criteria>
