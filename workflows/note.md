<purpose>
Captura de ideias sem fricção. Uma chamada Write, uma linha de confirmação. Sem perguntas, sem prompts.

**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Executa inline — sem Task, sem AskUserQuestion, sem Bash.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="storage_format">
**Formato de armazenamento de notas.**

As notas são armazenadas como arquivos markdown individuais:

- **Escopo do projeto**: `.planning/notes/{YYYY-MM-DD}-{slug}.md` — usado quando `.planning/` existe no diretório atual
- **Escopo global**: `$HOME/.claude/notes/{YYYY-MM-DD}-{slug}.md` — fallback quando não há `.planning/`, ou quando a flag `--global` está presente

Cada arquivo de nota:

```markdown
---
date: "YYYY-MM-DD HH:mm"
promoted: false
---

{texto da nota verbatim}
```

**Flag `--global`**: Remova `--global` de qualquer lugar em `$ARGUMENTS` antes de analisar. Quando presente, force o escopo global independentemente de `.planning/` existir.

**Importante**: NÃO crie `.planning/` se não existir. Recorra ao escopo global silenciosamente.
</step>

<step name="parse_subcommand">
**Analise o subcomando de $ARGUMENTS (após remover --global).**

| Condição | Subcomando |
|----------|------------|
| Os argumentos são exatamente `list` (insensível a maiúsculas) | **list** |
| Os argumentos são exatamente `promote <N>` onde N é um número | **promote** |
| Os argumentos estão vazios (sem texto algum) | **list** |
| Qualquer outra coisa | **append** (o texto É a nota) |

**Crítico**: `list` é apenas um subcomando quando é o ARGUMENTO COMPLETO. `/gsd-note list of groceries` salva uma nota com o texto "list of groceries". O mesmo para `promote` — apenas um subcomando quando seguido de exatamente um número.
</step>

<step name="append">
**Subcomando: append — criar um arquivo de nota com timestamp.**

1. Determine o escopo (projeto ou global) conforme o formato de armazenamento acima
2. Garanta que o diretório de notas exista (`.planning/notes/` ou `$HOME/.claude/notes/`)
3. Gere slug: primeiras ~4 palavras significativas do texto da nota, minúsculas, separadas por hífen (remova artigos/preposições do início)
4. Gere o nome do arquivo: `{YYYY-MM-DD}-{slug}.md`
   - Se um arquivo com esse nome já existir, acrescente `-2`, `-3`, etc.
5. Escreva o arquivo com frontmatter e texto da nota (veja o formato de armazenamento)
6. Confirme com exatamente uma linha: `Anotado ({escopo}): {texto da nota}`
   - Onde `{escopo}` é "projeto" ou "global"

**Restrições:**
- **Nunca modifique o texto da nota** — capture verbatim, incluindo erros de digitação
- **Nunca faça perguntas** — apenas escreva e confirme
- **Formato de timestamp**: Use hora local, `YYYY-MM-DD HH:mm` (24 horas, sem segundos)
</step>

<step name="list">
**Subcomando: list — mostrar notas de ambos os escopos.**

1. Glob `.planning/notes/*.md` (se o diretório existir) — notas do projeto
2. Glob `$HOME/.claude/notes/*.md` (se o diretório existir) — notas globais
3. Para cada arquivo, leia o frontmatter para obter o status de `date` e `promoted`
4. Exclua arquivos onde `promoted: true` das contagens ativas (mas ainda mostre-os, esmaecidos)
5. Ordene por data, numere todas as entradas ativas sequencialmente começando em 1
6. Se o total de entradas ativas > 20, mostre apenas as últimas 10 com uma nota sobre quantas foram omitidas

**Formato de exibição:**

```
Notas:

Projeto (.planning/notes/):
  1. [2026-02-08 14:32] refatorar o sistema de hooks para suportar validadores assíncronos
  2. [promovido] [2026-02-08 14:40] adicionar rate limiting aos endpoints da API
  3. [2026-02-08 15:10] considerar adicionar uma flag --dry-run ao build

Global ($HOME/.claude/notes/):
  4. [2026-02-08 10:00] ideia entre projetos sobre configuração compartilhada

{count} nota(s) ativa(s). Use `/gsd-note promote <N>` para converter em tarefa.
```

Se um escopo não tiver diretório ou entradas, mostre: `(sem notas)`
</step>

<step name="promote">
**Subcomando: promote — converter uma nota em tarefa.**

1. Execute a lógica **list** para construir o índice numerado (ambos os escopos)
2. Encontre a entrada N da lista numerada
3. Se N for inválido ou se referir a uma nota já promovida, informe o usuário e pare
4. **Requer diretório `.planning/`** — se não existir, avise: "Tarefas requerem um projeto GSD. Execute `/gsd-new-project` para inicializar um."
5. Garanta que o diretório `.planning/todos/pending/` exista
6. Gere o ID da tarefa: `{NNN}-{slug}` onde NNN é o próximo número sequencial (varra `.planning/todos/pending/` e `.planning/todos/completed/` para o maior número existente, incremente em 1, preencha com zeros até 3 dígitos) e slug são as primeiras ~4 palavras significativas do texto da nota
7. Extraia o texto da nota do arquivo fonte (corpo após o frontmatter)
8. Crie `.planning/todos/pending/{id}.md`:

```yaml
---
title: "{texto da nota}"
status: pending
priority: P2
source: "promoted from /gsd-note"
created: {YYYY-MM-DD}
theme: general
---

## Objetivo

{texto da nota}

## Contexto

Promovido de nota rápida capturada em {data original}.

## Critérios de Aceitação

- [ ] {critério principal derivado do texto da nota}
```

9. Marque o arquivo de nota fonte como promovido: atualize seu frontmatter para `promoted: true`
10. Confirme: `Nota {N} promovida para tarefa {id}: {texto da nota}`
</step>

</process>

<edge_cases>
1. **"list" como texto de nota**: `/gsd-note list of things` salva a nota "list of things" (subcomando apenas quando `list` é o argumento completo)
2. **Sem `.planning/`**: Recorre ao `$HOME/.claude/notes/` global — funciona em qualquer diretório
3. **Promover sem projeto**: Avisa que tarefas requerem `.planning/`, sugere `/gsd-new-project`
4. **Arquivos grandes**: `list` mostra as últimas 10 quando há > 20 entradas ativas
5. **Slugs duplicados**: Acrescente `-2`, `-3` etc. ao nome do arquivo se o slug já for usado na mesma data
6. **Posição de `--global`**: Removido de qualquer lugar — `--global minha ideia` e `minha ideia --global` salvam "minha ideia" globalmente
7. **Promover já promovido**: Informe ao usuário "Nota {N} já foi promovida" e pare
8. **Texto de nota vazio após remover flags**: Trate como subcomando `list`
</edge_cases>

<success_criteria>
- [ ] Append: Arquivo de nota escrito com frontmatter correto e texto verbatim
- [ ] Append: Nenhuma pergunta feita — captura instantânea
- [ ] List: Ambos os escopos exibidos com numeração sequencial
- [ ] List: Notas promovidas exibidas mas esmaecidas
- [ ] Promote: Tarefa criada com formato correto
- [ ] Promote: Nota fonte marcada como promovida
- [ ] Fallback global: Funciona quando não há `.planning/`
</success_criteria>
</output>
