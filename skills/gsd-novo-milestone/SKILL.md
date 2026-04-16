---
name: gsd-novo-milestone
description: "Iniciar um novo ciclo de milestone — atualizar PROJECT.md e encaminhar para requisitos"
argument-hint: "[nome do milestone, ex.: 'v1.1 Notificações']"
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion
---

<objective>
Iniciar um novo milestone: questionamento → pesquisa (opcional) → requisitos → roadmap.

Equivalente brownfield do new-project. O projeto existe e PROJECT.md tem histórico. Coleta "o que vem a seguir", atualiza PROJECT.md e então executa o ciclo requisitos → roadmap.

**Cria/Atualiza:**
- `.planning/PROJECT.md` — atualizado com os objetivos do novo milestone
- `.planning/research/` — pesquisa de domínio (opcional, apenas para funcionalidades NOVAS)
- `.planning/REQUIREMENTS.md` — requisitos com escopo para este milestone
- `.planning/ROADMAP.md` — estrutura de fases (continua a numeração)
- `.planning/STATE.md` — reiniciado para o novo milestone

**Após:** `/gsd-plan-phase [N]` para iniciar a execução.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/new-milestone.md
@$HOME/.claude/get-shit-done/references/questioning.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
@$HOME/.claude/get-shit-done/templates/project.md
@$HOME/.claude/get-shit-done/templates/requirements.md
</execution_context>

<context>
Nome do milestone: $ARGUMENTS (opcional - será solicitado se não fornecido)

Contexto do projeto e do milestone são resolvidos internamente pelo workflow (`init new-milestone`) e delegados via blocos `<files_to_read>` onde subagentes são usados.
</context>

<process>
Executar o workflow new-milestone de @$HOME/.claude/get-shit-done/workflows/new-milestone.md do início ao fim.
Preservar todos os pontos de controle do workflow (validação, questionamento, pesquisa, requisitos, aprovação do roadmap, commits).
</process>
</content>
