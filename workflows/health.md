<purpose>
Valida a integridade do diretório `.planning/` e reporta problemas acionáveis. Verifica arquivos ausentes, configurações inválidas, estado inconsistente e planos órfãos. Opcionalmente repara problemas corrigíveis automaticamente.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt de invocação antes de começar.
</required_reading>

<process>

<step name="parse_args">
**Analise os argumentos:**

Verifique se a flag `--repair` está presente nos argumentos do comando.

```
REPAIR_FLAG=""
if arguments contain "--repair"; then
  REPAIR_FLAG="--repair"
fi
```
</step>

<step name="run_health_check">
**Execute a validação de saúde:**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" validate health $REPAIR_FLAG
```

Analise a saída JSON:
- `status`: "healthy" | "degraded" | "broken"
- `errors[]`: Problemas críticos (code, message, fix, repairable)
- `warnings[]`: Problemas não críticos
- `info[]`: Notas informativas
- `repairable_count`: Número de problemas corrigíveis automaticamente
- `repairs_performed[]`: Ações tomadas se --repair foi usado
</step>

<step name="format_output">
**Formate e exiba os resultados:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD Health Check
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: SAUDÁVEL | DEGRADADO | QUEBRADO
Erros: N | Avisos: N | Info: N
```

**Se reparos foram realizados:**
```
## Reparos Realizados

- ✓ config.json: Criado com valores padrão
- ✓ STATE.md: Regenerado a partir do roadmap
```

**Se erros existirem:**
```
## Erros

- [E001] config.json: Erro de parse JSON na linha 5
  Correção: Execute /gsd-health --repair para redefinir aos valores padrão

- [E002] PROJECT.md não encontrado
  Correção: Execute /gsd-new-project para criar
```

**Se avisos existirem:**
```
## Avisos

- [W002] STATE.md referencia a fase 5, mas apenas as fases 1-3 existem
  Correção: Revise STATE.md manualmente antes de alterá-lo; o repair não sobrescreverá um STATE.md existente

- [W005] O diretório de fase "1-setup" não segue o formato NN-name
  Correção: Renomeie para corresponder ao padrão (ex.: 01-setup)
```

**Se info existir:**
```
## Info

- [I001] 02-implementation/02-01-PLAN.md não tem SUMMARY.md
  Nota: Pode estar em progresso
```

**Rodapé (se problemas reparáveis existirem e --repair NÃO foi usado):**
```
---
N problemas podem ser reparados automaticamente. Execute: /gsd-health --repair
```
</step>

<step name="offer_repair">
**Se problemas reparáveis existirem e --repair NÃO foi usado:**

Pergunte ao usuário se deseja executar os reparos:

```
Deseja executar /gsd-health --repair para corrigir N problemas automaticamente?
```

Se sim, execute novamente com a flag --repair e exiba os resultados.
</step>

<step name="verify_repairs">
**Se reparos foram realizados:**

Execute novamente a verificação de saúde sem --repair para confirmar que os problemas foram resolvidos:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" validate health
```

Reporte o status final.
</step>

</process>

<error_codes>

| Código | Severidade | Descrição | Reparável |
|------|----------|-------------|------------|
| E001 | error | Diretório .planning/ não encontrado | Não |
| E002 | error | PROJECT.md não encontrado | Não |
| E003 | error | ROADMAP.md não encontrado | Não |
| E004 | error | STATE.md não encontrado | Sim |
| E005 | error | Erro de parse do config.json | Sim |
| W001 | warning | PROJECT.md com seção obrigatória ausente | Não |
| W002 | warning | STATE.md referencia fase inválida | Não |
| W003 | warning | config.json não encontrado | Sim |
| W004 | warning | config.json com valor de campo inválido | Não |
| W005 | warning | Incompatibilidade de nomenclatura do diretório de fase | Não |
| W006 | warning | Fase no ROADMAP mas sem diretório | Não |
| W007 | warning | Fase no disco mas não no ROADMAP | Não |
| W008 | warning | config.json: workflow.nyquist_validation ausente (padrão habilitado mas agentes podem pular) | Sim |
| W009 | warning | Fase tem Validation Architecture no RESEARCH.md mas sem VALIDATION.md | Não |
| I001 | info | Plano sem SUMMARY (pode estar em progresso) | Não |

</error_codes>

<repair_actions>

| Ação | Efeito | Risco |
|--------|--------|------|
| createConfig | Cria config.json com valores padrão | Nenhum |
| resetConfig | Apaga + recria config.json | Perde configurações personalizadas |
| regenerateState | Cria STATE.md a partir da estrutura do ROADMAP quando estiver ausente | Perde histórico de sessão |
| addNyquistKey | Adiciona workflow.nyquist_validation: true ao config.json | Nenhum — corresponde ao padrão existente |

**Não reparável (risco muito alto):**
- Conteúdo de PROJECT.md, ROADMAP.md
- Renomeação de diretório de fase
- Limpeza de planos órfãos

</repair_actions>

<stale_task_cleanup>
**Específico para Windows:** Verifique diretórios de task do Claude Code obsoletos que se acumulam em crash/freeze.
Estes são deixados para trás quando subagentes são forçadamente encerrados e consomem espaço em disco.

Quando `--repair` estiver ativo, detecte e limpe:

```bash
# Verifique diretórios de task obsoletos (mais de 24 horas)
TASKS_DIR="$HOME/.claude/tasks"
if [ -d "$TASKS_DIR" ]; then
  STALE_COUNT=$( (find "$TASKS_DIR" -maxdepth 1 -type d -mtime +1 2>/dev/null || true) | wc -l )
  if [ "$STALE_COUNT" -gt 0 ]; then
    echo "⚠️  Found $STALE_COUNT stale task directories in $HOME/.claude/tasks/"
    echo "   These are leftover from crashed subagent sessions."
    echo "   Run: rm -rf $HOME/.claude/tasks/*  (safe — only affects dead sessions)"
  fi
fi
```

Reporte como diagnóstico de info: `I002 | info | Diretórios de task de subagente obsoletos encontrados | Sim (--repair os remove)`
</stale_task_cleanup>
