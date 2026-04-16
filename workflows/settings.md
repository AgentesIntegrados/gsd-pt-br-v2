<purpose>
Configuração interativa dos agentes de fluxo de trabalho GSD (research, plan_check, verifier) e seleção de perfil de modelo via prompt de múltiplas perguntas. Atualiza .planning/config.json com as preferências do usuário. Opcionalmente, salva as configurações como padrões globais (~/.gsd/defaults.json) para projetos futuros.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="ensure_and_load_config">
Garanta que a config exista e carregue o estado atual:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-ensure-section
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Cria `.planning/config.json` com padrões se estiver ausente e carrega os valores de configuração atuais.
</step>

<step name="read_current">
```bash
cat .planning/config.json
```

Analise os valores atuais (padrão para `true` se não estiver presente):
- `workflow.research` — spawnar pesquisador durante plan-phase
- `workflow.plan_check` — spawnar verificador de plano durante plan-phase
- `workflow.verifier` — spawnar verificador durante execute-phase
- `workflow.nyquist_validation` — pesquisa de arquitetura de validação durante plan-phase (padrão: true se ausente)
- `workflow.ui_phase` — gerar contratos de design UI-SPEC.md para fases frontend (padrão: true se ausente)
- `workflow.ui_safety_gate` — solicitar para executar /gsd-ui-phase antes de planejar fases frontend (padrão: true se ausente)
- `workflow.ai_integration_phase` — seleção de framework + estratégia de avaliação para fases AI (padrão: true se ausente)
- `model_profile` — qual modelo cada agente usa (padrão: `balanced`)
- `git.branching_strategy` — abordagem de branching (padrão: `"none"`)
- `workflow.use_worktrees` — se os agentes executores paralelos rodam em isolamento de worktree (padrão: `true`)
</step>

<step name="present_settings">

**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Use AskUserQuestion com os valores atuais pré-selecionados:

```
AskUserQuestion([
  {
    question: "Qual perfil de modelo para os agentes?",
    header: "Modelo",
    multiSelect: false,
    options: [
      { label: "Quality", description: "Opus em todo lugar exceto verificação (custo mais alto)" },
      { label: "Balanced (Recomendado)", description: "Opus para planejamento, Sonnet para pesquisa/execução/verificação" },
      { label: "Budget", description: "Sonnet para escrita, Haiku para pesquisa/verificação (custo mais baixo)" },
      { label: "Inherit", description: "Usar o modelo da sessão atual para todos os agentes (melhor para OpenRouter, modelos locais ou troca de modelo em tempo de execução)" }
    ]
  },
  {
    question: "Spawnar Pesquisador de Planos? (pesquisa domínio antes do planejamento)",
    header: "Research",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Pesquisar objetivos da fase antes do planejamento" },
      { label: "Não", description: "Pular pesquisa, planejar diretamente" }
    ]
  },
  {
    question: "Spawnar Verificador de Planos? (verifica planos antes da execução)",
    header: "Verificação de Plano",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Verificar se os planos atendem aos objetivos da fase" },
      { label: "Não", description: "Pular verificação de plano" }
    ]
  },
  {
    question: "Spawnar Verificador de Execução? (verifica conclusão da fase)",
    header: "Verificador",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Verificar must-haves após a execução" },
      { label: "Não", description: "Pular verificação pós-execução" }
    ]
  },
  {
    question: "Pipeline de avanço automático? (discuss → plan → execute automaticamente)",
    header: "Auto",
    multiSelect: false,
    options: [
      { label: "Não (Recomendado)", description: "Manual /clear + colar entre etapas" },
      { label: "Sim", description: "Encadear etapas via subagentes Task() (mesmo isolamento)" }
    ]
  },
  {
    question: "Habilitar Validação Nyquist? (pesquisa cobertura de testes durante planejamento)",
    header: "Nyquist",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Pesquisar cobertura de testes automatizados durante plan-phase. Adiciona requisitos de validação aos planos. Bloqueia aprovação se tarefas não tiverem verify automatizado." },
      { label: "Não", description: "Pular pesquisa de validação. Bom para prototipagem rápida ou fases sem testes." }
    ]
  },
  {
    question: "Habilitar Fase UI? (gera contratos de design UI-SPEC.md para fases frontend)",
    header: "Fase UI",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Gerar contratos de design UI antes de planejar fases frontend. Bloqueia espaçamento, tipografia, cor e redação." },
      { label: "Não", description: "Pular geração de UI-SPEC. Bom para projetos apenas backend ou fases de API." }
    ]
  },
  {
    question: "Habilitar Gate de Segurança UI? (solicita executar /gsd-ui-phase antes de planejar fases frontend)",
    header: "Gate UI",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "plan-phase pergunta para executar /gsd-ui-phase primeiro quando indicadores frontend detectados." },
      { label: "Não", description: "Sem prompt — plan-phase prossegue sem verificação de UI-SPEC." }
    ]
  },
  {
    question: "Habilitar Fase AI? (seleção de framework + estratégia de avaliação para fases AI)",
    header: "Fase AI",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Executar /gsd-ai-phase antes de planejar fases de sistema AI. Apresenta o framework certo, pesquisa seus docs e projeta a estratégia de avaliação." },
      { label: "Não", description: "Pular contrato de design AI. Bom para fases não-AI ou quando o framework já está decidido." }
    ]
  },
  {
    question: "Estratégia de branching git?",
    header: "Branching",
    multiSelect: false,
    options: [
      { label: "Nenhuma (Recomendado)", description: "Commitar diretamente no branch atual" },
      { label: "Por Fase", description: "Criar branch para cada fase (gsd/phase-{N}-{name})" },
      { label: "Por Milestone", description: "Criar branch para todo o milestone (gsd/{version}-{name})" }
    ]
  },
  {
    question: "Habilitar avisos de janela de contexto? (injeta mensagens de aviso quando o contexto está ficando cheio)",
    header: "Avisos de Ctx",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Avisar quando o uso de contexto exceder 65%. Ajuda a evitar perda de trabalho." },
      { label: "Não", description: "Desabilitar avisos. Permite que o Claude atinja auto-compact naturalmente. Bom para execuções longas sem supervisão." }
    ]
  },
  {
    question: "Pesquisar melhores práticas antes de fazer perguntas? (busca na web durante new-project e discuss-phase)",
    header: "Pesquisar Qs",
    multiSelect: false,
    options: [
      { label: "Não (Recomendado)", description: "Fazer perguntas diretamente. Mais rápido, usa menos tokens." },
      { label: "Sim", description: "Pesquisar na web por melhores práticas antes de cada grupo de perguntas. Perguntas mais informadas mas usa mais tokens." }
    ]
  },
  {
    question: "Pular discuss-phase no modo autônomo? (usar objetivos de fase do ROADMAP como spec)",
    header: "Pular Discuss",
    multiSelect: false,
    options: [
      { label: "Não (Recomendado)", description: "Executar discuss inteligente antes de cada fase — apresenta áreas cinzas e captura decisões." },
      { label: "Sim", description: "Pular discuss em /gsd-autonomous — encadear diretamente para planejar. Melhor para trabalho backend/pipeline onde as descrições de fase são o spec." }
    ]
  },
  {
    question: "Usar git worktrees para isolamento de agentes paralelos?",
    header: "Worktrees",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Cada executor paralelo roda no seu próprio branch de worktree — sem conflitos entre agentes." },
      { label: "Não", description: "Desabilitar isolamento de worktree. Agentes rodam sequencialmente na árvore de trabalho principal. Use se EnterWorktree cria branches a partir da base errada (problema multiplataforma conhecido)." }
    ]
  }
])
```
</step>

