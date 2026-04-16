---
name: gsd-nota
description: "Captura de ideias sem fricção. Adicionar, listar ou promover notas para todos."
argument-hint: "<texto> | list | promote <N> [--global]"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
---

<objective>
Captura de ideias sem fricção — uma chamada Write, uma linha de confirmação.

Três subcomandos:
- **append** (padrão): Salvar um arquivo de nota com timestamp. Sem perguntas, sem formatação.
- **list**: Exibir todas as notas dos escopos do projeto e global.
- **promote**: Converter uma nota em um todo estruturado.

Executa inline — sem Task, sem AskUserQuestion, sem Bash.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/note.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
Executar o workflow note de @$HOME/.claude/get-shit-done/workflows/note.md do início ao fim.
Capturar a nota, listar notas ou promover para todo — dependendo dos argumentos.
</process>
</content>
