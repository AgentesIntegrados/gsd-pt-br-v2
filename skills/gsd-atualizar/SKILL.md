---
name: gsd-atualizar
description: "Atualizar o GSD para a versão mais recente com exibição do changelog"
allowed-tools:
  - Bash
  - AskUserQuestion
---


<objective>
Verificar atualizações do GSD, instalar se disponível e exibir o que mudou.

Encaminha para o fluxo de trabalho update que gerencia:
- Detecção de versão (instalação local vs global)
- Verificação de versão via npm
- Busca e exibição do changelog
- Confirmação do usuário com aviso de instalação limpa
- Execução da atualização e limpeza de cache
- Lembrete de reinicialização
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/update.md
</execution_context>

<process>
**Seguir o fluxo de trabalho update** de `@$HOME/.claude/get-shit-done/workflows/update.md`.

O fluxo de trabalho gerencia toda a lógica, incluindo:
1. Detecção da versão instalada (local/global)
2. Verificação da versão mais recente via npm
3. Comparação de versões
4. Busca e extração do changelog
5. Exibição do aviso de instalação limpa
6. Confirmação do usuário
7. Execução da atualização
8. Limpeza do cache
</process>
