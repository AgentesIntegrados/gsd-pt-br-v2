<purpose>
Remover uma fase futura do roadmap e renumerar as fases subsequentes. Protegido por verificações de segurança para prevenir remoção de fases com trabalho concluído.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Analisar $ARGUMENTS para número de fase alvo.

Se nenhum argumento fornecido: exibir uso:
```
Uso: /gsd-remove-phase {N}

Exemplo:
  /gsd-remove-phase 5
  
Remove a Fase 5 do roadmap e renumera as fases subsequentes.
Nota: Não é possível remover fases com trabalho concluído.
```
Sair.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► REMOVER FASE {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="validate_target">
Verificar que a fase pode ser removida:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${TARGET_PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

**Verificação de segurança 1: A fase existe?**
Se `phase_found` for false: exibir "Fase {N} não encontrada no roadmap." e sair.

**Verificação de segurança 2: A fase tem trabalho concluído?**
```bash
ls .planning/phases/$(printf '%02d' ${TARGET_PHASE})-*/*-SUMMARY.md 2>/dev/null | wc -l
```
Se SUMMARY.md encontrado: exibir
```
╔══════════════════════════════════════════════════════════════╗
║  ERRO DE SEGURANÇA                                           ║
╚══════════════════════════════════════════════════════════════╝

A Fase {N} tem trabalho concluído ({M} planos executados).
Não é possível remover fases com trabalho concluído.

Para descontinuar trabalho futuro: edite ROADMAP.md diretamente.
```
Sair.

**Verificação de segurança 3: Outras fases dependem desta?**
```bash
grep -l "Depende de: Fase ${TARGET_PHASE}\|depends_on.*${TARGET_PHASE}" .planning/ROADMAP.md 2>/dev/null
```
Se dependências encontradas: avisar mas não bloquear.

Exibir o que mudará:
```
Remover Fase {N}: "{nome}" irá:
  ✗ Remover entrada da fase do ROADMAP.md
  ✗ Remover diretório .planning/phases/{NN}-{slug}/ (se vazio)
  → Renumerar Fase {N+1} → Fase {N}
  → Renumerar Fase {N+2} → Fase {N+1}
  ...
```
</step>

<step name="confirm_removal">
**TRILHO DE SEGURANÇA: Sempre confirmar remoção de fase (ação destrutiva).**

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Perguntar via AskUserQuestion:
- header: "Confirmar Remoção"
- question: "Tem certeza de que deseja remover a Fase {N}: '{nome}'?"
- options:
  - "Sim, remover" — "Prosseguir com remoção e renumeração"
  - "Cancelar" — "Não fazer alterações"

Se Cancelar: sair com "Remoção cancelada. Nenhuma alteração feita."
</step>

<step name="execute_removal">
Remover a fase via gsd-tools:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase remove "${TARGET_PHASE}"
```

Isso vai:
- Remover a entrada da fase do ROADMAP.md
- Renumerar todas as fases subsequentes
- Atualizar contagens de progresso

Renomear diretórios de fase afetados:
```bash
# Renomear da posição mais baixa para cima para evitar colisões
for i in {(TARGET_PHASE+1)..max_phase}; do
  OLD_DIR=".planning/phases/$(printf '%02d' $i)-*"
  NEW_NUM=$(printf '%02d' $((i-1)))
  if ls -d .planning/phases/$(printf '%02d' $i)-* 2>/dev/null | head -1; then
    OLD_NAME=$(ls -d .planning/phases/$(printf '%02d' $i)-* | head -1 | xargs basename | cut -d'-' -f2-)
    mv ".planning/phases/$(printf '%02d' $i)-${OLD_NAME}" ".planning/phases/${NEW_NUM}-${OLD_NAME}"
  fi
done
```

Remover diretório da fase alvo se vazio:
```bash
PHASE_DIR=".planning/phases/$(printf '%02d' ${TARGET_PHASE})-*"
if ls -d $PHASE_DIR 2>/dev/null; then
  rmdir $PHASE_DIR 2>/dev/null || echo "Diretório não vazio — deixado em vigor"
fi
```
</step>

<step name="update_state">
Atualizar STATE.md:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Last Activity" "$(date +%Y-%m-%d)"
```

Se a fase removida era a fase atual: atualizar fase atual para próxima:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Current Phase" "$(printf '%02d' ${TARGET_PHASE})"
```
</step>

<step name="commit_changes">
Commitar alterações:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: remover Fase ${TARGET_PHASE} '${PHASE_NAME}', renumerar fases subsequentes" \
  --files .planning/ROADMAP.md .planning/STATE.md
```
</step>

<step name="report">
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► FASE REMOVIDA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Removida: Fase {N} — {nome}
Fases renumeradas: {M} fases

───────────────────────────────────────────────────────────────

## ▶ Próximos passos

- `/gsd-progress` — ver roadmap atualizado
- `/gsd-plan-phase {N}` — planejar a próxima fase

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Fase alvo validada (existe, sem trabalho concluído)
- [ ] Dependências verificadas e avisado se encontradas
- [ ] Confirmação do usuário obtida (ação destrutiva)
- [ ] ROADMAP.md atualizado (fase removida, subsequentes renumeradas)
- [ ] Diretórios de fase renumerados
- [ ] STATE.md atualizado
- [ ] Alterações commitadas
</success_criteria>
