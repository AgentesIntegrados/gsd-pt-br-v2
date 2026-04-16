<purpose>
Lista todos os todos pendentes, permite seleção, carrega o contexto completo do todo selecionado e roteia para a ação apropriada.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="init_context">
Carregue o contexto de todos:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init todos)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extraia do JSON de inicialização: `todo_count`, `todos`, `pending_dir`.

Se `todo_count` for 0:
```
Nenhum todo pendente.

Os todos são capturados durante sessões de trabalho com /gsd-add-todo.

---

Você gostaria de:

1. Continuar com a fase atual (/gsd-progress)
2. Adicionar um todo agora (/gsd-add-todo)
```

Encerrar.
</step>

<step name="parse_filter">
Verifique o filtro de área nos argumentos:
- `/gsd-check-todos` → mostrar todos
- `/gsd-check-todos api` → filtrar apenas área:api
</step>

<step name="list_todos">
Use o array `todos` do contexto de inicialização (já filtrado por área se especificado).

Analise e exiba como lista numerada:

```
Todos Pendentes:

1. Adicionar refresh de token de auth (api, há 2d)
2. Corrigir problema de z-index no modal (ui, há 1d)
3. Refatorar pool de conexão do banco de dados (database, há 5h)

---

Responda com um número para ver os detalhes, ou:
- `/gsd-check-todos [área]` para filtrar por área
- `q` para sair
```

Formate a idade como tempo relativo a partir do timestamp de criação.
</step>

<step name="handle_selection">
Aguarde o usuário responder com um número.

Se válido: carregue o todo selecionado, prossiga.
Se inválido: "Seleção inválida. Responda com um número (1-[N]) ou `q` para sair."
</step>

<step name="load_context">
Leia o arquivo do todo completamente. Exiba:

```
## [title]

**Área:** [area]
**Criado:** [date] (há [tempo relativo])
**Arquivos:** [lista ou "Nenhum"]

### Problema
[conteúdo da seção de problema]

### Solução
[conteúdo da seção de solução]
```

Se o campo `files` tiver entradas, leia e resuma brevemente cada um.
</step>

<step name="check_roadmap">
Verifique se há roadmap (pode usar init progress ou verificar a existência do arquivo diretamente):

Se `.planning/ROADMAP.md` existir:
1. Verifique se a área do todo corresponde a uma fase futura
2. Verifique se os arquivos do todo se sobrepõem ao escopo de uma fase
3. Anote qualquer correspondência para as opções de ação
</step>

<step name="offer_actions">
**Se o todo corresponde a uma fase do roadmap:**


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON de inicialização for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada AskUserQuestion por uma lista numerada em texto simples e peça ao usuário que digite o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Use AskUserQuestion:
- header: "Ação"
- question: "Este todo está relacionado à Fase [N]: [nome]. O que você gostaria de fazer?"
- options:
  - "Trabalhar nisso agora" — mover para concluído, começar a trabalhar
  - "Adicionar ao plano da fase" — incluir ao planejar a Fase [N]
  - "Discutir abordagem" — pensar antes de decidir
  - "Deixar para depois" — voltar à lista

**Se não há correspondência no roadmap:**

Use AskUserQuestion:
- header: "Ação"
- question: "O que você gostaria de fazer com este todo?"
- options:
  - "Trabalhar nisso agora" — mover para concluído, começar a trabalhar
  - "Criar uma fase" — /gsd-add-phase com este escopo
  - "Discutir abordagem" — pensar antes de decidir
  - "Deixar para depois" — voltar à lista
</step>

<step name="execute_action">
**Trabalhar nisso agora:**
```bash
mv ".planning/todos/pending/[filename]" ".planning/todos/completed/"
```
Atualize a contagem de todos no STATE.md. Apresente o contexto de problema/solução. Comece a trabalhar ou pergunte como prosseguir.

**Adicionar ao plano da fase:**
Anote a referência do todo nas notas de planejamento da fase. Mantenha em pending. Volte à lista ou saia.

**Criar uma fase:**
Exiba: `/gsd-add-phase [descrição do todo]`
Mantenha em pending. O usuário executa o comando em novo contexto.

**Discutir abordagem:**
Mantenha em pending. Inicie discussão sobre o problema e as abordagens.

**Deixar para depois:**
Volte para a etapa list_todos.
</step>

<step name="update_state">
Após qualquer ação que altere a contagem de todos:

Re-execute `init todos` para obter a contagem atualizada e atualize a seção "### Pending Todos" do STATE.md se existir.
</step>

<step name="git_commit">
Se o todo foi movido para done/, faça commit da mudança:

```bash
git rm --cached .planning/todos/pending/[filename] 2>/dev/null || true
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: start work on todo - [title]" --files .planning/todos/completed/[filename] .planning/STATE.md
```

A ferramenta respeita a config `commit_docs` e o gitignore automaticamente.

Confirme: "Committed: docs: start work on todo - [title]"
</step>

</process>

<success_criteria>
- [ ] Todos os todos pendentes listados com título, área e idade
- [ ] Filtro de área aplicado se especificado
- [ ] Contexto completo do todo selecionado carregado
- [ ] Contexto do roadmap verificado para correspondência de fase
- [ ] Ações apropriadas oferecidas
- [ ] Ação selecionada executada
- [ ] STATE.md atualizado se a contagem de todos mudou
- [ ] Mudanças commitadas no git (se o todo foi movido para done/)
</success_criteria>
