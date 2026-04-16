---
name: gsd-revisao
description: "Solicitar revisão por pares entre IAs dos planos de fase por CLIs externas de IA"
argument-hint: "--phase N [--gemini] [--claude] [--codex] [--opencode] [--qwen] [--cursor] [--all]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
---


<objective>
Invocar CLIs externas de IA (Gemini, Claude, Codex, OpenCode, Qwen Code, Cursor) para revisar planos de fase de forma independente.
Produz um REVIEWS.md estruturado com feedback por revisor que pode ser reinserido no
planejamento via /gsd-plan-phase --reviews.

**Fluxo:** Detectar CLIs → Construir prompt de revisão → Invocar cada CLI → Coletar respostas → Escrever REVIEWS.md
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/review.md
</execution_context>

<context>
Número da fase: extraído de $ARGUMENTS (obrigatório)

**Flags:**
- `--gemini` — Incluir revisão da CLI do Gemini
- `--claude` — Incluir revisão da CLI do Claude (usa sessão separada)
- `--codex` — Incluir revisão da CLI do Codex
- `--opencode` — Incluir revisão do OpenCode (usa modelo da configuração OpenCode do usuário)
- `--qwen` — Incluir revisão do Qwen Code (modelos Qwen da Alibaba)
- `--cursor` — Incluir revisão do agente Cursor
- `--all` — Incluir todas as CLIs disponíveis
</context>

<process>
Executar o fluxo de trabalho review de @$HOME/.claude/get-shit-done/workflows/review.md do início ao fim.
</process>
