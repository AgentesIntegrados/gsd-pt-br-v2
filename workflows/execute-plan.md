<purpose>
Executa um prompt de fase (PLAN.md) e cria o resumo de resultado (SUMMARY.md).
</purpose>

<required_reading>
Leia STATE.md antes de qualquer operação para carregar o contexto do projeto.
Leia config.json para as configurações de comportamento do planejamento.

@$HOME/.claude/get-shit-done/references/git-integration.md
</required_reading>

<available_agent_types>
Tipos válidos de subagente GSD (use nomes exatos — não recorra a 'general-purpose'):
- gsd-executor — Executa tarefas do plano, faz commits, cria SUMMARY.md
</available_agent_types>

<process>

<step name="init_context" priority="first">
Carregue o contexto de execução (apenas caminhos para minimizar o contexto do orquestrador):

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extraia do JSON init: `executor_model`, `commit_docs`, `sub_repos`, `phase_dir`, `phase_number`, `plans`, `summaries`, `incomplete_plans`, `state_path`, `config_path`.

Se `.planning/` estiver ausente: erro.
</step>

<step name="identify_plan">
```bash
# Use plans/summaries do JSON INIT, ou liste os arquivos
(ls .planning/phases/XX-name/*-PLAN.md 2>/dev/null || true) | sort
(ls .planning/phases/XX-name/*-SUMMARY.md 2>/dev/null || true) | sort
```

Encontre o primeiro PLAN sem SUMMARY correspondente. Fases decimais suportadas (`01.1-hotfix/`):

```bash
PHASE=$(echo "$PLAN_PATH" | grep -oE '[0-9]+(\.[0-9]+)?-[0-9]+')
# configurações do config podem ser obtidas via gsd-tools config-get se necessário
```

<if mode="yolo">
Aprovação automática: `⚡ Execute {phase}-{plan}-PLAN.md [Plan X of Y for Phase Z]` → parse_segments.
</if>

<if mode="interactive" OR="custom with gates.execute_next_plan true">
Apresente a identificação do plano e aguarde confirmação.
</if>
</step>

<step name="record_start_time">
```bash
PLAN_START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_START_EPOCH=$(date +%s)
```
</step>

<step name="parse_segments">
```bash
# Conte as tarefas — encontre a tag <task em qualquer nível de indentação
TASK_COUNT=$(grep -cE '^\s*<task[[:space:]>]' .planning/phases/XX-name/{phase}-{plan}-PLAN.md 2>/dev/null || echo "0")
INLINE_THRESHOLD=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.inline_plan_threshold --default 2 2>/dev/null || echo "2")
grep -n "type=\"checkpoint" .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```

