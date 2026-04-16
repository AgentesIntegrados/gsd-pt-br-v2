<purpose>

Executar fases de milestone de forma autônoma — todas as fases restantes, um intervalo via `--from N`/`--to N`, ou uma única fase via `--only N`. Para cada fase incompleta: discutir → planejar → executar usando invocações Skill() diretas. Pausa apenas para decisões explícitas do usuário (aceitação de área cinzenta, bloqueadores, solicitações de validação). Relê o ROADMAP.md após cada fase para capturar fases inseridas dinamicamente.

</purpose>

<required_reading>

Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

</required_reading>

<process>

<step name="initialize" priority="first">

## 1. Inicializar

Analise `$ARGUMENTS` para as flags `--from N`, `--to N`, `--only N` e `--interactive`:

```bash
FROM_PHASE=""
if echo "$ARGUMENTS" | grep -qE '\-\-from\s+[0-9]'; then
  FROM_PHASE=$(echo "$ARGUMENTS" | grep -oE '\-\-from\s+[0-9]+\.?[0-9]*' | awk '{print $2}')
fi

TO_PHASE=""
if echo "$ARGUMENTS" | grep -qE '\-\-to\s+[0-9]'; then
  TO_PHASE=$(echo "$ARGUMENTS" | grep -oE '\-\-to\s+[0-9]+\.?[0-9]*' | awk '{print $2}')
fi

ONLY_PHASE=""
if echo "$ARGUMENTS" | grep -qE '\-\-only\s+[0-9]'; then
  ONLY_PHASE=$(echo "$ARGUMENTS" | grep -oE '\-\-only\s+[0-9]+\.?[0-9]*' | awk '{print $2}')
  FROM_PHASE="$ONLY_PHASE"
fi

INTERACTIVE=""
if echo "$ARGUMENTS" | grep -q '\-\-interactive'; then
  INTERACTIVE="true"
fi
```

Quando `--only` estiver definido, defina também `FROM_PHASE` com o mesmo valor para que a lógica de filtro existente seja aplicada.

Quando `--interactive` estiver definido, a discussão é executada inline com perguntas (sem resposta automática), enquanto o planejamento e a execução são despachados como agentes em segundo plano. Isso mantém o contexto principal enxuto — apenas as conversas de discussão se acumulam — enquanto preserva a entrada do usuário em todas as decisões de design.

Bootstrap via inicialização no nível de milestone:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init milestone-op)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `milestone_version`, `milestone_name`, `phase_count`, `completed_phases`, `roadmap_exists`, `state_exists`, `commit_docs`.

**Se `roadmap_exists` for falso:** Erro — "Nenhum ROADMAP.md encontrado. Execute `/gsd-new-milestone` primeiro."
**Se `state_exists` for falso:** Erro — "Nenhum STATE.md encontrado. Execute `/gsd-new-milestone` primeiro."

Exibir banner de inicialização:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Milestone: {milestone_version} — {milestone_name}
 Fases: {phase_count} total, {completed_phases} concluídas
```

Se `ONLY_PHASE` estiver definido, exibir: `Modo fase única: Fase ${ONLY_PHASE}`
Senão se `FROM_PHASE` estiver definido, exibir: `Iniciando a partir da fase ${FROM_PHASE}`
Se `TO_PHASE` estiver definido, exibir: `Parando após a fase ${TO_PHASE}`
Se `INTERACTIVE` estiver definido, exibir: `Modo: Interativo (discussão inline, planejamento+execução em segundo plano)`

</step>

<step name="discover_phases">

## 2. Descobrir Fases

Executar descoberta de fases:

```bash
ROADMAP=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Analisar o array JSON `phases`.

**Filtrar para fases incompletas:** Manter apenas fases onde `disk_status !== "complete"` OU `roadmap_complete === false`.

**Aplicar filtro `--from N`:** Se `FROM_PHASE` foi fornecido, filtrar adicionalmente as fases onde `number < FROM_PHASE` (usar comparação numérica — lida com fases decimais como "5.1").

**Aplicar filtro `--to N`:** Se `TO_PHASE` foi fornecido, filtrar adicionalmente as fases onde `number > TO_PHASE` (usar comparação numérica). Isso limita a execução às fases até a fase alvo.

**Aplicar filtro `--only N`:** Se `ONLY_PHASE` foi fornecido, filtrar adicionalmente as fases onde `number != ONLY_PHASE`. Isso significa que a lista de fases conterá exatamente uma fase (ou zero se já estiver completa).

**Se `TO_PHASE` estiver definido e nenhuma fase restar** (todas as fases até N já estão concluídas):

```
Todas as fases até ${TO_PHASE} já estão concluídas. Nada a fazer.
```

Sair limpo.

**Se `ONLY_PHASE` estiver definido e nenhuma fase restar** (fase já completa):

```
A fase ${ONLY_PHASE} já está concluída. Nada a fazer.
```

Sair limpo.

**Ordenar por `number`** em ordem numérica crescente.

**Se nenhuma fase incompleta restar:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ COMPLETO 🎉
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Todas as fases concluídas! Nada mais a fazer.
```

Sair limpo.

**Exibir plano de fases:**

```
## Plano de Fases

