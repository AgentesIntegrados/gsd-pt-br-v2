<purpose>
Marcar um milestone como completo. Arquivar artefatos de planejamento, criar tag git e preparar para o próximo milestone. Fecha o loop plan → execute → verify → ship → complete.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar estado do milestone:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init milestone-op "${MILESTONE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON: `milestone_version`, `milestone_name`, `phases_complete`, `phases_total`, `all_phases_done`, `commit_docs`.

Carregar configuração:
```bash
CONFIG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
```

Extrair: `branching_strategy`, `branch_name`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CONCLUIR MILESTONE {version}: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="preflight_checks">
Verificar se o trabalho está pronto para conclusão:

1. **Todas as fases concluídas?**
   Se `all_phases_done` for false: exibir fases pendentes e avisar.
   Perguntar ao usuário se deseja continuar mesmo assim.

2. **PRs mergeados?**
   ```bash
   gh pr list --state open 2>/dev/null | head -5
   ```
   Se PRs abertos existirem: avisar que PRs estão pendentes.

3. **Árvore de trabalho limpa?**
   ```bash
   git status --short
   ```
   Se alterações não commitadas existirem: pedir ao usuário que commite ou dê stash primeiro.

4. **Verificações passadas?**
   Checar VERIFICATION.md mais recente por `status: passed` ou `status: human_needed`.
   Se nenhuma verificação encontrada: avisar.
</step>

<step name="generate_summary">
Gerar resumo do milestone a partir dos artefatos de planejamento:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" milestone summary "${MILESTONE_VERSION}"
```

Incluir:
- Total de fases e planos executados
- Requisitos concluídos
- Decisões-chave registradas
- Arquivos criados/modificados
- Commits contados

Exibir resumo ao usuário antes de continuar.
</step>

<step name="update_state">
Marcar o milestone como completo no STATE.md:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Status" "Milestone ${MILESTONE_VERSION} concluído"
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Last Activity" "$(date +%Y-%m-%d)"
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" milestone mark-complete "${MILESTONE_VERSION}"
```

Atualizar ROADMAP.md para marcar o milestone como concluído:
- Adicionar data de conclusão
- Atualizar indicador de status
</step>

<step name="create_git_tag">
Criar tag git para o milestone:

```bash
TAG_NAME="v${MILESTONE_VERSION}"
TAG_MESSAGE="Milestone ${MILESTONE_VERSION}: ${MILESTONE_NAME}

${SUMMARY_TEXT}"

git tag -a "${TAG_NAME}" -m "${TAG_MESSAGE}"
```

Se a tag já existir: avisar e perguntar se sobrescreve.

Reportar: "Tag `{tag}` criada"
</step>

<step name="archive_artifacts">
**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Perguntar se deve arquivar artefatos de planejamento:

```
AskUserQuestion:
  question: "Arquivar artefatos de planejamento deste milestone?"
  options:
    - label: "Sim, arquivar"
      description: "Mover .planning/ para .planning/archive/v{version}/"
    - label: "Não, manter"
      description: "Manter .planning/ no lugar para referência"
```

Se "Sim":
```bash
mkdir -p .planning/archive
cp -r .planning .planning/archive/v${MILESTONE_VERSION}
```

Reportar: "Artefatos arquivados em `.planning/archive/v{version}/`"
</step>

<step name="commit_completion">
Se `commit_docs` for true:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: milestone ${MILESTONE_VERSION} concluído" --files .planning/STATE.md .planning/ROADMAP.md
```
</step>

<step name="report">
```
───────────────────────────────────────────────────────────────

## ✓ Milestone {version}: {Nome} — Concluído

Fases: {N}/{N} concluídas
Planos: {N} executados
Requisitos: {N} entregues
Tag git: v{version}
Artefatos: {arquivado/mantido}

## ▶ Próximos passos

- `/gsd-new-milestone` — iniciar próximo milestone
- `/gsd-progress` — ver estado do projeto
- Publicar a tag: `git push origin v{version}`

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Verificações preliminares passadas (fases, PRs, árvore de trabalho, verificações)
- [ ] Resumo do milestone gerado a partir dos artefatos
- [ ] STATE.md e ROADMAP.md atualizados com status de conclusão
- [ ] Tag git criada com mensagem de resumo
- [ ] Artefatos arquivados (se usuário confirmou)
- [ ] Usuário sabe os próximos passos
</success_criteria>
