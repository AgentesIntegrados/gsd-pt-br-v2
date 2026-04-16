<purpose>
Auditoria retroativa da cobertura de avaliação de uma fase de IA implementada. Comando standalone que funciona em qualquer fase GSD gerenciada com IA. Produz um EVAL-REVIEW.md pontuado com análise de lacunas e plano de remediação.

Use após /gsd-execute-phase para verificar se a estratégia de avaliação do AI-SPEC.md foi realmente implementada. Espelha o padrão de /gsd-ui-review e /gsd-validate-phase.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ai-evals.md
</required_reading>

<process>

## 0. Inicializar

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `commit_docs`.

```bash
AUDITOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-eval-auditor --raw)
```

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUDITORIA DE EVAL — FASE {N}: {name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 1. Detectar Estado de Entrada

```bash
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
AI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-AI-SPEC.md 2>/dev/null | head -1)
EVAL_REVIEW_FILE=$(ls "${PHASE_DIR}"/*-EVAL-REVIEW.md 2>/dev/null | head -1)
```

**Estado A** — AI-SPEC.md + SUMMARY.md existem: Auditoria completa contra spec
**Estado B** — SUMMARY.md existe, sem AI-SPEC.md: Auditoria contra melhores práticas gerais
**Estado C** — Sem SUMMARY.md: Encerre — "Fase {N} não executada. Execute /gsd-execute-phase {N} primeiro."


**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON init for `true`. Quando TEXT_MODE estiver ativo, substitua toda chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
**Se `EVAL_REVIEW_FILE` não estiver vazio:** Use AskUserQuestion:
- header: "Revisão de Eval Existente"
- question: "EVAL-REVIEW.md já existe para a Fase {N}."
- options:
  - "Re-audit — executar nova auditoria"
  - "View — exibir revisão atual e encerrar"

Se "View": exiba o arquivo, encerre.
Se "Re-audit": continue.

**Se Estado B (sem AI-SPEC.md):** Avise:
```
Nenhum AI-SPEC.md encontrado para a Fase {N}.
A auditoria avaliará contra as melhores práticas gerais de avaliação de IA em vez de um plano específico da fase.
Considere executar /gsd-ai-integration-phase {N} antes da implementação da próxima vez.
```
Continue (não bloqueante).

## 2. Reunir Caminhos de Contexto

Construa a lista de arquivos para o auditor:
- AI-SPEC.md (se existir — a estratégia de avaliação planejada)
- Todos os arquivos SUMMARY.md no diretório da fase
- Todos os arquivos PLAN.md no diretório da fase

## 3. Iniciar gsd-eval-auditor

```
◆ Iniciando auditor de eval...
```

Construa o prompt:

```markdown
Leia $HOME/.claude/agents/gsd-eval-auditor.md para as instruções.

<objective>
Conduza auditoria de cobertura de avaliação da Fase {phase_number}: {phase_name}
{Se AI-SPEC existir: "Audite contra o plano de avaliação do AI-SPEC.md."}
{Se sem AI-SPEC: "Audite contra as melhores práticas gerais de avaliação de IA."}
</objective>

<files_to_read>
- {summary_paths}
- {plan_paths}
- {ai_spec_path se existir}
</files_to_read>

<input>
ai_spec_path: {ai_spec_path ou "none"}
phase_dir: {phase_dir}
phase_number: {phase_number}
phase_name: {phase_name}
padded_phase: {padded_phase}
state: {A ou B}
</input>
```

Inicie como Task com o modelo `AUDITOR_MODEL`.

## 4. Analisar Resultado do Auditor

Leia o EVAL-REVIEW.md escrito. Extraia:
- `overall_score`
- `verdict` (PRODUCTION READY | NEEDS WORK | SIGNIFICANT GAPS | NOT IMPLEMENTED)
- `critical_gap_count`

## 5. Exibir Resumo

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUDITORIA DE EVAL CONCLUÍDA — FASE {N}: {name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Pontuação: {overall_score}/100
◆ Veredicto: {verdict}
◆ Lacunas Críticas: {critical_gap_count}
◆ Saída: {eval_review_path}

{Se PRODUCTION READY:}
  Próximo passo: /gsd-plan-phase (próxima fase) ou deploy

{Se NEEDS WORK:}
  Trate as lacunas críticas no EVAL-REVIEW.md e depois re-execute /gsd-eval-review {N}

{Se SIGNIFICANT GAPS ou NOT IMPLEMENTED:}
  Revise o plano de avaliação do AI-SPEC.md. Dimensões críticas de avaliação não estão implementadas.
  Não faça deploy até que as lacunas sejam tratadas.
```

## 6. Commit

**Se `commit_docs` for true:**
```bash
git add "${EVAL_REVIEW_FILE}"
git commit -m "docs({phase_slug}): add EVAL-REVIEW.md — score {overall_score}/100 ({verdict})"
```

</process>

<success_criteria>
- [ ] Estado de execução da fase detectado corretamente
- [ ] Presença de AI-SPEC.md tratada (com ou sem)
- [ ] gsd-eval-auditor iniciado com contexto correto
- [ ] EVAL-REVIEW.md escrito (pelo auditor)
- [ ] Pontuação e veredicto exibidos ao usuário
- [ ] Próximos passos apropriados apresentados com base no veredicto
- [ ] Commit realizado se commit_docs estiver habilitado
</success_criteria>
