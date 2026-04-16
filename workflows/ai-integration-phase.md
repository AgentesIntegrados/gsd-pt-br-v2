<purpose>
Gera um contrato de design de IA (AI-SPEC.md) para fases que envolvem a construção de sistemas de IA. Orquestra gsd-framework-selector → gsd-ai-researcher → gsd-domain-researcher → gsd-eval-planner com um gate de validação. Inserido entre discuss-phase e plan-phase no ciclo de vida GSD.

AI-SPEC.md bloqueia quatro coisas antes que o planejador crie tarefas:
1. Seleção de framework (com justificativa e alternativas)
2. Guia de implementação (sintaxe correta, padrões, armadilhas da documentação oficial)
3. Contexto de domínio (ingredientes de rubrica de praticantes, modos de falha, restrições regulatórias)
4. Estratégia de avaliação (dimensões, rubricas, ferramentas, dataset de referência, guardrails)

Isso previne as duas falhas mais comuns no desenvolvimento de IA: escolher o framework errado para o caso de uso, e tratar a avaliação como um pós-pensamento.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ai-frameworks.md
@$HOME/.claude/get-shit-done/references/ai-evals.md
</required_reading>

<process>

## 1. Inicializar

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "$PHASE")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_context`, `has_research`, `commit_docs`.

**Caminhos de arquivo:** `state_path`, `roadmap_path`, `requirements_path`, `context_path`.

Resolva os modelos dos agentes:
```bash
SELECTOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-framework-selector --raw)
RESEARCHER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-ai-researcher --raw)
DOMAIN_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-domain-researcher --raw)
PLANNER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-eval-planner --raw)
```

Verifique a config:
```bash
AI_PHASE_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.ai_integration_phase 2>/dev/null || echo "true")
```

**Se `AI_PHASE_ENABLED` for `false`:**
```
A fase de IA está desabilitada na config. Habilite via /gsd-settings.
```
Encerre o workflow.

**Se `planning_exists` for false:** Erro — execute `/gsd-new-project` primeiro.

## 2. Analisar e Validar Fase

Extraia o número da fase de $ARGUMENTS. Se não fornecido, detecte a próxima fase não planejada.

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}")
```

**Se `found` for false:** Erro com as fases disponíveis.

## 3. Verificar Pré-requisitos

**Se `has_context` for false:**
```
Nenhum CONTEXT.md encontrado para a Fase {N}.
Recomendado: execute /gsd-discuss-phase {N} primeiro para capturar preferências de framework.
Continuando sem decisões do usuário — o seletor de framework fará todas as perguntas.
```
Continue (não bloqueante).

## 4. Verificar AI-SPEC Existente

```bash
AI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-AI-SPEC.md 2>/dev/null | head -1)
```


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON de inicialização for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada AskUserQuestion por uma lista numerada em texto simples e peça ao usuário que digite o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
**Se existir:** Use AskUserQuestion:
- header: "AI-SPEC Existente"
- question: "AI-SPEC.md já existe para a Fase {N}. O que você gostaria de fazer?"
- options:
  - "Atualizar — re-executar com o existente como base"
  - "Visualizar — exibir o AI-SPEC atual e sair"
  - "Pular — manter o AI-SPEC atual e sair"

Se "Visualizar": exiba o conteúdo do arquivo, saia.
Se "Pular": saia.
Se "Atualizar": continue para a etapa 5.

## 5. Inicializar gsd-framework-selector

