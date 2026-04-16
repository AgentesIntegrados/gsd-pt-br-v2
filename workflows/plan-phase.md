<purpose>
Criar prompts de fase executáveis (arquivos PLAN.md) para uma fase do roadmap com pesquisa e verificação integradas. Fluxo padrão: Pesquisa (se necessário) -> Planejamento -> Verificação -> Concluído. Orquestra os agentes gsd-phase-researcher, gsd-planner e gsd-plan-checker com um ciclo de revisão (máximo 3 iterações).
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

@$HOME/.claude/get-shit-done/references/ui-brand.md
@$HOME/.claude/get-shit-done/references/revision-loop.md
@$HOME/.claude/get-shit-done/references/gate-prompts.md
@$HOME/.claude/get-shit-done/references/agent-contracts.md
@$HOME/.claude/get-shit-done/references/gates.md
</required_reading>

<available_agent_types>
Tipos válidos de subagentes GSD (use os nomes exatos — não use 'general-purpose' como alternativa):
- gsd-phase-researcher — Pesquisa abordagens técnicas para uma fase
- gsd-pattern-mapper — Analisa a base de código em busca de padrões existentes, produz PATTERNS.md
- gsd-planner — Cria planos detalhados a partir do escopo da fase
- gsd-plan-checker — Revisa a qualidade do plano antes da execução
</available_agent_types>

<process>

## 1. Inicializar

Carregue todo o contexto em uma única chamada (apenas caminhos para minimizar o contexto do orquestrador):

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_RESEARCHER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-researcher 2>/dev/null)
AGENT_SKILLS_PLANNER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-planner 2>/dev/null)
AGENT_SKILLS_CHECKER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-checker 2>/dev/null)
CONTEXT_WINDOW=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get context_window 2>/dev/null || echo "200000")
TDD_MODE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.tdd_mode 2>/dev/null || echo "false")
```

Quando `TDD_MODE` for `true`, o agente planejador recebe instrução para aplicar `type: tdd` às tarefas elegíveis usando heurísticas de `references/tdd.md`. O `<required_reading>` do planejador é estendido para incluir `@$HOME/.claude/get-shit-done/references/tdd.md`, disponibilizando as regras de aplicação dos gates durante o planejamento.

Quando `CONTEXT_WINDOW >= 500000`, o prompt do planejador inclui os 3 arquivos CONTEXT.md e SUMMARY.md das fases anteriores mais recentes MAIS quaisquer fases explicitamente listadas no campo `Depends on:` da fase atual no ROADMAP.md. Dependências explícitas sempre carregam independentemente da recência (ex: Fase 7 declarando `Depends on: Phase 2` sempre vê o contexto da Fase 2). A recência limitada mantém o orçamento de contexto do planejador focado no trabalho recente.

Analise o JSON para: `researcher_model`, `planner_model`, `checker_model`, `research_enabled`, `plan_checker_enabled`, `nyquist_validation_enabled`, `commit_docs`, `text_mode`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_reviews`, `has_plans`, `plan_count`, `planning_exists`, `roadmap_exists`, `phase_req_ids`, `response_language`.

**Se `response_language` estiver definido:** Inclua `response_language: {value}` em todos os prompts de subagentes gerados para que qualquer saída voltada ao usuário permaneça no idioma configurado.

**Caminhos de arquivos (para blocos <files_to_read>):** `state_path`, `roadmap_path`, `requirements_path`, `context_path`, `research_path`, `verification_path`, `uat_path`, `reviews_path`. Estes são null caso os arquivos não existam.

**Se `planning_exists` for false:** Erro — execute `/gsd-new-project` primeiro.

## 2. Analisar e Normalizar Argumentos

Extraia de $ARGUMENTS: número da fase (inteiro ou decimal como `2.1`), flags (`--research`, `--skip-research`, `--gaps`, `--skip-verify`, `--skip-ui`, `--prd <filepath>`, `--reviews`, `--text`, `--bounce`, `--skip-bounce`).

Defina `TEXT_MODE=true` se `--text` estiver presente em $ARGUMENTS OU se `text_mode` do JSON de init for `true`. Quando `TEXT_MODE` estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário que digite o número de sua escolha. Isso é necessário para sessões remotas do Claude Code (modo `/rc`) onde menus TUI não funcionam através do Claude App.

Extraia `--prd <filepath>` de $ARGUMENTS. Se presente, defina PRD_FILE com o caminho do arquivo.

**Se não houver número de fase:** Detecte a próxima fase não planejada a partir do roadmap.

**Se `phase_found` for false:** Valide se a fase existe no ROADMAP.md. Se válida, crie o diretório usando `phase_slug` e `padded_phase` do init:
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```

**Artefatos existentes do init:** `has_research`, `has_plans`, `plan_count`.

## 2.5. Validar Pré-requisito `--reviews`

**Ignorar se:** Não há flag `--reviews`.

**Se `--reviews` E `--gaps`:** Erro — não é possível combinar `--reviews` com `--gaps`. Estes são modos conflitantes.

**Se `--reviews` E `has_reviews` for false (sem REVIEWS.md no diretório da fase):**

Erro:
```
No REVIEWS.md found for Phase {N}. Run reviews first:

/gsd-review --phase {N}

Then re-run /gsd-plan-phase {N} --reviews
```
Encerre o workflow.

## 3. Validar Fase

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

**Se `found` for false:** Erro com fases disponíveis. **Se `found` for true:** Extraia `phase_number`, `phase_name`, `goal` do JSON.

## 3.5. Tratar Caminho Expresso PRD

**Ignorar se:** Não há flag `--prd` nos argumentos.

**Se `--prd <filepath>` fornecido:**

1. Leia o arquivo PRD:
```bash
PRD_CONTENT=$(cat "$PRD_FILE" 2>/dev/null)
if [ -z "$PRD_CONTENT" ]; then
  echo "Error: PRD file not found: $PRD_FILE"
  exit 1
fi
```

2. Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PRD EXPRESS PATH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Using PRD: {PRD_FILE}
Generating CONTEXT.md from requirements...
```

3. Analise o conteúdo do PRD e gere o CONTEXT.md. O orquestrador deve:
   - Extrair todos os requisitos, histórias de usuário, critérios de aceite e restrições do PRD
   - Mapear cada um como uma decisão bloqueada (tudo no PRD é tratado como decisão bloqueada)
   - Identificar áreas que o PRD não cobre e marcar como "Claude's Discretion"
   - **Extrair refs canônicas** do ROADMAP.md para esta fase, mais quaisquer specs/ADRs referenciados no PRD — expandir para caminhos completos de arquivo (OBRIGATÓRIO)
   - Criar CONTEXT.md no diretório da fase

