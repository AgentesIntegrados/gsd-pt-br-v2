<purpose>
Criar arquivos de handoff estruturados `.planning/HANDOFF.json` e `.continue-here.md` para preservar o estado completo do trabalho entre sessões. O JSON fornece estado legível por máquina para `/gsd-resume-work`; o markdown fornece contexto legível por humanos.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="detect">
## Detecção de Contexto

Determine que tipo de trabalho está sendo pausado e defina o destino do handoff de acordo:

```bash
# Verificar fase ativa
phase=$(( ls -lt .planning/phases/*/PLAN.md 2>/dev/null || true ) | head -1 | grep -oP 'phases/\K[^/]+' || true)

# Verificar spike ativo
spike=$(( ls -lt .planning/spikes/*/SPIKE.md .planning/spikes/*/DESIGN.md 2>/dev/null || true ) | head -1 | grep -oP 'spikes/\K[^/]+' || true)

# Verificar deliberação ativa
deliberation=$(ls .planning/deliberations/*.md 2>/dev/null | head -1 || true)
```

- **Trabalho de fase**: diretório de fase ativo → handoff para `.planning/phases/XX-name/.continue-here.md`
- **Trabalho de spike**: diretório de spike ativo ou arquivos relacionados a spike (sem fase ativa) → handoff para `.planning/spikes/SPIKE-NNN/.continue-here.md` (criar diretório se necessário)
- **Trabalho de deliberação**: arquivo de deliberação ativo (sem fase/spike) → handoff para `.planning/deliberations/.continue-here.md`
- **Trabalho de pesquisa**: notas de pesquisa existem mas sem fase/spike/deliberação → handoff para `.planning/.continue-here.md`
- **Padrão**: sem contexto detectável → handoff para `.planning/.continue-here.md`, note a ambiguidade em `<current_state>`

Se uma fase for detectada, prossiga com o caminho de handoff da fase. Caso contrário, use o primeiro caminho não-fase correspondente acima.
</step>

<step name="gather">
**Colete o estado completo para handoff:**

1. **Posição atual**: Qual fase, qual plano, qual tarefa
2. **Trabalho concluído**: O que foi feito nesta sessão
3. **Trabalho restante**: O que ainda falta no plano/fase atual
4. **Decisões tomadas**: Decisões-chave e justificativas
5. **Bloqueios/problemas**: Qualquer coisa emperrada
6. **Ações humanas pendentes**: Coisas que precisam de intervenção manual (configuração de MCP, chaves de API, aprovações, testes manuais)
7. **Processos em segundo plano**: Quaisquer servidores/observadores em execução que faziam parte do fluxo de trabalho
8. **Arquivos modificados**: O que foi alterado mas não commitado
9. **Restrições bloqueantes**: Anti-padrões ou falhas metodológicas encontradas durante esta sessão que um agente retomando DEVE conhecer antes de prosseguir. Inclua apenas itens descobertos por falha real — não avisos ou previsões. Atribua a cada restrição uma `severity`:
   - `blocking` — O agente retomando DEVE demonstrar compreensão antes de prosseguir. Os fluxos de trabalho discuss-phase e execute-phase vão aplicar uma verificação obrigatória de entendimento.
   - `advisory` — Contexto importante, mas não bloqueia a retomada.

Peça ao usuário esclarecimentos se necessário através de perguntas conversacionais.

**Inspecione também arquivos SUMMARY.md para completions falsas:**
```bash
# Verificar conteúdo placeholder em resumos existentes
grep -l "To be filled\|placeholder\|TBD" .planning/phases/*/*.md 2>/dev/null || true
```
Reporte quaisquer resumos com conteúdo placeholder como itens incompletos.
</step>

<step name="write_structured">
**Escreva handoff estruturado em `.planning/HANDOFF.json`:**

```bash
timestamp=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" current-timestamp full --raw)
```

```json
{
  "version": "1.0",
  "timestamp": "{timestamp}",
  "phase": "{phase_number}",
  "phase_name": "{phase_name}",
  "phase_dir": "{phase_dir}",
  "plan": {current_plan_number},
  "task": {current_task_number},
  "total_tasks": {total_task_count},
  "status": "paused",
  "completed_tasks": [
    {"id": 1, "name": "{task_name}", "status": "done", "commit": "{short_hash}"},
    {"id": 2, "name": "{task_name}", "status": "done", "commit": "{short_hash}"},
    {"id": 3, "name": "{task_name}", "status": "in_progress", "progress": "{what_done}"}
  ],
  "remaining_tasks": [
    {"id": 4, "name": "{task_name}", "status": "not_started"},
    {"id": 5, "name": "{task_name}", "status": "not_started"}
  ],
  "blockers": [
    {"description": "{bloqueio}", "type": "technical|human_action|external", "workaround": "{se houver}"}
  ],
  "human_actions_pending": [
    {"action": "{o que precisa ser feito}", "context": "{por que}", "blocking": true}
  ],
  "decisions": [
    {"decision": "{o quê}", "rationale": "{por que}", "phase": "{phase_number}"}
  ],
  "uncommitted_files": [],
  "next_action": "{primeira ação específica ao retomar}",
  "context_notes": "{estado mental, abordagem, o que você estava pensando}"
}
```
</step>

