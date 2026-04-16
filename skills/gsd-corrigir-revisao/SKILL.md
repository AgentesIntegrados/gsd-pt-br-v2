---
name: gsd-corrigir-revisao
description: "Corrigir automaticamente problemas encontrados pela revisão de código no REVIEW.md. Dispara agente corretor, commita cada correção atomicamente e produz resumo em REVIEW-FIX.md."
argument-hint: "<phase-number> [--all] [--auto]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Edit
  - Task
---

<objective>
Corrigir automaticamente problemas encontrados pela revisão de código. Lê o REVIEW.md da fase especificada, dispara o agente gsd-code-fixer para aplicar as correções e produz o resumo em REVIEW-FIX.md.

Argumentos:
- Número da fase (obrigatório) — qual REVIEW.md corrigir (ex.: "2" ou "02")
- `--all` (opcional) — incluir achados de nível Info no escopo de correção (padrão: apenas Critical + Warning)
- `--auto` (opcional) — habilitar loop de iteração correção + revisão, limitado a 3 iterações

Resultado: {padded_phase}-REVIEW-FIX.md no diretório da fase + resumo inline das correções aplicadas
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/code-review-fix.md
</execution_context>

<context>
Phase: $ARGUMENTS (primeiro argumento posicional é o número da fase)

Flags opcionais extraídas de $ARGUMENTS:
- `--all` — Incluir achados de nível Info no escopo de correção. O comportamento padrão corrige apenas Critical + Warning.
- `--auto` — Habilitar loop de iteração correção + revisão. Após aplicar as correções, re-executar code-review na mesma profundidade. Se novos problemas forem encontrados, iterar. Limitar a 3 iterações no total. Sem esta flag, apenas uma passagem de correção.

Os arquivos de contexto (CLAUDE.md, REVIEW.md, estado da fase) são resolvidos dentro do fluxo via `gsd-tools init phase-op` e delegados ao agente via blocos de configuração.
</context>

<process>
Este comando é uma camada fina de despacho. Analisa os argumentos e delega ao fluxo.

Executar o fluxo code-review-fix de @$HOME/.claude/get-shit-done/workflows/code-review-fix.md do início ao fim.

O fluxo (não este comando) aplica estes gates:
- Validação de fase (antes do gate de configuração)
- Verificação do gate de configuração (workflow.code_review)
- Verificação de existência do REVIEW.md (erro se ausente)
- Verificação de status do REVIEW.md (ignorar se limpo/ignorado)
- Disparo de agente (gsd-code-fixer)
- Loop de iteração (se --auto, limitado a 3 iterações)
- Apresentação de resultados (resumo inline + próximos passos)
</process>
