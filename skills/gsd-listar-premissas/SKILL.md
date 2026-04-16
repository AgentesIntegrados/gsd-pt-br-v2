---
name: gsd-listar-premissas
description: "Expõe as suposições do Claude sobre a abordagem de uma fase antes do planejamento"
argument-hint: "[fase]"
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
---


<objective>
Analisa uma fase e apresenta as suposições do Claude sobre abordagem técnica, ordem de implementação, limites de escopo, áreas de risco e dependências.

Propósito: Ajudar o usuário a ver o que o Claude pensa ANTES de o planejamento começar — permitindo correções de curso quando as suposições estiverem erradas.
Saída: Apenas saída conversacional (sem criação de arquivo) — termina com o prompt "O que você acha?"
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/list-phase-assumptions.md
</execution_context>

<context>
Número da fase: $ARGUMENTS (obrigatório)

O estado do projeto e o roadmap são carregados dentro do fluxo usando leituras específicas.
</context>

<process>
1. Valide o argumento de número de fase (erro se ausente ou inválido)
2. Verifique se a fase existe no roadmap
3. Siga o fluxo list-phase-assumptions.md:
   - Analise a descrição do roadmap
   - Exponha suposições sobre: abordagem técnica, ordem de implementação, escopo, riscos, dependências
   - Apresente as suposições de forma clara
   - Promova "O que você acha?"
4. Colete feedback e ofereça os próximos passos
</process>

<success_criteria>

- Fase validada contra o roadmap
- Suposições expostas em cinco áreas
- Usuário solicitado a dar feedback
- Usuário sabe os próximos passos (discutir contexto, planejar fase ou corrigir suposições)
  </success_criteria>