**Roteamento primário: limiar de contagem de tarefas (#1979)**

Se `INLINE_THRESHOLD > 0` E `TASK_COUNT <= INLINE_THRESHOLD`: Use o Padrão C (inline) independentemente do tipo de checkpoint. Planos pequenos executam mais rápido inline — evita o overhead de ~14K tokens do spawn de subagente e preserva o cache do prompt. Configure o limiar via `workflow.inline_plan_threshold` (padrão: 2, defina como `0` para sempre usar subagentes).

Caso contrário: aplique o roteamento baseado em checkpoints abaixo.

**Roteamento baseado em checkpoints (planos com mais tarefas que o limiar):**

| Checkpoints | Padrão | Execução |
|-------------|---------|-----------|
| Nenhum | A (autônomo) | Subagente único: plano completo + SUMMARY + commit |
| Somente verify | B (segmentado) | Segmentos entre checkpoints. Após none/human-verify → SUBAGENT. Após decision/human-action → MAIN |
| Decision | C (main) | Executa inteiramente no contexto principal |

**Padrão A:** init_agent_tracking → captura `EXPECTED_BASE=$(git rev-parse HEAD)` → spawna Task(subagent_type="gsd-executor", model=executor_model) com prompt: execute o plano em [path], autônomo, todas as tarefas + SUMMARY + commit, siga regras de desvio/autenticação, reporte: nome do plano, tarefas, caminho do SUMMARY, hash do commit → rastreie agent_id → aguarde → atualize rastreamento → reporte. **Inclua `isolation="worktree"` somente se `workflow.use_worktrees` não for `false`** (leia via `config-get workflow.use_worktrees`). **Ao usar `isolation="worktree"`, inclua um bloco `<worktree_branch_check>` no prompt** instruindo o executor a executar `git merge-base HEAD {EXPECTED_BASE}` e, se o resultado diferir de `{EXPECTED_BASE}`, fazer hard-reset do branch com `git reset --hard {EXPECTED_BASE}` antes de iniciar o trabalho (seguro — executa antes de qualquer trabalho do agente), depois verificar com `[ "$(git rev-parse HEAD)" != "{EXPECTED_BASE}" ] && exit 1`. Isso corrige um problema conhecido onde `EnterWorktree` cria branches a partir de `main` em vez do HEAD do branch de feature (afeta todas as plataformas).

**Padrão B:** Execute segmento por segmento. Segmentos autônomos: spawne subagente para as tarefas atribuídas apenas (sem SUMMARY/commit). Checkpoints: contexto principal. Após todos os segmentos: agregue arquivos/desvios/decisões → crie SUMMARY → commit → self-check:
   - Verifique se key-files.created existem no disco com `[ -f ]`
   - Verifique `git log --oneline --all --grep="{phase}-{plan}"` retorna ≥1 commit
   - Execute TODOS os `<acceptance_criteria>` de cada tarefa novamente — se algum falhar, corrija antes de finalizar o SUMMARY
   - Execute novamente os comandos `<verification>` do nível do plano — registre os resultados no SUMMARY
   - Anexe `## Self-Check: PASSED` ou `## Self-Check: FAILED` ao SUMMARY

**Padrão C:** Execute no contexto principal usando o fluxo padrão (step name="execute").

Contexto fresco por subagente preserva a qualidade máxima. O contexto principal permanece enxuto.
</step>

<step name="init_agent_tracking">
```bash
if [ ! -f .planning/agent-history.json ]; then
  echo '{"version":"1.0","max_entries":50,"entries":[]}' > .planning/agent-history.json
fi
rm -f .planning/current-agent-id.txt
if [ -f .planning/current-agent-id.txt ]; then
  INTERRUPTED_ID=$(cat .planning/current-agent-id.txt)
  echo "Found interrupted agent: $INTERRUPTED_ID"
fi
```

Se interrompido: pergunte ao usuário se deseja retomar (parâmetro `resume` da Task) ou iniciar do zero.

**Protocolo de rastreamento:** No spawn: escreva o agent_id em `current-agent-id.txt`, anexe a agent-history.json: `{"agent_id":"[id]","task_description":"[desc]","phase":"[phase]","plan":"[plan]","segment":[num|null],"timestamp":"[ISO]","status":"spawned","completion_timestamp":null}`. Na conclusão: status → "completed", defina completion_timestamp, delete current-agent-id.txt. Poda: se entries > max_entries, remova o "completed" mais antigo (nunca "spawned").

Execute para Padrão A/B antes do spawn. Padrão C: ignore.
</step>

<step name="segment_execution">
Somente Padrão B (checkpoints verify-only). Ignore para A/C.

1. Analise o mapa de segmentos: localizações e tipos dos checkpoints
2. Por segmento:
   - Rota subagente: spawne gsd-executor para as tarefas atribuídas apenas. Prompt: intervalo de tarefas, caminho do plano, leia o plano completo para contexto, execute as tarefas atribuídas, rastreie desvios, SEM SUMMARY/commit. Rastreie via protocolo de agente.
   - Rota principal: execute tarefas usando o fluxo padrão (step name="execute")
3. Após TODOS os segmentos: agregue arquivos/desvios/decisões → crie SUMMARY.md → commit → self-check:
   - Verifique se key-files.created existem no disco com `[ -f ]`
   - Verifique `git log --oneline --all --grep="{phase}-{plan}"` retorna ≥1 commit
   - Execute novamente TODOS os `<acceptance_criteria>` de cada tarefa — se algum falhar, corrija antes de finalizar o SUMMARY
   - Execute novamente os comandos `<verification>` do nível do plano — registre os resultados no SUMMARY
   - Anexe `## Self-Check: PASSOU` ou `## Self-Check: FALHOU` ao SUMMARY

   **Bug conhecido do Claude Code (classifyHandoffIfNeeded):** Se algum agente de segmento reportar "failed" com `classifyHandoffIfNeeded is not defined`, isso é um bug de runtime do Claude Code — não uma falha real. Execute verificações pontuais; se passarem, trate como bem-sucedido.




</step>

<step name="load_prompt">
```bash
cat .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```
Este É o conjunto de instruções de execução. Siga exatamente. Se o plano referencia CONTEXT.md: respeite a visão do usuário durante todo o processo.

**Se o plano contiver um bloco `<interfaces>`:** Estas são definições de tipo e contratos pré-extraídos. Use-os diretamente — NÃO releia os arquivos de origem para descobrir tipos. O planejador já extraiu o que você precisa.
</step>

<step name="previous_phase_check">
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases list --type summaries --raw
# Extraia o segundo SUMMARY mais recente do resultado JSON
```

**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON init for `true`. Quando TEXT_MODE estiver ativo, substitua toda chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Se o SUMMARY anterior tiver "Issues Encountered" ou bloqueadores em "Next Phase Readiness" não resolvidos: AskUserQuestion(header="Problemas Anteriores", options: "Prosseguir mesmo assim" | "Tratar primeiro" | "Revisar anterior").
</step>

<step name="execute">
Desvios são normais — trate-os pelas regras abaixo.

1. Leia os arquivos @context do prompt
2. **Ferramentas MCP:** Se CLAUDE.md ou as instruções do projeto referenciarem ferramentas MCP (ex.: jCodeMunch para navegação de código), prefira-as em vez de Grep/Glob quando disponíveis. Recorra a Grep/Glob se as ferramentas MCP não estiverem acessíveis.
3. Por tarefa:
   - **Gate obrigatório read_first:** Se a tarefa tiver um campo `<read_first>`, você DEVE ler cada arquivo listado ANTES de fazer qualquer edição. Isso não é opcional. Não pule arquivos porque você "já sabe" o que está neles — leia-os. Os arquivos read_first estabelecem a fonte verdadeira para a tarefa.
   - `type="auto"`: se `tdd="true"` → execução TDD. Implemente com regras de desvio + gates de autenticação. Verifique os critérios de conclusão. Faça commit (veja task_commit). Rastreie o hash para o Summary.
   - `type="checkpoint:*"`: PARE → checkpoint_protocol → aguarde o usuário → continue apenas após confirmação.
   - **GATE RÍGIDO — verificação de acceptance_criteria:** Após concluir cada tarefa, se ela tiver `<acceptance_criteria>`, você DEVE executar um loop de verificação antes de prosseguir:
     1. Para cada critério: execute o grep, verificação de arquivo ou comando CLI que prove que passou
     2. Registre cada resultado como PASSOU ou FALHOU com a saída do comando
     3. Se QUALQUER critério falhar: corrija a implementação imediatamente, depois execute TODOS os critérios novamente
     4. Repita até que todos os critérios passem — você está BLOQUEADO de iniciar a próxima tarefa até que este gate seja liberado
     5. Se um critério não puder ser satisfeito após 2 tentativas de correção, registre-o como desvio com justificativa — NÃO o ignore silenciosamente
     Este gate não é consultivo. Uma tarefa com critérios de aceitação com falha é uma tarefa incompleta.
3. Execute as verificações `<verification>`
4. Confirme que `<success_criteria>` foi atendido
5. Documente desvios no Summary
</step>

<authentication_gates>

## Gates de Autenticação

Erros de autenticação durante a execução NÃO são falhas — são pontos de interação esperados.

**Indicadores:** "Not authenticated", "Unauthorized", 401/403, "Please run {tool} login", "Set {ENV_VAR}"

**Protocolo:**
1. Reconheça o gate de autenticação (não é um bug)
2. PARE a execução da tarefa
3. Crie um checkpoint dinâmico:human-action com os passos exatos de autenticação
4. Aguarde o usuário se autenticar
5. Verifique se as credenciais funcionam
6. Tente novamente a tarefa original
7. Continue normalmente

**Exemplo:** `vercel --yes` → "Not authenticated" → checkpoint pedindo ao usuário para `vercel login` → verifique com `vercel whoami` → tente o deploy novamente → continue

**No Summary:** Documente como fluxo normal em "## Gates de Autenticação", não como desvios.

</authentication_gates>

<deviation_rules>

## Regras de Desvio

Aplique as regras de desvio da definição do agente gsd-executor (fonte única de verdade):
- **Regras 1-3** (bugs, ausências críticas, bloqueadores): corrija automaticamente, teste, verifique, registre como desvios
- **Regra 4** (mudanças arquiteturais): PARE, apresente a decisão ao usuário, aguarde aprovação
- **Limite de escopo**: não corrija automaticamente problemas pré-existentes não relacionados à tarefa atual
- **Limite de tentativas de correção**: máximo de 3 tentativas por desvio antes de escalar
- **Prioridade**: Regra 4 (PARE) > Regras 1-3 (automático) > incerto → Regra 4

</deviation_rules>

<deviation_documentation>

## Documentando Desvios

O Summary DEVE incluir a seção de desvios. Nenhum? → `## Desvios do Plano\n\nNenhum - o plano foi executado exatamente como escrito.`

Por desvio: **[Regra N - Categoria] Título** — Encontrado durante: Tarefa X | Problema | Correção | Arquivos modificados | Verificação | Hash do commit

Termine com: **Total de desvios:** N corrigidos automaticamente (detalhamento). **Impacto:** avaliação.

</deviation_documentation>

<tdd_plan_execution>
## Execução TDD

Para planos `type: tdd` — RED-GREEN-REFACTOR:

1. **Infraestrutura** (somente no primeiro plano TDD): detecte o projeto, instale o framework, configure, verifique suite vazia
2. **RED:** Leia `<behavior>` → teste(s) com falha → execute (DEVE falhar) → commit: `test({phase}-{plan}): add failing test for [feature]`
3. **GREEN:** Leia `<implementation>` → código mínimo → execute (DEVE passar) → commit: `feat({phase}-{plan}): implement [feature]`
4. **REFACTOR:** Limpe → testes DEVEM passar → commit: `refactor({phase}-{plan}): clean up [feature]`

Erros: RED não falha → investigue o teste/feature existente. GREEN não passa → depure, itere. REFACTOR quebra → desfaça.

Veja `$HOME/.claude/get-shit-done/references/tdd.md` para estrutura.
</tdd_plan_execution>

<precommit_failure_handling>
## Tratamento de Falhas em Pre-commit Hook

Seus commits podem acionar pre-commit hooks. Hooks de autocorreção se tratam de forma transparente — os arquivos são corrigidos e re-staged automaticamente.

**Se estiver executando como agente executor paralelo (spawned por execute-phase):**
Use `--no-verify` em todos os commits. Pre-commit hooks causam contenção de build lock quando múltiplos agentes fazem commit simultaneamente (ex.: conflitos de cargo lock em projetos Rust). O orquestrador valida uma vez após todos os agentes concluírem.

**Se estiver executando como executor único (modo sequencial):**
Se um commit for BLOQUEADO por um hook:

1. O comando `git commit` falha com saída de erro do hook
2. Leia o erro — ele informa exatamente qual hook e o que falhou
3. Corrija o problema (erro de tipo, violação de lint, vazamento de secret, etc.)
4. `git add` os arquivos corrigidos
5. Tente o commit novamente
6. Orçamento de 1-2 ciclos de tentativa por commit
</precommit_failure_handling>

<task_commit>
## Protocolo de Commit de Tarefa

Siga o protocolo de commit de tarefa da definição do agente gsd-executor (fonte única de verdade):
- Stage arquivos individualmente (NUNCA `git add .` ou `git add -A`)
- Formato: `{type}({phase}-{plan}): {descrição concisa}` com bullet points para mudanças principais
- Tipos: feat, fix, test, refactor, perf, docs, style, chore
- Sub-repos: use `commit-to-subrepo` quando `sub_repos` estiver configurado
- Registre o hash do commit para rastreamento no SUMMARY
- Verifique arquivos gerados não rastreados após cada commit

</task_commit>

<step name="checkpoint_protocol">
Em `type="checkpoint:*"`: automatize tudo o que for possível primeiro. Checkpoints são apenas para verificação/decisões.

Exiba: caixa `CHECKPOINT: [Tipo]` → Progresso {X}/{Y} → Nome da tarefa → conteúdo específico do tipo → `SUA AÇÃO: [sinal]`

| Tipo | Conteúdo | Sinal de retomada |
|------|---------|---------------|
| human-verify (90%) | O que foi construído + passos de verificação (comandos/URLs) | "approved" ou descreva os problemas |
| decision (9%) | Decisão necessária + contexto + opções com prós/contras | "Select: option-id" |
| human-action (1%) | O que foi automatizado + UM passo manual + plano de verificação | "done" |

Após a resposta: verifique se especificado. Passou → continue. Falhou → informe, aguarde. AGUARDE o usuário — NÃO invente a conclusão.

Veja $HOME/.claude/get-shit-done/references/checkpoints.md para detalhes.
</step>

<step name="checkpoint_return_for_orchestrator">
Quando spawned via Task e ao atingir um checkpoint: retorne o estado estruturado (não pode interagir com o usuário diretamente).

**Retorno obrigatório:** 1) Tabela de Tarefas Concluídas (hashes + arquivos) 2) Tarefa Atual (o que está bloqueando) 3) Detalhes do Checkpoint (conteúdo voltado ao usuário) 4) Aguardando (o que é necessário do usuário)

