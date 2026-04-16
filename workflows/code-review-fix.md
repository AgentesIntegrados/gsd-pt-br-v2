<purpose>
Corrigir automaticamente as descobertas do REVIEW.md. Lê o arquivo de revisão gerado por `/gsd-code-review`, aplica correções para descobertas de prioridade alta e crítica, e atualiza o REVIEW.md com o status das correções.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar contexto do projeto:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_FIXER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-code-fixer 2>/dev/null)
FIXER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-code-fixer --raw)
```

Extrair do JSON: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `commit_docs`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CORREÇÃO DE CÓDIGO — FASE {N}: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="locate_review">
Localizar o REVIEW.md para esta fase:

```bash
REVIEW_FILE=$(ls "${PHASE_DIR}"/*-REVIEW.md 2>/dev/null | head -1)
```

**Se não encontrado:**
```
Nenhum REVIEW.md encontrado para a Fase {N}.
Execute primeiro /gsd-code-review {N} para gerar descobertas.
```
Sair.

**Se encontrado:** Ler o conteúdo completo do REVIEW.md.

Contar descobertas por prioridade: Crítico, Alto, Médio, Baixo, Info.

Exibir:
```
◆ Arquivo de revisão: {caminho para REVIEW.md}
  Crítico: {N} | Alto: {N} | Médio: {N} | Baixo: {N}
```
</step>

<step name="confirm_scope">
**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Perguntar via AskUserQuestion:
- header: "Escopo da Correção"
- question: "Quais descobertas corrigir automaticamente?"
- options:
  - "Apenas Crítico e Alto" — "Corrigir os {N} problemas de prioridade mais alta"
  - "Todos (Crítico + Alto + Médio)" — "Corrigir todos os {N} problemas não informativos"
  - "Cancelar" — "Não fazer alterações agora"

Se "Cancelar": sair com "Nenhuma alteração feita."
</step>

<step name="spawn_fixer">
Exibir:
```
◆ Iniciando corretor de código...
  Corrigindo: {N} descobertas
  Modelo: {model}
```

Construir prompt:

```markdown
Leia $HOME/.claude/agents/gsd-code-fixer.md para instruções.

<objetivo>
Corrigir descobertas de revisão de código para Fase {phase_number}: {phase_name}
Aplicar apenas correções seguras — NÃO refatorar além do escopo das descobertas.
</objetivo>

<arquivo_de_revisao>
{conteúdo completo do REVIEW.md}
</arquivo_de_revisao>

<escopo>
{prioridades selecionadas pelo usuário: Crítico+Alto ou Crítico+Alto+Médio}
</escopo>

${AGENT_SKILLS_FIXER}

<config>
review_file: {caminho do REVIEW.md}
phase_dir: {phase_dir}
</config>
```

```
Task(
  prompt=fixer_prompt,
  subagent_type="gsd-code-fixer",
  model="{FIXER_MODEL}",
  description="Correção de código Fase {N}"
)
```
</step>

<step name="handle_return">
**Se `## CORREÇÃO CONCLUÍDA`:**

Exibir resumo das correções:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CÓDIGO CORRIGIDO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Fase {N}: {Nome}** — {N} descobertas corrigidas

| Resultado    | Contagem |
|-------------|---------|
| Corrigido   | {N}     |
| Pulado      | {N}     |
| Requer Human| {N}     |

Descobertas puladas ou que requerem revisão humana estão marcadas no REVIEW.md.
```

**Se `## CORREÇÃO PARCIAL`:**
Exibir o que foi corrigido e o que foi pulado com motivos.

**Se `## CORREÇÃO BLOQUEADA`:**
Exibir detalhes do bloqueio. Oferecer opções para resolver manualmente.
</step>

<step name="commit_fixes">
Se correções foram aplicadas:

```bash
# Commitar alterações nos arquivos de código
git add {arquivos_modificados}
git commit -m "fix(${padded_phase}): aplicar correções de revisão de código"

# Atualizar REVIEW.md com status
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): atualizar REVIEW.md com status das correções" --files "${PHASE_DIR}/${PADDED_PHASE}-REVIEW.md"
```
</step>

<step name="report">
```
───────────────────────────────────────────────────────────────

## Próximos passos

- `/gsd-code-review {fase}` — re-revisar para verificar correções
- `/gsd-verify-phase {fase}` — verificar implementação da fase
- Revisar descobertas puladas no: {caminho do REVIEW.md}

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] REVIEW.md localizado para a fase alvo
- [ ] Escopo de correção confirmado pelo usuário
- [ ] gsd-code-fixer iniciado com contexto correto
- [ ] Correções aplicadas aos arquivos de código-fonte
- [ ] REVIEW.md atualizado com status das correções
- [ ] Alterações commitadas (código + REVIEW.md)
- [ ] Resumo de correções exibido ao usuário
</success_criteria>
