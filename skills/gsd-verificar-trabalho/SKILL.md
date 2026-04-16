---
name: gsd-verificar-trabalho
description: "Validar funcionalidades construídas por meio de UAT conversacional"
argument-hint: "[phase number, e.g., '4']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Edit
  - Write
  - Task
---

<objective>
Validar funcionalidades construídas por meio de testes conversacionais com estado persistente.

Propósito: Confirmar que o que o Claude construiu realmente funciona do ponto de vista do usuário. Um teste por vez, respostas em texto simples, sem interrogatório. Quando problemas são encontrados, diagnosticar automaticamente, planejar correções e preparar para execução.

Resultado: {phase_num}-UAT.md rastreando todos os resultados de teste. Se problemas forem encontrados: lacunas diagnosticadas, planos de correção verificados prontos para /gsd-execute-phase
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/verify-work.md
@$HOME/.claude/get-shit-done/templates/UAT.md
</execution_context>

<context>
Fase: $ARGUMENTS (opcional)
- Se fornecido: Testar fase específica (ex.: "4")
- Se não fornecido: Verificar sessões ativas ou solicitar a fase

Os arquivos de contexto são resolvidos dentro do fluxo de trabalho (`init verify-work`) e delegados via blocos `<files_to_read>`.
</context>

<process>
Executar o fluxo de trabalho verify-work de @$HOME/.claude/get-shit-done/workflows/verify-work.md do início ao fim.
Preservar todos os portões do fluxo de trabalho (gerenciamento de sessão, apresentação de testes, diagnóstico, planejamento de correções, roteamento).
</process>