O orquestrador analisa → apresenta ao usuário → spawna continuação fresca com o estado das suas tarefas concluídas. Você NÃO será retomado. No contexto principal: use o checkpoint_protocol acima.
</step>

<step name="verification_failure_gate">
Se a verificação falhar:

**Verifique se o node repair está habilitado** (padrão: ativo):
```bash
NODE_REPAIR=$(node "./.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.node_repair 2>/dev/null || echo "true")
```

Se `NODE_REPAIR` for `true`: invoque `@./.claude/get-shit-done/workflows/node-repair.md` com:
- FAILED_TASK: número da tarefa, nome, critérios de conclusão
- ERROR: resultado esperado vs. atual
- PLAN_CONTEXT: nomes das tarefas adjacentes + objetivo da fase
- REPAIR_BUDGET: `workflow.node_repair_budget` da config (padrão: 2)

O node repair tentará RETRY, DECOMPOSE ou PRUNE autonomamente. Só chega a este gate novamente se o orçamento de repair for esgotado (ESCALATE).

Se `NODE_REPAIR` for `false` OU o repair retornar ESCALATE: PARE. Apresente: "Verificação falhou para Tarefa [X]: [nome]. Esperado: [critérios]. Atual: [resultado]. Repair tentado: [resumo do que foi tentado]." Opções: Tentar novamente | Pular (marcar como incompleto) | Parar (investigar). Se pulado → SUMMARY "Issues Encountered".
</step>

