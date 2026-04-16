<purpose>
Detectar o estado atual do projeto e avançar automaticamente para o próximo passo lógico do fluxo GSD.
Lê o estado do projeto para determinar a progressão: discuss → plan → execute → verify → complete.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="detect_state">
Leia o estado do projeto para determinar a posição atual:

```bash
# Obter snapshot do estado
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state json 2>/dev/null || echo "{}"
```

Leia também:
- `.planning/STATE.md` — fase atual, progresso, contagens de planos
- `.planning/ROADMAP.md` — estrutura de milestones e lista de fases

Extraia:
- `current_phase` — qual fase está ativa
- `plan_of` / `plans_total` — progresso de execução do plano
- `progress` — porcentagem geral
- `status` — ativo, pausado, etc.

Se nenhum diretório `.planning/` existir:
```
Nenhum projeto GSD detectado. Execute `/gsd-new-project` para começar.
```
Sair.
</step>

<step name="safety_gates">
Execute verificações de parada antes de rotear. Saia na primeira ocorrência a menos que `--force` tenha sido passado.

Se a flag `--force` foi passada, pule todos os gates e o guarda consecutivo.
Imprima um aviso de uma linha: `⚠ --force: ignorando gates de segurança`
Em seguida, vá diretamente para `determine_next_action`.

**Gate 1: Checkpoint não resolvido**
Verifique se `.planning/.continue-here.md` existe:
```bash
[ -f .planning/.continue-here.md ]
```
Se encontrado:
```
⛔ Parada forçada: Checkpoint não resolvido

`.planning/.continue-here.md` existe — uma sessão anterior deixou
trabalho inacabado que precisa de revisão manual antes de avançar.

Leia o arquivo, resolva o problema e depois exclua-o para continuar.
Use `--force` para ignorar esta verificação.
```
Sair (não rotear).

**Gate 2: Estado de erro**
Verifique se STATE.md contém `status: error` ou `status: failed`:
Se encontrado:
```
⛔ Parada forçada: Projeto em estado de erro

STATE.md mostra status: {status}. Resolva o erro antes de avançar.
Execute `/gsd-health` para diagnosticar, ou corrija manualmente STATE.md.
Use `--force` para ignorar esta verificação.
```
Sair.

**Gate 3: Verificação não conferida**
Verifique se a fase atual tem um VERIFICATION.md com itens `FAIL` sem substituições:
Se encontrado:
```
⛔ Parada forçada: Falhas de verificação não conferidas

VERIFICATION.md da fase {N} tem {count} itens FAIL não resolvidos.
Corrija as falhas ou adicione substituições antes de avançar para a próxima fase.
Use `--force` para ignorar esta verificação.
```
Sair.

**Varredura de completude de fases anteriores:**
Após passar pelos três gates de parada forçada, varra todas as fases que precedem a fase atual na ordem do ROADMAP.md em busca de trabalho incompleto. Use o output existente de `gsd-tools.cjs phase json <N>` para inspecionar cada fase anterior.

Detecte três categorias de trabalho incompleto:
1. **Planos sem resumos** — um PLAN.md existe em um diretório de fase anterior mas nenhum SUMMARY.md correspondente existe (execução iniciada mas não concluída).
2. **Falhas de verificação não substituídas** — uma fase anterior tem um VERIFICATION.md com itens `FAIL` sem anotação de substituição.
3. **CONTEXT.md sem planos** — um diretório de fase anterior tem um CONTEXT.md mas nenhum arquivo PLAN.md (discussão aconteceu, planejamento nunca executou).

Se nenhum trabalho anterior incompleto for encontrado, continue para `determine_next_action` silenciosamente sem interrupção.

