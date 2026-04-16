<purpose>
Criar um diretório de workspace isolado com cópias de repositórios git (worktrees ou clones) e um diretório `.planning/` independente. Suporta orquestração de múltiplos repositórios e isolamento de branches de funcionalidades em repositório único.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 1. Configuração

**PRIMEIRO PASSO OBRIGATÓRIO — Execute o comando de inicialização:**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init new-workspace)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise o JSON para: `default_workspace_base`, `child_repos`, `child_repo_count`, `worktree_available`, `is_git_repo`, `cwd_repo_name`, `project_root`.

## 2. Analisar Argumentos

Extrair de $ARGUMENTS:
- `--name` → `WORKSPACE_NAME` (obrigatório)
- `--repos` → `REPO_LIST` (caminhos ou nomes separados por vírgula)
- `--path` → `TARGET_PATH` (padrão: `$default_workspace_base/$WORKSPACE_NAME`)
- `--strategy` → `STRATEGY` (padrão: `worktree`)
- `--branch` → `BRANCH_NAME` (padrão: `workspace/$WORKSPACE_NAME`)
- `--auto` → pular perguntas interativas

**Se `--name` estiver ausente e não for `--auto`:**


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Use AskUserQuestion:
- header: "Nome do Workspace"
- question: "Como este workspace deve se chamar?"
- requireAnswer: true

## 3. Selecionar Repositórios

**Se `--repos` for fornecido:** Analise os valores separados por vírgula. Para cada valor:
- Se for um caminho absoluto, use-o diretamente
- Se for um caminho relativo ou nome, resolva em relação a `$project_root`
- Caso especial: `.` significa o repositório atual (use `$project_root`, nomeie como `$cwd_repo_name`)

**Se `--repos` NÃO for fornecido e não for `--auto`:**

**Se `child_repo_count` > 0:**

Apresente os repositórios filhos para seleção:

Use AskUserQuestion:
- header: "Selecionar Repositórios"
- question: "Quais repositórios devem ser incluídos no workspace?"
- options: Listar cada repositório filho do array `child_repos` por nome
- multiSelect: true

**Se `child_repo_count` for 0 e `is_git_repo` for true:**

Use AskUserQuestion:
- header: "Repositório Atual"
- question: "Nenhum repositório filho encontrado. Criar um workspace com o repositório atual?"
- options:
  - "Sim — criar workspace com o repositório atual" → usar repositório atual
  - "Cancelar" → sair

**Se `child_repo_count` for 0 e `is_git_repo` for false:**

Erro:
```
Nenhum repositório git encontrado no diretório atual e este não é um repositório git.

Execute este comando a partir de um diretório contendo repositórios git, ou especifique repositórios explicitamente:
  /gsd-new-workspace --name meu-workspace --repos /caminho/para/repo1,/caminho/para/repo2
```
Sair.

**Se `--auto` e `--repos` NÃO for fornecido:**

Erro:
```
Erro: --auto requer --repos para especificar quais repositórios incluir.

Uso:
  /gsd-new-workspace --name meu-workspace --repos repo1,repo2 --auto
```
Sair.

## 4. Selecionar Estratégia

**Se `--strategy` for fornecido:** Use-o (validar: deve ser `worktree` ou `clone`).

**Se `--strategy` NÃO for fornecido e não for `--auto`:**

Use AskUserQuestion:
- header: "Estratégia"
- question: "Como os repositórios devem ser copiados para o workspace?"
- options:
  - "Worktree (recomendado) — leve, compartilha objetos .git com o repositório fonte" → `worktree`
  - "Clone — cópia totalmente independente, sem conexão com o repositório fonte" → `clone`

**Se `--auto`:** Padrão para `worktree`.

## 5. Validar

Antes de criar qualquer coisa, valide:

1. **Caminho de destino** — não deve existir ou deve estar vazio:
```bash
if [ -d "$TARGET_PATH" ] && [ "$(ls -A "$TARGET_PATH" 2>/dev/null)" ]; then
  echo "Erro: O caminho de destino já existe e não está vazio: $TARGET_PATH"
  echo "Escolha um --name ou --path diferente."
  exit 1
fi
```

