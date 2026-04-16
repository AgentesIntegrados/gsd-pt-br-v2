<purpose>
Listar todos os workspaces GSD disponíveis no diretório atual. Exibir status, progresso e localização de cada workspace.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="scan_workspaces">
Verificar workspaces GSD disponíveis:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" workspaces list --json 2>/dev/null
```

Se o comando falhar ou nenhum workspace encontrado:
```bash
# Fallback: verificar manualmente por diretórios .planning
find . -maxdepth 3 -name "STATE.md" -path "*/.planning/*" 2>/dev/null | head -20
```
</step>

<step name="display_workspaces">
Exibir resultados:

**Se nenhum workspace encontrado:**
```
Nenhum workspace GSD encontrado neste diretório.

Para criar um workspace:
  /gsd-new-workspace "{nome}" — criar novo workspace

Para inicializar um projeto neste diretório:
  /gsd-new-project
```

**Se um workspace encontrado (sem modo workspace):**
```
Workspace GSD atual:

Localização: {caminho}
Milestone:   {versão} — {nome}
Fase atual:  Fase {N} — {nome}
Status:      {status}
Progresso:   [{barra de progresso}] {N}/{M} fases
```

**Se múltiplos workspaces encontrados:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► WORKSPACES DISPONÍVEIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| # | Nome | Milestone | Fase Atual | Progresso | Status |
|---|------|-----------|------------|-----------|--------|
{para cada workspace}

Para alternar para um workspace:
  /gsd-new-workspace --switch "{nome}"

Para ver detalhes de um workspace:
  /gsd-progress --workspace "{nome}"
```
</step>

<step name="show_details_if_requested">
Se `--details` em $ARGUMENTS, para cada workspace exibir:

```
## Workspace: {nome}
Caminho: {caminho}
Milestone: {versão} — {nome}
Fases:
  {lista de fases com status}
Última atividade: {data}
Branches git: {branches relacionados}
```
</step>

</process>

<success_criteria>
- [ ] Busca de workspaces executada
- [ ] Lista de workspaces exibida com detalhes de status
- [ ] Próximas ações sugeridas para cada cenário
</success_criteria>
