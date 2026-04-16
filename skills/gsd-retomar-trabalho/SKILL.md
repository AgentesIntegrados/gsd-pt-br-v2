---
name: gsd-retomar-trabalho
description: "Retomar o trabalho da sessão anterior com restauração completa do contexto"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
  - SlashCommand
---


<objective>
Restaurar o contexto completo do projeto e retomar o trabalho de forma contínua a partir da sessão anterior.

Encaminha para o fluxo de trabalho resume-project que gerencia:

- Carregamento do STATE.md (ou reconstrução se ausente)
- Detecção de checkpoints (arquivos .continue-here)
- Detecção de trabalho incompleto (PLAN sem SUMMARY)
- Apresentação de status
- Roteamento para próxima ação com base no contexto
  </objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/resume-project.md
</execution_context>

<process>
**Seguir o fluxo de trabalho resume-project** de `@$HOME/.claude/get-shit-done/workflows/resume-project.md`.

O fluxo de trabalho gerencia toda a lógica de retomada, incluindo:

1. Verificação de existência do projeto
2. Carregamento ou reconstrução do STATE.md
3. Detecção de checkpoints e trabalho incompleto
4. Apresentação visual do status
5. Oferta de opções com base no contexto (verifica CONTEXT.md antes de sugerir plan vs discuss)
6. Encaminhamento para o próximo comando apropriado
7. Atualizações de continuidade da sessão
   </process>