| # | Fase | Status |
|---|-------|--------|
| 5 | Skill Scaffolding & Phase Discovery | Em Andamento |
| 6 | Smart Discuss | Não Iniciada |
| 7 | Auto-Chain Refinements | Não Iniciada |
| 8 | Lifecycle Orchestration | Não Iniciada |
```

**Buscar detalhes de cada fase:**

```bash
DETAIL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM})
```

Extrair `phase_name`, `goal`, `success_criteria` de cada uma. Armazenar para uso em execute_phase e mensagens de transição.

</step>

<step name="execute_phase">

## 3. Executar Fase

Para a fase atual, exibir o banner de progresso:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ Fase {N}/{T}: {Nome} [████░░░░] {P}%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Onde N = número da fase atual (do ROADMAP, ex.: 63), T = total de fases do milestone (de `phase_count` analisado na etapa de inicialização, ex.: 67). **Importante:** T deve ser `phase_count` (o número total de fases neste milestone), NÃO a contagem de fases restantes/incompletas. Quando as fases são numeradas de 61 a 67, T=7 e o banner deve mostrar `Fase 63/7` (fase 63, 7 total no milestone), não `Fase 63/3` (o que confundiria 3 restantes com 3 total). P = porcentagem de todas as fases do milestone concluídas até agora. Calcule P como: (número de fases com `disk_status` "complete" do último `roadmap analyze` / T × 100). Use █ para segmentos preenchidos e ░ para vazios na barra de progresso (8 caracteres de largura).

**Exibição alternativa quando os números de fase excedem o total** (ex.: projetos com múltiplos milestones onde as fases são numeradas globalmente): Se N > T (número de fase excede a contagem de fases do milestone), use o formato `Fase {N} ({posição}/{T})` onde `posição` é o índice baseado em 1 desta fase entre as fases incompletas sendo processadas. Isso evita exibições confusas como "Fase 63/5".

**3a. Smart Discuss**

Verificar se CONTEXT.md já existe para esta fase:

```bash
PHASE_STATE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Analisar `has_context` do JSON.

**Se has_context for verdadeiro:** Pular discuss — contexto já reunido. Exibir:

```
Fase ${PHASE_NUM}: Contexto existe — pulando discuss.
```

Prosseguir para 3b.

**Se has_context for falso:** Verificar se discuss está desabilitado via configurações:

```bash
SKIP_DISCUSS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.skip_discuss 2>/dev/null || echo "false")
```

**Se SKIP_DISCUSS for `true`:** Pular discuss completamente — a descrição da fase do ROADMAP é a especificação. Exibir:

```
Fase ${PHASE_NUM}: Discuss ignorado (workflow.skip_discuss=true) — usando objetivo da fase do ROADMAP como especificação.
```

Escrever um CONTEXT.md mínimo para que o plan-phase downstream tenha entrada válida. Obter detalhes da fase:

```bash
DETAIL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM})
```

Extrair `goal` e `requirements` do JSON. Escrever `${phase_dir}/${padded_phase}-CONTEXT.md` com:

```markdown
# Fase {PHASE_NUM}: {Nome da Fase} - Contexto

**Coletado:** {data}
**Status:** Pronto para planejamento
**Modo:** Gerado automaticamente (discuss ignorado via workflow.skip_discuss)

<domain>
## Escopo da Fase

{objetivo da descrição da fase no ROADMAP}

</domain>

<decisions>
## Decisões de Implementação

### A Critério do Claude
Todas as escolhas de implementação ficam a critério do Claude — a fase de discuss foi ignorada conforme configuração do usuário. Use o objetivo da fase no ROADMAP, critérios de sucesso e convenções do código para guiar as decisões.

</decisions>

<code_context>
## Informações do Código Existente

O contexto do código será reunido durante a pesquisa da fase de planejamento.

</code_context>

<specifics>
## Ideias Específicas

Sem requisitos específicos — fase de discuss ignorada. Consulte a descrição da fase no ROADMAP e os critérios de sucesso.

</specifics>

<deferred>
## Ideias Adiadas

Nenhuma — fase de discuss ignorada.

</deferred>
```

Confirmar o contexto mínimo:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${PADDED_PHASE}): contexto gerado automaticamente (discuss ignorado)" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Prosseguir para 3b.

**Se SKIP_DISCUSS for `false` (ou não definido):**

**IMPORTANTE — O discuss deve ser de passagem única no modo autônomo.**
A etapa de discuss no modo `--auto` NÃO DEVE fazer loop. Se CONTEXT.md já existir após o término do discuss, NÃO re-invoque o discuss para a mesma fase. A verificação `has_context` abaixo é autoridade — uma vez verdadeira, o discuss está concluído para esta fase independentemente de "lacunas" percebidas no arquivo de contexto.

**Se `INTERACTIVE` estiver definido:** Execute o skill padrão de discuss-phase inline (faz perguntas interativas, aguarda respostas do usuário). Isso preserva a entrada do usuário em todas as decisões de design enquanto mantém planejamento+execução fora do contexto principal:

```
Skill(skill="gsd:discuss-phase", args="${PHASE_NUM}")
```

**Se `INTERACTIVE` NÃO estiver definido:** Execute a etapa smart_discuss para esta fase (propostas em tabelas em lote, otimizado automaticamente).

Após o término do discuss (em qualquer modo), verificar se o contexto foi escrito:

