---
name: gsd-pesquisar-fase
description: "Pesquisar como implementar uma fase (standalone - normalmente use /gsd-plan-phase)"
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Bash
  - Task
---


<objective>
Pesquisar como implementar uma fase. Cria o agente gsd-phase-researcher com o contexto da fase.

**Nota:** Este é um comando de pesquisa standalone. Para a maioria dos fluxos de trabalho, use `/gsd-plan-phase` que integra a pesquisa automaticamente.

**Use este comando quando:**
- Você quer pesquisar sem planejar ainda
- Você quer fazer nova pesquisa depois que o planejamento estiver completo
- Você precisa investigar antes de decidir se uma fase é viável

**Papel do orquestrador:** Analisar a fase, validar contra o roadmap, verificar pesquisas existentes, coletar contexto, criar agente pesquisador, apresentar resultados.

**Por que subagente:** Pesquisa consome contexto rapidamente (WebSearch, consultas ao Context7, verificação de fontes). Contexto limpo de 200k para investigação. O contexto principal permanece enxuto para interação com o usuário.
</objective>

<available_agent_types>
Valid GSD subagent types (use exact names — do not fall back to 'general-purpose'):
- gsd-phase-researcher — Researches technical approaches for a phase
</available_agent_types>

<context>
Número da fase: $ARGUMENTS (obrigatório)

Normalizar a entrada da fase na etapa 1 antes de qualquer busca de diretório.
</context>

<process>

## 0. Inicializar Contexto

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "$ARGUMENTS")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `phase_dir`, `phase_number`, `phase_name`, `phase_found`, `commit_docs`, `has_research`, `state_path`, `requirements_path`, `context_path`, `research_path`.

Resolver modelo do pesquisador:
```bash
RESEARCHER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-phase-researcher --raw)
```

## 1. Validar Fase

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}")
```

**Se `found` for falso:** Exibir erro e encerrar. **Se `found` for verdadeiro:** Extrair `phase_number`, `phase_name`, `goal` do JSON.

## 2. Verificar Pesquisa Existente

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null
```

**Se existir:** Oferecer: 1) Atualizar pesquisa, 2) Ver existente, 3) Pular. Aguardar resposta.

**Se não existir:** Continuar.

## 3. Coletar Contexto da Fase

Usar caminhos do INIT (não incluir conteúdo dos arquivos inline no contexto do orquestrador):
- `requirements_path`
- `context_path`
- `state_path`

Apresentar resumo com a descrição da fase e quais arquivos o pesquisador carregará.

## 4. Criar Agente gsd-phase-researcher

Modos de pesquisa: ecosystem (padrão), feasibility, implementation, comparison.

```markdown
<research_type>
Phase Research — investigating HOW to implement a specific phase well.
</research_type>

<key_insight>
The question is NOT "which library should I use?"

The question is: "What do I not know that I don't know?"

For this phase, discover:
- What's the established architecture pattern?
- What libraries form the standard stack?
- What problems do people commonly hit?
- What's SOTA vs what Claude's training thinks is SOTA?
- What should NOT be hand-rolled?
</key_insight>

<objective>
Research implementation approach for Phase {phase_number}: {phase_name}
Mode: ecosystem
</objective>

<files_to_read>
- {requirements_path} (Requirements)
- {context_path} (Phase context from discuss-phase, if exists)
- {state_path} (Prior project decisions and blockers)
</files_to_read>

<additional_context>
**Phase description:** {phase_description}
</additional_context>

<downstream_consumer>
Your RESEARCH.md will be loaded by `/gsd-plan-phase` which uses specific sections:
- `## Standard Stack` → Plans use these libraries
- `## Architecture Patterns` → Task structure follows these
- `## Don't Hand-Roll` → Tasks NEVER build custom solutions for listed problems
- `## Common Pitfalls` → Verification steps check for these
- `## Code Examples` → Task actions reference these patterns

Be prescriptive, not exploratory. "Use X" not "Consider X or Y."
</downstream_consumer>

<quality_gate>
Before declaring complete, verify:
- [ ] All domains investigated (not just some)
- [ ] Negative claims verified with official docs
- [ ] Multiple sources for critical claims
- [ ] Confidence levels assigned honestly
- [ ] Section names match what plan-phase expects
</quality_gate>

<output>
Write to: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</output>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  description="Research Phase {phase}"
)
```

## 5. Tratar Retorno do Agente

**`## RESEARCH COMPLETE`:** Exibir resumo, oferecer: Planejar fase, Aprofundar pesquisa, Revisar completo, Concluído.

**`## CHECKPOINT REACHED`:** Apresentar ao usuário, obter resposta, criar agente de continuação.

**`## RESEARCH INCONCLUSIVE`:** Mostrar o que foi tentado, oferecer: Adicionar contexto, Tentar modo diferente, Manual.

## 6. Criar Agente de Continuação

```markdown
<objective>
Continue research for Phase {phase_number}: {phase_name}
</objective>

<prior_state>
<files_to_read>
- .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md (Existing research)
</files_to_read>
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>
```

```
Task(
  prompt=continuation_prompt,
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  description="Continue research Phase {phase}"
)
```

</process>

<success_criteria>
- [ ] Fase validada contra o roadmap
- [ ] Pesquisa existente verificada
- [ ] gsd-phase-researcher criado com contexto
- [ ] Checkpoints tratados corretamente
- [ ] Usuário informado sobre próximas etapas
</success_criteria>
