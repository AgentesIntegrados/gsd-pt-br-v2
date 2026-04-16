<purpose>
Exibir estatísticas abrangentes do projeto incluindo fases, planos, requisitos, métricas git e linha do tempo.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="gather_stats">
Reunir estatísticas do projeto:

```bash
STATS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" stats json)
if [[ "$STATS" == @file:* ]]; then STATS=$(cat "${STATS#@file:}"); fi
```

Extrair campos do JSON: `milestone_version`, `milestone_name`, `phases`, `phases_completed`, `phases_total`, `total_plans`, `total_summaries`, `percent`, `plan_percent`, `requirements_total`, `requirements_complete`, `git_commits`, `git_first_commit_date`, `last_activity`.
</step>

<step name="present_stats">
Apresentar ao usuário com este formato:

```
# 📊 Estatísticas do Projeto — {milestone_version} {milestone_name}

## Progresso
[████████░░] X/Y fases (Z%)

## Planos
X/Y planos concluídos (Z%)

## Fases
| Fase | Nome | Planos | Concluídos | Status |
|------|------|--------|-----------|--------|
| ...  | ...  | ...    | ...       | ...    |

## Requisitos
✅ X/Y requisitos concluídos

## Git
- **Commits:** N
- **Início:** AAAA-MM-DD
- **Última atividade:** AAAA-MM-DD

## Linha do Tempo
- **Idade do projeto:** N dias
```

Se nenhum diretório `.planning/` existir, informar o usuário para executar `/gsd-new-project` primeiro.
</step>

</process>

<success_criteria>
- [ ] Estatísticas coletadas do estado do projeto
- [ ] Resultados formatados de forma clara
- [ ] Exibidos ao usuário
</success_criteria>
