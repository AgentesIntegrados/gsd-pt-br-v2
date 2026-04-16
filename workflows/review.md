<purpose>
Revisão por pares entre IAs — invoca CLIs de IA externas para revisar independentemente os planos de fase.
Cada CLI recebe o mesmo prompt (contexto do PROJECT.md, planos de fase, requisitos) e
produz feedback estruturado. Os resultados são combinados em REVIEWS.md para o planejador
incorporar via flag --reviews.

Isso implementa revisão adversarial: diferentes modelos de IA identificam diferentes pontos cegos.
Um plano que sobrevive à revisão de 2-3 sistemas de IA independentes é mais robusto.
</purpose>

<process>

<step name="detect_clis">
Verificar quais CLIs de IA estão disponíveis no sistema:

```bash
# Verificar cada CLI
command -v gemini >/dev/null 2>&1 && echo "gemini:available" || echo "gemini:missing"
command -v claude >/dev/null 2>&1 && echo "claude:available" || echo "claude:missing"
command -v codex >/dev/null 2>&1 && echo "codex:available" || echo "codex:missing"
command -v coderabbit >/dev/null 2>&1 && echo "coderabbit:available" || echo "coderabbit:missing"
command -v opencode >/dev/null 2>&1 && echo "opencode:available" || echo "opencode:missing"
command -v qwen >/dev/null 2>&1 && echo "qwen:available" || echo "qwen:missing"
command -v cursor >/dev/null 2>&1 && echo "cursor:available" || echo "cursor:missing"
```

Analisar flags de `$ARGUMENTS`:
- `--gemini` → incluir Gemini
- `--claude` → incluir Claude
- `--codex` → incluir Codex
- `--coderabbit` → incluir CodeRabbit
- `--opencode` → incluir OpenCode
- `--qwen` → incluir Qwen Code
- `--cursor` → incluir Cursor
- `--all` → incluir todos os disponíveis
- Sem flags → incluir todos os disponíveis

Se nenhuma CLI estiver disponível:
```
Nenhuma CLI de IA externa encontrada. Instale ao menos uma:
- gemini: https://github.com/google-gemini/gemini-cli
- codex: https://github.com/openai/codex
- claude: https://github.com/anthropics/claude-code
- opencode: https://opencode.ai (usa modelos da assinatura GitHub Copilot)
- qwen: https://github.com/nicepkg/qwen-code (modelos Alibaba Qwen)
- cursor: https://cursor.com (modo agente do Cursor IDE)

Depois execute /gsd-review novamente.
```
Sair.

Determinar qual CLI pular com base no ambiente de execução atual:

```bash
# Detecção de runtime baseada no ambiente (ordem de prioridade)
if [ "$ANTIGRAVITY_AGENT" = "1" ]; then
  # Antigravity é um cliente separado — todas as CLIs são externas, não pular nenhuma
  SELF_CLI="none"
elif [ -n "$CURSOR_SESSION_ID" ]; then
  # Executando dentro do agente Cursor — pular cursor para independência
  SELF_CLI="cursor"
elif [ -n "$CLAUDE_CODE_ENTRYPOINT" ]; then
  # Executando dentro do Claude Code CLI — pular claude para independência
  SELF_CLI="claude"
else
  # Outros ambientes (Gemini CLI, Codex CLI, etc.)
  # Usar auto-identificação da IA para decidir qual CLI pular
  SELF_CLI="auto"
fi
```

Regras:
- Se `SELF_CLI="none"` → invocar TODAS as CLIs disponíveis (sem pular)
- Se `SELF_CLI="claude"` → pular claude, usar gemini/codex
- Se `SELF_CLI="auto"` → a IA em execução se identifica e pula sua própria CLI
- Pelo menos uma CLI DIFERENTE deve estar disponível para a revisão prosseguir.
</step>

<step name="gather_context">
Coletar artefatos da fase para o prompt de revisão:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Ler do init: `phase_dir`, `phase_number`, `padded_phase`.

Depois ler:
1. `.planning/PROJECT.md` (primeiras 80 linhas — contexto do projeto)
2. Seção da fase em `.planning/ROADMAP.md`
3. Todos os arquivos `*-PLAN.md` no diretório da fase
4. `*-CONTEXT.md` se presente (decisões do usuário)
5. `*-RESEARCH.md` se presente (pesquisa do domínio)
6. `.planning/REQUIREMENTS.md` (requisitos que esta fase aborda)
</step>

<step name="build_prompt">
Construir um prompt de revisão estruturado:

```markdown
# Solicitação de Revisão Cross-AI de Plano

You are reviewing implementation plans for a software project phase.
Provide structured feedback on plan quality, completeness, and risks.

## Project Context
{first 80 lines of PROJECT.md}

## Phase {N}: {phase name}
### Roadmap Section
{roadmap phase section}

### Requirements Addressed
{requirements for this phase}

### User Decisions (CONTEXT.md)
{context if present}

### Research Findings
{research if present}

### Plans to Review
{all PLAN.md contents}

## Review Instructions

Analyze each plan and provide:

1. **Summary** — One-paragraph assessment
2. **Strengths** — What's well-designed (bullet points)
3. **Concerns** — Potential issues, gaps, risks (bullet points with severity: HIGH/MEDIUM/LOW)
4. **Suggestions** — Specific improvements (bullet points)
5. **Risk Assessment** — Overall risk level (LOW/MEDIUM/HIGH) with justification

Focus on:
- Missing edge cases or error handling
- Dependency ordering issues
- Scope creep or over-engineering
- Security considerations
- Performance implications
- Whether the plans actually achieve the phase goals

Output your review in markdown format.
```

Escrever em arquivo temporário: `/tmp/gsd-review-prompt-{phase}.md`
</step>

