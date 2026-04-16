---
name: gsd-auditoria-milestone
description: "Auditar a conclusão do milestone em relação à intenção original antes de arquivar"
argument-hint: "[version]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Task
  - Write
---

<objective>
Verificar se o milestone atingiu sua definição de pronto. Checar cobertura de requisitos, integração entre fases e fluxos de ponta a ponta.

**Este comando É o orquestrador.** Lê os arquivos VERIFICATION.md existentes (fases já verificadas durante execute-phase), agrega dívida técnica e lacunas adiadas, e então dispara o verificador de integração para checar a conexão entre fases.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/audit-milestone.md
</execution_context>

<context>
Version: $ARGUMENTS (opcional — padrão é o milestone atual)

Os arquivos centrais de planejamento são resolvidos no fluxo (`init milestone-op`) e carregados apenas quando necessário.

**Trabalho Concluído:**
Glob: .planning/phases/*/*-SUMMARY.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>

<process>
Executar o fluxo audit-milestone de @$HOME/.claude/get-shit-done/workflows/audit-milestone.md do início ao fim.
Preservar todos os gates do fluxo (determinação de escopo, leitura de verificação, checagem de integração, cobertura de requisitos, roteamento).
</process>
