<purpose>
Pesquisa como implementar uma fase. Instancia o agente gsd-phase-researcher com o contexto da fase.

Comando de pesquisa independente. Para a maioria dos fluxos, use `/gsd-plan-phase` que integra a pesquisa automaticamente.
</purpose>

<available_agent_types>
Tipos de subagentes GSD válidos (use os nomes exatos — não use 'general-purpose' como alternativa):
- gsd-phase-researcher — Pesquisa abordagens técnicas para uma fase
</available_agent_types>

<process>

## Passo 0: Resolver Perfil de Modelo

@$HOME/.claude/get-shit-done/references/model-profile-resolution.md

Resolver modelo para:
- `gsd-phase-researcher`

## Passo 1: Normalizar e Validar a Fase

@$HOME/.claude/get-shit-done/references/phase-argument-parsing.md

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

Se `found` for false: exibir erro e sair.

## Passo 2: Verificar Pesquisa Existente

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null || true
```

Se existir: oferecer opções de atualizar/visualizar/pular.

## Passo 3: Coletar Contexto da Fase

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# Extrair: phase_dir, padded_phase, phase_number, state_path, requirements_path, context_path
AGENT_SKILLS_RESEARCHER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-researcher 2>/dev/null)
```

## Passo 4: Instanciar Pesquisador

```
Task(
  prompt="<objective>
Research implementation approach for Phase {phase}: {name}
</objective>

<files_to_read>
- {context_path} (USER DECISIONS from /gsd-discuss-phase)
- {requirements_path} (Project requirements)
- {state_path} (Project decisions and history)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<additional_context>
Phase description: {description}
</additional_context>

<output>
Write to: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</output>",
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}"
)
```

## Passo 5: Tratar Retorno

- `## RESEARCH COMPLETE` — Exibir resumo, oferecer: Planejar/Aprofundar/Revisar/Concluir
- `## CHECKPOINT REACHED` — Apresentar ao usuário, instanciar continuação
- `## RESEARCH INCONCLUSIVE` — Mostrar tentativas, oferecer: Adicionar contexto/Tentar modo diferente/Manual

</process>
