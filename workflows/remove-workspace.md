<purpose>
Remover um workspace GSD e limpar worktrees git associados. Verificações de segurança previnem remoção acidental de workspaces com trabalho concluído não salvo.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Analisar $ARGUMENTS para nome do workspace alvo.

Se nenhum argumento fornecido: exibir uso:
```
Uso: /gsd-remove-workspace "{nome}"

Exemplo:
  /gsd-remove-workspace "backend-api"
  
Remove um workspace GSD e seus worktrees git associados.
```
Sair.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REMOVER WORKSPACE: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="locate_workspace">
Localizar o workspace:

```bash
WORKSPACES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workspaces list --json 2>/dev/null)
```

Encontrar o workspace com o nome especificado.

Se não encontrado: exibir "Workspace '{nome}' não encontrado." com lista de workspaces disponíveis e sair.

Extrair: `workspace_path`, `workspace_state`, `git_worktree`, `phases_complete`, `phases_total`.
</step>

<step name="safety_checks">
Executar verificações de segurança:

**Verificação 1: Trabalho não mergeado?**
```bash
if [ -n "${git_worktree}" ]; then
  git -C "${workspace_path}" log origin/main..HEAD --oneline 2>/dev/null | wc -l
fi
```

Se commits não mergeados encontrados:
```
⚠️ Aviso: Este workspace tem {N} commits não mergeados com o branch principal.
   Certifique-se de que o trabalho foi mergeado ou publicado antes de remover.
```

**Verificação 2: Trabalho em andamento?**
```bash
STATUS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workspaces status "${WORKSPACE_NAME}" --raw)
```

Se status for "em_andamento" ou "executando":
```
⚠️ Aviso: Este workspace parece estar com trabalho ativo em andamento.
   Fase atual: {current_phase} — {current_phase_name}
   Status: {status}
```

**Verificação 3: Worktree tem alterações não commitadas?**
```bash
if [ -n "${git_worktree}" ]; then
  git -C "${workspace_path}" status --short 2>/dev/null
fi
```

Se alterações não commitadas: exibir aviso.

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Mostrar resumo e perguntar via AskUserQuestion:
- header: "Confirmar Remoção"
- question: "Remover workspace '{nome}' e seus worktrees?"
- options:
  - "Sim, remover tudo" — "Remover workspace e worktrees git"
  - "Manter worktrees, remover apenas registro" — "Limpar registro GSD mas manter arquivos"
  - "Cancelar" — "Não fazer alterações"

Se Cancelar: sair.
</step>

<step name="remove_workspace">
**Se "Remover tudo":**

Remover worktree git se existir:
```bash
if [ -n "${git_worktree}" ]; then
  git worktree remove "${workspace_path}" --force 2>/dev/null || true
fi
```

Remover registro do workspace do GSD:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workspaces remove "${WORKSPACE_NAME}"
```

**Se "Manter worktrees, remover apenas registro":**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workspaces remove "${WORKSPACE_NAME}" --keep-files
```
</step>

<step name="update_state">
Atualizar state se necessário:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Last Activity" "$(date +%Y-%m-%d)"
```
</step>

<step name="report">
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► WORKSPACE REMOVIDO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Workspace removido: {nome}
{Se worktree removido: "Worktree git removido: {caminho}"}
{Se apenas registro: "Registro removido — arquivos preservados em: {caminho}"}

───────────────────────────────────────────────────────────────

## ▶ Próximos passos

- `/gsd-list-workspaces` — ver workspaces restantes
- `/gsd-new-workspace "{nome}"` — criar novo workspace
- `/gsd-progress` — ver estado do projeto

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Workspace alvo localizado
- [ ] Verificações de segurança executadas (commits não mergeados, trabalho em andamento, alterações não commitadas)
- [ ] Confirmação do usuário obtida
- [ ] Worktree git removido (se selecionado)
- [ ] Registro do workspace removido
- [ ] Usuário informado sobre o que foi removido
</success_criteria>
