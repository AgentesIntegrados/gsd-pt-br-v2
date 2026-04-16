---
name: gsd-revisao-codigo
description: "Revisar arquivos-fonte alterados durante uma fase em busca de bugs, problemas de segurança e qualidade de código"
argument-hint: "<phase-number> [--depth=quick|standard|deep] [--files file1,file2,...]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---

<objective>
Revisar arquivos-fonte alterados durante uma fase em busca de bugs, vulnerabilidades de segurança e problemas de qualidade de código.

Dispara o agente gsd-code-reviewer para analisar o código no nível de profundidade especificado. Produz o artefato REVIEW.md no diretório da fase com achados classificados por severidade.

Argumentos:
- Número da fase (obrigatório) — qual fase revisar (ex.: "2" ou "02")
- `--depth=quick|standard|deep` (opcional) — nível de profundidade da revisão, substitui a configuração workflow.code_review_depth
  - quick: Apenas correspondência de padrões (~2 min)
  - standard: Análise por arquivo com verificações específicas por linguagem (~5-15 min, padrão)
  - deep: Análise entre arquivos incluindo grafos de importação e cadeias de chamada (~15-30 min)
- `--files file1,file2,...` (opcional) — lista explícita de arquivos separados por vírgula, ignora escopo por SUMMARY/git (precedência mais alta para escopo)

Resultado: {padded_phase}-REVIEW.md no diretório da fase + resumo inline dos achados
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/code-review.md
</execution_context>

<context>
Phase: $ARGUMENTS (primeiro argumento posicional é o número da fase)

Flags opcionais extraídas de $ARGUMENTS:
- `--depth=VALUE` — Substituição de profundidade (quick|standard|deep). Se fornecido, substitui a configuração workflow.code_review_depth.
- `--files=file1,file2,...` — Substituição explícita de lista de arquivos. Tem a precedência mais alta para escopo de arquivos. Quando fornecido, o fluxo ignora completamente a extração de SUMMARY.md e o fallback de git diff.

Os arquivos de contexto (CLAUDE.md, SUMMARY.md, estado da fase) são resolvidos dentro do fluxo via `gsd-tools init phase-op` e delegados ao agente via blocos `<files_to_read>`.
</context>

<process>
Este comando é uma camada fina de despacho. Analisa os argumentos e delega ao fluxo.

Executar o fluxo code-review de @$HOME/.claude/get-shit-done/workflows/code-review.md do início ao fim.

O fluxo (não este comando) aplica estes gates:
- Validação de fase (antes do gate de configuração)
- Verificação do gate de configuração (workflow.code_review)
- Escopo de arquivos (substituição --files > SUMMARY.md > fallback git diff)
- Verificação de escopo vazio (ignorar se sem arquivos)
- Disparo de agente (gsd-code-reviewer)
- Apresentação de resultados (resumo inline + próximos passos)
</process>
