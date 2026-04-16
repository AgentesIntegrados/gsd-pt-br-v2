---
name: gsd-executar-fase
description: "Executa todos os planos de uma fase com paralelização em ondas"
argument-hint: "<número-da-fase> [--wave N] [--gaps-only] [--interactive] [--tdd]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoWrite
  - AskUserQuestion
---

<objective>
Executa todos os planos de uma fase usando execução paralela em ondas.

O orquestrador permanece enxuto: descobre os planos, analisa dependências, agrupa em ondas, cria subagentes e coleta resultados. Cada subagente carrega o contexto completo de execução de plano e gerencia seu próprio plano.

Filtro de onda opcional:
- `--wave N` executa apenas a onda `N` para controle de ritmo, gerenciamento de cota ou implantação gradual
- a verificação/conclusão da fase só acontece quando não restam planos incompletos após a onda selecionada terminar

Regra de uso de flags:
- As flags opcionais documentadas abaixo são comportamentos disponíveis, não comportamentos ativos implícitos
- Uma flag só está ativa quando seu token literal aparece em `$ARGUMENTS`
- Se uma flag documentada estiver ausente de `$ARGUMENTS`, trate-a como inativa

Orçamento de contexto: ~15% orquestrador, 100% novo por subagente.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-phase.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<runtime_note>
**Copilot (VS Code):** Use `vscode_askquestions` wherever this workflow calls `AskUserQuestion`. They are equivalent — `vscode_askquestions` is the VS Code Copilot implementation of the same interactive question API.
</runtime_note>

<context>
Fase: $ARGUMENTS

**Flags opcionais disponíveis (apenas documentação — não ficam ativas automaticamente):**
- `--wave N` — Executa apenas a onda `N` da fase. Use quando quiser controlar o ritmo de execução ou permanecer dentro dos limites de uso.
- `--gaps-only` — Executa apenas planos de fechamento de lacunas (planos com `gap_closure: true` no frontmatter). Use após o verify-work criar planos de correção.
- `--interactive` — Executa planos sequencialmente de forma inline (sem subagentes) com checkpoints do usuário entre tarefas. Menor uso de tokens, estilo programação em par. Melhor para fases pequenas, correções de bugs e lacunas de verificação.

**As flags ativas devem ser derivadas de `$ARGUMENTS`:**
- `--wave N` só está ativa se o token literal `--wave` estiver presente em `$ARGUMENTS`
- `--gaps-only` só está ativa se o token literal `--gaps-only` estiver presente em `$ARGUMENTS`
- `--interactive` só está ativa se o token literal `--interactive` estiver presente em `$ARGUMENTS`
- Se nenhum desses tokens aparecer, execute o fluxo padrão de execução completa da fase sem filtragem por flag
- Não infira que uma flag está ativa apenas porque está documentada neste prompt

Os arquivos de contexto são resolvidos dentro do fluxo via `gsd-tools init execute-phase` e blocos `<files_to_read>` por subagente.
</context>

<process>
Execute o fluxo execute-phase a partir de @$HOME/.claude/get-shit-done/workflows/execute-phase.md do início ao fim.
Preserve todos os portões do fluxo (execução em ondas, tratamento de checkpoints, verificação, atualizações de estado, roteamento).
</process>