<step name="record_completion_time">
```bash
PLAN_END_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_END_EPOCH=$(date +%s)

DURATION_SEC=$(( PLAN_END_EPOCH - PLAN_START_EPOCH ))
DURATION_MIN=$(( DURATION_SEC / 60 ))

if [[ $DURATION_MIN -ge 60 ]]; then
  HRS=$(( DURATION_MIN / 60 ))
  MIN=$(( DURATION_MIN % 60 ))
  DURATION="${HRS}h ${MIN}m"
else
  DURATION="${DURATION_MIN} min"
fi
```
</step>

<step name="generate_user_setup">
```bash
grep -A 50 "^user_setup:" .planning/phases/XX-name/{phase}-{plan}-PLAN.md | head -50
```

Se user_setup existir: crie `{phase}-USER-SETUP.md` usando o template `$HOME/.claude/get-shit-done/templates/user-setup.md`. Por serviço: tabela de env vars, checklist de configuração de conta, config do dashboard, notas de dev local, comandos de verificação. Status "Incompleto". Defina `USER_SETUP_CREATED=true`. Se vazio/ausente: ignore.
</step>

<step name="create_summary">
Crie `{phase}-{plan}-SUMMARY.md` em `.planning/phases/XX-name/`. Use `$HOME/.claude/get-shit-done/templates/summary.md`.