<step name="invoke_reviewers">
Ler preferências de modelo da configuração de planejamento. Valores nulos/ausentes voltam para os padrões da CLI.

```bash
GEMINI_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get review.models.gemini --raw 2>/dev/null || true)
CLAUDE_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get review.models.claude --raw 2>/dev/null || true)
CODEX_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get review.models.codex --raw 2>/dev/null || true)
OPENCODE_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get review.models.opencode --raw 2>/dev/null || true)
```

Para cada CLI selecionada, invocar em sequência (não em paralelo — evitar limites de taxa):

**Gemini:**
```bash
if [ -n "$GEMINI_MODEL" ] && [ "$GEMINI_MODEL" != "null" ]; then
  gemini -m "$GEMINI_MODEL" -p "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-gemini-{phase}.md
else
  gemini -p "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-gemini-{phase}.md
fi
```

**Claude (sessão separada):**
```bash
if [ -n "$CLAUDE_MODEL" ] && [ "$CLAUDE_MODEL" != "null" ]; then
  claude --model "$CLAUDE_MODEL" -p "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-claude-{phase}.md
else
  claude -p "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-claude-{phase}.md
fi
```

**Codex:**
```bash
if [ -n "$CODEX_MODEL" ] && [ "$CODEX_MODEL" != "null" ]; then
  codex exec --model "$CODEX_MODEL" --skip-git-repo-check "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-codex-{phase}.md
else
  codex exec --skip-git-repo-check "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-codex-{phase}.md
fi
```

**CodeRabbit:**

Nota: O CodeRabbit revisa o diff git atual/árvore de trabalho — não aceita prompt ou flag de modelo. Pode levar até 5 minutos. Use `timeout: 360000` na chamada da ferramenta Bash.

```bash
coderabbit review --prompt-only 2>/dev/null > /tmp/gsd-review-coderabbit-{phase}.md
```

**OpenCode (via GitHub Copilot):**
```bash
if [ -n "$OPENCODE_MODEL" ] && [ "$OPENCODE_MODEL" != "null" ]; then
  cat /tmp/gsd-review-prompt-{phase}.md | opencode run --model "$OPENCODE_MODEL" - 2>/dev/null > /tmp/gsd-review-opencode-{phase}.md
else
  cat /tmp/gsd-review-prompt-{phase}.md | opencode run - 2>/dev/null > /tmp/gsd-review-opencode-{phase}.md
fi
if [ ! -s /tmp/gsd-review-opencode-{phase}.md ]; then
  echo "OpenCode review failed or returned empty output." > /tmp/gsd-review-opencode-{phase}.md
fi
```

**Qwen Code:**
```bash
qwen "$(cat /tmp/gsd-review-prompt-{phase}.md)" 2>/dev/null > /tmp/gsd-review-qwen-{phase}.md
if [ ! -s /tmp/gsd-review-qwen-{phase}.md ]; then
  echo "Qwen review failed or returned empty output." > /tmp/gsd-review-qwen-{phase}.md
fi
```

**Cursor:**
```bash
cat /tmp/gsd-review-prompt-{phase}.md | cursor agent -p --mode ask --trust 2>/dev/null > /tmp/gsd-review-cursor-{phase}.md
if [ ! -s /tmp/gsd-review-cursor-{phase}.md ]; then
  echo "Cursor review failed or returned empty output." > /tmp/gsd-review-cursor-{phase}.md
fi
```

Se uma CLI falhar, registrar o erro e continuar com as demais CLIs.

Exibir progresso:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REVISÃO CROSS-AI — Fase {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Revisando com {CLI}... concluído ✓
◆ Revisando com {CLI}... concluído ✓
```
</step>

<step name="write_reviews">
Combinar todas as respostas de revisão em `{phase_dir}/{padded_phase}-REVIEWS.md`:

```markdown
---
phase: {N}
reviewers: [gemini, claude, codex, coderabbit, opencode, qwen, cursor]
reviewed_at: {ISO timestamp}
plans_reviewed: [{list of PLAN.md files}]
---

# Revisão Cross-AI de Plano — Fase {N}

## Revisão do Gemini

{gemini review content}

---

## Revisão do Claude

{claude review content}

---

## Revisão do Codex

{codex review content}

---

## Revisão do CodeRabbit

{coderabbit review content}

---

## Revisão do OpenCode

{opencode review content}

---

## Revisão do Qwen

{qwen review content}

---

## Revisão do Cursor

{cursor review content}

---

## Resumo Consensual

{síntese das preocupações comuns entre todos os revisores}

### Pontos Fortes em Consenso
{pontos fortes mencionados por 2+ revisores}

### Preocupações em Consenso
{preocupações levantadas por 2+ revisores — maior prioridade}

### Visões Divergentes
{onde os revisores discordaram — vale investigar}
```

Commit:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: cross-AI review for phase {N}" --files {phase_dir}/{padded_phase}-REVIEWS.md
```
</step>

<step name="present_results">
Exibir resumo:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REVISÃO CONCLUÍDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Fase {N} revisada por {count} sistemas de IA.

Preocupações em consenso:
{top 3 preocupações compartilhadas}

Revisão completa: {padded_phase}-REVIEWS.md

Para incorporar o feedback no planejamento:
  /gsd-plan-phase {N} --reviews
```

Limpar arquivos temporários.
</step>

</process>

<success_criteria>
- [ ] Ao menos uma CLI externa invocada com sucesso
- [ ] REVIEWS.md escrito com feedback estruturado
- [ ] Resumo consensual sintetizado a partir de múltiplos revisores
- [ ] Arquivos temporários limpos
- [ ] Usuário sabe como usar o feedback (/gsd-plan-phase --reviews)
</success_criteria>
