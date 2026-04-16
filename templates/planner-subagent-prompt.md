# Template de Prompt do Subagente Planejador

Template para spawnar o agente gsd-planner. O agente contém toda a expertise de planejamento — este template fornece apenas o contexto de planejamento.

---

## Template

```markdown
<planning_context>

**Fase:** {phase_number}
**Modo:** {standard | gap_closure}

**Estado do Projeto:**
@.planning/STATE.md

**Roadmap:**
@.planning/ROADMAP.md

**Requisitos (se existir):**
@.planning/REQUIREMENTS.md

**Contexto da Fase (se existir):**
@.planning/phases/{phase_dir}/{phase_num}-CONTEXT.md

**Pesquisa (se existir):**
@.planning/phases/{phase_dir}/{phase_num}-RESEARCH.md

**Fechamento de Lacunas (se no modo --gaps):**
@.planning/phases/{phase_dir}/{phase_num}-VERIFICATION.md
@.planning/phases/{phase_dir}/{phase_num}-UAT.md

</planning_context>

<downstream_consumer>
Saída consumida por /gsd-execute-phase
Os planos devem ser prompts executáveis com:
- Frontmatter (wave, depends_on, files_modified, autonomous)
- Tarefas em formato XML
- Critérios de verificação
- must_haves para verificação retroativa a partir do objetivo
</downstream_consumer>

<quality_gate>
Antes de retornar PLANEJAMENTO CONCLUÍDO:
- [ ] Arquivos PLAN.md criados no diretório da fase
- [ ] Cada plano possui frontmatter válido
- [ ] As tarefas são específicas e acionáveis
- [ ] Dependências corretamente identificadas
- [ ] Waves atribuídas para execução paralela
- [ ] must_haves derivados do objetivo da fase
</quality_gate>
```

---

## Placeholders

| Placeholder | Origem | Exemplo |
|-------------|--------|---------|
| `{phase_number}` | Do roadmap/argumentos | `5` ou `2.1` |
| `{phase_dir}` | Nome do diretório da fase | `05-user-profiles` |
| `{phase}` | Prefixo da fase | `05` |
| `{standard \| gap_closure}` | Flag de modo | `standard` |

---

## Uso

**A partir de /gsd-plan-phase (modo padrão):**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-planner",
  description="Planejar Fase {phase}"
)
```

**A partir de /gsd-plan-phase --gaps (modo de fechamento de lacunas):**
```python
Task(
  prompt=filled_template,  # com mode: gap_closure
  subagent_type="gsd-planner",
  description="Planejar lacunas para a Fase {phase}"
)
```

---

## Continuação

Para checkpoints, spawne um novo agente com:

```markdown
<objective>
Continuar o planejamento para a Fase {phase_number}: {phase_name}
</objective>

<prior_state>
Diretório da fase: @.planning/phases/{phase_dir}/
Planos existentes: @.planning/phases/{phase_dir}/*-PLAN.md
</prior_state>

<checkpoint_response>
**Tipo:** {checkpoint_type}
**Resposta:** {user_response}
</checkpoint_response>

<mode>
Continuar: {standard | gap_closure}
</mode>
```

---

**Nota:** Metodologia de planejamento, divisão de tarefas, análise de dependências, atribuição de waves, detecção TDD e derivação retroativa a partir do objetivo estão incorporadas ao agente gsd-planner. Este template apenas passa o contexto.