**Frontmatter:** phase, plan, subsystem, tags | requires/provides/affects | tech-stack.added/patterns | key-files.created/modified | key-decisions | requirements-completed (**DEVE** copiar o array `requirements` do frontmatter do PLAN.md verbatim) | duration ($DURATION), completed ($PLAN_END_TIME date).

Título: `# Phase [X] Plan [Y]: [Name] Summary`

Uma linha SUBSTANTIVA: "Auth JWT com rotação de refresh usando a biblioteca jose" não "Autenticação implementada"

Inclua: duração, horários de início/fim, contagem de tarefas, contagem de arquivos.

Próximo: mais planos → "Pronto para {next-plan}" | último → "Fase completa, pronto para o próximo passo".
</step>

<step name="update_current_position">
**Ignore este passo se estiver em modo paralelo** (o orquestrador em execute-phase.md
gerencia as atualizações de STATE.md/ROADMAP.md centralmente após mesclar as worktrees para
evitar conflitos de merge).

Atualize STATE.md usando gsd-tools:

```bash
# Detecte automaticamente o modo paralelo: .git é um arquivo em worktrees, um diretório no repo principal
IS_WORKTREE=$([ -f .git ] && echo "true" || echo "false")

# Ignore em modo paralelo — o orquestrador gerencia STATE.md centralmente
if [ "$IS_WORKTREE" != "true" ]; then
  # Avance o contador de planos (trata o caso especial do último plano)
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state advance-plan

  # Recalcule a barra de progresso a partir do estado em disco
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update-progress

  # Registre métricas de execução
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-metric \
    --phase "${PHASE}" --plan "${PLAN}" --duration "${DURATION}" \
    --tasks "${TASK_COUNT}" --files "${FILE_COUNT}"
fi
```
</step>

