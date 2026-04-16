---
name: gsd-perfil-usuario
description: "Gerar perfil comportamental do desenvolvedor e criar artefatos descobríveis pelo Claude"
argument-hint: "[--questionnaire] [--refresh]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Task
---


<objective>
Gerar um perfil comportamental do desenvolvedor a partir da análise de sessões (ou questionário) e produzir artefatos (USER-PROFILE.md, /gsd-dev-preferences, seção CLAUDE.md) que personalizam as respostas do Claude.

Encaminha para o workflow profile-user que orquestra o fluxo completo: verificação de consentimento, análise de sessões ou fallback para questionário, geração do perfil, exibição dos resultados e seleção de artefatos.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/profile-user.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Flags de $ARGUMENTS:
- `--questionnaire` -- Pular análise de sessões completamente, usar apenas o caminho do questionário
- `--refresh` -- Reconstruir perfil mesmo quando já existe, fazer backup do perfil antigo, exibir diff de dimensões
</context>

<process>
Executar o workflow profile-user do início ao fim.

O workflow trata de toda a lógica incluindo:
1. Inicialização e detecção de perfil existente
2. Verificação de consentimento antes da análise de sessões
3. Varredura de sessões e verificações de suficiência de dados
4. Análise de sessões (agente profiler) ou fallback para questionário
5. Resolução de divisão entre projetos
6. Escrita do perfil em USER-PROFILE.md
7. Exibição dos resultados com cartão de relatório e destaques
8. Seleção de artefatos (dev-preferences, seções CLAUDE.md)
9. Geração sequencial de artefatos
10. Resumo com diff de atualização (se aplicável)
</process>
</content>
