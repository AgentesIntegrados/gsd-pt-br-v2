---
name: gsd-adicionar-tarefa
description: "Registrar ideia ou tarefa como todo a partir do contexto da conversa atual"
argument-hint: "[optional description]"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---


<objective>
Registrar uma ideia, tarefa ou problema que surge durante uma sessão GSD como um todo estruturado para trabalho futuro.

Encaminha para o fluxo add-todo que cuida de:
- Criação da estrutura de diretórios
- Extração de conteúdo dos argumentos ou da conversa
- Inferência de área a partir de caminhos de arquivo
- Detecção e resolução de duplicatas
- Criação de arquivo todo com frontmatter
- Atualizações em STATE.md
- Commits git
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/add-todo.md
</execution_context>

<context>
Arguments: $ARGUMENTS (descrição opcional do todo)

O estado é resolvido no fluxo via `init todos` e leituras direcionadas.
</context>

<process>
**Seguir o fluxo add-todo** de `@$HOME/.claude/get-shit-done/workflows/add-todo.md`.

O fluxo cuida de toda a lógica, incluindo:
1. Garantia de diretório
2. Verificação de área existente
3. Extração de conteúdo (argumentos ou conversa)
4. Inferência de área
5. Verificação de duplicatas
6. Criação de arquivo com geração de slug
7. Atualizações em STATE.md
8. Commits git
</process>