<step name="extract_decisions_and_issues">
A partir do SUMMARY: extraia decisões e adicione ao STATE.md:

```bash
# Adicione cada decisão das key-decisions do SUMMARY
# Prefira arquivos de entrada para texto shell-safe (preserva `$`, `*`, etc. exatamente)
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state add-decision \
  --phase "${PHASE}" --summary-file "${DECISION_TEXT_FILE}" --rationale-file "${RATIONALE_FILE}"

# Adicione bloqueadores se encontrados
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state add-blocker --text-file "${BLOCKER_TEXT_FILE}"
```
</step>

<step name="update_session_continuity">
Atualize as informações da sessão usando gsd-tools:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Completed ${PHASE}-${PLAN}-PLAN.md" \
  --resume-file "None"
```

Mantenha STATE.md com menos de 150 linhas.
</step>

<step name="issues_review_gate">
Se o SUMMARY "Issues Encountered" ≠ "None": yolo → registre e continue. Interativo → apresente os problemas e aguarde confirmação.
</step>

<step name="update_roadmap">
**Ignore este passo se estiver em modo paralelo** (o orquestrador gerencia as atualizações do ROADMAP.md
centralmente após mesclar as worktrees).

```bash
# Detecte automaticamente o modo paralelo: .git é um arquivo em worktrees, um diretório no repo principal
IS_WORKTREE=$([ -f .git ] && echo "true" || echo "false")

