---
name: gsd-proximo
description: "Avançar automaticamente para o próximo passo lógico no workflow GSD"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - SlashCommand
---

<objective>
Detectar o estado atual do projeto e invocar automaticamente o próximo passo lógico do workflow GSD.
Nenhum argumento necessário — lê STATE.md, ROADMAP.md e diretórios de fase para determinar o que vem a seguir.

Projetado para workflows multi-projeto rápidos onde lembrar em qual fase/etapa você está é uma sobrecarga.

Suporta a flag `--force` para contornar pontos de controle de segurança (checkpoint, estado de erro, falhas de verificação e varredura de completude de fases anteriores).

Antes de encaminhar para o próximo passo, varre todas as fases anteriores em busca de trabalho incompleto: planos que rodaram sem produzir resumos, falhas de verificação sem override e fases onde houve discussão mas o planejamento nunca foi executado. Quando trabalho incompleto é encontrado, exibe um relatório estruturado e oferece três opções: adiar as lacunas para o backlog e continuar, parar e resolver manualmente, ou forçar o avanço sem registrar. Quando as fases anteriores estão limpas, encaminha silenciosamente sem interrupção.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/next.md
</execution_context>

<process>
Executar o workflow next de @$HOME/.claude/get-shit-done/workflows/next.md do início ao fim.
</process>
</content>
