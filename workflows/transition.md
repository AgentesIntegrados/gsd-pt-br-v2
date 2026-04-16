<internal_workflow>

**Este é um fluxo de trabalho INTERNO — NÃO é um comando voltado ao usuário.**

Não existe comando `/gsd-transition`. Este fluxo de trabalho é invocado automaticamente por
`execute-phase` durante o auto-avanço, ou inline pelo orquestrador após a verificação da fase.
Os usuários nunca devem ser instruídos a executar `/gsd-transition`.

**Comandos válidos para progressão de fase:**
- `/gsd-discuss-phase {N}` — discutir uma fase antes de planejar
- `/gsd-plan-phase {N}` — planejar uma fase
- `/gsd-execute-phase {N}` — executar uma fase
- `/gsd-progress` — ver progresso do roadmap

</internal_workflow>

<required_reading>

**Leia esses arquivos AGORA:**

1. `.planning/STATE.md`
2. `.planning/PROJECT.md`
3. `.planning/ROADMAP.md`
4. Arquivos de plano da fase atual (`*-PLAN.md`)
5. Arquivos de resumo da fase atual (`*-SUMMARY.md`)

</required_reading>

<purpose>

Marcar a fase atual como concluída e avançar para a próxima. Este é o ponto natural onde o rastreamento de progresso e a evolução do PROJECT.md acontecem.

"Planejar a próxima fase" = "a fase atual está concluída"

</purpose>

<process>

<step name="load_project_state" priority="first">

Antes da transição, ler o estado do projeto:

```bash
cat .planning/STATE.md 2>/dev/null || true
cat .planning/PROJECT.md 2>/dev/null || true
```

Analisar a posição atual para verificar que estamos transicionando a fase correta.
Anotar o contexto acumulado que pode precisar de atualização após a transição.

</step>

<step name="verify_completion">

Verificar que a fase atual tem todos os resumos de planos:

```bash
(ls .planning/phases/XX-current/*-PLAN.md 2>/dev/null || true) | sort
(ls .planning/phases/XX-current/*-SUMMARY.md 2>/dev/null || true) | sort
```

**Lógica de verificação:**

- Contar arquivos PLAN
- Contar arquivos SUMMARY
- Se as contagens corresponderem: todos os planos concluídos
- Se as contagens não corresponderem: incompleto

<config-check>

```bash
cat .planning/config.json 2>/dev/null || true
```

</config-check>

**Verificar dívida de verificação nesta fase:**

```bash
# Contar itens pendentes na fase atual
OUTSTANDING=""
for f in .planning/phases/XX-current/*-UAT.md .planning/phases/XX-current/*-VERIFICATION.md; do
  [ -f "$f" ] || continue
  grep -q "result: pending\|result: blocked\|status: partial\|status: human_needed\|status: diagnosed" "$f" && OUTSTANDING="$OUTSTANDING\n$(basename $f)"
done
```

**Se OUTSTANDING não estiver vazio:**

Adicionar à mensagem de confirmação de conclusão (independente do modo):

```
Itens de verificação pendentes nesta fase:
{lista de nomes de arquivos}

Estes serão carregados como dívida. Revise: `/gsd-audit-uat`
```

Isso NÃO bloqueia a transição — garante que o usuário veja a dívida antes de confirmar.

**Se todos os planos estiverem concluídos:**

<if mode="yolo">

```
⚡ Auto-aprovado: Transição Fase [X] → Fase [X+1]
Fase [X] concluída — todos os [Y] planos finalizados.

Prosseguindo para marcar como concluída e avançar...
```

Prosseguir diretamente para a etapa cleanup_handoff.

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

Perguntar: "Fase [X] concluída — todos os [Y] planos finalizados. Pronto para marcar como concluída e passar para a Fase [X+1]?"

Aguardar confirmação antes de prosseguir.

</if>

**Se os planos estiverem incompletos:**

**TRILHO DE SEGURANÇA: always_confirm_destructive se aplica aqui.**
Pular planos incompletos é destrutivo — SEMPRE solicitar independente do modo.

Apresentar:

```
A Fase [X] tem planos incompletos:
- {fase}-01-SUMMARY.md ✓ Concluído
- {fase}-02-SUMMARY.md ✗ Ausente
- {fase}-03-SUMMARY.md ✗ Ausente

⚠️ Trilho de segurança: Pular planos requer confirmação (ação destrutiva)

Opções:
1. Continuar a fase atual (executar os planos restantes)
2. Marcar como concluída assim mesmo (pular os planos restantes)
3. Revisar o que falta
```

Aguardar a decisão do usuário.

</step>

<step name="cleanup_handoff">

Verificar handoffs pendentes:

```bash
ls .planning/phases/XX-current/.continue-here*.md 2>/dev/null || true
```

Se encontrados, deletar — a fase está concluída, handoffs estão desatualizados.

</step>

<step name="update_roadmap_and_state">

**Delegar atualizações do ROADMAP.md e STATE.md para gsd-tools:**

```bash
TRANSITION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase complete "${current_phase}")
```