2. **Os repositórios fonte existem e são repositórios git** — para cada caminho de repositório:
```bash
if [ ! -d "$REPO_PATH/.git" ]; then
  echo "Erro: Não é um repositório git: $REPO_PATH"
  exit 1
fi
```

3. **Disponibilidade do Worktree** — se a estratégia for `worktree` e `worktree_available` for false:
```
Erro: git não está disponível. Instale git ou use --strategy clone.
```

Reporte todos os erros de validação de uma vez, não um por vez.

## 6. Criar Workspace

```bash
mkdir -p "$TARGET_PATH"
```

### Para cada repositório:

**Estratégia Worktree:**
```bash
cd "$SOURCE_REPO_PATH"
git worktree add "$TARGET_PATH/$REPO_NAME" -b "$BRANCH_NAME" 2>&1
```

Se `git worktree add` falhar porque o branch já existe, tente com um branch com timestamp:
```bash
TIMESTAMP=$(date +%Y%m%d%H%M%S)
git worktree add "$TARGET_PATH/$REPO_NAME" -b "${BRANCH_NAME}-${TIMESTAMP}" 2>&1
```

Se isso também falhar, reporte o erro e continue com os repositórios restantes.

**Estratégia Clone:**
```bash
git clone "$SOURCE_REPO_PATH" "$TARGET_PATH/$REPO_NAME" 2>&1
cd "$TARGET_PATH/$REPO_NAME"
git checkout -b "$BRANCH_NAME" 2>&1
```

Registre os resultados: quais repositórios tiveram sucesso, quais falharam, qual branch foi usado.

## 7. Escrever WORKSPACE.md

Escreva o manifesto do workspace em `$TARGET_PATH/WORKSPACE.md`:

```markdown
# Workspace: $WORKSPACE_NAME

Criado: $DATE
Estratégia: $STRATEGY

## Repositórios Membros

| Repositório | Fonte | Branch | Estratégia |
|-------------|-------|--------|------------|
| $REPO_NAME | $SOURCE_PATH | $BRANCH | $STRATEGY |
...para cada repositório...

## Notas

[Adicione contexto sobre o propósito deste workspace]
```

## 8. Inicializar .planning/

```bash
mkdir -p "$TARGET_PATH/.planning"
```

## 9. Relatório e Próximos Passos

**Se todos os repositórios tiveram sucesso:**

```
Workspace criado: $TARGET_PATH

  Repositórios: $REPO_COUNT
  Estratégia: $STRATEGY
  Branch: $BRANCH_NAME

Próximos passos:
  cd "$TARGET_PATH"
  /gsd-new-project    # Inicializar GSD no workspace
```

**Se alguns repositórios falharam:**

```
Workspace criado com $SUCCESS_COUNT de $TOTAL_COUNT repositórios: $TARGET_PATH

  Com sucesso: repo1, repo2
  Com falha: repo3 (branch já existe), repo4 (não é um repositório git)

Próximos passos:
  cd "$TARGET_PATH"
  /gsd-new-project    # Inicializar GSD no workspace
```

**Ofereça inicializar o GSD (se não for `--auto`):**

Use AskUserQuestion:
- header: "Inicializar GSD"
- question: "Deseja inicializar um projeto GSD no novo workspace?"
- options:
  - "Sim — executar /gsd-new-project" → dizer ao usuário para `cd "$TARGET_PATH"` primeiro, depois executar `/gsd-new-project`
  - "Não — vou configurar depois" → concluído

</process>

<success_criteria>
- [ ] Diretório do workspace criado no caminho de destino
- [ ] Todos os repositórios especificados copiados (worktree ou clone) para o workspace
- [ ] Manifesto WORKSPACE.md escrito com tabela de repositórios correta
- [ ] Diretório `.planning/` inicializado na raiz do workspace
- [ ] Usuário informado sobre o caminho do workspace e próximos passos
</success_criteria>
</output>
