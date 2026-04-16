---
name: gsd-discutir-fase
description: "Coletar contexto da fase através de perguntas adaptativas antes do planejamento. Use --auto para pular perguntas interativas (Claude escolhe os padrões recomendados). Use --chain para discuss interativo seguido de plan+execute automáticos. Use --power para geração em massa de perguntas em um arquivo para resposta no seu ritmo."
argument-hint: "<phase> [--auto] [--chain] [--batch] [--analyze] [--text] [--power]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---


<objective>
Extrair decisões de implementação que os agentes de downstream precisam — o pesquisador e o planejador usarão CONTEXT.md para saber o que investigar e quais escolhas estão bloqueadas.

**Como funciona:**
1. Carregar contexto anterior (PROJECT.md, REQUIREMENTS.md, STATE.md, arquivos CONTEXT.md anteriores)
2. Explorar a base de código em busca de ativos e padrões reutilizáveis
3. Analisar a fase — ignorar áreas cinzas já decididas em fases anteriores
4. Apresentar áreas cinzas restantes — o usuário escolhe quais discutir
5. Aprofundar cada área selecionada até estar satisfeito
6. Criar CONTEXT.md com decisões que guiam a pesquisa e o planejamento

**Resultado:** `{phase_num}-CONTEXT.md` — decisões claras o suficiente para que os agentes de downstream possam agir sem precisar perguntar ao usuário novamente
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/discuss-phase.md
@$HOME/.claude/get-shit-done/workflows/discuss-phase-assumptions.md
@$HOME/.claude/get-shit-done/workflows/discuss-phase-power.md
@$HOME/.claude/get-shit-done/templates/context.md
</execution_context>

<runtime_note>
**Copilot (VS Code):** Use `vscode_askquestions` wherever this workflow calls `AskUserQuestion`. They are equivalent — `vscode_askquestions` is the VS Code Copilot implementation of the same interactive question API.
</runtime_note>

<context>
Número da fase: $ARGUMENTS (obrigatório)

Os arquivos de contexto são resolvidos no fluxo usando `init phase-op` e chamadas de ferramentas de roadmap/estado.
</context>

<process>
**Roteamento de modo:**
```bash
DISCUSS_MODE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.discuss_mode 2>/dev/null || echo "discuss")
```

Se `DISCUSS_MODE` for `"assumptions"`: Ler e executar @$HOME/.claude/get-shit-done/workflows/discuss-phase-assumptions.md do início ao fim.

Se `DISCUSS_MODE` for `"discuss"` (ou não definido, ou qualquer outro valor): Ler e executar @$HOME/.claude/get-shit-done/workflows/discuss-phase.md do início ao fim.

**OBRIGATÓRIO:** Os arquivos de execution_context listados acima SÃO as instruções. Ler o arquivo de fluxo ANTES de qualquer ação. As seções de objetivo e success_criteria neste arquivo de comando são resumos — o arquivo de fluxo contém o processo passo a passo completo com todos os comportamentos necessários, verificações de configuração e padrões de interação. Não improvisar a partir do resumo.
</process>

<success_criteria>
- Contexto anterior carregado e aplicado (sem re-perguntar sobre questões já decididas)
- Áreas cinzas identificadas por análise inteligente
- Usuário escolheu quais áreas discutir
- Cada área selecionada explorada até a satisfação
- Desvios de escopo redirecionados para ideias adiadas
- CONTEXT.md registra decisões, não visão vaga
- Usuário conhece os próximos passos
</success_criteria>
