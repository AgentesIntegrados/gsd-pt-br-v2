<purpose>
Captura uma ideia, tarefa ou problema que surge durante uma sessão GSD como um todo estruturado para trabalho futuro. Permite o fluxo "pensamento → captura → continuar" sem perder contexto.
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

Extraia do JSON de inicialização: `commit_docs`, `date`, `timestamp`, `todo_count`, `todos`, `pending_dir`, `todos_dir_exists`.

Garanta que os diretórios existam:
```bash
mkdir -p .planning/todos/pending .planning/todos/completed
```

Anote as áreas existentes do array todos para consistência na etapa infer_area.
</step>

<step name="extract_content">
**Com argumentos:** Use como título/foco.
- `/gsd-add-todo Adicionar refresh de token de auth` → title = "Adicionar refresh de token de auth"

**Sem argumentos:** Analise a conversa recente para extrair:
- O problema específico, ideia ou tarefa discutida
- Caminhos de arquivo relevantes mencionados
- Detalhes técnicos (mensagens de erro, números de linha, restrições)

Formule:
- `title`: título descritivo de 3 a 10 palavras (verbo de ação preferido)
- `problem`: O que está errado ou por que isso é necessário
- `solution`: Dicas de abordagem ou "TBD" se for apenas uma ideia
- `files`: Caminhos relevantes com números de linha da conversa
</step>

<step name="infer_area">
Infira a área a partir dos caminhos de arquivo:

| Padrão de caminho | Área |
|--------------|------|
| `src/api/*`, `api/*` | `api` |
| `src/components/*`, `src/ui/*` | `ui` |
| `src/auth/*`, `auth/*` | `auth` |
| `src/db/*`, `database/*` | `database` |
| `tests/*`, `__tests__/*` | `testing` |
| `docs/*` | `docs` |
| `.planning/*` | `planning` |
| `scripts/*`, `bin/*` | `tooling` |
| Sem arquivos ou indefinido | `general` |

Use a área existente da etapa 2 se houver correspondência semelhante.
</step>

<step name="check_duplicates">
```bash
# Pesquise palavras-chave do título nos todos existentes
grep -l -i "[palavras-chave do título]" .planning/todos/pending/*.md 2>/dev/null || true
```

Se um possível duplicado for encontrado:
1. Leia o todo existente
2. Compare o escopo


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON de inicialização for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada AskUserQuestion por uma lista numerada em texto simples e peça ao usuário que digite o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Se houver sobreposição, use AskUserQuestion:
- header: "Duplicado?"
- question: "Todo semelhante já existe: [title]. O que você gostaria de fazer?"
- options:
  - "Pular" — manter o todo existente
  - "Substituir" — atualizar o existente com novo contexto
  - "Adicionar mesmo assim" — criar como todo separado
</step>

<step name="create_file">
Use os valores do contexto de inicialização: `timestamp` e `date` já estão disponíveis.

Gere o slug para o título:
```bash
slug=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" generate-slug "$title" --raw)
```

Escreva em `.planning/todos/pending/${date}-${slug}.md`:

```markdown
---
created: [timestamp]
title: [title]
area: [area]
files:
  - [file:lines]
---

## Problema

[descrição do problema — contexto suficiente para um Claude futuro entender semanas depois]

## Solução

[dicas de abordagem ou "TBD"]
```
</step>

<step name="update_state">
Se `.planning/STATE.md` existir:

1. Use `todo_count` do contexto de inicialização (ou re-execute `init todos` se a contagem mudou)
2. Atualize "### Pending Todos" em "## Accumulated Context"
</step>

<step name="git_commit">
Faça commit do todo e de qualquer estado atualizado:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: capture todo - [title]" --files .planning/todos/pending/[filename] .planning/STATE.md
```

A ferramenta respeita a config `commit_docs` e o gitignore automaticamente.

Confirme: "Committed: docs: capture todo - [title]"
</step>

<step name="confirm">
```
Todo salvo: .planning/todos/pending/[filename]

  [title]
  Área: [area]
  Arquivos: [count] referenciados

---

Você gostaria de:

1. Continuar com o trabalho atual
2. Adicionar outro todo
3. Ver todos os todos (/gsd-check-todos)
```
</step>

</process>

<success_criteria>
- [ ] Estrutura de diretórios existe
- [ ] Arquivo de todo criado com frontmatter válido
- [ ] Seção de problema tem contexto suficiente para um Claude futuro
- [ ] Sem duplicatas (verificado e resolvido)
- [ ] Área consistente com todos existentes
- [ ] STATE.md atualizado se existir
- [ ] Todo e estado commitados no git
</success_criteria>
