---
name: gsd-planejar-fase
description: "Criar plano detalhado da fase (PLAN.md) com loop de verificação"
argument-hint: "[fase] [--auto] [--research] [--skip-research] [--gaps] [--skip-verify] [--prd <arquivo>] [--reviews] [--text] [--tdd]"
agent: gsd-planner
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
  - WebFetch
  - mcp__context7__*
---

<objective>
Criar prompts executáveis de fase (arquivos PLAN.md) para uma fase do roadmap com pesquisa integrada e verificação.

**Fluxo padrão:** Pesquisa (se necessário) → Plano → Verificar → Concluído

**Papel do orquestrador:** Analisar argumentos, validar fase, pesquisar domínio (a menos que pulado), iniciar gsd-planner, verificar com gsd-plan-checker, iterar até passar ou atingir o máximo de iterações, apresentar resultados.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/plan-phase.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<runtime_note>
**Copilot (VS Code):** Use `vscode_askquestions` wherever this workflow calls `AskUserQuestion`. They are equivalent — `vscode_askquestions` is the VS Code Copilot implementation of the same interactive question API. Do not skip questioning steps because `AskUserQuestion` appears unavailable; use `vscode_askquestions` instead.
</runtime_note>

<context>
Número da fase: $ARGUMENTS (opcional — detecta automaticamente a próxima fase não planejada se omitido)

**Flags:**
- `--research` — Forçar nova pesquisa mesmo se RESEARCH.md já existir
- `--skip-research` — Pular pesquisa, ir direto para o planejamento
- `--gaps` — Modo de fechamento de lacunas (lê VERIFICATION.md, pula pesquisa)
- `--skip-verify` — Pular loop de verificação
- `--prd <arquivo>` — Usar um arquivo PRD/critérios de aceitação em vez de discuss-phase. Converte requisitos em CONTEXT.md automaticamente. Pula o discuss-phase completamente.
- `--reviews` — Replanejar incorporando feedback de revisão cross-IA do REVIEWS.md (produzido por `/gsd-review`)
- `--text` — Usar listas numeradas em texto simples em vez de menus TUI (obrigatório para sessões remotas `/rc`)

Normalizar entrada de fase no passo 2 antes de qualquer consulta de diretório.
</context>

<process>
Executar o workflow plan-phase de @$HOME/.claude/get-shit-done/workflows/plan-phase.md do início ao fim.
Preservar todos os pontos de controle do workflow (validação, pesquisa, planejamento, loop de verificação, roteamento).
</process>
</content>
