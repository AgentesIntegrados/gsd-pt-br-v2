---
name: gsd-verificar-tarefas
description: "Listar todos pendentes e selecionar um para trabalhar"
argument-hint: "[area filter]"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---


<objective>
Listar todos os todos pendentes, permitir seleção, carregar o contexto completo do todo selecionado e encaminhar para a ação apropriada.

Encaminha para o fluxo check-todos que cuida de:
- Contagem e listagem de todos com filtragem por área
- Seleção interativa com carregamento completo de contexto
- Verificação de correlação com o roadmap
- Roteamento de ação (trabalhar agora, adicionar à fase, brainstormar, criar fase)
- Atualizações em STATE.md e commits git
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/check-todos.md
</execution_context>

<context>
Arguments: $ARGUMENTS (filtro de área opcional)

O estado dos todos e a correlação com o roadmap são carregados no fluxo usando `init todos` e leituras direcionadas.
</context>

<process>
**Seguir o fluxo check-todos** de `@$HOME/.claude/get-shit-done/workflows/check-todos.md`.

O fluxo cuida de toda a lógica, incluindo:
1. Verificação de existência de todos
2. Filtragem por área
3. Listagem e seleção interativa
4. Carregamento completo de contexto com resumos de arquivos
5. Verificação de correlação com o roadmap
6. Oferta e execução de ações
7. Atualizações em STATE.md
8. Commits git
</process>
