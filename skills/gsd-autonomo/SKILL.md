---
name: gsd-autonomo
description: "Executar todas as fases restantes de forma autônoma — discuss→plan→execute por fase"
argument-hint: "[--from N] [--to N] [--only N] [--interactive]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
  - Agent
---

<objective>
Executar todas as fases restantes do milestone de forma autônoma. Para cada fase: discuss → plan → execute. Pausa apenas para decisões do usuário (aceitação de áreas cinzas, bloqueadores, solicitações de validação).

Usa a descoberta de fases do ROADMAP.md e invocações planas de Skill() para cada comando de fase. Após a conclusão de todas as fases: auditoria do milestone → conclusão → limpeza.

**Cria/Atualiza:**
- `.planning/STATE.md` — atualizado após cada fase
- `.planning/ROADMAP.md` — progresso atualizado após cada fase
- Artefatos de fase — CONTEXT.md, PLANs, SUMMARYs por fase

**Após:** O milestone está completo e limpo.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/autonomous.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Flags opcionais:
- `--from N` — iniciar a partir da fase N em vez da primeira fase incompleta.
- `--to N` — parar após a conclusão da fase N (interromper em vez de avançar para a próxima fase).
- `--only N` — executar apenas a fase N (modo de fase única).
- `--interactive` — executar discuss de forma interativa com perguntas (não respondidas automaticamente) e, em seguida, despachar plan→execute como agentes em segundo plano. Mantém o contexto principal enxuto enquanto preserva a entrada do usuário nas decisões.

O contexto do projeto, a lista de fases e o estado são resolvidos dentro do fluxo usando comandos init (`gsd-tools.cjs init milestone-op`, `gsd-tools.cjs roadmap analyze`). Não é necessário carregar contexto antecipadamente.
</context>

<process>
Executar o fluxo autonomous de @$HOME/.claude/get-shit-done/workflows/autonomous.md do início ao fim.
Preservar todos os gates do fluxo (descoberta de fases, execução por fase, tratamento de bloqueadores, exibição de progresso).
</process>
