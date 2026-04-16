---
name: gsd-gerenciador
description: "Central de comando interativa para gerenciar múltiplas fases em um único terminal"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Skill
  - Task
---

<objective>
Central de comando em terminal único para gerenciar um milestone. Exibe um painel com todas as fases e indicadores visuais de status, recomenda as próximas ações ideais e despacha trabalho — discussões rodam inline, planejamento/execução rodam como agentes em segundo plano.

Projetado para usuários avançados que querem paralelizar trabalho entre fases em um único terminal: discutir uma fase enquanto outra planeja ou executa em segundo plano.

**Cria/Atualiza:**
- Nenhum arquivo criado diretamente — despacha para comandos GSD existentes via Skill() e agentes Task em segundo plano.
- Lê `.planning/STATE.md`, `.planning/ROADMAP.md`, diretórios de fase para status.

**Após:** O usuário sai quando terminar de gerenciar, ou todas as fases concluem e o ciclo de vida do milestone é sugerido.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/manager.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Nenhum argumento necessário. Requer um milestone ativo com ROADMAP.md e STATE.md.

Contexto do projeto, lista de fases, dependências e recomendações são resolvidos internamente pelo workflow usando `gsd-tools.cjs init manager`. Não é necessário carregar contexto antecipadamente.
</context>

<process>
Executar o workflow manager de @$HOME/.claude/get-shit-done/workflows/manager.md do início ao fim.
Manter o loop de atualização do painel até que o usuário saia ou todas as fases sejam concluídas.
</process>
</content>
