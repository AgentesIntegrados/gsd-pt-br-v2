<purpose>
Verificar o estado do projeto e rotear para a próxima ação. Lê STATE.md e ROADMAP.md para determinar onde o projeto está e o que vem a seguir.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="load_state">
Carregar estado completo do projeto:

```bash
PROGRESS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" progress full --json)
if [[ "$PROGRESS" == @file:* ]]; then PROGRESS=$(cat "${PROGRESS#@file:}"); fi
```

Extrair: `milestone_version`, `milestone_name`, `current_phase`, `current_phase_name`, `status`, `phases`, `next_action`, `progress_percent`.

Se nenhum projeto encontrado:
```
Nenhum projeto GSD encontrado neste diretório.

Para começar:
  /gsd-new-project — inicializar novo projeto
  /gsd-resume-project — retomar projeto existente
```
Sair.
</step>

<step name="display_progress">
Exibir progresso atual:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PROGRESSO — {milestone_version} {milestone_name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[{barra_de_progresso}] {progress_percent}% — Fase {current_phase}: {current_phase_name}

## Fases

{Para cada fase:}
{✓/→/○} Fase {N}: {nome}
   {Se concluída: "Concluída — {N} planos"}
   {Se atual: "Em andamento — {status}"}
   {Se futura: "Pendente"}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="determine_next_action">
Determinar e apresentar próxima ação com base no status:

**Status: "Pronto para discutir":**
```
## ▶ Próxima Ação

**Discutir Fase {N}** — capturar contexto e decisões

`/clear` então: `/gsd-discuss-phase {N}`

Ou pule a discussão: `/gsd-plan-phase {N}`
```

**Status: "Pronto para planejar":**
```
## ▶ Próxima Ação

**Planejar Fase {N}: {nome}** — criar planos detalhados de execução

`/clear` então: `/gsd-plan-phase {N}`
```

**Status: "Pronto para executar":**
```
## ▶ Próxima Ação

**Executar Fase {N}: {nome}** — {N} planos prontos para execução

`/clear` então: `/gsd-execute-phase {N}`
```

**Status: "Executando":**
```
## ▶ Próxima Ação

**Fase {N} em andamento** — {N}/{M} planos concluídos

`/clear` então: `/gsd-execute-phase {N}` (retomar)
```

**Status: "Pronto para verificar":**
```
## ▶ Próxima Ação

**Verificar Fase {N}** — UAT e testes

`/clear` então: `/gsd-verify-work {N}`
```

**Status: "Pronto para publicar":**
```
## ▶ Próxima Ação

**Publicar Fase {N}** — criar PR com corpo rico

`/clear` então: `/gsd-ship {N}`
```

**Status: "Milestone completo":**
```
## ▶ Próxima Ação

**Concluir Milestone {version}** — arquivar e preparar para próximo milestone

`/clear` então: `/gsd-complete-milestone {version}`
```
</step>

<step name="show_alternatives">
Exibir ações alternativas disponíveis:

```
───────────────────────────────────────────────────────────────

**Também disponível:**
- `/gsd-stats` — estatísticas detalhadas e métricas git
- `/gsd-manager` — central de comando interativa
- `/gsd-audit-milestone` — auditoria de qualidade antes de concluir
- `/gsd-settings` — ajustar configurações do fluxo de trabalho

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Estado do projeto carregado
- [ ] Nenhum projeto tratado graciosamente
- [ ] Progresso exibido com fases
- [ ] Próxima ação determinada com base no status
- [ ] Alternativas exibidas
</success_criteria>
