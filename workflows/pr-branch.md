<purpose>
Criar um branch de PR limpo filtrando os commits transitórios de .planning/ gerados pelo fluxo de trabalho GSD. Produz um branch de PR contendo apenas commits de código, tornando o histórico mais limpo para revisão.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar estado do projeto:

```bash
CONFIG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
BASE_BRANCH=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get git.base_branch 2>/dev/null || echo "")
if [ -z "$BASE_BRANCH" ] || [ "$BASE_BRANCH" = "null" ]; then
  BASE_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|^refs/remotes/origin/||')
  BASE_BRANCH="${BASE_BRANCH:-main}"
fi

CURRENT_BRANCH=$(git branch --show-current)
PHASE_ARG=$(echo "$ARGUMENTS" | grep -oE '[0-9]+' | head -1)
```

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CRIAR BRANCH DE PR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="verify_prerequisites">
Verificações pré-requisito:

1. **Árvore de trabalho limpa?**
   ```bash
   git status --short
   ```
   Se alterações não commitadas: pedir ao usuário que commite ou dê stash primeiro.

2. **No branch certo?**
   Se no `${BASE_BRANCH}`: avisar — deve estar em um branch de feature.

3. **Origin configurado?**
   ```bash
   git remote -v | head -2
   ```
   Se sem origin: erro — não é possível criar branch de PR.

4. **Commits GSD existem?**
   ```bash
   git log --oneline ${BASE_BRANCH}..HEAD | head -20
   ```
   Contar commits de docs GSD (padrão `docs(NN`).
</step>

<step name="identify_commits">
Categorizar commits entre o branch atual e o base:

```bash
git log --oneline ${BASE_BRANCH}..HEAD
```

**Commits de código** (manter no branch de PR):
- Commits sem prefixo `docs(`
- Commits `feat(`, `fix(`, `refactor(`, `test(`, `chore(`

**Commits de documentação GSD** (filtrar do branch de PR):
- Commits `docs(NN):` onde NN é um número de fase
- Commits de planejamento (STATE.md, ROADMAP.md, PLAN.md, SUMMARY.md, etc.)

Exibir categorização:
```
Commits para o branch de PR: {N} commits de código
Commits filtrados: {M} commits de docs GSD

Commits de código:
  {hash} {mensagem}
  ...

Commits GSD (filtrados):
  {hash} {mensagem}
  ...
```
</step>

<step name="create_pr_branch">
**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que Digite o número de sua escolha.

Perguntar via AskUserQuestion:
- header: "Branch de PR"
- question: "Criar branch de PR limpo?"
- options:
  - "Criar branch de PR" — "Criar gsd/pr-{branch-atual} com apenas commits de código"
  - "Ver diff primeiro" — "Mostrar diff do que vai para o branch de PR"
  - "Cancelar" — "Não fazer alterações"

Se "Cancelar": sair.

Se "Ver diff": exibir diff e perguntar novamente.

Se "Criar":

```bash
PR_BRANCH="gsd/pr-${CURRENT_BRANCH}"

# Criar novo branch a partir do base
git checkout -b "${PR_BRANCH}" "${BASE_BRANCH}"

# Cherry-pick apenas commits de código
for hash in {commits de código em ordem cronológica}; do
  git cherry-pick ${hash}
done
```

Se cherry-pick falhar: exibir erro e oferecer opções de resolução.
</step>

<step name="push_and_report">
Após criação do branch bem-sucedida:

```bash
git push --set-upstream origin "${PR_BRANCH}"
```

Exibir:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► BRANCH DE PR CRIADO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Branch: {PR_BRANCH}
Base:   {BASE_BRANCH}
Commits incluídos: {N} commits de código
Commits filtrados: {M} commits de docs GSD

───────────────────────────────────────────────────────────────

## ▶ Próximos passos

Criar PR:
  gh pr create --base {BASE_BRANCH} --head {PR_BRANCH}

Ou use /gsd-ship para criar PR com corpo rico gerado automaticamente.

───────────────────────────────────────────────────────────────
```

Retornar ao branch original:
```bash
git checkout "${CURRENT_BRANCH}"
```
</step>

</process>

<success_criteria>
- [ ] Pré-requisitos verificados (árvore limpa, branch, origin)
- [ ] Commits categorizados (código vs. docs GSD)
- [ ] Usuário confirmou criação do branch de PR
- [ ] Branch de PR criado com apenas commits de código
- [ ] Branch enviado para origin
- [ ] Instruções para criar PR exibidas
- [ ] Retornou ao branch original
</success_criteria>