<step name="write">
**Escreva handoff no caminho determinado na etapa de detecção** (ex: `.planning/phases/XX-name/.continue-here.md`, `.planning/spikes/SPIKE-NNN/.continue-here.md`, ou `.planning/.continue-here.md`):

```markdown
---
context: [phase|spike|deliberation|research|default]
phase: XX-name
task: 3
total_tasks: 7
status: in_progress
last_updated: [timestamp de current-timestamp]
---

# RESTRIÇÕES BLOQUEANTES — Leia Antes de Qualquer Coisa

> Estas não são sugestões. Cada restrição abaixo foi descoberta por falha.
> Reconheça cada uma explicitamente antes de prosseguir.

- [ ] RESTRIÇÃO: [nome] — [o que é] — [mitigação estrutural necessária]

**Não prossiga até que todas as caixas estejam marcadas.**

_Se nenhuma restrição foi identificada ainda, remova esta seção._

## Anti-Padrões Críticos

| Padrão | Descrição | Severidade | Mecanismo de Prevenção |
|--------|-----------|------------|------------------------|
| [nome do padrão] | [o que é e como se manifestou] | blocking | [passo estrutural que previne recorrência — não reconhecimento] |
| [nome do padrão] | [o que é e como se manifestou] | advisory | [orientação para evitá-lo] |

**Valores de severidade:** `blocking` — o agente retomando deve passar em verificação de entendimento antes de prosseguir. `advisory` — contexto importante, não bloqueia a retomada.

_Remova linhas que não se aplicam. Os fluxos de trabalho discuss-phase e execute-phase analisam esta tabela e aplicam verificação obrigatória de entendimento para quaisquer linhas `blocking`._

<current_state>
[Onde exatamente estamos? Contexto imediato]
</current_state>

<completed_work>

Tarefas Concluídas:
- Tarefa 1: [nome] - Concluída
- Tarefa 2: [nome] - Concluída
- Tarefa 3: [nome] - Em andamento, [o que foi feito]
</completed_work>

<remaining_work>

- Tarefa 3: [o que falta]
- Tarefa 4: Não iniciada
- Tarefa 5: Não iniciada
</remaining_work>

<decisions_made>

- Decidiu usar [X] porque [motivo]
- Escolheu [abordagem] em vez de [alternativa] porque [motivo]
</decisions_made>

<blockers>
- [Bloqueio 1]: [status/solução alternativa]
</blockers>

## Leitura Obrigatória (em ordem)
<!-- Liste documentos que o agente retomando deve ler antes de agir -->
1. [documento] — [por que é importante]
1. `.planning/METHODOLOGY.md` (se existir) — lentes analíticas do projeto; aplique antes de qualquer análise de suposições

## Anti-Padrões Críticos (NÃO repita estes)
<!-- Erros descobertos nesta sessão que devem ser estruturalmente evitados -->
- [ANTI-PADRÃO]: [o que é] → [mitigação estrutural]

## Estado da Infraestrutura
<!-- Serviços em execução, estado externo, especificidades do ambiente -->
- [serviço/env]: [estado atual]

## Crítica Pré-Execução Necessária
<!-- Preencha APENAS se pausar entre design e execução (ex: design de spike concluído, ainda não executado) -->
- Artefato de design: [caminho]
- Foco da crítica: [perguntas-chave que o crítico deve sondar]
- Gate: NÃO comece a execução até que a crítica esteja completa e o design revisado

<context>
[Estado mental, o que você estava pensando, o plano]
</context>

<next_action>
Comece com: [primeira ação específica ao retomar]
</next_action>
```

Seja específico o suficiente para que um Claude novo entenda imediatamente.

Use `current-timestamp` para o campo last_updated. Você pode usar init todos (que fornece timestamps) ou chamar diretamente:
```bash
timestamp=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" current-timestamp full --raw)
```
</step>

<step name="commit">
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "wip: [context-name] pausado em [X]/[Y]" --files [handoff-path] .planning/HANDOFF.json
```
</step>

<step name="confirm">
```
✓ Handoff criado:
  - .planning/HANDOFF.json (estruturado, legível por máquina)
  - [handoff-path] (legível por humanos)

Estado atual:

- Contexto: [phase|spike|deliberation|research]
- Local: [XX-name ou SPIKE-NNN]
- Tarefa: [X] de [Y]
- Status: [in_progress/blocked]
- Bloqueios: [count] ({human_actions_pending count} precisam de ação humana)
- Commitado como WIP

Para retomar: /gsd-resume-work

```
</step>

</process>

<success_criteria>
- [ ] Contexto detectado (phase/spike/deliberation/research/default)
- [ ] .continue-here.md criado no caminho correto para o contexto detectado
- [ ] Seções de Leitura Obrigatória, Anti-Padrões e Estado da Infraestrutura preenchidas
- [ ] Seção de Crítica Pré-Execução preenchida se pausar entre design e execução
- [ ] Commitado como WIP
- [ ] Usuário sabe a localização e como retomar
</success_criteria>
</output>
