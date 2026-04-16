<purpose>
Criar um pull request a partir do trabalho concluído de fase/milestone, gerar um corpo rico de PR a partir dos artefatos de planejamento, opcionalmente executar revisão de código e preparar para merge. Fecha o loop plan → execute → verify → ship.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Analisar argumentos e carregar estado do projeto:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analisar do JSON de init: `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `commit_docs`.

Também carregar config para estratégia de branching:
```bash
CONFIG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
```

Extrair: `branching_strategy`, `branch_name`.

Detectar branch base para PRs e merges:
```bash
BASE_BRANCH=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get git.base_branch 2>/dev/null || echo "")
if [ -z "$BASE_BRANCH" ] || [ "$BASE_BRANCH" = "null" ]; then
  BASE_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|^refs/remotes/origin/||')
  BASE_BRANCH="${BASE_BRANCH:-main}"
fi
```
</step>

<step name="preflight_checks">
Verificar se o trabalho está pronto para publicar:

1. **Verificação passou?**
   ```bash
   VERIFICATION=$(cat ${PHASE_DIR}/*-VERIFICATION.md 2>/dev/null)
   ```
   Verificar `status: passed` ou `status: human_needed` (com aprovação humana).
   Se nenhum VERIFICATION.md ou status for `gaps_found`: avisar e pedir confirmação do usuário.

2. **Árvore de trabalho limpa?**
   ```bash
   git status --short
   ```
   Se alterações não commitadas existirem: pedir ao usuário que commite ou dê stash primeiro.

3. **No branch correto?**
   ```bash
   CURRENT_BRANCH=$(git branch --show-current)
   ```
   Se no `${BASE_BRANCH}`: avisar — deve estar em um branch de feature.
   Se branching_strategy for `none`: oferecer criar um branch agora.

4. **Remote configurado?**
   ```bash
   git remote -v | head -2
   ```
   Detectar remote `origin`. Se sem remote: erro — não é possível criar PR.

5. **gh CLI disponível?**
   ```bash
   which gh && gh auth status 2>&1
   ```
   Se `gh` não encontrado ou não autenticado: fornecer instruções de configuração e sair.
</step>

<step name="push_branch">
Enviar o branch atual para o remote:

```bash
git push origin ${CURRENT_BRANCH} 2>&1
```

Se push falhar (ex.: sem upstream): definir upstream:
```bash
git push --set-upstream origin ${CURRENT_BRANCH} 2>&1
```

Reportar: "Branch `{branch}` enviado para origin ({commit_count} commits à frente de ${BASE_BRANCH})"
</step>

<step name="generate_pr_body">
Gerar automaticamente um corpo rico de PR a partir dos artefatos de planejamento:

**1. Título:**
```
Fase {phase_number}: {phase_name}
```
Ou para milestone: `Milestone {version}: {name}`

**2. Seção de Resumo:**
Ler ROADMAP.md para meta da fase. Ler VERIFICATION.md para status de verificação.

```markdown
## Resumo

**Fase {N}: {Nome}**
**Meta:** {meta do ROADMAP.md}
**Status:** Verificado ✓

{Um parágrafo sintetizado dos arquivos SUMMARY.md — o que foi construído}
```

**3. Seção de Alterações:**
Para cada SUMMARY.md no diretório da fase:
```markdown
## Alterações

### Plano {plan_id}: {plan_name}
{one_liner do frontmatter do SUMMARY.md}

**Arquivos-chave:**
{key-files.created e key-files.modified do frontmatter do SUMMARY.md}
```

**4. Seção de Requisitos:**
```markdown
## Requisitos Atendidos

{REQ-IDs do frontmatter do plano, vinculados às descrições do REQUIREMENTS.md}
```

**5. Seção de Testes:**
```markdown
## Verificação

- [x] Verificação automatizada: {aprovado/reprovado do VERIFICATION.md}
- {itens de verificação humana do VERIFICATION.md, se houver}
```

**6. Seção de Decisões:**
```markdown
## Decisões-Chave

{Decisões do contexto acumulado do STATE.md relevantes para esta fase}
```
</step>

<step name="create_pr">
Criar o PR usando o corpo gerado:

```bash
gh pr create \
  --title "Fase ${PHASE_NUMBER}: ${PHASE_NAME}" \
  --body "${PR_BODY}" \
  --base ${BASE_BRANCH}
```

Se a flag `--draft` foi passada: adicionar `--draft`.

Reportar: "PR #{número} criado: {url}"
</step>

<step name="optional_review">

**Comando de revisão de código externo (sub-etapa automatizada):**

Antes de solicitar ao usuário, verificar se um comando de revisão externo está configurado:

```bash
REVIEW_CMD=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.code_review_command --default "" 2>/dev/null)
```

Se `REVIEW_CMD` for não vazio e diferente de `"null"`, executar a revisão externa:

1. **Gerar diff e estatísticas:**
   ```bash
   DIFF=$(git diff ${BASE_BRANCH}...HEAD)
   DIFF_STATS=$(git diff --stat ${BASE_BRANCH}...HEAD)
   ```

2. **Carregar contexto da fase do STATE.md:**
   ```bash
   STATE_STATUS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load 2>/dev/null | head -20)
   ```

3. **Construir prompt de revisão e enviar via stdin:**
   Construir um prompt de revisão contendo o diff, estatísticas do diff e contexto da fase, então enviar para o comando configurado:
   ```bash
   REVIEW_PROMPT="Você está revisando um pull request.\n\nEstatísticas do diff:\n${DIFF_STATS}\n\nContexto da fase:\n${STATE_STATUS}\n\nDiff completo:\n${DIFF}\n\nResponda com JSON: { \"verdict\": \"APPROVED\" ou \"REVISE\", \"confidence\": 0-100, \"summary\": \"...\", \"issues\": [{\"severity\": \"...\", \"file\": \"...\", \"line_range\": \"...\", \"description\": \"...\", \"suggestion\": \"...\"}] }"
   REVIEW_OUTPUT=$(echo "${REVIEW_PROMPT}" | timeout 120 ${REVIEW_CMD} 2>/tmp/gsd-review-stderr.log)
   REVIEW_EXIT=$?
   ```

4. **Tratar timeout (120s) e falha.**

5. **Analisar resultado JSON:**
   - Se `verdict` for `"APPROVED"`: reportar aprovação com confiança e resumo.
   - Se `verdict` for `"REVISE"`: reportar problemas encontrados, listar cada um com severidade, arquivo, intervalo de linhas, descrição e sugestão.

---

**Opções de revisão manual:**

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

```
AskUserQuestion:
  question: "PR criado. Executar uma revisão de código antes do merge?"
  options:
    - label: "Pular revisão"
      description: "PR está pronto — mergear quando CI passar"
    - label: "Auto-revisão"
      description: "Vou revisar o diff no PR eu mesmo"
    - label: "Solicitar revisão"
      description: "Solicitar revisão de um colega"
```

**Se "Solicitar revisão":**
```bash
gh pr edit ${PR_NUMBER} --add-reviewer "${REVIEWER}"
```

**Se "Auto-revisão":**
Reportar a URL do PR e sugerir: "Revise o diff em {url}/files"
</step>

<step name="track_shipping">
Atualizar STATE.md para refletir a ação de publicação:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Last Activity" "$(date +%Y-%m-%d)"
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Status" "Fase ${PHASE_NUMBER} publicada — PR #${PR_NUMBER}"
```

Se `commit_docs` for true:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): publicar fase ${PHASE_NUMBER} — PR #${PR_NUMBER}" --files .planning/STATE.md
```
</step>

<step name="report">
```
───────────────────────────────────────────────────────────────

## ✓ Fase {X}: {Nome} — Publicada

PR: #{número} ({url})
Branch: {branch} → ${BASE_BRANCH}
Commits: {contagem}
Verificação: ✓ Passou
Requisitos: {N} REQ-IDs atendidos

Próximos passos:
- Revisar/aprovar PR
- Mergear quando CI passar
- /gsd-complete-milestone (se última fase do milestone)
- /gsd-progress (para ver o que vem a seguir)

───────────────────────────────────────────────────────────────
```
</step>

</process>

<offer_next>
Após publicar:

- /gsd-complete-milestone — se todas as fases do milestone estiverem concluídas
- /gsd-progress — ver estado geral do projeto
- /gsd-execute-phase {próxima} — continuar para a próxima fase
</offer_next>

<success_criteria>
- [ ] Verificações preliminares passadas (verificação, árvore limpa, branch, remote, gh)
- [ ] Branch enviado para remote
- [ ] PR criado com corpo rico gerado automaticamente
- [ ] STATE.md atualizado com status de publicação
- [ ] Usuário sabe número do PR e próximos passos
</success_criteria>
