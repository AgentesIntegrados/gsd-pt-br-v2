<purpose>
Gerar um contrato de design de UI (UI-SPEC.md) para fases frontend. Orquestra gsd-ui-researcher e gsd-ui-checker com loop de revisão. Inserido entre discuss-phase e plan-phase no ciclo de vida.

UI-SPEC.md trava espaçamento, tipografia, cor, copywriting e decisões de sistema de design antes que o planejador crie as tarefas. Isso previne dívida de design causada por decisões de estilo ad-hoc durante a execução.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<available_agent_types>
Tipos de subagentes GSD válidos (use nomes exatos — não use substitutos genéricos):
- gsd-ui-researcher — Pesquisa abordagens de UI/UX
- gsd-ui-checker — Revisa a qualidade da implementação de UI
</available_agent_types>

<process>

## 1. Inicializar

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_UI=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-ui-researcher 2>/dev/null)
AGENT_SKILLS_UI_CHECKER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-ui-checker 2>/dev/null)
```

Analisar JSON para: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_context`, `has_research`, `commit_docs`.

Resolver modelos de agentes de UI:

```bash
UI_RESEARCHER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-ui-researcher --raw)
UI_CHECKER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-ui-checker --raw)
```

Verificar config:

```bash
UI_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ui_phase 2>/dev/null || echo "true")
```

**Se `UI_ENABLED` for `false`:**
```
Fase de UI está desativada na config. Ative via /gsd-settings.
```
Sair do fluxo de trabalho.

## 2. Analisar e Validar Fase

Extrair número de fase de $ARGUMENTS. Se não fornecido, detectar a próxima fase não planejada.

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

**Se `found` for false:** Erro com as fases disponíveis.

## 3. Verificar Pré-requisitos

**Se `has_context` for false:**
```
Nenhum CONTEXT.md encontrado para a Fase {N}.
Recomendado: execute /gsd-discuss-phase {N} primeiro para capturar preferências de design.
Continuando sem decisões do usuário — o pesquisador de UI fará todas as perguntas.
```
Continuar (não-bloqueante).

## 4. Verificar UI-SPEC Existente

```bash
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
```

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

**Se existir:** Usar AskUserQuestion:
- header: "UI-SPEC Existente"
- question: "UI-SPEC.md já existe para a Fase {N}. O que você gostaria de fazer?"
- options:
  - "Atualizar — re-executar pesquisador com existente como base"
  - "Visualizar — exibir UI-SPEC atual e sair"
  - "Pular — manter UI-SPEC atual, prosseguir para verificação"

Se "Visualizar": exibir conteúdo do arquivo, sair.
Se "Pular": prosseguir para o passo 7 (verificador).
Se "Atualizar": continuar para o passo 5.

## 5. Iniciar gsd-ui-researcher

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CONTRATO DE DESIGN DE UI — FASE {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando pesquisador de UI...
```

```
Task(
  prompt=ui_research_prompt,
  subagent_type="gsd-ui-researcher",
  model="{UI_RESEARCHER_MODEL}",
  description="Contrato de Design de UI Fase {N}"
)
```

## 6. Lidar com Retorno do Pesquisador

**Se `## UI-SPEC CONCLUÍDO`:**
Exibir confirmação. Continuar para o passo 7.

**Se `## UI-SPEC BLOQUEADO`:**
Exibir detalhes do bloqueio e opções. Sair do fluxo de trabalho.

## 7. Iniciar gsd-ui-checker

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO UI-SPEC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando verificador de UI...
```

```
Task(
  prompt=ui_checker_prompt,
  subagent_type="gsd-ui-checker",
  model="{UI_CHECKER_MODEL}",
  description="Verificar UI-SPEC Fase {N}"
)
```

## 8. Lidar com Retorno do Verificador

**Se `## UI-SPEC VERIFICADO`:**
Exibir resultados das dimensões. Prosseguir para o passo 10.

**Se `## PROBLEMAS ENCONTRADOS`:**
Exibir problemas bloqueantes. Prosseguir para o passo 9.

## 9. Loop de Revisão (Máx 2 Iterações)

Rastrear `revision_count` (começa em 0).

**Se `revision_count` < 2:**
- Incrementar `revision_count`
- Re-iniciar gsd-ui-researcher com contexto de revisão
- Após retorno do pesquisador → re-iniciar verificador (passo 7)

**Se `revision_count` >= 2:**
```
Máximo de iterações de revisão atingido. Problemas restantes:

{lista de problemas restantes}

Opções:
1. Forçar aprovação — prosseguir com UI-SPEC atual
2. Editar manualmente — abrir UI-SPEC.md no editor, re-executar /gsd-ui-phase
3. Abandonar — sair sem aprovar
```

Usar AskUserQuestion para a escolha.

## 10. Apresentar Status Final

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► UI-SPEC PRONTO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {N}: {Nome}** — contrato de design de UI aprovado

Dimensões: 6/6 aprovadas

───────────────────────────────────────────────────────────────

## ▶ Próximos Passos

`/clear` então: `/gsd-plan-phase {N}`

───────────────────────────────────────────────────────────────
```

## 11. Commit (se configurado)

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): contrato de design de UI" --files "${PHASE_DIR}/${PADDED_PHASE}-UI-SPEC.md"
```

## 12. Atualizar State

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Fase ${PHASE} UI-SPEC aprovada" \
  --resume-file "${PHASE_DIR}/${PADDED_PHASE}-UI-SPEC.md"
```

</process>

<success_criteria>
- [ ] Config verificada (sair se ui_phase estiver desativada)
- [ ] Fase validada contra o roadmap
- [ ] Pré-requisitos verificados (CONTEXT.md, RESEARCH.md — avisos não-bloqueantes)
- [ ] UI-SPEC existente tratada (atualizar/visualizar/pular)
- [ ] gsd-ui-researcher iniciado com contexto correto
- [ ] UI-SPEC.md criado no local correto
- [ ] gsd-ui-checker iniciado com UI-SPEC.md
- [ ] Todas as 6 dimensões avaliadas
- [ ] Loop de revisão se BLOQUEADO (máx 2 iterações)
- [ ] Status final exibido com próximos passos
- [ ] UI-SPEC.md commitado (se commit_docs habilitado)
- [ ] State atualizado
</success_criteria>
