---
name: gsd-fazer
description: "Encaminha texto livre para o comando GSD correto automaticamente"
argument-hint: "<descrição do que você quer fazer>"
allowed-tools:
  - Read
  - Bash
  - AskUserQuestion
---

<objective>
Analisa a entrada em linguagem natural e despacha para o comando GSD mais adequado.

Funciona como um despachante inteligente — nunca executa o trabalho diretamente. Identifica a intenção e a corresponde ao melhor comando GSD usando regras de roteamento, confirma a correspondência e repassa a execução.

Use quando você sabe o que quer fazer, mas não sabe qual comando `/gsd-*` executar.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/do.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
Execute o fluxo "do" a partir de @$HOME/.claude/get-shit-done/workflows/do.md do início ao fim.
Encaminhe a intenção do usuário para o melhor comando GSD e invoque-o.
</process>
</output>