Se trabalho anterior incompleto for encontrado, mostre um relatório estruturado de completude:
```
⚠ Fase anterior tem trabalho incompleto

Fase {N} — "{name}" tem itens não resolvidos:
  • Plano {N}-{M} ({slug}): executado mas sem SUMMARY.md
  [... itens adicionais ...]

Avançar antes de resolver estes itens pode causar:
  • Lacunas de verificação — a verificação de fases futuras não terá visibilidade do que as fases anteriores entregaram
  • Perda de contexto — planos executados sem resumos não deixam registro para agentes futuros

Opções:
  [C] Continuar e deferir estes itens para o backlog
  [S] Parar e resolver manualmente (recomendado)
  [F] Forçar avanço sem registrar o deferimento

Escolha [S]:
```

**Se o usuário escolher "Parar" (S ou Enter/padrão):** Sair sem rotear.

**Se o usuário escolher "Continuar e deferir" (C):**
1. Para cada item incompleto, crie uma entrada de backlog em `ROADMAP.md` sob `## Backlog` usando o esquema de numeração `999.x` existente:
```markdown
### Fase 999.{N}: Acompanhamento — Planos incompletos da Fase {src} (BACKLOG)

**Objetivo:** Resolver planos que foram executados sem produzir resumos durante a execução da Fase {src}
**Fase de origem:** {src}
**Diferido em:** {date} durante o avanço /gsd-next para a Fase {dest}
**Planos:**
- [ ] {N}-{M}: {slug} (executado, sem SUMMARY.md)
```
2. Comite o registro de deferimento:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: diferir itens incompletos da Fase {src} para backlog"
```
3. Continue roteando para `determine_next_action` imediatamente — sem segundo prompt.

**Se o usuário escolher "Forçar" (F):** Continue para `determine_next_action` sem registrar deferimento.
</step>

<step name="determine_next_action">
Aplique regras de roteamento com base no estado:

**Rota 1: Nenhuma fase existe ainda → discuss**
Se o ROADMAP tem fases mas nenhum diretório de fase existe no disco:
→ Próxima ação: `/gsd-discuss-phase <primeira-fase>`

**Rota 2: Fase existe mas não tem CONTEXT.md ou RESEARCH.md → discuss**
Se o diretório da fase atual existe mas não tem CONTEXT.md nem RESEARCH.md:
→ Próxima ação: `/gsd-discuss-phase <fase-atual>`

**Rota 3: Fase tem contexto mas sem planos → plan**
Se a fase atual tem CONTEXT.md (ou RESEARCH.md) mas nenhum arquivo PLAN.md:
→ Próxima ação: `/gsd-plan-phase <fase-atual>`

**Rota 4: Fase tem planos mas resumos incompletos → execute**
Se existem planos mas nem todos têm resumos correspondentes:
→ Próxima ação: `/gsd-execute-phase <fase-atual>`

**Rota 5: Todos os planos têm resumos → verify e complete**
Se todos os planos na fase atual têm resumos:
→ Próxima ação: `/gsd-verify-work`

**Rota 6: Fase completa, próxima fase existe → advance**
Se a fase atual está completa e a próxima fase existe no ROADMAP:
→ Próxima ação: `/gsd-discuss-phase <próxima-fase>`

**Rota 7: Todas as fases completas → complete milestone**
Se todas as fases estão completas:
→ Próxima ação: `/gsd-complete-milestone`

**Rota 8: Pausado → resume**
Se STATE.md mostra paused_at:
→ Próxima ação: `/gsd-resume-work`
</step>

<step name="show_and_execute">
Exiba a determinação:

```
## GSD Next

**Atual:** Fase [N] — [name] | [progress]%
**Status:** [descrição do status]

▶ **Próximo passo:** `/gsd-[command] [args]`
  [Explicação de uma linha sobre por que este é o próximo passo]
```

Em seguida, invoque imediatamente o comando determinado via SlashCommand.
Não peça confirmação — o objetivo de `/gsd-next` é avançar sem fricção.
</step>

</process>

<success_criteria>
- [ ] Estado do projeto corretamente detectado
- [ ] Próxima ação corretamente determinada pelas regras de roteamento
- [ ] Comando invocado imediatamente sem confirmação do usuário
- [ ] Status claro exibido antes de invocar
</success_criteria>
</output>
