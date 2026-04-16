---
name: gsd-revisar-backlog
description: "Revisar e promover itens do backlog para o milestone ativo"
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---


<objective>
Revisar todos os itens do backlog 999.x e opcionalmente promovê-los para a
sequência ativa do milestone ou remover entradas obsoletas.
</objective>

<process>

1. **Listar itens do backlog:**
   ```bash
   ls -d .planning/phases/999* 2>/dev/null || echo "No backlog items found"
   ```

2. **Ler ROADMAP.md** e extrair todas as entradas de fase 999.x:
   ```bash
   cat .planning/ROADMAP.md
   ```
   Mostrar cada item do backlog com sua descrição, qualquer contexto acumulado (CONTEXT.md, RESEARCH.md) e data de criação.

3. **Apresentar a lista ao usuário** via AskUserQuestion:
   - Para cada item do backlog, mostrar: número da fase, descrição, artefatos acumulados
   - Opções por item: **Promover** (mover para ativo), **Manter** (deixar no backlog), **Remover** (excluir)

4. **Para itens a PROMOVER:**
   - Encontrar o próximo número de fase sequencial no milestone ativo
   - Renomear o diretório de `999.x-slug` para `{new_num}-slug`:
     ```bash
     NEW_NUM=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase add "${DESCRIPTION}" --raw)
     ```
   - Mover artefatos acumulados para o novo diretório de fase
   - Atualizar ROADMAP.md: mover a entrada da seção `## Backlog` para a lista de fases ativas
   - Remover o marcador `(BACKLOG)`
   - Adicionar campo `**Depends on:**` apropriado

5. **Para itens a REMOVER:**
   - Excluir o diretório da fase
   - Remover a entrada da seção `## Backlog` do ROADMAP.md

6. **Confirmar alterações:**
   ```bash
   node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: review backlog — promoted N, removed M" --files .planning/ROADMAP.md
   ```

7. **Relatar resumo:**
   ```
   ## Revisão do Backlog Concluída

   Promovidos: {lista de itens promovidos com novos números de fase}
   Mantidos: {lista de itens que permanecem no backlog}
   Removidos: {lista de itens excluídos}
   ```

</process>
