<purpose>
Orquestrar agentes de debug paralelos para diagnosticar gaps de UAT. Cada gap recebe seu próprio agente debug que investiga a causa raiz. Chamado a partir de verify-work quando problemas são encontrados.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar o arquivo UAT com gaps:

```bash
UAT_FILE=$(ls "${PHASE_DIR}"/*-UAT.md 2>/dev/null | head -1)
```

Se não encontrado: sair com "Nenhum arquivo UAT encontrado para a Fase {N}."

Ler a seção `## Gaps` do arquivo UAT.
Extrair cada gap como: `{ truth, status, reason, severity, test, artifacts, missing }`.

Exibir:
```
◆ Diagnosticando {N} gaps...
  Iniciando agentes debug paralelos
```
</step>

<step name="spawn_parallel_debuggers">
Para cada gap, iniciar um agente gsd-debugger em paralelo:

```bash
DEBUGGER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-debugger --raw)
AGENT_SKILLS_DEBUGGER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-debugger 2>/dev/null)
```

Para cada gap `{i}`:

```
Task(
  prompt="""
Leia $HOME/.claude/agents/gsd-debugger.md para instruções.

<objetivo>
Diagnosticar causa raiz do gap de UAT #{i}
Fase {phase_number}: {phase_name}
</objetivo>

<gap>
Verdade: {truth}
Status: {status}
Razão: {reason}
Severidade: {severity}
Teste: {test}
</gap>

<contexto>
{conteúdo dos SUMMARY.md relevantes}
{conteúdo dos PLAN.md relevantes}
</contexto>

${AGENT_SKILLS_DEBUGGER}

<instrucoes>
1. Investigar por que o comportamento esperado não está ocorrendo
2. Identificar a causa raiz específica (não apenas sintomas)
3. Listar os arquivos relacionados ao problema
4. Sugerir uma correção específica
5. Retornar diagnóstico estruturado
</instrucoes>
""",
  subagent_type="gsd-debugger",
  model="{DEBUGGER_MODEL}",
  description="Diagnosticar gap #{i}: {truth}"
)
```

Iniciar todos os agentes em paralelo. Aguardar todos os retornos.
</step>

<step name="collect_diagnoses">
Coletar e estruturar os retornos de diagnóstico de cada agente:

Para cada agente:
- Extrair: `root_cause`, `affected_files`, `suggested_fix`, `confidence`
- Associar de volta ao gap original

Exibir progresso:
```
✓ Gap #1: {raiz}
✓ Gap #2: {raiz}
...
```
</step>

<step name="update_uat_file">
Atualizar a seção `## Gaps` do arquivo UAT com diagnósticos:

Para cada gap, adicionar campos de diagnóstico:

```yaml
- truth: "{expected behavior from test}"
  status: failed
  reason: "Usuário reportou: {verbatim user response}"
  severity: {inferred}
  test: {N}
  artifacts: [{arquivos afetados do diagnóstico}]
  missing: [{o que está faltando do diagnóstico}]
  root_cause: "{causa raiz do agente debug}"
  suggested_fix: "{correção sugerida}"
  diagnosis_confidence: {high|medium|low}
```

Commitar arquivo UAT atualizado:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "test(${padded_phase}): diagnósticos de debug adicionados ao UAT" --files "${UAT_FILE}"
```
</step>

<step name="report_diagnoses">
Exibir relatório de diagnóstico consolidado:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DIAGNÓSTICO CONCLUÍDO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{N} gaps diagnosticados:

| # | Verdade | Causa Raiz | Severidade | Confiança |
|---|---------|------------|-----------|-----------|
| 1 | {truth} | {raiz}     | {sev}     | {conf}    |
| 2 | {truth} | {raiz}     | {sev}     | {conf}    |
...
```

Retornar diagnósticos estruturados ao fluxo de trabalho verify-work para planejamento de correções.
</step>

</process>

<success_criteria>
- [ ] Arquivo UAT carregado com gaps
- [ ] Agentes debug paralelos iniciados por gap
- [ ] Todos os diagnósticos coletados
- [ ] Arquivo UAT atualizado com causas raiz
- [ ] Relatório de diagnóstico exibido
- [ ] Diagnósticos estruturados retornados para verify-work
</success_criteria>
