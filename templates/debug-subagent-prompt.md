# Template de Prompt do Subagente de Debug

Template para spawnar o agente gsd-debugger. O agente contém toda a expertise de debugging — este template fornece apenas o contexto do problema.

---

## Template

```markdown
<objective>
Investigar o problema: {issue_id}

**Resumo:** {issue_summary}
</objective>

<symptoms>
expected: {expected}
actual: {actual}
errors: {errors}
reproduction: {reproduction}
timeline: {timeline}
</symptoms>

<mode>
symptoms_prefilled: {true_or_false}
goal: {find_root_cause_only | find_and_fix}
</mode>

<debug_file>
Criar: .planning/debug/{slug}.md
</debug_file>
```

---

## Placeholders

| Placeholder | Origem | Exemplo |
|-------------|--------|---------|
| `{issue_id}` | Atribuído pelo orquestrador | `auth-screen-dark` |
| `{issue_summary}` | Descrição do usuário | `Tela de auth está muito escura` |
| `{expected}` | Dos sintomas | `Ver o logo claramente` |
| `{actual}` | Dos sintomas | `A tela está escura` |
| `{errors}` | Dos sintomas | `Nenhum no console` |
| `{reproduction}` | Dos sintomas | `Abrir a página /auth` |
| `{timeline}` | Dos sintomas | `Após deploy recente` |
| `{goal}` | Definido pelo orquestrador | `find_and_fix` |
| `{slug}` | Gerado | `auth-screen-dark` |

---

## Uso

**De /gsd-debug:**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-debugger",
  description="Debug {slug}"
)
```

**De diagnose-issues (UAT):**
```python
Task(prompt=template, subagent_type="gsd-debugger", description="Debug UAT-001")
```

---

## Continuação

Para checkpoints, spawne um novo agente com:

```markdown
<objective>
Continuar o debug de {slug}. As evidências estão no arquivo de debug.
</objective>

<prior_state>
Arquivo de debug: @.planning/debug/{slug}.md
</prior_state>

<checkpoint_response>
**Tipo:** {checkpoint_type}
**Resposta:** {user_response}
</checkpoint_response>

<mode>
goal: {goal}
</mode>
```
