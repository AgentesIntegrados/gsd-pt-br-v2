<purpose>
Revisar arquivos de código-fonte para bugs, problemas de segurança e violações de padrões. Gera REVIEW.md com descobertas priorizadas para correção. Funciona em projetos gerenciados pelo GSD ou qualquer base de código.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar contexto do projeto e resolver o modelo do agente:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_REVIEWER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-code-reviewer 2>/dev/null)
REVIEWER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-code-reviewer --raw)
```

Extrair do JSON: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `commit_docs`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REVISÃO DE CÓDIGO — FASE {N}: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="detect_scope">
Determinar os arquivos a revisar:

**Se fase fornecida:** Usar arquivos listados nos SUMMARY.md da fase.
```bash
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
```
Extrair `key-files.created` e `key-files.modified` do frontmatter de cada SUMMARY.md.

**Se nenhuma fase fornecida:** Usar `git diff` para obter arquivos modificados recentemente.
```bash
CHANGED_FILES=$(git diff --name-only HEAD~1 HEAD 2>/dev/null || git diff --name-only --cached)
```

**Se --files fornecido em $ARGUMENTS:** Usar essa lista de arquivos explicitamente.

Filtrar para tipos de código-fonte: `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`, `.java`, `.rb`, `.php`, `.cs`, `.cpp`, `.c`, `.h`.

Se nenhum arquivo encontrado: exibir "Nenhum arquivo de código-fonte encontrado para revisar." e sair.
</step>

<step name="spawn_reviewer">
Exibir:
```
◆ Iniciando revisor de código...
  Arquivos: {N} arquivos para revisar
  Modelo: {model}
```

Construir prompt:

```markdown
Leia $HOME/.claude/agents/gsd-code-reviewer.md para instruções.

<objetivo>
Revisar arquivos de código-fonte quanto a bugs, problemas de segurança e violações de padrões.
Fase {phase_number}: {phase_name}
</objetivo>

<arquivos_para_revisar>
{lista de caminhos de arquivos, um por linha}
</arquivos_para_revisar>

<contexto_do_projeto>
{se fase GSD: conteúdo dos SUMMARY.md}
{se disponível: conteúdo do STATE.md}
</contexto_do_projeto>

${AGENT_SKILLS_REVIEWER}

<saída>
Escrever em: {phase_dir}/{padded_phase}-REVIEW.md
Prioridade: CRÍTICO → ALTO → MÉDIO → BAIXO → INFO
</saída>

<config>
commit_docs: {commit_docs}
phase_dir: {phase_dir}
padded_phase: {padded_phase}
</config>
```

```
Task(
  prompt=reviewer_prompt,
  subagent_type="gsd-code-reviewer",
  model="{REVIEWER_MODEL}",
  description="Revisão de código Fase {N}"
)
```
</step>

<step name="handle_return">
**Se `## REVISÃO CONCLUÍDA`:**

Exibir resumo da revisão:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REVISÃO DE CÓDIGO CONCLUÍDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {N}: {Nome}** — {N} descobertas

| Prioridade | Contagem |
|-----------|---------|
| Crítico   | {N}     |
| Alto      | {N}     |
| Médio     | {N}     |
| Baixo     | {N}     |
| Info      | {N}     |

Arquivo de revisão: {caminho para REVIEW.md}
```

Exibir as 3 principais descobertas críticas/altas, se existirem.

**Se `## REVISÃO BLOQUEADA`:**

Exibir detalhes do bloqueio e orientação. Sair do fluxo de trabalho.
</step>

<step name="offer_autofix">
**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Se houver descobertas de prioridade Crítica ou Alta:

Perguntar via AskUserQuestion:
- header: "Correção Automática"
- question: "Descobertas de alta prioridade encontradas. Corrigir automaticamente?"
- options:
  - "Corrigir tudo" — "Corrigir todas as descobertas críticas e altas automaticamente"
  - "Revisar primeiro" — "Ver REVIEW.md antes de decidir"
  - "Pular correção" — "Manter apenas o REVIEW.md"

Se "Corrigir tudo": Invocar `/gsd-code-review-fix` passando o caminho do REVIEW.md.
Se "Revisar primeiro": Exibir URL do REVIEW.md e aguardar.
Se "Pular correção": Continuar para o relatório final.
</step>

<step name="commit_review">
Se `commit_docs` for true:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): revisão de código" --files "${PHASE_DIR}/${PADDED_PHASE}-REVIEW.md"
```
</step>

<step name="report">
```
───────────────────────────────────────────────────────────────

## Próximos passos

- `/gsd-code-review-fix {fase}` — corrigir descobertas automaticamente
- Revisar o REVIEW.md completo em: {caminho}
- `/gsd-verify-phase {fase}` — verificar após aplicar correções

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Escopo de revisão determinado (fase, diff ou arquivos explícitos)
- [ ] Arquivos de código-fonte identificados
- [ ] gsd-code-reviewer iniciado com contexto correto
- [ ] REVIEW.md criado com descobertas priorizadas
- [ ] Resumo exibido ao usuário
- [ ] Opção de correção automática oferecida para problemas críticos/altos
- [ ] REVIEW.md commitado se commit_docs estiver habilitado
</success_criteria>