```bash
PHASE_STATE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Verificar `has_context`. Se falso → ir para handle_blocker: "O discuss da fase ${PHASE_NUM} não produziu CONTEXT.md."

**3a.5. Contrato de Design de UI (Fases Frontend)**

Verificar se esta fase tem indicadores frontend e se um UI-SPEC já existe:

```bash
PHASE_SECTION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM} 2>/dev/null)
echo "$PHASE_SECTION" | grep -iE "UI|interface|frontend|component|layout|page|screen|view|form|dashboard|widget" > /dev/null 2>&1
HAS_UI=$?
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
```

Verificar se o workflow de fase de UI está habilitado:

```bash
UI_PHASE_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_phase 2>/dev/null || echo "true")
```

**Se `HAS_UI` for 0 (indicadores frontend encontrados) E `UI_SPEC_FILE` estiver vazio (sem UI-SPEC) E `UI_PHASE_CFG` não for `false`:**

Exibir:

```
Fase ${PHASE_NUM}: Fase frontend detectada — gerando contrato de design de UI...
```

```
Skill(skill="gsd-ui-phase", args="${PHASE_NUM}")
```

Verificar se o UI-SPEC foi criado:

```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
```

**Se `UI_SPEC_FILE` ainda estiver vazio após ui-phase:** Exibir aviso `Fase ${PHASE_NUM}: A geração de UI-SPEC não produziu saída — continuando sem contrato de design.` e prosseguir para 3b.

**Se `HAS_UI` for 1 (sem indicadores frontend) OU `UI_SPEC_FILE` não estiver vazio (UI-SPEC já existe) OU `UI_PHASE_CFG` for `false`:** Pular silenciosamente para 3b.

**3b. Planejar**

**Se `INTERACTIVE` estiver definido:** Despachar planejamento como agente em segundo plano para manter o contexto principal enxuto. Enquanto o planejamento executa, o workflow pode imediatamente começar a discutir a próxima fase (ver etapa 4).

```
Agent(
  description="Planejar fase ${PHASE_NUM}: ${PHASE_NAME}",
  run_in_background=true,
  prompt="Executar plan-phase para a fase ${PHASE_NUM}: Skill(skill=\"gsd:plan-phase\", args=\"${PHASE_NUM}\")"
)
```

Armazenar o task_id do agente. Após o término do discuss para a próxima fase (ou se não houver próxima fase), aguardar que o agente de planejamento termine antes de prosseguir para execução.

**Se `INTERACTIVE` NÃO estiver definido (padrão):** Executar planejamento inline como antes.

```
Skill(skill="gsd-plan-phase", args="${PHASE_NUM}")
```

Verificar se o planejamento produziu saída — re-executar `init phase-op` e verificar `has_plans`. Se falso → ir para handle_blocker: "A fase de planejamento ${PHASE_NUM} não produziu nenhum plano."

**3c. Executar**

**Se `INTERACTIVE` estiver definido:** Aguardar que o agente de planejamento conclua (se ainda não tiver), verificar se os planos existem, e então despachar execução como agente em segundo plano:

```
Agent(
  description="Executar fase ${PHASE_NUM}: ${PHASE_NAME}",
  run_in_background=true,
  prompt="Executar execute-phase para a fase ${PHASE_NUM}: Skill(skill=\"gsd:execute-phase\", args=\"${PHASE_NUM} --no-transition\")"
)
```

Armazenar o task_id do agente. O workflow pode agora começar a discutir a próxima fase enquanto esta fase executa em segundo plano. Antes de iniciar o roteamento pós-execução para esta fase, aguardar que o agente de execução conclua.

**Se `INTERACTIVE` NÃO estiver definido (padrão):** Executar inline como antes.

```
Skill(skill="gsd-execute-phase", args="${PHASE_NUM} --no-transition")
```

**3c.5. Revisão e Correção de Código**

Invocar automaticamente a cadeia de revisão e correção de código. O modo autônomo encadeia revisão e correção (ao contrário de execute-phase/quick que apenas sugere a correção).

**Gate de configuração:**
```bash
CODE_REVIEW_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.code_review 2>/dev/null || echo "true")
```
Se `"false"`: exibir "Revisão de código ignorada (workflow.code_review=false)" e prosseguir para 3d.

```
Skill(skill="gsd:code-review", args="${PHASE_NUM}")
```

Analisar status do frontmatter do REVIEW.md. Se "clean" ou "skipped": prosseguir para 3d. Se encontrar ocorrências: invocar automaticamente:
```
Skill(skill="gsd:code-review-fix", args="${PHASE_NUM} --auto")
```

**Tratamento de erros:** Se qualquer Skill falhar, capturar o erro, exibir como não bloqueante e prosseguir para 3d.

**3d. Roteamento Pós-Execução**

**Se `INTERACTIVE` estiver definido:** Aguardar que o agente de execução conclua antes de ler os resultados de verificação.

Após o retorno de execute-phase (ou a conclusão do agente de execução), ler o resultado da verificação:

```bash
VERIFY_STATUS=$(grep "^status:" "${PHASE_DIR}"/*-VERIFICATION.md 2>/dev/null | head -1 | cut -d: -f2 | tr -d ' ')
```

Onde `PHASE_DIR` vem da chamada `init phase-op` já feita na etapa 3a. Se a variável não estiver no escopo, re-buscar:

```bash
PHASE_STATE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Analisar `phase_dir` do JSON.

**Se VERIFY_STATUS estiver vazio** (sem VERIFICATION.md ou sem campo de status):

Ir para handle_blocker: "A execução da fase ${PHASE_NUM} não produziu resultados de verificação."

**Se `passed`:**

Exibir:
```
Fase ${PHASE_NUM} ✅ ${PHASE_NAME} — Verificação aprovada
```

Prosseguir para a etapa de iteração.

**Se `human_needed`:**

Ler a seção human_verification do VERIFICATION.md para obter a contagem e os itens que requerem teste manual.

**Modo de texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário para digitar o número de sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.

Exibir os itens e perguntar ao usuário via AskUserQuestion:
- **question:** "A fase ${PHASE_NUM} tem itens que precisam de verificação manual. Validar agora ou continuar para a próxima fase?"
- **options:** "Validar agora" / "Continuar sem validação"

Em **"Validar agora"**: Apresentar os itens específicos da seção human_verification do VERIFICATION.md. Após a revisão do usuário, perguntar:
- **question:** "Resultado da validação?"
- **options:** "Tudo certo — continuar" / "Encontrei problemas"

Em "Tudo certo — continuar": Exibir `Fase ${PHASE_NUM} ✅ Validação humana aprovada` e prosseguir para a etapa de iteração.

Em "Encontrei problemas": Ir para handle_blocker com os problemas relatados pelo usuário como descrição.

Em **"Continuar sem validação"**: Exibir `Fase ${PHASE_NUM} ⏭ Validação humana adiada` e prosseguir para a etapa de iteração.

**Se `gaps_found`:**

Ler o resumo de lacunas do VERIFICATION.md (pontuação e itens ausentes). Exibir:
```
⚠ Fase ${PHASE_NUM}: ${PHASE_NAME} — Lacunas Encontradas
Pontuação: {N}/{M} must-haves verificados
```

Perguntar ao usuário via AskUserQuestion:
- **question:** "Lacunas encontradas na fase ${PHASE_NUM}. Como prosseguir?"
- **options:** "Executar fechamento de lacunas" / "Continuar sem corrigir" / "Parar modo autônomo"

Em **"Executar fechamento de lacunas"**: Executar ciclo de fechamento de lacunas (limite: 1 tentativa):

```
Skill(skill="gsd-plan-phase", args="${PHASE_NUM} --gaps")
```

Verificar se os planos de lacuna foram criados — re-executar `init phase-op ${PHASE_NUM}` e verificar `has_plans`. Se nenhum novo plano de lacuna → ir para handle_blocker: "O planejamento de fechamento de lacunas para a fase ${PHASE_NUM} não produziu planos."

Re-executar:
```
Skill(skill="gsd-execute-phase", args="${PHASE_NUM} --no-transition")
```

Re-ler o status de verificação:
```bash
VERIFY_STATUS=$(grep "^status:" "${PHASE_DIR}"/*-VERIFICATION.md 2>/dev/null | head -1 | cut -d: -f2 | tr -d ' ')
```

Se `passed` ou `human_needed`: Roteiar normalmente (continuar ou perguntar ao usuário como acima).

Se ainda `gaps_found` após esta tentativa: Exibir "As lacunas persistem após a tentativa de fechamento." e perguntar via AskUserQuestion:
- **question:** "O fechamento de lacunas não resolveu totalmente os problemas. Como prosseguir?"
- **options:** "Continuar mesmo assim" / "Parar modo autônomo"

Em "Continuar mesmo assim": Prosseguir para a etapa de iteração.
Em "Parar modo autônomo": Ir para handle_blocker.

Isso limita o fechamento de lacunas a 1 tentativa automática para evitar loops infinitos.

Em **"Continuar sem corrigir"**: Exibir `Fase ${PHASE_NUM} ⏭ Lacunas adiadas` e prosseguir para a etapa de iteração.

Em **"Parar modo autônomo"**: Ir para handle_blocker com "Usuário parou — lacunas permanecem na fase ${PHASE_NUM}".

**3d.5. Revisão de UI (Fases Frontend)**

> Executar após qualquer roteamento de execução bem-sucedido (aprovado, human_needed aceito, ou lacunas adiadas/aceitas) — antes de prosseguir para a etapa de iteração.

Verificar se esta fase teve um UI-SPEC (criado na etapa 3a.5 ou pré-existente):

```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
```

Verificar se a revisão de UI está habilitada:

```bash
UI_REVIEW_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_review 2>/dev/null || echo "true")
```

**Se `UI_SPEC_FILE` não estiver vazio E `UI_REVIEW_CFG` não for `false`:**

Exibir:

```
Fase ${PHASE_NUM}: Fase frontend com UI-SPEC — executando auditoria de revisão de UI...
```

```
Skill(skill="gsd-ui-review", args="${PHASE_NUM}")
```

Exibir o resumo do resultado da revisão (pontuação do UI-REVIEW.md se produzido). Continuar para a etapa de iteração independentemente da pontuação — a revisão de UI é consultiva, não bloqueante.

**Se `UI_SPEC_FILE` estiver vazio OU `UI_REVIEW_CFG` for `false`:** Pular silenciosamente para a etapa de iteração.

</step>

<step name="smart_discuss">

## Smart Discuss

Executar smart discuss para a fase atual. Propõe respostas de áreas cinzentas em tabelas em lote — o usuário aceita ou substitui por área. Produz saída CONTEXT.md idêntica ao discuss-phase regular.

> **Nota:** O smart discuss é uma variante otimizada para modo autônomo do skill `gsd-discuss-phase`. Produz saída CONTEXT.md idêntica mas usa propostas em tabelas em lote em vez de questionamento sequencial. O skill original `gsd-discuss-phase` permanece inalterado (conforme CTRL-03). Milestones futuros podem extrair isso para um arquivo de skill separado.

**Entradas:** `PHASE_NUM` de execute_phase. Executar init para obter caminhos de fase:

```bash
PHASE_STATE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op ${PHASE_NUM})
```

Analisar do JSON: `phase_dir`, `phase_slug`, `padded_phase`, `phase_name`.

---

### Sub-etapa 1: Carregar contexto anterior

Ler contexto do nível do projeto e de fases anteriores para evitar re-perguntar questões já decididas.

**Ler arquivos do projeto:**

```bash
cat .planning/PROJECT.md 2>/dev/null || true
cat .planning/REQUIREMENTS.md 2>/dev/null || true
cat .planning/STATE.md 2>/dev/null || true
```

Extrair destes:
- **PROJECT.md** — Visão, princípios, inegociáveis, preferências do usuário
- **REQUIREMENTS.md** — Critérios de aceitação, restrições, must-haves vs nice-to-haves
- **STATE.md** — Progresso atual, decisões registradas até o momento

**Ler todos os arquivos CONTEXT.md anteriores:**

```bash
(find .planning/phases -name "*-CONTEXT.md" 2>/dev/null || true) | sort
```

Para cada CONTEXT.md onde o número da fase < fase atual:
- Ler a seção `<decisions>` — estas são preferências bloqueadas
- Ler `<specifics>` — referências específicas ou momentos "Eu quero como X"
- Notar padrões (ex.: "usuário consistentemente prefere UI mínima", "usuário rejeitou saída verbosa")

**Construir contexto interno prior_decisions** (não escrever em arquivo):

```
<prior_decisions>
## Nível do Projeto
- [Princípio ou restrição chave do PROJECT.md]
- [Requisito que afeta esta fase do REQUIREMENTS.md]

## De Fases Anteriores
### Fase N: [Nome]
- [Decisão relevante para a fase atual]
- [Preferência que estabelece um padrão]
</prior_decisions>
```

Se não existir contexto anterior, continuar sem — esperado para fases iniciais.

---

### Sub-etapa 2: Explorar Base de Código

Varredura leve do código para informar a identificação de áreas cinzentas e propostas. Manter abaixo de ~5% do contexto.

**Verificar mapas de código existentes:**

```bash
ls .planning/codebase/*.md 2>/dev/null || true
```

**Se mapas de código existirem:** Ler os mais relevantes (CONVENTIONS.md, STRUCTURE.md, STACK.md com base no tipo de fase). Extrair componentes reutilizáveis, padrões estabelecidos, pontos de integração. Pular para construir contexto abaixo.

**Se não houver mapas de código, fazer grep direcionado:**

Extrair termos-chave do objetivo da fase. Buscar arquivos relacionados:

```bash
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" 2>/dev/null | head -10 || true
ls src/components/ src/hooks/ src/lib/ src/utils/ 2>/dev/null || true
```

Ler os 3-5 arquivos mais relevantes para entender os padrões existentes.

**Construir codebase_context interno** (não escrever em arquivo):
- **Ativos reutilizáveis** — componentes, hooks, utilitários existentes utilizáveis nesta fase
- **Padrões estabelecidos** — como o código faz gerenciamento de estado, estilização, busca de dados
- **Pontos de integração** — onde o novo código se conecta (rotas, nav, providers)

---

### Sub-etapa 3: Analisar Fase e Gerar Propostas

**Obter detalhes da fase:**

```bash
DETAIL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase ${PHASE_NUM})
```

Extrair `goal`, `requirements`, `success_criteria` da resposta JSON.

**Detecção de infraestrutura — verificar PRIMEIRO antes de gerar áreas cinzentas:**

Uma fase é infraestrutura pura quando TODOS estes são verdadeiros:
1. Palavras-chave do objetivo correspondem a: "scaffolding", "plumbing", "setup", "configuration", "migration", "refactor", "rename", "restructure", "upgrade", "infrastructure"
2. E os critérios de sucesso são todos técnicos: "arquivo existe", "teste passa", "config válida", "comando executa"
3. E nenhum comportamento voltado ao usuário é descrito (sem "usuários podem", "exibe", "mostra", "apresenta")

**Se for infraestrutura pura:** Pular Sub-etapa 4. Ir diretamente para Sub-etapa 5 com CONTEXT.md mínimo. Exibir:

```
Fase ${PHASE_NUM}: Fase de infraestrutura — pulando discuss, escrevendo contexto mínimo.
```

Usar estes padrões para o CONTEXT.md:
- `<domain>`: Escopo da fase do objetivo do ROADMAP
- `<decisions>`: Subseção única "### A Critério do Claude" — "Todas as escolhas de implementação ficam a critério do Claude — fase de infraestrutura pura"
- `<code_context>`: O que o explorador de código encontrou
- `<specifics>`: "Sem requisitos específicos — fase de infraestrutura"
- `<deferred>`: "Nenhum"

**Se NÃO for infraestrutura — gerar propostas de áreas cinzentas:**

Determinar o tipo de domínio a partir do objetivo da fase:
- Algo que os usuários **VEEM** → visual: layout, interações, estados, densidade
- Algo que os usuários **CHAMAM** → interface: contratos, respostas, erros, auth
- Algo que os usuários **EXECUTAM** → execução: invocação, saída, modos de comportamento, flags
- Algo que os usuários **LEEM** → conteúdo: estrutura, tom, profundidade, fluxo
- Algo sendo **ORGANIZADO** → organização: critérios, agrupamento, exceções, nomeação

Verificar prior_decisions — pular áreas cinzentas já decididas em fases anteriores.

Gerar **3-4 áreas cinzentas** com **~4 perguntas cada**. Para cada pergunta:
- **Pré-selecionar uma resposta recomendada** com base em: decisões anteriores (consistência), padrões do código (reutilização), convenções do domínio (abordagens padrão), critérios de sucesso do ROADMAP
- Gerar **1-2 alternativas** por pergunta
- **Anotar** com contexto de decisão anterior ("Você decidiu X na Fase N") e contexto de código ("O Componente Y existe com variantes Z") onde relevante

---

### Sub-etapa 4: Apresentar Propostas por Área

Apresentar áreas cinzentas **uma de cada vez**. Para cada área (M de N):

Exibir uma tabela:

```
### Área Cinzenta {M}/{N}: {Nome da Área}

| # | Pergunta | ✅ Recomendado | Alternativa(s) |
|---|----------|---------------|-----------------|
| 1 | {pergunta} | {resposta} — {justificativa} | {alt1}; {alt2} |
| 2 | {pergunta} | {resposta} — {justificativa} | {alt1} |
| 3 | {pergunta} | {resposta} — {justificativa} | {alt1}; {alt2} |
| 4 | {pergunta} | {resposta} — {justificativa} | {alt1} |
```

Depois perguntar ao usuário via **AskUserQuestion**:
- **header:** "Área {M}/{N}"
- **question:** "Aceitar estas respostas para {Nome da Área}?"
- **options:** Construir dinamicamente — sempre "Aceitar tudo" primeiro, depois "Alterar P1" até "Alterar PN" para cada pergunta (até 4), depois "Discutir mais" por último. Limitar a 6 opções explícitas (AskUserQuestion adiciona "Outro" automaticamente).

**Em "Aceitar tudo":** Registrar todas as respostas recomendadas para esta área. Mover para a próxima área.

**Em "Alterar PN":** Usar AskUserQuestion com as alternativas para essa pergunta específica:
- **header:** "{Nome da Área}"
- **question:** "P{N}: {texto da pergunta}"
- **options:** Listar as 1-2 alternativas mais "Você decide" (mapeia para Critério do Claude)

Registrar a escolha do usuário. Re-exibir a tabela atualizada com a alteração refletida. Re-apresentar o prompt de aceitação completo para que o usuário possa fazer alterações adicionais ou aceitar.

**Em "Discutir mais":** Alternar para modo interativo apenas para esta área — fazer perguntas uma de cada vez usando AskUserQuestion com 2-3 opções concretas por pergunta mais "Você decide". Após 4 perguntas, perguntar:
- **header:** "{Nome da Área}"
- **question:** "Mais perguntas sobre {nome da área}, ou seguir para a próxima?"
- **options:** "Mais perguntas" / "Próxima área"

Se "Mais perguntas", fazer mais 4. Se "Próxima área", exibir tabela de resumo final das respostas capturadas para esta área e seguir em frente.

**Em "Outro" (texto livre):** Interpretar como uma solicitação de alteração específica ou feedback geral. Incorporar nas decisões da área, re-exibir tabela atualizada, re-apresentar prompt de aceitação.

**Tratamento de expansão de escopo:** Se o usuário mencionar algo fora do domínio da fase:

```
"{Funcionalidade}" parece uma nova capacidade — pertence à sua própria fase.
Vou anotá-la como uma ideia adiada.

Voltando a {área atual}: {retornar à pergunta atual}"
```

Rastrear ideias adiadas internamente para inclusão no CONTEXT.md.

---

### Sub-etapa 5: Escrever CONTEXT.md

Após todas as áreas serem resolvidas (ou pulo de infraestrutura), escrever o arquivo CONTEXT.md.

**Caminho do arquivo:** `${phase_dir}/${padded_phase}-CONTEXT.md`

Use **exatamente** esta estrutura (idêntica à saída do discuss-phase):

```markdown
# Fase {PHASE_NUM}: {Nome da Fase} - Contexto

**Coletado:** {data}
**Status:** Pronto para planejamento

<domain>
## Escopo da Fase

{Declaração de escopo da análise — o que esta fase entrega}

</domain>

<decisions>
## Decisões de Implementação

### {Nome da Área 1}
- {Resposta aceita/escolhida para P1}
- {Resposta aceita/escolhida para P2}
- {Resposta aceita/escolhida para P3}
- {Resposta aceita/escolhida para P4}

### {Nome da Área 2}
- {Resposta aceita/escolhida para P1}
- {Resposta aceita/escolhida para P2}
...

### A Critério do Claude
{Quaisquer respostas "Você decide" coletadas — note que o Claude tem flexibilidade aqui}

</decisions>

<code_context>
## Informações do Código Existente

### Ativos Reutilizáveis
- {Do explorador de código — componentes, hooks, utilitários}

### Padrões Estabelecidos
- {Do explorador de código — gerenciamento de estado, estilização, busca de dados}

### Pontos de Integração
- {Do explorador de código — onde o novo código se conecta}

</code_context>

<specifics>
## Ideias Específicas

{Quaisquer referências específicas ou "Eu quero como X" da discussão}
{Se nenhuma: "Sem requisitos específicos — aberto a abordagens padrão"}

</specifics>

<deferred>
## Ideias Adiadas

{Ideias capturadas mas fora do escopo para esta fase}
{Se nenhuma: "Nenhuma — a discussão permaneceu dentro do escopo da fase"}

</deferred>
```

Escrever o arquivo.

**Confirmar:**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${PADDED_PHASE}): contexto do smart discuss" --files "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Exibir confirmação:

```
Criado: {caminho}
Decisões capturadas: {count} em {area_count} áreas
```

</step>

<step name="iterate">

## 4. Iterar

**Se `ONLY_PHASE` estiver definido:** Não iterar. Prosseguir diretamente para a etapa de ciclo de vida (que sai limpo por modo de fase única).

**Se `TO_PHASE` estiver definido e o número da fase atual >= `TO_PHASE`:** A fase alvo foi alcançada. Não iterar mais. Exibir:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ --to ${TO_PHASE} ATINGIDO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Concluído até a fase ${TO_PHASE} conforme solicitado.
 As fases restantes não foram executadas.

 Retomar com: /gsd-autonomous --from ${next_incomplete_phase}
```

Prosseguir diretamente para a etapa de ciclo de vida (que lida com conclusão parcial — pula auditoria/conclusão/limpeza já que nem todas as fases estão concluídas). Sair limpo.

**Caso contrário:** Após cada fase ser concluída, re-ler o ROADMAP.md para capturar fases inseridas durante a execução (fases decimais como 5.1):

```bash
ROADMAP=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze)
```

Re-filtrar fases incompletas usando a mesma lógica de discover_phases:
- Manter fases onde `disk_status !== "complete"` OU `roadmap_complete === false`
- Aplicar filtro `--from N` se originalmente fornecido
- Aplicar filtro `--to N` se originalmente fornecido
- Ordenar por número crescente

Ler STATE.md novamente:

```bash
cat .planning/STATE.md
```

Verificar bloqueadores na seção Bloqueadores/Preocupações. Se bloqueadores forem encontrados, ir para handle_blocker com a descrição do bloqueador.

Se fases incompletas restarem: prosseguir para a próxima fase, voltar ao loop de execute_phase.

**Sobreposição no modo interativo:** Quando `INTERACTIVE` estiver definido, a etapa de iteração habilita paralelismo de pipeline:
1. Após o término do discuss para a Fase N, despachar planejamento+execução como agentes em segundo plano
2. Imediatamente iniciar o discuss para a Fase N+1 (a próxima fase incompleta) enquanto a Fase N constrói
3. Antes de iniciar o planejamento para a Fase N+1, aguardar que o agente de execução da Fase N conclua e lidar com seu roteamento pós-execução (verificação, fechamento de lacunas, etc.)

Isso significa que o usuário está sempre respondendo perguntas de discuss (leves, interativas) enquanto o trabalho pesado (planejamento, geração de código) executa em segundo plano. O contexto principal acumula apenas conversas de discuss — os contextos de planejamento e execução são isolados em seus agentes.

Se todas as fases estiverem concluídas, prosseguir para a etapa de ciclo de vida.

</step>

<step name="lifecycle">

## 5. Ciclo de Vida

**Se `ONLY_PHASE` estiver definido:** Pular ciclo de vida. Uma única fase não aciona auditoria/conclusão/limpeza. Exibir:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ FASE ${ONLY_PHASE} CONCLUÍDA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Fase ${ONLY_PHASE}: ${PHASE_NAME} — Concluída
 Modo: Fase única (--only)

 Ciclo de vida ignorado — execute /gsd-autonomous sem --only
 após todas as fases concluírem para acionar auditoria/conclusão/limpeza.
```

Sair limpo.

**Caso contrário:** Após todas as fases concluírem, executar a sequência do ciclo de vida do milestone: auditoria → conclusão → limpeza.

Exibir banner de transição do ciclo de vida:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ CICLO DE VIDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Todas as fases concluídas → Iniciando ciclo de vida: auditoria → conclusão → limpeza
 Milestone: {milestone_version} — {milestone_name}
```

**5a. Auditoria**

```
Skill(skill="gsd-audit-milestone")
```

Após a conclusão da auditoria, detectar o resultado:

```bash
AUDIT_FILE=".planning/v${milestone_version}-MILESTONE-AUDIT.md"
AUDIT_STATUS=$(grep "^status:" "${AUDIT_FILE}" 2>/dev/null | head -1 | cut -d: -f2 | tr -d ' ')
```

**Se AUDIT_STATUS estiver vazio** (sem arquivo de auditoria ou sem campo de status):

Ir para handle_blocker: "A auditoria não produziu resultados — arquivo de auditoria ausente ou malformado."

**Se `passed`:**

Exibir:
```
Auditoria ✅ aprovada — prosseguindo para conclusão do milestone
```

Prosseguir para 5b (sem pausa do usuário — conforme CTRL-01).

**Se `gaps_found`:**

Ler o resumo de lacunas do arquivo de auditoria. Exibir:
```
⚠ Auditoria: Lacunas Encontradas
```

Perguntar ao usuário via AskUserQuestion:
- **question:** "A auditoria do milestone encontrou lacunas. Como prosseguir?"
- **options:** "Continuar mesmo assim — aceitar lacunas" / "Parar — corrigir lacunas manualmente"

Em **"Continuar mesmo assim"**: Exibir `Auditoria ⏭ Lacunas aceitas — prosseguindo para conclusão do milestone` e prosseguir para 5b.

Em **"Parar"**: Ir para handle_blocker com "Usuário parou — lacunas de auditoria permanecem. Execute /gsd-audit-milestone para revisar, depois /gsd-complete-milestone quando estiver pronto."

**Se `tech_debt`:**

Ler o resumo de dívida técnica do arquivo de auditoria. Exibir:
```
⚠ Auditoria: Dívida Técnica Identificada
```

Mostrar o resumo e perguntar ao usuário via AskUserQuestion:
- **question:** "A auditoria do milestone encontrou dívida técnica. Como prosseguir?"
- **options:** "Continuar com dívida técnica" / "Parar — endereçar a dívida primeiro"

Em **"Continuar com dívida técnica"**: Exibir `Auditoria ⏭ Dívida técnica reconhecida — prosseguindo para conclusão do milestone` e prosseguir para 5b.

Em **"Parar"**: Ir para handle_blocker com "Usuário parou — dívida técnica a endereçar. Execute /gsd-audit-milestone para revisar os detalhes."

**5b. Concluir Milestone**

```
Skill(skill="gsd-complete-milestone", args="${milestone_version}")
```

Após o retorno de complete-milestone, verificar se produziu saída:

```bash
ls .planning/milestones/v${milestone_version}-ROADMAP.md 2>/dev/null || true
```

Se o arquivo de arquivo não existir, ir para handle_blocker: "A conclusão do milestone não produziu os arquivos de arquivo esperados."

**5c. Limpeza**

```
Skill(skill="gsd-cleanup")
```

A limpeza mostra seu próprio dry-run e pede aprovação do usuário internamente — esta é uma pausa aceitável conforme CTRL-01, pois é uma decisão explícita sobre exclusão de arquivos.

**5d. Conclusão Final**

Exibir banner de conclusão final:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ COMPLETO 🎉
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Milestone: {milestone_version} — {milestone_name}
 Status: Concluído ✅
 Ciclo de vida: auditoria ✅ → conclusão ✅ → limpeza ✅

 Entregue! 🚀
```

</step>

<step name="handle_blocker">

## 6. Tratar Bloqueador

Quando qualquer operação de fase falha ou um bloqueador é detectado, apresentar 3 opções via AskUserQuestion:

**Prompt:** "A fase {N} ({Nome}) encontrou um problema: {descrição}"

**Opções:**
1. **"Corrigir e tentar novamente"** — Re-executar a etapa com falha (discuss, plan ou execute) para esta fase
2. **"Pular esta fase"** — Marcar fase como ignorada, continuar para a próxima fase incompleta
3. **"Parar modo autônomo"** — Exibir resumo do progresso até agora e sair limpo

**Em "Corrigir e tentar novamente":** Voltar para a etapa com falha dentro de execute_phase. Se a mesma etapa falhar novamente após a tentativa, re-apresentar essas opções.

**Em "Pular esta fase":** Registrar `Fase {N} ⏭ {Nome} — Ignorada pelo usuário` e prosseguir para iterar.

**Em "Parar modo autônomo":** Exibir resumo do progresso:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUTONOMOUS ▸ PARADO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Concluídas: {lista de fases concluídas}
 Ignoradas: {lista de fases ignoradas}
 Restantes: {lista de fases restantes}

 Retomar com: /gsd-autonomous ${ONLY_PHASE ? "--only " + ONLY_PHASE : "--from " + next_phase}${TO_PHASE ? " --to " + TO_PHASE : ""}
```

</step>

</process>

<success_criteria>
- [ ] Todas as fases incompletas executadas em ordem (smart discuss → ui-phase → plan → execute → ui-review cada uma)
- [ ] Smart discuss propõe respostas de áreas cinzentas em tabelas, usuário aceita ou substitui por área
- [ ] Banners de progresso exibidos entre fases
- [ ] Execute-phase invocado com --no-transition (autônomo gerencia transições)
- [ ] Verificação pós-execução lê VERIFICATION.md e roteia por status
- [ ] Verificação aprovada → continuação automática para a próxima fase
- [ ] Verificação human-needed → usuário solicitado a validar ou pular
- [ ] Gaps-found → usuário recebe opções de fechamento de lacunas, continuar ou parar
- [ ] Fechamento de lacunas limitado a 1 tentativa (previne loops infinitos)
- [ ] Falhas de plan-phase e execute-phase roteiam para handle_blocker
- [ ] ROADMAP.md relido após cada fase (captura fases inseridas)
- [ ] STATE.md verificado para bloqueadores antes de cada fase
- [ ] Bloqueadores tratados via escolha do usuário (tentar novamente / pular / parar)
- [ ] Conclusão final ou resumo de parada exibido
- [ ] Após todas as fases concluírem, etapa de ciclo de vida é invocada (não sugestão manual)
- [ ] Banner de transição de ciclo de vida exibido antes da auditoria
- [ ] Auditoria invocada via Skill(skill="gsd-audit-milestone")
- [ ] Roteamento do resultado da auditoria: passed → continuar automático, gaps_found → usuário decide, tech_debt → usuário decide
- [ ] Falha técnica de auditoria (sem arquivo/sem status) roteia para handle_blocker
- [ ] Complete-milestone invocado via Skill() com arg ${milestone_version}
- [ ] Cleanup invocado via Skill() — confirmação interna é aceitável (CTRL-01)
- [ ] Banner de conclusão final exibido após ciclo de vida
- [ ] Barra de progresso usa número de fase / total de fases do milestone (não posição entre incompletas), com exibição alternativa quando números de fase excedem o total
- [ ] Smart discuss documenta relacionamento com discuss-phase com nota CTRL-03
- [ ] Fases frontend recebem UI-SPEC gerado antes do planejamento (etapa 3a.5) se ainda não presente
- [ ] Fases frontend recebem auditoria de revisão de UI após execução bem-sucedida (etapa 3d.5) se UI-SPEC existir
- [ ] Fase de UI e revisão de UI respeitam toggles de configuração workflow.ui_phase e workflow.ui_review
- [ ] Revisão de UI é consultiva (não bloqueante) — fase prossegue para iteração independentemente da pontuação
- [ ] `--only N` restringe execução a exatamente uma fase
- [ ] `--only N` pula etapa de ciclo de vida (auditoria/conclusão/limpeza)
- [ ] `--only N` sai limpo após a conclusão de uma única fase
- [ ] `--only N` em fase já concluída sai com mensagem
- [ ] Mensagem de retomada do handle_blocker com `--only N` usa flag --only
- [ ] `--to N` para execução após a conclusão da fase N (para na etapa de iteração)
- [ ] `--to N` filtra fases com número > N durante a descoberta
- [ ] `--to N` exibe "Parando após a fase N" no banner de inicialização
- [ ] `--to N` em alvo já concluído sai com mensagem "já concluído"
- [ ] `--to N` compatível com `--from N` (executar fases de M a N)
- [ ] Mensagem de retomada do handle_blocker com `--to N` preserva flag --to
- [ ] `--to N` pula ciclo de vida quando nem todas as fases do milestone estão concluídas
- [ ] `--interactive` executa discuss inline via gsd:discuss-phase (faz perguntas, aguarda usuário)
- [ ] `--interactive` despacha plan e execute como agentes em segundo plano (isolamento de contexto)
- [ ] `--interactive` habilita paralelismo de pipeline: discuss Fase N+1 enquanto Fase N constrói
- [ ] `--interactive` contexto principal acumula apenas conversas de discuss (enxuto)
- [ ] `--interactive` aguarda agentes em segundo plano antes do roteamento pós-execução
- [ ] `--interactive` compatível com flags `--only`, `--from` e `--to`
</success_criteria>
