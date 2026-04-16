---
name: gsd-pausar
description: "Criar handoff de contexto ao pausar trabalho no meio de uma fase"
allowed-tools:
  - Read
  - Write
  - Bash
---


<objective>
Criar o arquivo de handoff `.continue-here.md` para preservar o estado completo do trabalho entre sessões.

Encaminha para o workflow pause-work que trata de:
- Detecção da fase atual a partir de arquivos recentes
- Coleta completa do estado (posição, trabalho concluído, trabalho restante, decisões, bloqueadores)
- Criação do arquivo de handoff com todas as seções de contexto
- Commit git como WIP
- Instruções para retomada
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/pause-work.md
</execution_context>

<context>
Estado e progresso da fase são coletados dentro do workflow com leituras direcionadas.
</context>

<process>
**Seguir o workflow pause-work** de `@$HOME/.claude/get-shit-done/workflows/pause-work.md`.

O workflow trata de toda a lógica incluindo:
1. Detecção do diretório de fase
2. Coleta do estado com esclarecimentos do usuário
3. Escrita do arquivo de handoff com timestamp
4. Commit git
5. Confirmação com instruções de retomada
</process>
</content>