# Ignore em modo paralelo — o orquestrador gerencia ROADMAP.md centralmente
if [ "$IS_WORKTREE" != "true" ]; then
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap update-plan-progress "${PHASE}"
fi
```
Conta arquivos PLAN vs SUMMARY no disco. Atualiza a linha da tabela de progresso com a contagem correta e o status (`In Progress` ou `Complete` com data).
</step>

<step name="update_requirements">
Marque os requisitos concluídos do campo `requirements:` do frontmatter do PLAN.md:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" requirements mark-complete ${REQ_IDS}
```

Extraia os IDs de requisito do frontmatter do plano (ex.: `requirements: [AUTH-01, AUTH-02]`). Se não houver campo requirements, ignore.
</step>

<step name="git_commit_metadata">
Código de tarefas já commitado por tarefa. Faça commit dos metadados do plano:

```bash
# Detecte automaticamente o modo paralelo: .git é um arquivo em worktrees, um diretório no repo principal
IS_WORKTREE=$([ -f .git ] && echo "true" || echo "false")

# Em modo paralelo: exclua STATE.md e ROADMAP.md (o orquestrador faz commit destes)
if [ "$IS_WORKTREE" = "true" ]; then
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}-{plan}): complete [plan-name] plan" --files .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/REQUIREMENTS.md
else
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}-{plan}): complete [plan-name] plan" --files .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md .planning/REQUIREMENTS.md
fi
```
</step>

<step name="update_codebase_map">
Se .planning/codebase/ não existir: ignore.

```bash
FIRST_TASK=$(git log --oneline --grep="feat({phase}-{plan}):" --grep="fix({phase}-{plan}):" --grep="test({phase}-{plan}):" --reverse | head -1 | cut -d' ' -f1)
git diff --name-only ${FIRST_TASK}^..HEAD 2>/dev/null || true
```

Atualize apenas mudanças estruturais: novo dir src/ → STRUCTURE.md | deps → STACK.md | padrão de arquivo → CONVENTIONS.md | cliente API → INTEGRATIONS.md | config → STACK.md | renomeado → atualize os caminhos. Ignore mudanças somente de código/bugfix/conteúdo.

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "" --files .planning/codebase/*.md --amend
```
</step>

<step name="offer_next">
Se `USER_SETUP_CREATED=true`: exiba `⚠️ CONFIGURAÇÃO DO USUÁRIO NECESSÁRIA` com caminho + tarefas de env/config NO TOPO.

```bash
(ls -1 .planning/phases/[current-phase-dir]/*-PLAN.md 2>/dev/null || true) | wc -l
(ls -1 .planning/phases/[current-phase-dir]/*-SUMMARY.md 2>/dev/null || true) | wc -l
```

| Condição | Rota | Ação |
|-----------|-------|--------|
| summaries < plans | **A: Mais planos** | Encontre o próximo PLAN sem SUMMARY. Yolo: continue automaticamente. Interativo: mostre o próximo plano, sugira `/gsd-execute-phase {phase}` + `/gsd-verify-work`. PARE aqui. |
| summaries = plans, atual < fase mais alta | **B: Fase concluída** | Mostre a conclusão, sugira `/gsd-plan-phase {Z+1}` + `/gsd-verify-work {Z}` + `/gsd-discuss-phase {Z+1}` |
| summaries = plans, atual = fase mais alta | **C: Marco concluído** | Mostre o banner, sugira `/gsd-complete-milestone` + `/gsd-verify-work` + `/gsd-add-phase` |

Todas as rotas: `/clear` primeiro para contexto limpo.
</step>

</process>

<success_criteria>

- Todas as tarefas do PLAN.md concluídas
- Todas as verificações passam
- USER-SETUP.md gerado se user_setup estiver no frontmatter
- SUMMARY.md criado com conteúdo substantivo
- STATE.md atualizado (posição, decisões, problemas, sessão) — exceto em modo paralelo (o orquestrador gerencia)
- ROADMAP.md atualizado — exceto em modo paralelo (o orquestrador gerencia)
- Se o mapa do codebase existir: mapa atualizado com as mudanças da execução (ou ignorado se sem mudanças significativas)
- Se USER-SETUP.md criado: destacado claramente na saída de conclusão
</success_criteria>