4. Escreva CONTEXT.md:
```markdown
# Phase [X]: [Name] - Context

**Gathered:** [date]
**Status:** Ready for planning
**Source:** PRD Express Path ({PRD_FILE})

<domain>
## Phase Boundary

[Extracted from PRD — what this phase delivers]

</domain>

<decisions>
## Implementation Decisions

{For each requirement/story/criterion in the PRD:}
### [Category derived from content]
- [Requirement as locked decision]

### Claude's Discretion
[Areas not covered by PRD — implementation details, technical choices]

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

[MANDATORY. Extract from ROADMAP.md and any docs referenced in the PRD.
Use full relative paths. Group by topic area.]

### [Topic area]
- `path/to/spec-or-adr.md` — [What it decides/defines]

[If no external specs: "No external specs — requirements fully captured in decisions above"]

</canonical_refs>

<specifics>
## Specific Ideas

[Any specific references, examples, or concrete requirements from PRD]

</specifics>

<deferred>
## Deferred Ideas

[Items in PRD explicitly marked as future/v2/out-of-scope]
[If none: "None — PRD covers phase scope"]

</deferred>

---

*Phase: XX-name*
*Context gathered: [date] via PRD Express Path*
```

5. Confirme:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): generate context from PRD" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

6. Defina `context_content` com o conteúdo do CONTEXT.md gerado e continue para o passo 5 (Tratar Pesquisa).

**Efeito:** Isso ignora completamente o passo 4 (Carregar CONTEXT.md) pois acabamos de criá-lo. O restante do workflow (pesquisa, planejamento, verificação) prossegue normalmente com o contexto derivado do PRD.

## 4. Carregar CONTEXT.md

**Ignorar se:** O caminho expresso PRD foi usado (CONTEXT.md já criado no passo 3.5).

Verifique `context_path` do JSON de init.

Se `context_path` não for null, exiba: `Using phase context from: ${context_path}`

**Se `context_path` for null (nenhum CONTEXT.md existe):**

Leia o modo discuss para o rótulo do gate de contexto:
```bash
DISCUSS_MODE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.discuss_mode 2>/dev/null || echo "discuss")
```

Se `TEXT_MODE` for true, apresente como lista numerada em texto simples:
```
No CONTEXT.md found for Phase {X}. Plans will use research and requirements only — your design preferences won't be included.

1. Continue without context — Plan using research + requirements only
[If DISCUSS_MODE is "assumptions":]
2. Gather context (assumptions mode) — Analyze codebase and surface assumptions before planning
[If DISCUSS_MODE is "discuss" or unset:]
2. Run discuss-phase first — Capture design decisions before planning

Enter number:
```

Caso contrário, use AskUserQuestion:
- header: "No context"
- question: "No CONTEXT.md found for Phase {X}. Plans will use research and requirements only — your design preferences won't be included. Continue or capture context first?"
- options:
  - "Continue without context" — Plan using research + requirements only
  Se `DISCUSS_MODE` for `"assumptions"`:
  - "Gather context (assumptions mode)" — Analyze codebase and surface assumptions before planning
  Se `DISCUSS_MODE` for `"discuss"` (ou não definido):
  - "Run discuss-phase first" — Capture design decisions before planning

Se "Continue without context": Avance para o passo 5.
Se "Run discuss-phase first":
  **IMPORTANTE:** NÃO invoque discuss-phase como uma chamada aninhada de Skill/Task — AskUserQuestion
  não funciona corretamente em subcontextos aninhados (#1009). Em vez disso, exiba o comando
  e encerre para que o usuário execute como um comando de nível superior:
  ```
  Run this command first, then re-run /gsd-plan-phase {X} ${GSD_WS}:

  /gsd-discuss-phase {X} ${GSD_WS}
  ```
  **Encerre o workflow plan-phase. Não continue.**

## 4.5. Verificar AI-SPEC

**Ignorar se:** `ai_integration_phase_enabled` da configuração for false, ou se a flag `--skip-ai-spec` estiver presente.

```bash
AI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-AI-SPEC.md 2>/dev/null | head -1)
AI_PHASE_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ai_integration_phase 2>/dev/null || echo "true")
```

**Ignorar se `AI_PHASE_CFG` for `false`.**

**Se `AI_SPEC_FILE` estiver vazio:** Verifique o objetivo da fase por palavras-chave de IA:
```bash
echo "${phase_goal}" | grep -qi "agent\|llm\|rag\|chatbot\|embedding\|langchain\|llamaindex\|crewai\|langgraph\|openai\|anthropic\|vector\|eval\|ai system"
```

**Se palavras-chave de IA foram detectadas E não há AI-SPEC.md:**
```
◆ Note: This phase appears to involve AI system development.
  Consider running /gsd-ai-integration-phase {N} before planning to:
  - Select the right framework for your use case
  - Research its docs and best practices
  - Design an evaluation strategy

  Continue planning without AI-SPEC? (non-blocking — /gsd-ai-integration-phase can be run after)
```

Use AskUserQuestion com as opções:
- "Continue — plan without AI-SPEC"
- "Stop — I'll run /gsd-ai-integration-phase {N} first"

Se "Stop": Encerre com lembrete de `/gsd-ai-integration-phase {N}`.
Se "Continue": Prossiga. (Não bloqueante — o planejador anotará que AI-SPEC está ausente.)

**Se `AI_SPEC_FILE` não estiver vazio:** Extraia o framework para o contexto do planejador:
```bash
FRAMEWORK_LINE=$(grep "Selected Framework:" "${AI_SPEC_FILE}" | head -1)
```
Passe `ai_spec_path` e `framework_line` para o planejador no passo 7 para que ele possa referenciar o contrato de design de IA.

## 5. Tratar Pesquisa

**Ignorar se:** Flag `--gaps`, flag `--skip-research` ou flag `--reviews`.

**Se `has_research` for true (do init) E não há flag `--research`:** Use a existente, pule para o passo 6.

**Se RESEARCH.md estiver ausente OU flag `--research` presente:**

**Se não houver flag explícita (`--research` ou `--skip-research`) e não for `--auto`:**
Pergunte ao usuário se deseja pesquisar, com uma recomendação contextual baseada na fase:

Se `TEXT_MODE` for true, apresente como lista numerada em texto simples:
```
Research before planning Phase {X}: {phase_name}?

1. Research first (Recommended) — Investigate domain, patterns, and dependencies before planning. Best for new features, unfamiliar integrations, or architectural changes.
2. Skip research — Plan directly from context and requirements. Best for bug fixes, simple refactors, or well-understood tasks.

Enter number:
```

Caso contrário, use AskUserQuestion:
```
AskUserQuestion([
  {
    question: "Research before planning Phase {X}: {phase_name}?",
    header: "Research",
    multiSelect: false,
    options: [
      { label: "Research first (Recommended)", description: "Investigate domain, patterns, and dependencies before planning. Best for new features, unfamiliar integrations, or architectural changes." },
      { label: "Skip research", description: "Plan directly from context and requirements. Best for bug fixes, simple refactors, or well-understood tasks." }
    ]
  }
])
```

Se o usuário selecionar "Skip research": pule para o passo 6.

**Se `--auto` e `research_enabled` for false:** Ignore a pesquisa silenciosamente (preserva o comportamento automatizado).

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCHING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning researcher...
```

### Gerar gsd-phase-researcher

```bash
PHASE_DESC=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" --pick section)
```

Prompt de pesquisa:

```markdown
<objective>
Research how to implement Phase {phase_number}: {phase_name}
Answer: "What do I need to know to PLAN this phase well?"
</objective>

<files_to_read>
- {context_path} (USER DECISIONS from /gsd-discuss-phase)
- {requirements_path} (Project requirements)
- {state_path} (Project decisions and history)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<additional_context>
**Phase description:** {phase_description}
**Phase requirement IDs (MUST address):** {phase_req_ids}

**Project instructions:** Read ./CLAUDE.md if exists — follow project-specific guidelines
**Project skills:** Check .claude/skills/ or .agents/skills/ directory (if either exists) — read SKILL.md files, research should account for project skill patterns
</additional_context>

<output>
Write to: {phase_dir}/{phase_num}-RESEARCH.md
</output>
```

```
Task(
  prompt=research_prompt,
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}",
  description="Research Phase {phase}"
)
```

### Tratar Retorno do Pesquisador

- **`## RESEARCH COMPLETE`:** Exiba confirmação, continue para o passo 6
- **`## RESEARCH BLOCKED`:** Exiba bloqueio, ofereça: 1) Fornecer contexto, 2) Pular pesquisa, 3) Abortar