O CLI gerencia:
- Marcar o checkbox da fase como `[x]` concluída com a data de hoje
- Atualizar contagem de planos para o final (ex.: "3/3 planos concluídos")
- Atualizar a tabela de Progresso (Status → Concluído, adicionando data)
- Avançar STATE.md para a próxima fase (Fase Atual, Status → Pronto para planejar, Plano Atual → Não iniciado)
- Detectar se esta é a última fase do milestone

Extrair do resultado: `completed_phase`, `plans_executed`, `next_phase`, `next_phase_name`, `is_last_phase`.

</step>

<step name="evolve_project">

Evoluir PROJECT.md para refletir os aprendizados da fase concluída.

**Ler resumos da fase:**

```bash
cat .planning/phases/XX-current/*-SUMMARY.md
```

**Avaliar mudanças de requisitos:**

1. **Requisitos validados?**
   - Algum requisito Ativo foi entregue nesta fase?
   - Mover para Validado com referência de fase: `- ✓ [Requisito] — Fase X`

2. **Requisitos invalidados?**
   - Algum requisito Ativo descoberto como desnecessário ou incorreto?
   - Mover para Fora do Escopo com motivo: `- [Requisito] — [por que invalidado]`

3. **Requisitos emergentes?**
   - Novos requisitos descobertos durante a construção?
   - Adicionar ao Ativo: `- [ ] [Novo requisito]`

4. **Decisões a registrar?**
   - Extrair decisões dos arquivos SUMMARY.md
   - Adicionar à tabela de Decisões-Chave com resultado se conhecido

5. **"O Que É" ainda é preciso?**
   - Se o produto mudou significativamente, atualizar a descrição

**Atualizar PROJECT.md:**

Fazer as edições inline. Atualizar o rodapé "Última atualização":

```markdown
---
*Última atualização: [data] após a Fase [X]*
```

</step>

<step name="update_current_position_after_transition">

**Nota:** Atualizações básicas de posição (Fase Atual, Status, Plano Atual, Última Atividade) já foram feitas pelo `gsd-tools phase complete` na etapa update_roadmap_and_state.

Verificar se as atualizações estão corretas lendo STATE.md. Se a barra de progresso precisar ser atualizada, usar:

```bash
PROGRESS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" progress bar --raw)
```

Atualizar a linha da barra de progresso no STATE.md com o resultado.

</step>

<step name="offer_next_phase">

**OBRIGATÓRIO: Verificar o status do milestone antes de apresentar os próximos passos.**

**Usar o resultado da transição de `gsd-tools phase complete`:**

O campo `is_last_phase` do resultado de phase complete informa diretamente:
- `is_last_phase: false` → Mais fases restam → Ir para **Rota A**
- `is_last_phase: true` → Última fase concluída → **Verificar colisões de workstream primeiro**

---

**Rota A: Mais fases restam no milestone**

Ler ROADMAP.md para obter o nome e a meta da próxima fase.

<if mode="yolo">

```
Fase [X] marcada como concluída.

Próxima: Fase [X+1] — [Nome]

⚡ Auto-continuando: Planejar Fase [X+1] em detalhes
```

Sair da skill e invocar SlashCommand("/gsd-plan-phase [X+1] --auto ${GSD_WS}")

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

```
## ✓ Fase [X] Concluída

---

## ▶ A Seguir

**Fase [X+1]: [Nome]** — [Meta do ROADMAP.md]

`/clear` então:

`/gsd-discuss-phase [X+1] ${GSD_WS}` — reunir contexto e clarificar abordagem

---

**Também disponível:**
- `/gsd-plan-phase [X+1] ${GSD_WS}` — pular discussão, planejar diretamente
- `/gsd-research-phase [X+1] ${GSD_WS}` — investigar incógnitas

---
```

</if>

---

**Rota B: Milestone concluído (todas as fases finalizadas)**

<if mode="yolo">

```
Fase {X} marcada como concluída.

🎉 Milestone {version} está 100% concluído — todas as {N} fases finalizadas!

⚡ Auto-continuando: Concluir milestone e arquivar
```

Sair da skill e invocar SlashCommand("/gsd-complete-milestone {version} ${GSD_WS}")

</if>

<if mode="interactive" OR="custom with gates.confirm_transition true">

```
## ✓ Fase {X}: {Nome da Fase} Concluída

🎉 Milestone {version} está 100% concluído — todas as {N} fases finalizadas!

---

## ▶ A Seguir

**Concluir Milestone {version}** — arquivar e preparar para o próximo

`/clear` então:

`/gsd-complete-milestone {version} ${GSD_WS}`

---
```

</if>

</step>

</process>

<success_criteria>

A transição está concluída quando:

- [ ] Resumos de planos da fase atual verificados (todos existem ou usuário escolheu pular)
- [ ] Handoffs obsoletos deletados
- [ ] ROADMAP.md atualizado com status de conclusão e contagem de planos
- [ ] PROJECT.md evoluído (requisitos, decisões, descrição se necessário)
- [ ] STATE.md atualizado (posição, referência do projeto, contexto, sessão)
- [ ] Tabela de progresso atualizada
- [ ] Usuário sabe os próximos passos

</success_criteria>