Exiba:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CONTRATO DE DESIGN DE IA — FASE {N}: {name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Etapa 1/4 — Seleção de Framework...
```

Inicie `gsd-framework-selector` com:
```markdown
Read $HOME/.claude/agents/gsd-framework-selector.md for instructions.

<objective>
Select the right AI framework for Phase {phase_number}: {phase_name}
Goal: {phase_goal}
</objective>

<files_to_read>
{context_path if exists}
{requirements_path if exists}
</files_to_read>

<phase_context>
Phase: {phase_number} — {phase_name}
Goal: {phase_goal}
</phase_context>
```

Analise a saída do seletor para: `primary_framework`, `system_type`, `model_provider`, `eval_concerns`, `alternative_framework`.

**Se o seletor falhar ou retornar vazio:** Encerre com erro — "Seleção de framework falhou. Re-execute /gsd-ai-integration-phase {N} ou responda a pergunta do framework em /gsd-discuss-phase {N} primeiro."

## 6. Inicializar AI-SPEC.md

Copie o template:
```bash
cp "$HOME/.claude/get-shit-done/templates/AI-SPEC.md" "${PHASE_DIR}/${PADDED_PHASE}-AI-SPEC.md"
```

Preencha os campos de cabeçalho:
- Número e nome da fase
- Classificação do sistema (do seletor)
- Framework selecionado (do seletor)
- Alternativa considerada (do seletor)

## 7. Inicializar gsd-ai-researcher

Exiba:
```
◆ Etapa 2/4 — Pesquisando docs do {primary_framework} + melhores práticas de sistemas de IA...
```

Inicie `gsd-ai-researcher` com:
```markdown
Read $HOME/.claude/agents/gsd-ai-researcher.md for instructions.

<objective>
Research {primary_framework} for Phase {phase_number}: {phase_name}
Write Sections 3 and 4 of AI-SPEC.md
</objective>

<files_to_read>
{ai_spec_path}
{context_path if exists}
</files_to_read>

<input>
framework: {primary_framework}
system_type: {system_type}
model_provider: {model_provider}
ai_spec_path: {ai_spec_path}
phase_context: Phase {phase_number}: {phase_name} — {phase_goal}
</input>
```

## 8. Inicializar gsd-domain-researcher

Exiba:
```
◆ Etapa 3/4 — Pesquisando contexto de domínio e critérios de avaliação de especialistas...
```

Inicie `gsd-domain-researcher` com:
```markdown
Read $HOME/.claude/agents/gsd-domain-researcher.md for instructions.

<objective>
Research the business domain and expert evaluation criteria for Phase {phase_number}: {phase_name}
Write Section 1b (Domain Context) of AI-SPEC.md
</objective>

<files_to_read>
{ai_spec_path}
{context_path if exists}
{requirements_path if exists}
</files_to_read>

<input>
system_type: {system_type}
phase_name: {phase_name}
phase_goal: {phase_goal}
ai_spec_path: {ai_spec_path}
</input>
```

## 9. Inicializar gsd-eval-planner

Exiba:
```
◆ Etapa 4/4 — Projetando estratégia de avaliação a partir do contexto de domínio + técnico...
```

Inicie `gsd-eval-planner` com:
```markdown
Read $HOME/.claude/agents/gsd-eval-planner.md for instructions.

<objective>
Design evaluation strategy for Phase {phase_number}: {phase_name}
Write Sections 5, 6, and 7 of AI-SPEC.md
AI-SPEC.md now contains domain context (Section 1b) — use it as your rubric starting point.
</objective>

<files_to_read>
{ai_spec_path}
{context_path if exists}
{requirements_path if exists}
</files_to_read>

<input>
system_type: {system_type}
framework: {primary_framework}
model_provider: {model_provider}
phase_name: {phase_name}
phase_goal: {phase_goal}
ai_spec_path: {ai_spec_path}
</input>
```

## 10. Validar Completude do AI-SPEC

Leia o AI-SPEC.md concluído. Verifique que:
- A Seção 2 tem um nome de framework (não placeholder)
- A Seção 1b tem pelo menos um ingrediente de rubrica de domínio (Bom/Ruim/Stakes)
- A Seção 3 tem um bloco de código não vazio (padrão de ponto de entrada)
- A Seção 4b tem um exemplo Pydantic
- A Seção 5 tem pelo menos uma linha na tabela de dimensões
- A Seção 6 tem pelo menos um guardrail ou nota explícita "N/A para ferramenta interna"
- A seção de checklist no final tem 3+ itens marcados

**Se a validação falhar:** Exiba as seções específicas faltando. Pergunte ao usuário se deseja re-executar a etapa específica ou continuar mesmo assim.

## 11. Commit

**Se `commit_docs` for true:**
```bash
git add "${AI_SPEC_FILE}"
git commit -m "docs({phase_slug}): generate AI-SPEC.md — {primary_framework} + domain context + eval strategy"
```

## 12. Exibir Conclusão

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AI-SPEC CONCLUÍDO — FASE {N}: {name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Framework: {primary_framework}
◆ Tipo de Sistema: {system_type}
◆ Domínio: {domain_vertical da Seção 1b}
◆ Dimensões de Avaliação: {eval_concerns}
◆ Rastreamento padrão: Arize Phoenix (ou ferramenta existente detectada)
◆ Saída: {ai_spec_path}

Próximo passo:
  /gsd-plan-phase {N}   — o planejador consumirá AI-SPEC.md
```

</process>

<success_criteria>
- [ ] Framework selecionado com justificativa (Seção 2)
- [ ] AI-SPEC.md criado a partir do template
- [ ] Docs do framework + melhores práticas de IA pesquisadas (Seções 3, 4, 4b preenchidas)
- [ ] Contexto de domínio + ingredientes de rubrica de especialista pesquisados (Seção 1b preenchida)
- [ ] Estratégia de avaliação fundamentada no contexto de domínio (Seções 5-7 preenchidas)
- [ ] Arize Phoenix (ou ferramenta detectada) definido como padrão de rastreamento na Seção 7
- [ ] AI-SPEC.md validado (Seções 1b, 2, 3, 4b, 5, 6 todas não vazias)
- [ ] Commitado se commit_docs habilitado
- [ ] Próximo passo apresentado ao usuário
</success_criteria>
