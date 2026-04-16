<purpose>
Central de comando interativa para gerenciar um milestone. Dashboard que exibe estado do projeto e oferece acesso rápido às ações mais comuns a partir de uma única interface.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar estado completo do projeto:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init milestone-op "${MILESTONE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
STATS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" stats json)
if [[ "$STATS" == @file:* ]]; then STATS=$(cat "${STATS#@file:}"); fi
```

Extrair do init JSON: `milestone_version`, `milestone_name`, `phases_complete`, `phases_total`.
Extrair do stats JSON: `git_commits`, `last_activity`, `requirements_complete`, `requirements_total`.
</step>

<step name="display_dashboard">
Exibir painel de controle do milestone:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► GERENCIADOR — {milestone_version} {milestone_name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Progresso
[{barra de progresso}] {phases_complete}/{phases_total} fases ({%})

## Fases

| Fase | Nome | Planos | Status | Verificação |
|------|------|--------|--------|-------------|
{para cada fase: número, nome, planos concluídos/total, status, resultado da verificação}

## Métricas

Requisitos: {requirements_complete}/{requirements_total}
Commits: {git_commits}
Última atividade: {last_activity}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="present_actions">
**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Usar AskUserQuestion para menu principal:

```
AskUserQuestion:
  question: "O que você quer fazer?"
  header: "Ações"
  options:
    - label: "Ver progresso detalhado"
      description: "Status completo de fases e planos"
    - label: "Planejar próxima fase"
      description: "Criar planos para a próxima fase não planejada"
    - label: "Executar fase"
      description: "Executar planos para uma fase específica"
    - label: "Verificar trabalho"
      description: "UAT e verificação de fase"
    - label: "Auditoria de milestone"
      description: "Verificar se o milestone está pronto para conclusão"
    - label: "Estatísticas do projeto"
      description: "Métricas detalhadas e estatísticas git"
    - label: "Configurações"
      description: "Ajustar configurações do fluxo de trabalho GSD"
    - label: "Sair"
      description: "Fechar o gerenciador"
```
</step>

<step name="handle_selection">
Com base na seleção do usuário:

**"Ver progresso detalhado":**
Executar fluxo de trabalho progress inline. Exibir estado de cada fase.
Retornar ao menu.

**"Planejar próxima fase":**
Identificar próxima fase não planejada do ROADMAP.md.
Exibir: "Próxima fase não planejada: Fase {N} — {nome}"
Perguntar confirmação, então invocar: `/gsd-plan-phase {N}`

**"Executar fase":**
Perguntar qual fase: exibir lista de fases com planos mas não executadas.
Invocar: `/gsd-execute-phase {N}`

**"Verificar trabalho":**
Perguntar qual fase verificar: exibir fases com trabalho concluído.
Invocar: `/gsd-verify-work {N}`

**"Auditoria de milestone":**
Invocar: `/gsd-audit-milestone`

**"Estatísticas do projeto":**
Invocar: `/gsd-stats`

**"Configurações":**
Invocar: `/gsd-settings`

**"Sair":**
```
Saindo do GSD Manager.
Use /gsd-progress para verificar o estado do projeto a qualquer momento.
```
Sair.
</step>

<step name="return_to_menu">
Após cada ação (exceto Sair), perguntar:

```
AskUserQuestion:
  question: "O que mais?"
  header: "Continuar"
  options:
    - label: "Voltar ao menu"
      description: "Ver opções do gerenciador"
    - label: "Sair"
      description: "Fechar o gerenciador"
```

Se "Voltar ao menu": atualizar dashboard e apresentar ações novamente.
Se "Sair": exibir mensagem de saída.
</step>

</process>

<success_criteria>
- [ ] Estado completo do projeto carregado
- [ ] Dashboard exibido com progresso do milestone
- [ ] Todas as 8 opções de ação apresentadas
- [ ] Cada seleção roteada para o fluxo de trabalho ou ação correto
- [ ] Retorno ao menu após cada ação
</success_criteria>
