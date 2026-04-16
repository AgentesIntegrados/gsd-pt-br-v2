<purpose>
Auditoria visual retroativa de 6 pilares do código frontend implementado. Comando autônomo que funciona em qualquer projeto — gerenciado pelo GSD ou não. Produz UI-REVIEW.md pontuado com descobertas acionáveis.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<available_agent_types>
Tipos de subagentes GSD válidos (use nomes exatos — não use substitutos genéricos):
- gsd-ui-auditor — Audita a UI contra requisitos de design
</available_agent_types>

<process>

## 0. Inicializar

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_UI_REVIEWER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-ui-reviewer 2>/dev/null)
```

Analisar: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `commit_docs`.

```bash
UI_AUDITOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-ui-auditor --raw)
```

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUDITORIA DE UI — FASE {N}: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 1. Detectar Estado da Entrada

```bash
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
UI_SPEC_FILE=$(ls "${PHASE_DIR}"/*-UI-SPEC.md 2>/dev/null | head -1)
UI_REVIEW_FILE=$(ls "${PHASE_DIR}"/*-UI-REVIEW.md 2>/dev/null | head -1)
```

**Se `SUMMARY_FILES` estiver vazio:** Sair — "Fase {N} não executada. Execute /gsd-execute-phase {N} primeiro."

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

**Se `UI_REVIEW_FILE` não estiver vazio:** Usar AskUserQuestion:
- header: "Revisão de UI Existente"
- question: "UI-REVIEW.md já existe para a Fase {N}."
- options:
  - "Re-auditar — executar auditoria nova"
  - "Visualizar — exibir revisão atual e sair"

Se "Visualizar": exibir arquivo, sair.
Se "Re-auditar": continuar.

## 2. Reunir Caminhos de Contexto

Construir lista de arquivos para o auditor:
- Todos os arquivos SUMMARY.md no diretório da fase
- Todos os arquivos PLAN.md no diretório da fase
- UI-SPEC.md (se existir — baseline de auditoria)
- CONTEXT.md (se existir — decisões bloqueadas)

## 3. Iniciar gsd-ui-auditor

```
◆ Iniciando auditor de UI...
```

```
Task(
  prompt=ui_audit_prompt,
  subagent_type="gsd-ui-auditor",
  model="{UI_AUDITOR_MODEL}",
  description="Auditoria de UI Fase {N}"
)
```

## 4. Lidar com Retorno

**Se `## REVISÃO DE UI CONCLUÍDA`:**

Exibir resumo de pontuação:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AUDITORIA DE UI CONCLUÍDA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {N}: {Nome}** — Total: {pontuação}/24

| Pilar | Pontuação |
|-------|-----------|
| Copywriting | {N}/4 |
| Visual | {N}/4 |
| Cor | {N}/4 |
| Tipografia | {N}/4 |
| Espaçamento | {N}/4 |
| Design de Experiência | {N}/4 |

Principais correções:
1. {correção}
2. {correção}
3. {correção}

Revisão completa: {caminho para UI-REVIEW.md}

───────────────────────────────────────────────────────────────

## ▶ Próximos passos

`/clear` então um dos:

- `/gsd-verify-work {N}` — teste UAT
- `/gsd-plan-phase {N+1}` — planejar próxima fase

───────────────────────────────────────────────────────────────
```

## Verificação Automatizada de UI (quando Playwright-MCP estiver disponível)

Se as ferramentas `mcp__playwright__*` estiverem acessíveis nesta sessão:

1. Navegar para cada componente de UI descrito no UI-SPEC.md da fase
2. Tirar screenshot de cada componente
3. Comparar contra os requisitos visuais da spec
4. Reportar discrepâncias automaticamente como descobertas adicionais no UI-REVIEW.md
5. Sinalizar itens que requerem julgamento humano como `needs_human_review: true`

Se o Playwright-MCP não estiver disponível, esta seção é completamente ignorada.

## 5. Commit (se configurado)

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): revisão de auditoria de UI" --files "${PHASE_DIR}/${PADDED_PHASE}-UI-REVIEW.md"
```

</process>

<success_criteria>
- [ ] Fase validada
- [ ] Arquivos SUMMARY.md encontrados (execução concluída)
- [ ] Revisão existente tratada (re-auditar/visualizar)
- [ ] gsd-ui-auditor iniciado com contexto correto
- [ ] UI-REVIEW.md criado no diretório da fase
- [ ] Resumo de pontuação exibido ao usuário
- [ ] Próximos passos apresentados
</success_criteria>