## 5.5. Criar Estratégia de Validação

Ignorar se `nyquist_validation_enabled` for false OU `research_enabled` for false.

Se `research_enabled` for false e `nyquist_validation_enabled` for true: avise "Nyquist validation enabled but research disabled — VALIDATION.md cannot be created without RESEARCH.md. Plans will lack validation requirements (Dimension 8)." Continue para o passo 6.

**Mas o Nyquist não é aplicável nesta execução** quando todos os seguintes forem verdadeiros:
- `research_enabled` é false
- `has_research` é false
- nenhuma flag `--research` foi fornecida

Nesse caso: **ignore completamente a criação da estratégia de validação**. **Não** espere `RESEARCH.md` ou `VALIDATION.md` para esta execução, e continue para o Passo 6.

```bash
grep -l "## Validation Architecture" "${PHASE_DIR}"/*-RESEARCH.md 2>/dev/null || true
```

**Se encontrado:**
1. Leia o template: `$HOME/.claude/get-shit-done/templates/VALIDATION.md`
2. Escreva em `${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md` (use a ferramenta Write)
3. Preencha o frontmatter: `{N}` → número da fase, `{phase-slug}` → slug, `{date}` → data atual
4. Verifique:
```bash
test -f "${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md" && echo "VALIDATION_CREATED=true" || echo "VALIDATION_CREATED=false"
```
5. Se `VALIDATION_CREATED=false`: PARE — não prossiga para o Passo 6
6. Se `commit_docs`: `commit "docs(phase-${PHASE}): add validation strategy"`

**Se não encontrado:** Avise e continue — os planos podem falhar na Dimensão 8.

## 5.55. Gate do Modelo de Ameaças de Segurança

> Ignorar se `workflow.security_enforcement` for explicitamente `false`. Ausente = habilitado.

```bash
SECURITY_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.security_enforcement --raw 2>/dev/null || echo "true")
SECURITY_ASVS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.security_asvs_level --raw 2>/dev/null || echo "1")
SECURITY_BLOCK=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.security_block_on --raw 2>/dev/null || echo "high")
```

**Se `SECURITY_CFG` for `false`:** Pule para o passo 5.6.

**Se `SECURITY_CFG` for `true`:** Exiba o banner:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SECURITY THREAT MODEL REQUIRED (ASVS L{SECURITY_ASVS})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Each PLAN.md must include a <threat_model> block.
Block on: {SECURITY_BLOCK} severity threats.
Opt out: set security_enforcement: false in .planning/config.json
```

Continue para o passo 5.6. A configuração de segurança é passada para o planejador no passo 8.

## 5.6. Gate do Contrato de Design de UI

> Ignorar se `workflow.ui_phase` for explicitamente `false` E `workflow.ui_safety_gate` for explicitamente `false` em `.planning/config.json`. Se as chaves estiverem ausentes, trate como habilitado.

```bash
UI_PHASE_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_phase 2>/dev/null || echo "true")
UI_GATE_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_safety_gate 2>/dev/null || echo "true")
```

**Se ambos forem `false`:** Pule para o passo 6.

Verifique se a fase possui indicadores de frontend:

```bash
PHASE_SECTION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" 2>/dev/null)
echo "$PHASE_SECTION" | grep -iE "UI|interface|frontend|component|layout|page|screen|view|form|dashboard|widget" > /dev/null 2>&1
HAS_UI=$?
```

**Se `HAS_UI` for 0 (indicadores de frontend encontrados):**

Verifique se existe UI-SPEC:
```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
```

**Se UI-SPEC.md encontrado:** Defina `UI_SPEC_PATH=$UI_SPEC_FILE`. Exiba: `Using UI design contract: ${UI_SPEC_PATH}`

**Se UI-SPEC.md ausente E flag `--skip-ui` presente em $ARGUMENTS:** Pule silenciosamente para o passo 6.

**Se UI-SPEC.md ausente E `UI_GATE_CFG` for `true`:**

Leia o estado de encadeamento automático:
```bash
AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
```

**Se `AUTO_CHAIN` for `true` (executando dentro de um pipeline `--chain` ou `--auto`):**

Gere UI-SPEC automaticamente sem solicitar confirmação:
```
Skill(skill="gsd-ui-phase", args="${PHASE} --auto ${GSD_WS}")
```
Após o retorno de `gsd-ui-phase`, releia:
```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
UI_SPEC_PATH="${UI_SPEC_FILE}"
```
Continue para o passo 6.

**Se `AUTO_CHAIN` for `false` (invocação manual):**

Produza este markdown diretamente (não como bloco de código):

```
## ⚠ UI-SPEC.md missing for Phase {N}
▶ Recommended next step:
`/gsd-ui-phase {N} ${GSD_WS}` — generate UI design contract before planning
───────────────────────────────────────────────
Also available:
- `/gsd-plan-phase {N} --skip-ui ${GSD_WS}` — plan without UI-SPEC (not recommended for frontend phases)
```

**Encerre o workflow plan-phase. Não continue.**

**Se `HAS_UI` for 1 (sem indicadores de frontend):** Pule silenciosamente para o passo 5.7.

## 5.7. Gate de Detecção de Schema Push

> Detecta arquivos relevantes a schema no escopo da fase e injeta uma tarefa `[BLOCKING]` obrigatória de schema push no plano. Previne verificação com falso positivo onde build/types passam porque os tipos TypeScript vêm da configuração, não do banco de dados ativo.

Verifique se algum arquivo no escopo da fase corresponde a padrões de schema:

```bash
PHASE_SECTION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" --pick section 2>/dev/null)
```

Examine `PHASE_SECTION`, `CONTEXT.md` (se carregado) e `RESEARCH.md` (se existir) em busca de caminhos de arquivo que correspondam a estes padrões ORM:

| ORM | File Patterns |
|-----|--------------|
| Payload CMS | `src/collections/**/*.ts`, `src/globals/**/*.ts` |
| Prisma | `prisma/schema.prisma`, `prisma/schema/*.prisma` |
| Drizzle | `drizzle/schema.ts`, `src/db/schema.ts`, `drizzle/*.ts` |
| Supabase | `supabase/migrations/*.sql` |
| TypeORM | `src/entities/**/*.ts`, `src/migrations/**/*.ts` |

Também verifique se algum arquivo PLAN.md existente para esta fase já referencia esses padrões de arquivo em `files_modified`.

**Se arquivos relevantes a schema forem detectados:**

Defina `SCHEMA_PUSH_REQUIRED=true` e `SCHEMA_ORM={detected_orm}`.

Determine o comando de push para o ORM detectado:

| ORM | Push Command | Non-TTY Workaround |
|-----|-------------|-------------------|
| Payload CMS | `npx payload migrate` | `CI=true PAYLOAD_MIGRATING=true npx payload migrate` |
| Prisma | `npx prisma db push` | `npx prisma db push --accept-data-loss` (if destructive) |
| Drizzle | `npx drizzle-kit push` | `npx drizzle-kit push` |
| Supabase | `supabase db push` | Set `SUPABASE_ACCESS_TOKEN` env var |
| TypeORM | `npx typeorm migration:run` | `npx typeorm migration:run -d src/data-source.ts` |

Injete o seguinte no prompt do planejador (passo 8) como restrição adicional:

```markdown
<schema_push_requirement>
**[BLOCKING] Schema Push Required**

