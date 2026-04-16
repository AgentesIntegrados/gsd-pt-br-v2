---
name: gsd-importar-gsd2
description: "Importa um projeto GSD-2 (.gsd/) de volta ao formato GSD v1 (.planning/)"
argument-hint: "[--path <dir>] [--force]"
allowed-tools:
  - Read
  - Write
  - Bash
---


<objective>
Migra reversamente um projeto GSD-2 (diretório `.gsd/`) de volta ao formato GSD v1 (`.planning/`).

Mapeia a hierarquia GSD-2 (Marco → Fatia → Tarefa) para a hierarquia GSD v1 (seções de Marco no ROADMAP.md → Fase → Plano), preservando o estado de conclusão, arquivos de pesquisa e resumos.
</objective>

<process>

1. **Localize o diretório .gsd/** — verifique o diretório de trabalho atual (ou o argumento `--path`):
   ```bash
   node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" from-gsd2 --dry-run
   ```
   Se nenhum `.gsd/` for encontrado, reporte o erro e pare.

2. **Exiba o preview do dry-run** — apresente a lista completa de arquivos e as estatísticas de migração ao usuário. Peça confirmação antes de escrever qualquer coisa.

3. **Execute a migração** após a confirmação:
   ```bash
   node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" from-gsd2
   ```
   Use `--force` se `.planning/` já existir e o usuário tiver confirmado a sobrescrita.

4. **Reporte o resultado** — exiba a contagem de `filesWritten`, o caminho `planningDir` e o resumo do preview.

</process>

<notes>
- A migração não é destrutiva: `.gsd/` nunca é modificado ou removido.
- Passe `--path <dir>` para migrar um projeto em um caminho diferente do diretório atual.
- As fatias são numeradas sequencialmente em todos os marcos (M001/S01 → fase 01, M001/S02 → fase 02, M002/S01 → fase 03, etc.).
- As tarefas dentro de cada fatia tornam-se planos (T01 → plano 01, T02 → plano 02, etc.).
- Fatias e tarefas concluídas mantêm seu estado de conclusão nos checkboxes do ROADMAP.md e nos arquivos SUMMARY.md.
- O ledger de custo/token do GSD-2, o estado do banco de dados e o estado da extensão do VS Code não podem ser migrados.
</notes>
