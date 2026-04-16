<purpose>
Verificar as mitigações de ameaças de uma fase concluída. Confirmar que as disposições do registro de ameaças do PLAN.md estão resolvidas. Atualizar SECURITY.md.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<available_agent_types>
Tipos de subagentes GSD válidos (use os nomes exatos — não use 'general-purpose' como alternativa):
- gsd-security-auditor — Verifica a cobertura de mitigação de ameaças
</available_agent_types>

<process>

## 0. Inicializar

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_AUDITOR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-security-auditor 2>/dev/null)
```

Analisar: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`.

```bash
AUDITOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-security-auditor --raw)
SECURITY_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.security_enforcement --raw 2>/dev/null || echo "true")
```

Se `SECURITY_CFG` for `false`: sair com "Aplicação de segurança desativada. Ative em /gsd-settings."

Exibir banner: `GSD > PROTEGER FASE {N}: {name}`

## 1. Detectar Estado de Entrada

```bash
SECURITY_FILE=$(ls "${PHASE_DIR}"/*-SECURITY.md 2>/dev/null | head -1)
PLAN_FILES=$(ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null)
SUMMARY_FILES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
```

- **Estado A** (`SECURITY_FILE` não vazio): Auditar existente
- **Estado B** (`SECURITY_FILE` vazio, `PLAN_FILES` e `SUMMARY_FILES` não vazios): Executar a partir dos artefatos
- **Estado C** (`SUMMARY_FILES` vazio): Sair — "Fase {N} não executada. Execute /gsd-execute-phase {N} primeiro."

## 2. Descoberta

### 2a. Ler Artefatos da Fase

Ler PLAN.md — extrair bloco `<threat_model>`: limites de confiança, registro STRIDE (`threat_id`, `category`, `component`, `disposition`, `mitigation_plan`).

### 2b. Ler Sinalizações de Ameaça do Resumo

Ler SUMMARY.md — extrair entradas de `## Threat Flags`.

### 2c. Construir Registro de Ameaças

Por ameaça: `{ threat_id, category, component, disposition, mitigation_pattern, files_to_check }`

## 3. Classificação de Ameaças

Classificar cada ameaça:

| Status | Critérios |
|--------|-----------|
| CLOSED | mitigação encontrada OU risco aceito documentado em SECURITY.md OU transferência documentada |
| OPEN | nenhuma das condições acima |

Construir: `{ threat_id, category, component, disposition, status, evidence }`

Se `threats_open: 0` → ir diretamente para o Passo 6.

## 4. Apresentar Plano de Ameaças


**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON do init for `true`. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número da escolha. Isso é obrigatório para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Chamar AskUserQuestion com tabela de ameaças e opções:
1. "Verificar todas as ameaças abertas" → Passo 5
2. "Aceitar todas as abertas — documentar no log de riscos aceitos" → adicionar ao SECURITY.md riscos aceitos, definir todas como CLOSED, Passo 6
3. "Cancelar" → sair

## 5. Instanciar gsd-security-auditor

```
Task(
  prompt="Read $HOME/.claude/agents/gsd-security-auditor.md for instructions.\n\n" +
    "<files_to_read>{PLAN, SUMMARY, impl files, SECURITY.md}</files_to_read>" +
    "<threat_register>{threat register}</threat_register>" +
    "<config>asvs_level: {SECURITY_ASVS}, block_on: {SECURITY_BLOCK_ON}</config>" +
    "<constraints>Never modify implementation files. Verify mitigations exist — do not scan for new threats. Escalate implementation gaps.</constraints>" +
    "${AGENT_SKILLS_AUDITOR}",
  subagent_type="gsd-security-auditor",
  model="{AUDITOR_MODEL}",
  description="Verify threat mitigations for Phase {N}"
)
```

Tratar retorno:
- `## SECURED` → registrar encerramentos → Passo 6
- `## OPEN_THREATS` → registrar fechados + abertos, apresentar ao usuário opção de aceitar/bloquear → Passo 6
- `## ESCALATE` → apresentar ao usuário → Passo 6

## 6. Escrever/Atualizar SECURITY.md

**Estado B (criar):**
1. Ler template de `$HOME/.claude/get-shit-done/templates/SECURITY.md`
2. Preencher: frontmatter, registro de ameaças, riscos aceitos, trilha de auditoria
3. Escrever em `${PHASE_DIR}/${PADDED_PHASE}-SECURITY.md`

**Estado A (atualizar):**
1. Atualizar status do registro de ameaças, anexar à trilha de auditoria:

```markdown
## Auditoria de Segurança {date}
| Métrica | Contagem |
|---------|----------|
| Ameaças encontradas | {N} |
| Fechadas | {M} |
| Abertas | {K} |
```

**PORTÃO OBRIGATÓRIO:** Se `threats_open > 0` após todas as opções esgotadas (usuário não aceitou, não todas verificadas como fechadas):

```
GSD > SEGURANÇA DA FASE {N} BLOQUEADA
{K} ameaças abertas — avanço da fase bloqueado até threats_open: 0
▶ Corrija as mitigações e execute novamente: /gsd-secure-phase {N}
▶ Ou documente os riscos aceitos em SECURITY.md e execute novamente.
```

NÃO emitir roteamento para próxima fase. Parar aqui.

## 7. Commit

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PHASE}): add/update security threat verification"
```

## 8. Resultados + Roteamento

**Protegida (threats_open: 0):**
```
GSD > FASE {N} COM AMEAÇAS VERIFICADAS
threats_open: 0 — todas as ameaças têm disposições.
▶ /gsd-validate-phase {N}    validar cobertura de testes
▶ /gsd-verify-work {N}       executar UAT
```

Exibir lembrete do `/clear`.

</process>

<success_criteria>
- [ ] Aplicação de segurança verificada — sair se false
- [ ] Estado de entrada detectado (A/B/C) — estado C sai sem erros
- [ ] Modelo de ameaça do PLAN.md analisado, registro construído
- [ ] Sinalizações de ameaça do SUMMARY.md incorporadas
- [ ] threats_open: 0 → pular diretamente para o Passo 6
- [ ] Portão de usuário com tabela de ameaças apresentado
- [ ] Auditor instanciado com contexto completo
- [ ] Todos os três formatos de retorno (SECURED/OPEN_THREATS/ESCALATE) tratados
- [ ] SECURITY.md criado ou atualizado
- [ ] threats_open > 0 BLOQUEIA o avanço (sem roteamento para próxima fase emitido)
- [ ] Resultados com roteamento apresentados no sucesso
</success_criteria>
