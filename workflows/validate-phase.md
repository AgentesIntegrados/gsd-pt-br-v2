<purpose>
Auditar lacunas de validação Nyquist para uma fase concluída. Gerar testes ausentes. Atualizar VALIDATION.md.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<available_agent_types>
Tipos de subagente GSD válidos (use os nomes exatos — não use 'general-purpose' como fallback):
- gsd-nyquist-auditor — Valida a cobertura de verificação
</available_agent_types>

<process>

## 0. Inicializar

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_AUDITOR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-nyquist-auditor 2>/dev/null)
```

Analisar: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`.

```bash
AUDITOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-nyquist-auditor --raw)
NYQUIST_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.nyquist_validation --raw)
```

Se `NYQUIST_CFG` for `false`: sair com "Validação Nyquist está desabilitada. Habilite via /gsd-settings."

Exibir banner: `GSD > VALIDAR FASE {N}: {name}`

## 1. Detectar Estado de Entrada

```bash
VALIDATION_FILE=$(ls "${PHASE_DIR}"/*-VALIDATION.md 2>/dev/null | head -1)
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
```

- **Estado A** (`VALIDATION_FILE` não-vazio): Auditar existente
- **Estado B** (`VALIDATION_FILE` vazio, `SUMMARY_FILES` não-vazio): Reconstruir a partir de artefatos
- **Estado C** (`SUMMARY_FILES` vazio): Sair — "Fase {N} não executada. Execute /gsd-execute-phase {N} ${GSD_WS} primeiro."

## 2. Descoberta

### 2a. Ler Artefatos da Fase

Ler todos os arquivos PLAN e SUMMARY. Extrair: listas de tarefas, IDs de requisitos, arquivos-chave alterados, blocos de verificação.

### 2b. Construir Mapa de Requisito-para-Tarefa

Por tarefa: `{ task_id, plan_id, wave, requirement_ids, has_automated_command }`

### 2c. Detectar Infraestrutura de Testes

Estado A: Analisar a partir da tabela de Infraestrutura de Testes do VALIDATION.md existente.
Estado B: Varredura de sistema de arquivos:

```bash
find . -name "pytest.ini" -o -name "jest.config.*" -o -name "vitest.config.*" -o -name "pyproject.toml" 2>/dev/null | head -10
find . \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" \) -not -path "*/node_modules/*" 2>/dev/null | head -40
```

### 2d. Referência Cruzada

Cruzar cada requisito com testes existentes por nome de arquivo, imports, descrições de testes. Registrar: requisito → arquivo_de_teste → status.

## 3. Análise de Lacunas

Classificar cada requisito:

| Status | Critério |
|--------|----------|
| COBERTO | Teste existe, tem como alvo o comportamento, passa sem erros |
| PARCIAL | Teste existe, falhando ou incompleto |
| AUSENTE | Nenhum teste encontrado |

Construir: `{ task_id, requirement, gap_type, suggested_test_path, suggested_command }`

Sem lacunas → pular para Etapa 6, definir `nyquist_compliant: true`.

## 4. Apresentar Plano de Lacunas


**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário para digitar o número de sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Chamar AskUserQuestion com tabela de lacunas e opções:
1. "Corrigir todas as lacunas" → Etapa 5
2. "Pular — marcar como somente manual" → adicionar a Manual-Only, Etapa 6
3. "Cancelar" → sair

## 5. Spawnar gsd-nyquist-auditor

```
Task(
  prompt="Read $HOME/.claude/agents/gsd-nyquist-auditor.md for instructions.\n\n" +
    "<files_to_read>{PLAN, SUMMARY, impl files, VALIDATION.md}</files_to_read>" +
    "<gaps>{gap list}</gaps>" +
    "<test_infrastructure>{framework, config, commands}</test_infrastructure>" +
    "<constraints>Never modify impl files. Max 3 debug iterations. Escalate impl bugs.</constraints>" +
    "${AGENT_SKILLS_AUDITOR}",
  subagent_type="gsd-nyquist-auditor",
  model="{AUDITOR_MODEL}",
  description="Fill validation gaps for Phase {N}"
)
```

Tratar retorno:
- `## GAPS FILLED` → registrar testes + atualizações de mapa, Etapa 6
- `## PARTIAL` → registrar resolvidos, mover escalados para somente manual, Etapa 6
- `## ESCALATE` → mover todos para somente manual, Etapa 6

## 6. Gerar/Atualizar VALIDATION.md

**Estado B (criar):**
1. Ler template de `$HOME/.claude/get-shit-done/templates/VALIDATION.md`
2. Preencher: frontmatter, Infraestrutura de Testes, Mapa Por-Tarefa, Somente Manual, Assinatura
3. Escrever em `${PHASE_DIR}/${PADDED_PHASE}-VALIDATION.md`

**Estado A (atualizar):**
1. Atualizar status do Mapa Por-Tarefa, adicionar escalados ao Somente Manual, atualizar frontmatter
2. Acrescentar trilha de auditoria:

```markdown
## Auditoria de Validação {date}
| Métrica | Contagem |
|---------|---------|
| Lacunas encontradas | {N} |
| Resolvidas | {M} |
| Escaladas | {K} |
```

## 7. Commitar

```bash
git add {test_files}
git commit -m "test(phase-${PHASE}): add Nyquist validation tests"

node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PHASE}): add/update validation strategy"
```

## 8. Resultados + Roteamento

**Conforme:**
```
GSD > FASE {N} ESTÁ EM CONFORMIDADE COM NYQUIST
Todos os requisitos têm verificação automatizada.
▶ Próximo: /gsd-audit-milestone ${GSD_WS}
```

**Parcial:**
```
GSD > FASE {N} VALIDADA (PARCIAL)
{M} automatizados, {K} somente manual.
▶ Tentar novamente: /gsd-validate-phase {N} ${GSD_WS}
```

Exibir lembrete de `/clear`.

</process>

<success_criteria>
- [ ] Configuração Nyquist verificada (sair se desabilitada)
- [ ] Estado de entrada detectado (A/B/C)
- [ ] Estado C sai normalmente
- [ ] Arquivos PLAN/SUMMARY lidos, mapa de requisitos construído
- [ ] Infraestrutura de testes detectada
- [ ] Lacunas classificadas (COBERTO/PARCIAL/AUSENTE)
- [ ] Gate do usuário com tabela de lacunas
- [ ] Auditor spawanado com contexto completo
- [ ] Os três formatos de retorno tratados
- [ ] VALIDATION.md criado ou atualizado
- [ ] Arquivos de teste commitados separadamente
- [ ] Resultados com roteamento apresentados
</success_criteria>
</output>
