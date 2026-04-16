# Git Planning Commit

Faça commit de artefatos de planejamento usando a CLI do gsd-tools, que verifica automaticamente a configuração `commit_docs` e o status do gitignore.

## Commit via CLI

Sempre use `gsd-tools.cjs commit` para arquivos `.planning/` — ele lida com as verificações de `commit_docs` e gitignore automaticamente:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({scope}): {description}" --files .planning/STATE.md .planning/ROADMAP.md
```

A CLI retornará `skipped` (com motivo) se `commit_docs` for `false` ou se `.planning/` estiver no gitignore. Nenhuma verificação condicional manual é necessária.

## Emendar o commit anterior

Para incorporar alterações de arquivos `.planning/` no commit anterior:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "" --files .planning/codebase/*.md --amend
```

## Padrões de Mensagem de Commit

| Comando | Escopo | Exemplo |
|---------|--------|---------|
| plan-phase | phase | `docs(phase-03): create authentication plans` |
| execute-phase | phase | `docs(phase-03): complete authentication phase` |
| new-milestone | milestone | `docs: start milestone v1.1` |
| remove-phase | chore | `chore: remove phase 17 (dashboard)` |
| insert-phase | phase | `docs: insert phase 16.1 (critical fix)` |
| add-phase | phase | `docs: add phase 07 (settings page)` |

## Quando Pular

- `commit_docs: false` na configuração
- `.planning/` está no gitignore
- Nenhuma alteração para commitar (verifique com `git status --porcelain .planning/`)
