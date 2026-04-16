---
name: gsd-adicionar-backlog
description: "Adicionar uma ideia ao backlog (numeração 999.x)"
argument-hint: "<description>"
allowed-tools:
  - Read
  - Write
  - Bash
---


<objective>
Adicionar um item ao backlog do roadmap usando numeração 999.x. Itens de backlog são
ideias sem sequência definida que ainda não estão prontas para planejamento ativo — ficam fora
da sequência normal de fases e acumulam contexto ao longo do tempo.
</objective>

<process>

1. **Ler ROADMAP.md** para encontrar entradas de backlog existentes:
   ```bash
   cat .planning/ROADMAP.md
   ```

2. **Encontrar o próximo número de backlog:**
   ```bash
   NEXT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase next-decimal 999 --raw)
   ```
   Se não existirem fases 999.x, começar em 999.1.

3. **Criar o diretório da fase:**
   ```bash
   SLUG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" generate-slug "$ARGUMENTS" --raw)
   mkdir -p ".planning/phases/${NEXT}-${SLUG}"
   touch ".planning/phases/${NEXT}-${SLUG}/.gitkeep"
   ```

4. **Adicionar ao ROADMAP.md** em uma seção `## Backlog`. Se a seção não existir, criá-la no final:

   ```markdown
   ## Backlog

   ### Fase {NEXT}: {description} (BACKLOG)

   **Objetivo:** [Registrado para planejamento futuro]
   **Requisitos:** A definir
   **Planos:** 0 planos

   Planos:
   - [ ] A definir (promover com /gsd-review-backlog quando pronto)
   ```

5. **Commitar:**
   ```bash
   node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: add backlog item ${NEXT} — ${ARGUMENTS}" --files .planning/ROADMAP.md ".planning/phases/${NEXT}-${SLUG}/.gitkeep"
   ```

6. **Relatório:**
   ```
   ## 📋 Item de Backlog Adicionado

   Fase {NEXT}: {description}
   Diretório: .planning/phases/{NEXT}-{slug}/

   Este item está no backlog.
   Use /gsd-discuss-phase {NEXT} para explorá-lo melhor.
   Use /gsd-review-backlog para promover itens ao milestone ativo.
   ```

</process>

<notes>
- A numeração 999.x mantém os itens de backlog fora da sequência ativa de fases
- Os diretórios de fase são criados imediatamente, para que /gsd-discuss-phase e /gsd-plan-phase funcionem neles
- Sem campo `Depends on:` — itens de backlog são sem sequência por definição
- Numeração esparsa é permitida (999.1, 999.3) — sempre usa next-decimal
</notes>
