---
name: gsd-novo-projeto
description: "Inicializar um novo projeto com coleta profunda de contexto e PROJECT.md"
argument-hint: "[--auto]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<runtime_note>
**Copilot (VS Code):** Use `vscode_askquestions` wherever this workflow calls `AskUserQuestion`. They are equivalent — `vscode_askquestions` is the VS Code Copilot implementation of the same interactive question API.
</runtime_note>

<context>
**Flags:**
- `--auto` — Modo automático. Após as perguntas de configuração, executa pesquisa → requisitos → roadmap sem interação adicional. Espera documento de ideia via referência @.
</context>

<objective>
Inicializar um novo projeto através de um fluxo unificado: questionamento → pesquisa (opcional) → requisitos → roadmap.

**Cria:**
- `.planning/PROJECT.md` — contexto do projeto
- `.planning/config.json` — preferências do workflow
- `.planning/research/` — pesquisa de domínio (opcional)
- `.planning/REQUIREMENTS.md` — requisitos com escopo definido
- `.planning/ROADMAP.md` — estrutura de fases
- `.planning/STATE.md` — memória do projeto

**Após este comando:** Execute `/gsd-plan-phase 1` para iniciar a execução.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/new-project.md
@$HOME/.claude/get-shit-done/references/questioning.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
@$HOME/.claude/get-shit-done/templates/project.md
@$HOME/.claude/get-shit-done/templates/requirements.md
</execution_context>

<process>
Executar o workflow new-project de @$HOME/.claude/get-shit-done/workflows/new-project.md do início ao fim.
Preservar todos os pontos de controle do workflow (validação, aprovações, commits, roteamento).
</process>
</content>