This phase modifies schema-relevant files ({detected_files}). The planner MUST include
a `[BLOCKING]` task that runs the database schema push command AFTER all schema file
modifications are complete but BEFORE verification.

- ORM detected: {SCHEMA_ORM}
- Push command: {push_command}
- Non-TTY workaround: {env_hint}
- If push requires interactive prompts that cannot be suppressed, flag the task for
  manual intervention with `autonomous: false`

This task is mandatory — the phase CANNOT pass verification without it. Build and
type checks will pass without the push (types come from config, not the live database),
creating a false-positive verification state.
</schema_push_requirement>
```

Exiba: `Schema files detected ({SCHEMA_ORM}) — [BLOCKING] push task will be injected into plans`

**Se nenhum arquivo relevante a schema for detectado:** Pule silenciosamente para o passo 6.

## 6. Verificar Planos Existentes

```bash
ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null || true
```

**Se existir E flag `--reviews` presente:** Ignore o prompt — vá diretamente para o replanejamento (o propósito de `--reviews` é replanejar com o feedback das revisões).

**Se existir E não houver flag `--reviews`:** Ofereça: 1) Adicionar mais planos, 2) Ver os existentes, 3) Replanejar do zero.

## 7. Usar Caminhos de Contexto do INIT

Extraia do JSON do INIT:

```bash
_gsd_field() { node -e "const o=JSON.parse(process.argv[1]); const v=o[process.argv[2]]; process.stdout.write(v==null?'':String(v))" "$1" "$2"; }
STATE_PATH=$(_gsd_field "$INIT" state_path)
ROADMAP_PATH=$(_gsd_field "$INIT" roadmap_path)
REQUIREMENTS_PATH=$(_gsd_field "$INIT" requirements_path)
RESEARCH_PATH=$(_gsd_field "$INIT" research_path)
VERIFICATION_PATH=$(_gsd_field "$INIT" verification_path)
UAT_PATH=$(_gsd_field "$INIT" uat_path)
CONTEXT_PATH=$(_gsd_field "$INIT" context_path)
REVIEWS_PATH=$(_gsd_field "$INIT" reviews_path)
PATTERNS_PATH=$(_gsd_field "$INIT" patterns_path)
```

## 7.5. Verificar Artefatos Nyquist

Ignorar se `nyquist_validation_enabled` for false OU `research_enabled` for false.

Também ignore se todos os seguintes forem verdadeiros:
- `research_enabled` é false
- `has_research` é false
- nenhuma flag `--research` foi fornecida

Nesse caminho sem pesquisa, os artefatos Nyquist **não são necessários** para esta execução.

```bash
VALIDATION_EXISTS=$(ls "${PHASE_DIR}"/*-VALIDATION.md 2>/dev/null | head -1)
```

Se ausente e o Nyquist ainda estiver habilitado/aplicável — pergunte ao usuário:
1. Reexecute: `/gsd-plan-phase {PHASE} --research ${GSD_WS}`
2. Desative o Nyquist com o comando exato:
   `node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow.nyquist_validation false`
3. Continue mesmo assim (os planos falharão na Dimensão 8)

Prossiga para o Passo 7.8 (ou Passo 8 se o mapeador de padrões estiver desabilitado) somente se o usuário selecionar 2 ou 3.

## 7.8. Gerar Agente gsd-pattern-mapper (Opcional)

**Ignore se** `workflow.pattern_mapper` estiver explicitamente definido como `false` no config.json (chave ausente = habilitado). Também ignore se não existir CONTEXT.md nem RESEARCH.md para esta fase (nada para extrair listas de arquivos).

Verifique a configuração:
```bash
PATTERN_MAPPER_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.pattern_mapper --default true 2>/dev/null)
```

**Se `PATTERN_MAPPER_CFG` for `false`:** Pule para o passo 8.

**Se PATTERNS.md já existir** (`PATTERNS_PATH` não vazio do passo 7): Pule para o passo 8 (use o existente).

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PATTERN MAPPING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning pattern mapper...
```

Prompt do mapeador de padrões:

```markdown
<pattern_mapping_context>
**Phase:** {phase_number} - {phase_name}
**Phase directory:** {phase_dir}
**Padded phase:** {padded_phase}

<files_to_read>
- {context_path} (USER DECISIONS from /gsd-discuss-phase)
- {research_path} (Technical Research)
</files_to_read>

**Output file:** {phase_dir}/{padded_phase}-PATTERNS.md

Extract the list of files to be created/modified from CONTEXT.md and RESEARCH.md. For each file, classify by role and data flow, find the closest existing analog in the codebase, extract concrete code excerpts, and produce PATTERNS.md.
</pattern_mapping_context>
```

Gere com:
```
Task(
  prompt="{above}",
  subagent_type="gsd-pattern-mapper",
  model="{researcher_model}",
)
```

**Tratar retorno:**
- **`## PATTERN MAPPING COMPLETE`:** Atualize `PATTERNS_PATH` para o caminho do arquivo criado, continue para o passo 8.
- **Qualquer erro ou retorno vazio:** Registre aviso, continue para o passo 8 sem padrões (não bloqueante).

Após a conclusão do mapeador de padrões, atualize a variável de caminho:
```bash
PATTERNS_PATH="${PHASE_DIR}/${PADDED_PHASE}-PATTERNS.md"
```

## 8. Gerar Agente gsd-planner

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PLANNING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning planner...
```

Prompt do planejador:

```markdown
<planning_context>
**Phase:** {phase_number}
**Mode:** {standard | gap_closure | reviews}

<files_to_read>
- {state_path} (Project State)
- {roadmap_path} (Roadmap)
- {requirements_path} (Requirements)
- {context_path} (USER DECISIONS from /gsd-discuss-phase)
- {research_path} (Technical Research)
- {PATTERNS_PATH} (Pattern Map — analog files and code excerpts, if exists)
- {verification_path} (Verification Gaps - if --gaps)
- {uat_path} (UAT Gaps - if --gaps)
- {reviews_path} (Cross-AI Review Feedback - if --reviews)
- {UI_SPEC_PATH} (UI Design Contract — visual/interaction specs, if exists)
${CONTEXT_WINDOW >= 500000 ? `
**Cross-phase context (1M model enrichment):**
- CONTEXT.md files from the 3 most recent completed phases (locked decisions — maintain consistency)
- SUMMARY.md files from the 3 most recent completed phases (what was built — reuse patterns, avoid duplication)
- CONTEXT.md and SUMMARY.md from any phases listed in the current phase's "Depends on:" field in ROADMAP.md (regardless of recency — explicit dependencies always load, deduplicated against the 3 most recent)
- Skip all other prior phases to stay within context budget
` : ''}
</files_to_read>

${AGENT_SKILLS_PLANNER}

**Phase requirement IDs (every ID MUST appear in a plan's `requirements` field):** {phase_req_ids}

**Project instructions:** Read ./CLAUDE.md if exists — follow project-specific guidelines
**Project skills:** Check .claude/skills/ or .agents/skills/ directory (if either exists) — read SKILL.md files, plans should account for project skill rules

${TDD_MODE === 'true' ? `
<tdd_mode_active>
**TDD Mode is ENABLED.** Apply TDD heuristics from @$HOME/.claude/get-shit-done/references/tdd.md to all eligible tasks:
- Business logic with defined I/O → type: tdd
- API endpoints with request/response contracts → type: tdd
- Data transformations, validation, algorithms → type: tdd
- UI, config, glue code, CRUD → standard plan (type: execute)
Each TDD plan gets one feature with RED/GREEN/REFACTOR gate sequence.
</tdd_mode_active>
` : ''}
</planning_context>

<downstream_consumer>
Output consumed by /gsd-execute-phase. Plans need:
- Frontmatter (wave, depends_on, files_modified, autonomous)
- Tasks in XML format with read_first and acceptance_criteria fields (MANDATORY on every task)
- Verification criteria
- must_haves for goal-backward verification
</downstream_consumer>

<deep_work_rules>
## Regras Anti-Execução Superficial (OBRIGATÓRIO)

Cada tarefa DEVE incluir estes campos — eles NÃO são opcionais:

1. **`<read_first>`** — Arquivos que o executor DEVE ler antes de tocar em qualquer coisa. Sempre inclua:
   - O arquivo sendo modificado (para que o executor veja o estado atual, não suposições)
   - Qualquer arquivo "fonte da verdade" referenciado no CONTEXT.md (implementações de referência, padrões existentes, arquivos de configuração, schemas)
   - Qualquer arquivo cujos padrões, assinaturas, tipos ou convenções devem ser replicados ou respeitados

2. **`<acceptance_criteria>`** — Condições verificáveis que provam que a tarefa foi feita corretamente. Regras:
   - Cada critério deve ser verificável com grep, leitura de arquivo, comando de teste ou saída de CLI
   - NUNCA use linguagem subjetiva ("parece correto", "configurado corretamente", "consistente com")
   - SEMPRE inclua strings exatas, padrões, valores ou saídas de comando que devem estar presentes
   - Exemplos:
     - Código: `auth.py contains def verify_token(` / `test_auth.py exits 0`
     - Config: `.env.example contains DATABASE_URL=` / `Dockerfile contains HEALTHCHECK`
     - Docs: `README.md contains '## Installation'` / `API.md lists all endpoints`
     - Infra: `deploy.yml has rollback step` / `docker-compose.yml has healthcheck for db`

3. **`<action>`** — Deve incluir valores CONCRETOS, não referências. Regras:
   - NUNCA diga "alinhe X com Y", "combine X com Y", "atualize para ser consistente" sem especificar o estado alvo exato
   - SEMPRE inclua os valores reais: chaves de configuração, assinaturas de função, instruções SQL, nomes de classe, caminhos de importação, variáveis de ambiente, etc.
   - Se CONTEXT.md tiver uma tabela comparativa ou valores esperados, copie-os verbatim na action
   - O executor deve ser capaz de completar a tarefa apenas com o texto da action, sem precisar ler CONTEXT.md ou arquivos de referência (read_first é para verificação, não descoberta)

**Por que isso importa:** Agentes executores trabalham a partir do texto do plano. Instruções vagas como "atualize a configuração para corresponder à produção" produzem mudanças de uma linha. Instruções concretas como "adicione DATABASE_URL=postgresql://... , defina POOL_SIZE=20, adicione REDIS_URL=redis://..." produzem trabalho completo. O custo de planos detalhados é muito menor que o custo de refazer uma execução superficial.
</deep_work_rules>

<quality_gate>
- [ ] Arquivos PLAN.md criados no diretório da fase
- [ ] Cada plano tem frontmatter válido
- [ ] As tarefas são específicas e acionáveis
- [ ] Cada tarefa tem `<read_first>` com pelo menos o arquivo sendo modificado
- [ ] Cada tarefa tem `<acceptance_criteria>` com condições verificáveis por grep
- [ ] Cada `<action>` contém valores concretos (sem "alinhe X com Y" sem especificar o quê)
- [ ] Dependências corretamente identificadas
- [ ] Waves atribuídas para execução paralela
- [ ] must_haves derivados do objetivo da fase
</quality_gate>
```

```
Task(
  prompt=filled_prompt,
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Plan Phase {phase}"
)
```

## 9. Tratar Retorno do Planejador

- **`## PLANNING COMPLETE`:** Exiba contagem de planos. Se `--skip-verify` ou `plan_checker_enabled` for false (do init): pule para o passo 13. Caso contrário: passo 10.
- **`## PHASE SPLIT RECOMMENDED`:** O planejador determinou que a fase excede o orçamento de contexto para implementação de alta fidelidade de todos os itens fonte. Trate no passo 9b.
- **`## ⚠ Source Audit: Unplanned Items Found`:** A auditoria de cobertura multi-fonte do planejador encontrou itens de REQUIREMENTS.md, RESEARCH.md, objetivo do ROADMAP ou decisões do CONTEXT.md que não estão cobertos por nenhum plano. Trate no passo 9c.
- **`## CHECKPOINT REACHED`:** Apresente ao usuário, obtenha resposta, gere continuação (passo 12)
- **`## PLANNING INCONCLUSIVE`:** Exiba tentativas, ofereça: Adicionar contexto / Tentar novamente / Manual

## 9b. Tratar Recomendação de Divisão de Fase

Quando o planejador retornar `## PHASE SPLIT RECOMMENDED`, significa que os itens fonte da fase excedem o orçamento de contexto para implementação de alta fidelidade. O planejador propõe agrupamentos.

**Extraia do retorno do planejador:**
- Sub-fases propostas (ex: "17a: processing core (D-01 to D-19)", "17b: billing + config UX (D-20 to D-27)")
- Quais itens fonte (REQ-IDs, decisões D-XX, itens de RESEARCH) vão em cada sub-fase
- Por que a divisão é necessária (estimativa de custo de contexto, contagem de arquivos)

**Apresente ao usuário:**
```
## Phase {X} exceeds context budget for full-fidelity implementation

The planner found {N} source items that exceed the context budget when
planned at full fidelity. Instead of reducing scope, we recommend splitting:

**Option 1: Split into sub-phases**
- Phase {X}a: {name} — {items} ({N} source items, ~{P}% context)
- Phase {X}b: {name} — {items} ({M} source items, ~{Q}% context)

**Option 2: Proceed anyway** (planner will attempt all, quality may degrade past 50% context)

**Option 3: Prioritize** — you choose which items to implement now,
rest become a follow-up phase
```

Use AskUserQuestion com essas 3 opções.

**Se "Split":** Use `/gsd-insert-phase` para criar as sub-fases, depois replaneje cada uma.
**Se "Proceed":** Retorne ao planejador com instrução para tentar todos os itens em alta fidelidade, aceitando mais planos/tarefas.
**Se "Prioritize":** Use AskUserQuestion (multiSelect) para deixar o usuário escolher quais itens são "agora" vs "depois". Crie CONTEXT.md para cada sub-fase com os itens selecionados.

## 9c. Tratar Lacunas da Auditoria de Fonte

Quando o planejador retornar `## ⚠ Source Audit: Unplanned Items Found`, significa que itens de REQUIREMENTS.md, RESEARCH.md, objetivo do ROADMAP ou decisões do CONTEXT.md não têm nenhum plano correspondente.

**Extraia do retorno do planejador:**
- Cada item não planejado com seu artefato fonte e seção
- As opções sugeridas pelo planejador (A: adicionar plano, B: dividir fase, C: adiar com confirmação)

**Apresente cada lacuna ao usuário.** Para cada item não planejado:

```
## ⚠ Unplanned: {item description}

Source: {RESEARCH.md / REQUIREMENTS.md / ROADMAP goal / CONTEXT.md}
Details: {why the planner flagged this}

Options:
1. Add a plan to cover this item (recommended)
2. Split phase — move to a sub-phase with related items
3. Defer — add to backlog (developer confirms this is intentional)
```

Use AskUserQuestion para cada lacuna (ou em lote se houver múltiplas).

**Se "Add plan":** Retorne ao planejador (passo 8) com instrução para adicionar planos cobrindo os itens ausentes, preservando os planos existentes.
**Se "Split":** Use `/gsd-insert-phase` para itens excedentes, depois replaneje.
**Se "Defer":** Registre no CONTEXT.md em `## Deferred Ideas` com a confirmação do desenvolvedor. Prossiga para o passo 10.

## 10. Gerar Agente gsd-plan-checker

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFYING PLANS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning plan checker...
```

Prompt do verificador:

```markdown
<verification_context>
**Phase:** {phase_number}
**Phase Goal:** {goal from ROADMAP}

<files_to_read>
- {PHASE_DIR}/*-PLAN.md (Plans to verify)
- {roadmap_path} (Roadmap)
- {requirements_path} (Requirements)
- {context_path} (USER DECISIONS from /gsd-discuss-phase)
- {research_path} (Technical Research — includes Validation Architecture)
</files_to_read>

${AGENT_SKILLS_CHECKER}

**Phase requirement IDs (MUST ALL be covered):** {phase_req_ids}

**Project instructions:** Read ./CLAUDE.md if exists — verify plans honor project guidelines
**Project skills:** Check .claude/skills/ or .agents/skills/ directory (if either exists) — verify plans account for project skill rules
</verification_context>

<expected_output>
- ## VERIFICATION PASSED — all checks pass
- ## ISSUES FOUND — structured issue list
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-plan-checker",
  model="{checker_model}",
  description="Verify Phase {phase} plans"
)
```

## 11. Tratar Retorno do Verificador

- **`## VERIFICATION PASSED`:** Exiba confirmação, prossiga para o passo 13.
- **`## ISSUES FOUND`:** Exiba problemas, verifique contagem de iterações, prossiga para o passo 12.

**Parceiro de raciocínio para trade-offs arquiteturais (condicional):**
Se `features.thinking_partner` estiver habilitado, examine os problemas do verificador em busca de palavras-chave de trade-off arquitetural
("architecture", "approach", "strategy", "pattern", "vs", "alternative"). Se encontrado:

```
The plan-checker flagged an architectural decision point:
{issue description}

Brief analysis:
- Option A: {approach_from_plan} — {pros/cons}
- Option B: {alternative_approach} — {pros/cons}
- Recommendation: {choice} aligned with {phase_goal}

Apply this to the revision? [Yes] / [No, I'll decide]
```

Se sim: inclua a recomendação no prompt de revisão. Se não: prossiga com o ciclo de revisão normalmente.
Se thinking_partner estiver desabilitado: ignore este bloco completamente.

## 12. Ciclo de Revisão (Máximo 3 Iterações)

Rastreie `iteration_count` (começa em 1 após o plano inicial + verificação).
Rastreie `prev_issue_count` (inicializado como `Infinity` antes do ciclo começar).
Rastreie `stall_reentry_count` (começa em 0; incrementado cada vez que "Adjust approach" reentra no passo 8).

**Se iteration_count < 3:**

Analise a contagem de problemas do retorno do verificador: conte entradas BLOCKER + WARNING no bloco YAML de problemas (saída estruturada do gsd-plan-checker). Se o retorno do verificador não contiver bloco YAML de problemas (ou seja, o plano foi aprovado sem problemas), trate `issue_count` como 0 e ignore a verificação de estagnação — o plano passou. Prossiga para o passo 13.

Exiba: `Revision iteration {N}/3 -- {blocker_count} blockers, {warning_count} warnings`

**Detecção de estagnação:** Se `issue_count >= prev_issue_count`:
  Exiba: `Revision loop stalled — issue count not decreasing ({issue_count} issues remain after {N} iterations)`

  **Se `stall_reentry_count < 2`:**
    Pergunte ao usuário:
      Question: "Issues remain after {N} revision attempts with no progress. Proceed with current output?"
      Options: "Proceed anyway" | "Adjust approach"
    Se "Proceed anyway": aceite os planos atuais e continue para o passo 13.
    Se "Adjust approach": incremente `stall_reentry_count`, abra discussão aberta, depois reentrada no passo 8 (replanejamento completo). Nota: a reentrada redefine `iteration_count` e `prev_issue_count`, mas `stall_reentry_count` persiste entre reentradas e é limitado a 2.

  **Se `stall_reentry_count >= 2`:**
    Exiba: `Stall persists after 2 re-planning attempts. The following issues could not be resolved automatically:`
    Liste os problemas restantes do verificador.
    Sugira: "Consider resolving these issues manually or running `/gsd-debug` to investigate root causes."
    Opções: "Proceed anyway" | "Abandon"
    Se "Proceed anyway": aceite os planos atuais e continue para o passo 13.
    Se "Abandon": pare o workflow.

Defina `prev_issue_count = issue_count`.

Prompt de revisão:

```markdown
<revision_context>
**Phase:** {phase_number}
**Mode:** revision

<files_to_read>
- {PHASE_DIR}/*-PLAN.md (Existing plans)
- {context_path} (USER DECISIONS from /gsd-discuss-phase)
</files_to_read>

${AGENT_SKILLS_PLANNER}

**Checker issues:** {structured_issues_from_checker}
</revision_context>

<instructions>
Make targeted updates to address checker issues.
Do NOT replan from scratch unless issues are fundamental.
Return what changed.
</instructions>
```

```
Task(
  prompt=revision_prompt,
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Revise Phase {phase} plans"
)
```

Após o planejador retornar -> gere o verificador novamente (passo 10), incremente iteration_count.

**Se iteration_count >= 3:**

Exiba: `Max iterations reached. {N} issues remain:` + lista de problemas

Ofereça: 1) Forçar prosseguimento, 2) Fornecer orientação e tentar novamente, 3) Abandonar

## 12.5. Plan Bounce (Refinamento Externo Opcional)

**Ignore se:** Flag `--skip-bounce`, flag `--gaps`, ou bounce não estiver ativado.

**Ativação:** O bounce é executado quando a flag `--bounce` estiver presente OU quando a configuração `workflow.plan_bounce` for `true`. A flag `--skip-bounce` sempre vence (desabilita o bounce mesmo se a configuração o habilitar). A flag `--gaps` também desabilita o bounce (o modo de fechamento de lacunas não deve modificar planos externamente).

**Pré-requisitos:** `workflow.plan_bounce_script` deve estar definido como um caminho de script válido. Se o bounce estiver ativado mas nenhum script estiver configurado, exiba um aviso e ignore:
```
⚠ Plan bounce activated but no script configured.
Set workflow.plan_bounce_script to the path of your refinement script.
Skipping bounce step.
```

**Leia a contagem de passes:**
```bash
BOUNCE_PASSES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.plan_bounce_passes --default 2)
BOUNCE_SCRIPT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.plan_bounce_script)
```

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► BOUNCING PLANS (External Refinement)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Script: ${BOUNCE_SCRIPT}
Max passes: ${BOUNCE_PASSES}
```

**Para cada arquivo PLAN.md no diretório da fase:**

1. **Backup:** Copie `*-PLAN.md` para `*-PLAN.pre-bounce.md`
```bash
cp "${PLAN_FILE}" "${PLAN_FILE%.md}.pre-bounce.md"
```

2. **Invoque o script de bounce:**
```bash
"${BOUNCE_SCRIPT}" "${PLAN_FILE}" "${BOUNCE_PASSES}"
```

3. **Valide o plano após bounce — integridade do frontmatter YAML:**
Após o script retornar, verifique se o arquivo pós-bounce ainda tem frontmatter YAML válido (delimitadores `---` de abertura e fechamento com conteúdo analisável entre eles). Se o plano pós-bounce quebrar a validação do frontmatter YAML, restaure o original do backup pre-bounce.md e continue para o próximo plano:
```
⚠ Bounced plan ${PLAN_FILE} has broken YAML frontmatter — restoring original from pre-bounce backup.
```

4. **Trate falha do script:** Se o script de bounce sair com código diferente de zero, restaure o plano original do backup pre-bounce.md e continue para o próximo plano:
```
⚠ Bounce script failed for ${PLAN_FILE} (exit code ${EXIT_CODE}) — restoring original from pre-bounce backup.
```

**Após todos os planos serem processados pelo bounce:**

5. **Execute novamente o verificador nos planos pós-bounce:** Gere gsd-plan-checker (igual ao passo 10) em todos os planos modificados. Se um plano pós-bounce falhar na verificação, restaure o original do backup pre-bounce.md:
```
⚠ Bounced plan ${PLAN_FILE} failed checker validation — restoring original from pre-bounce backup.
```

6. **Confirme os planos pós-bounce que sobreviveram:** Se pelo menos um plano sobreviveu tanto à validação do frontmatter quanto à nova execução do verificador, confirme as mudanças:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "refactor(${padded_phase}): bounce plans through external refinement" --files "${PHASE_DIR}/*-PLAN.md"
```

Exiba o resumo:
```
Plan bounce complete: {survived}/{total} plans refined
```

**Limpeza:** Remova todos os arquivos de backup `*-PLAN.pre-bounce.md` após a conclusão do passo de bounce (independentemente de os planos terem sobrevivido ou sido restaurados).

## 13. Gate de Cobertura de Requisitos

Após os planos passarem pelo verificador (ou o verificador ser ignorado), verifique se todos os requisitos da fase estão cobertos por pelo menos um plano.

**Ignore se:** `phase_req_ids` for null ou TBD (nenhum requisito mapeado para esta fase).

**Passo 1: Extraia IDs de requisitos reivindicados pelos planos**
```bash
# Collect all requirement IDs from plan frontmatter
PLAN_REQS=$(grep -h "requirements_addressed\|requirements:" ${PHASE_DIR}/*-PLAN.md 2>/dev/null | tr -d '[]' | tr ',' '\n' | sed 's/^[[:space:]]*//' | sort -u)
```

**Passo 2: Compare com os requisitos da fase do ROADMAP**

Para cada REQ-ID em `phase_req_ids`:
- Se REQ-ID aparecer em `PLAN_REQS` → coberto ✓
- Se REQ-ID NÃO aparecer em nenhum plano → não coberto ✗

**Passo 3: Verifique features do CONTEXT.md contra objetivos do plano**

Leia a seção `<decisions>` do CONTEXT.md. Extraia nomes de features/capacidades. Verifique cada um contra blocos `<objective>` do plano. Features não mencionadas em nenhum objetivo de plano → potencialmente removidas.

**Passo 4: Relatório**

Se todos os requisitos estiverem cobertos e não houver features removidas:
```
✓ Requirements coverage: {N}/{N} REQ-IDs covered by plans
```
→ Prossiga para o passo 14.

Se houver lacunas:
```
## ⚠ Requirements Coverage Gap

{M} of {N} phase requirements are not assigned to any plan:

| REQ-ID | Description | Plans |
|--------|-------------|-------|
| {id} | {from REQUIREMENTS.md} | None |

{K} CONTEXT.md features not found in plan objectives:
- {feature_name} — described in CONTEXT.md but no plan covers it

Options:
1. Re-plan to include missing requirements (recommended)
2. Move uncovered requirements to next phase
3. Proceed anyway — accept coverage gaps
```

Se `TEXT_MODE` for true, apresente como lista numerada em texto simples (as opções já estão mostradas no bloco acima). Caso contrário, use AskUserQuestion para apresentar as opções.

## 13b. Registrar Conclusão do Planejamento no STATE.md

Após os planos passarem por todos os gates, registre que o planejamento está completo para que STATE.md reflita o novo status da fase:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state planned-phase --phase "${PHASE_NUMBER}" --name "${PHASE_NAME}" --plans "${PLAN_COUNT}"
```

Isso atualiza STATUS para "Ready to execute", define a contagem correta de planos e marca o timestamp de Last Activity.

## 14. Apresentar Status Final

Encaminhe para `<offer_next>` OU `auto_advance` dependendo das flags/configuração.

## 15. Verificação de Avanço Automático

Verifique o gatilho de avanço automático:

1. Analise as flags `--auto` e `--chain` de $ARGUMENTS
2. **Sincronize a flag de encadeamento com a intenção** — se o usuário invocou manualmente (sem `--auto` e sem `--chain`), limpe a flag efêmera de encadeamento de qualquer cadeia `--auto` interrompida anteriormente. Isso NÃO toca em `workflow.auto_advance` (a preferência de configuração persistente do usuário):
   ```bash
   if [[ ! "$ARGUMENTS" =~ --auto ]] && [[ ! "$ARGUMENTS" =~ --chain ]]; then
     node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. Leia tanto a flag de encadeamento quanto a preferência do usuário:
   ```bash
   AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se a flag `--auto` ou `--chain` estiver presente E `AUTO_CHAIN` não for true:** Persista a flag de encadeamento na configuração (trata invocação direta sem discuss-phase anterior):
```bash
if ([[ "$ARGUMENTS" =~ --auto ]] || [[ "$ARGUMENTS" =~ --chain ]]) && [[ "$AUTO_CHAIN" != "true" ]]; then
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
fi
```

**Se a flag `--auto` ou `--chain` estiver presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true:**

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTO-ADVANCING TO EXECUTE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Plans ready. Launching execute-phase...
```

Lance execute-phase usando a ferramenta Skill para evitar sessões Task aninhadas (que causam travamentos de runtime devido ao aninhamento profundo de agentes):
```
Skill(skill="gsd-execute-phase", args="${PHASE} --auto --no-transition ${GSD_WS}")
```

A flag `--no-transition` instrui o execute-phase a retornar o status após a verificação em vez de encadear mais adiante. Isso mantém a cadeia de avanço automático plana — cada fase é executada no mesmo nível de aninhamento em vez de gerar agentes Task mais profundos.

**Trate o retorno do execute-phase:**
- **PHASE COMPLETE** → Exiba o resumo final:
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► PHASE ${PHASE} COMPLETE ✓
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Auto-advance pipeline finished.

  Next: /gsd-discuss-phase ${NEXT_PHASE} --auto ${GSD_WS}
  ```
- **GAPS FOUND / VERIFICATION FAILED** → Exiba o resultado, pare a cadeia:
  ```
  Auto-advance stopped: Execution needs review.

  Review the output above and continue manually:
  /gsd-execute-phase ${PHASE} ${GSD_WS}
  ```

**Se nem `--auto` nem a configuração estiver habilitada:**
Encaminhe para `<offer_next>` (comportamento existente).

</process>

<offer_next>
Produza este markdown diretamente (não como bloco de código):

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PHASE {X} PLANNED ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {N} plan(s) in {M} wave(s)

| Wave | Plans | What it builds |
|------|-------|----------------|
| 1    | 01, 02 | [objectives] |
| 2    | 03     | [objective]  |

Research: {Completed | Used existing | Skipped}
Verification: {Passed | Passed with override | Skipped}

───────────────────────────────────────────────────────────────

## ▶ Próximos Passos

**Execute a Fase {X}** — execute todos os {N} planos

/clear e então:

/gsd-execute-phase {X} ${GSD_WS}

───────────────────────────────────────────────────────────────

**Também disponível:**
- cat .planning/phases/{phase-dir}/*-PLAN.md — revisar planos
- /gsd-plan-phase {X} --research — pesquisar novamente primeiro
- /gsd-review --phase {X} --all — revisão por pares dos planos com IAs externas
- /gsd-plan-phase {X} --reviews — replanejar incorporando o feedback das revisões

───────────────────────────────────────────────────────────────
</offer_next>

<windows_troubleshooting>
**Usuários Windows:** Se o plan-phase travar durante a geração de agentes (comum no Windows devido a
deadlocks de stdio com servidores MCP — veja o issue do Claude Code anthropics/claude-code#28126):

1. **Force-kill:** Feche o terminal (Ctrl+C pode não funcionar)
2. **Limpe processos órfãos:**
   ```powershell
   # Kill orphaned node processes from stale MCP servers
   Get-Process node -ErrorAction SilentlyContinue | Where-Object {$_.StartTime -lt (Get-Date).AddHours(-1)} | Stop-Process -Force
   ```
3. **Limpe diretórios de tarefas obsoletos:**
   ```powershell
   # Remove stale subagent task dirs (Claude Code never cleans these on crash)
   Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\tasks\*" -ErrorAction SilentlyContinue
   ```
4. **Reduza a contagem de servidores MCP:** Desabilite temporariamente servidores MCP não essenciais no settings.json
5. **Tente novamente:** Reinicie o Claude Code e execute `/gsd-plan-phase` novamente

Se os travamentos persistirem, tente `--skip-research` para reduzir a cadeia de agentes de 3 para 2:
```
/gsd-plan-phase N --skip-research
```
</windows_troubleshooting>

<success_criteria>
- [ ] Diretório .planning/ validado
- [ ] Fase validada contra o roadmap
- [ ] Diretório da fase criado se necessário
- [ ] CONTEXT.md carregado cedo (passo 4) e passado para TODOS os agentes
- [ ] Pesquisa concluída (a menos que --skip-research ou --gaps ou já exista)
- [ ] gsd-phase-researcher gerado com CONTEXT.md
- [ ] Planos existentes verificados
- [ ] gsd-planner gerado com CONTEXT.md + RESEARCH.md
- [ ] Planos criados (PLANNING COMPLETE ou CHECKPOINT tratado)
- [ ] gsd-plan-checker gerado com CONTEXT.md
- [ ] Verificação passou OU override do usuário OU máximo de iterações com decisão do usuário
- [ ] Usuário vê status entre gerações de agentes
- [ ] Usuário conhece os próximos passos
</success_criteria>
