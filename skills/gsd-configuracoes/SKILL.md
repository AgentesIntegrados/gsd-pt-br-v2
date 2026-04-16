---
name: gsd-configuracoes
description: "Configurar toggles de fluxo de trabalho e perfil de modelo do GSD"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---


<objective>
Configuração interativa dos agentes de fluxo de trabalho e perfil de modelo do GSD via prompt de múltiplas perguntas.

Encaminha para o fluxo de trabalho settings que gerencia:
- Garantia de existência da configuração
- Leitura e análise das configurações atuais
- Prompt interativo com 5 perguntas (model, research, plan_check, verifier, branching)
- Mesclagem e escrita da configuração
- Exibição de confirmação com referências rápidas de comandos
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/settings.md
</execution_context>

<process>
**Seguir o fluxo de trabalho settings** de `@$HOME/.claude/get-shit-done/workflows/settings.md`.

O fluxo de trabalho gerencia toda a lógica, incluindo:
1. Criação do arquivo de configuração com padrões se ausente
2. Leitura da configuração atual
3. Apresentação interativa das configurações com pré-seleção
4. Análise das respostas e mesclagem da configuração
5. Escrita do arquivo
6. Exibição de confirmação
</process>