<step name="update_config">
Mescle as novas configurações no config.json existente:

```json
{
  ...existing_config,
  "model_profile": "quality" | "balanced" | "budget" | "adaptive" | "inherit",
  "workflow": {
    "research": true/false,
    "plan_check": true/false,
    "verifier": true/false,
    "auto_advance": true/false,
    "nyquist_validation": true/false,
    "ui_phase": true/false,
    "ui_safety_gate": true/false,
    "ai_integration_phase": true/false,
    "text_mode": true/false,
    "research_before_questions": true/false,
    "discuss_mode": "discuss" | "assumptions",
    "skip_discuss": true/false,
    "use_worktrees": true/false
  },
  "git": {
    "branching_strategy": "none" | "phase" | "milestone",
    "quick_branch_template": <string|null>
  },
  "hooks": {
    "context_warnings": true/false,
    "workflow_guard": true/false
  }
}
```

Escreva a config atualizada em `.planning/config.json`.
</step>

<step name="save_as_defaults">
Pergunte se deseja salvar essas configurações como padrões globais para projetos futuros:

```
AskUserQuestion([
  {
    question: "Salvar essas configurações como padrões para todos os novos projetos?",
    header: "Padrões",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Novos projetos começam com essas configurações (salvo em ~/.gsd/defaults.json)" },
      { label: "Não", description: "Aplicar apenas a este projeto" }
    ]
  }
])
```

Se "Sim": escreva o mesmo objeto de config (menos campos específicos do projeto como `brave_search`) em `~/.gsd/defaults.json`:

```bash
mkdir -p ~/.gsd
```
</step>

<step name="confirm">
Exiba:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CONFIGURAÇÕES ATUALIZADAS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Configuração            | Valor |
|-------------------------|-------|
| Perfil de Modelo        | {quality/balanced/budget/inherit} |
| Pesquisador de Planos   | {Ativado/Desativado} |
| Verificador de Planos   | {Ativado/Desativado} |
| Verificador de Execução | {Ativado/Desativado} |
| Avanço Automático       | {Ativado/Desativado} |
| Validação Nyquist       | {Ativado/Desativado} |
| Fase UI                 | {Ativado/Desativado} |
| Gate de Segurança UI    | {Ativado/Desativado} |
| Fase AI                 | {Ativado/Desativado} |
| Branching Git           | {Nenhum/Por Fase/Por Milestone} |
| Pular Discuss           | {Ativado/Desativado} |
| Avisos de Contexto      | {Ativado/Desativado} |
| Salvo como Padrões      | {Sim/Não} |

Essas configurações se aplicam às futuras execuções de /gsd-plan-phase e /gsd-execute-phase.

Comandos rápidos:
- /gsd-set-profile <profile> — trocar perfil de modelo
- /gsd-plan-phase --research — forçar pesquisa
- /gsd-plan-phase --skip-research — pular pesquisa
- /gsd-plan-phase --skip-verify — pular verificação de plano
```
</step>

</process>

<success_criteria>
- [ ] Config atual lida
- [ ] Usuário apresentado com 14 configurações (perfil + 11 toggles de fluxo de trabalho + branching git + avisos de ctx)
- [ ] Config atualizada com seções model_profile, workflow e git
- [ ] Usuário oferecido para salvar como padrões globais (~/.gsd/defaults.json)
- [ ] Alterações confirmadas ao usuário
</success_criteria>
</output>
