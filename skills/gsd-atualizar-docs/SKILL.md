---
name: gsd-atualizar-docs
description: "Gera ou atualiza a documentação do projeto verificada contra o código-fonte"
argument-hint: "[--force] [--verify-only]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

<objective>
Gera e atualiza até 9 arquivos de documentação para o projeto atual. Cada tipo de documento é escrito por um subagente gsd-doc-writer que explora o código-fonte diretamente — sem caminhos inventados, endpoints fantasma ou assinaturas desatualizadas.

Regra de uso de flags:
- As flags opcionais documentadas abaixo são comportamentos disponíveis, não comportamentos ativos implícitos
- Uma flag só está ativa quando seu token literal aparece em `$ARGUMENTS`
- Se uma flag documentada estiver ausente de `$ARGUMENTS`, trate-a como inativa
- `--force`: ignora prompts de preservação, regenera todos os documentos independentemente do conteúdo existente ou marcadores GSD
- `--verify-only`: verifica a precisão dos documentos existentes contra o código-fonte, sem geração (verificação completa requer o verificador da Fase 4)
- Se `--force` e `--verify-only` ambos aparecerem em `$ARGUMENTS`, `--force` tem precedência
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/docs-update.md
</execution_context>

<context>
Argumentos: $ARGUMENTS

**Flags opcionais disponíveis (apenas documentação — não ficam ativas automaticamente):**
- `--force` — Regenera todos os documentos. Sobrescreve documentos manuais e GSD. Sem prompts de preservação.
- `--verify-only` — Verifica a precisão dos documentos existentes contra o código-fonte. Nenhum arquivo é escrito. Reporta a contagem de marcadores VERIFY. A verificação completa do código-fonte requer o agente gsd-doc-verifier (Fase 4).

**As flags ativas devem ser derivadas de `$ARGUMENTS`:**
- `--force` só está ativa se o token literal `--force` estiver presente em `$ARGUMENTS`
- `--verify-only` só está ativa se o token literal `--verify-only` estiver presente em `$ARGUMENTS`
- Se nenhum token aparecer, execute o fluxo padrão de geração completa por fases
- Não infira que uma flag está ativa apenas porque está documentada neste prompt
</context>

<process>
Execute o fluxo docs-update a partir de @$HOME/.claude/get-shit-done/workflows/docs-update.md do início ao fim.
Preserve todos os portões do fluxo (verification_check, tratamento de flags, execução em ondas, despacho para monorepo, commit, relatório).
</process>
</output>
