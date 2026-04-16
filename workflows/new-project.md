<purpose>
Inicializa um novo projeto por meio de um fluxo unificado: questionamento, pesquisa (opcional), requisitos, roadmap. Este é o momento de maior alavancagem em qualquer projeto — um questionamento profundo aqui significa planos melhores, execução melhor e resultados melhores. Um único workflow leva você da ideia ao pronto-para-planejar.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo `execution_context` do prompt que invocou este workflow antes de começar.
</required_reading>

<available_agent_types>
Tipos válidos de subagente GSD (use os nomes exatos — não recorra a 'general-purpose'):
- gsd-project-researcher — Pesquisa decisões técnicas no nível do projeto
- gsd-research-synthesizer — Sintetiza descobertas de agentes de pesquisa paralelos
- gsd-roadmapper — Cria roadmaps de execução em fases
</available_agent_types>

<auto_mode>

## Detecção do Modo Automático

Verifique se a flag `--auto` está presente em $ARGUMENTS.

**Se modo automático:**

- Pule a oferta de mapeamento brownfield (assuma greenfield)
- Pule o questionamento profundo (extraia o contexto do documento fornecido)
- Config: modo YOLO é implícito (pule essa pergunta), mas pergunte sobre granularidade/git/agentes PRIMEIRO (Passo 2a)
- Após a config: execute os Passos 6-9 automaticamente com padrões inteligentes:
  - Pesquisa: Sempre sim
  - Requisitos: Inclua todos os table stakes + funcionalidades do documento fornecido
  - Aprovação de requisitos: Aprovação automática
  - Aprovação do roadmap: Aprovação automática

**Requisito de documento:**
O modo automático requer um documento de ideia — seja:

- Referência a arquivo: `/gsd-new-project --auto @prd.md`
- Texto colado/escrito no prompt

Se nenhum conteúdo de documento for fornecido, erro:

```
Error: --auto requires an idea document.

Usage:
  /gsd-new-project --auto @your-idea.md
  /gsd-new-project --auto [paste or write your idea here]

The document should describe what you want to build.
```

</auto_mode>

<process>

## 1. Configuração Inicial

**PRIMEIRO PASSO OBRIGATÓRIO — Execute estas verificações antes de QUALQUER interação com o usuário:**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init new-project)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_RESEARCHER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-project-researcher 2>/dev/null)
AGENT_SKILLS_SYNTHESIZER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-synthesizer 2>/dev/null)
AGENT_SKILLS_ROADMAPPER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-roadmapper 2>/dev/null)
```

Analise o JSON para: `researcher_model`, `synthesizer_model`, `roadmapper_model`, `commit_docs`, `project_exists`, `has_codebase_map`, `planning_exists`, `has_existing_code`, `has_package_file`, `is_brownfield`, `needs_codebase_map`, `has_git`, `project_path`.

**Detecte o runtime e defina o nome do arquivo de instruções:**

Derive `RUNTIME` a partir do caminho `execution_context` do prompt que invocou:
- Caminho contém `/.codex/` → `RUNTIME=codex`
- Caminho contém `/.gemini/` → `RUNTIME=gemini`
- Caminho contém `/.config/opencode/` ou `/.opencode/` → `RUNTIME=opencode`
- Caso contrário → `RUNTIME=claude`

Se o caminho `execution_context` não estiver disponível, use variáveis de ambiente como fallback:
```bash
if [ -n "$CODEX_HOME" ]; then RUNTIME="codex"
elif [ -n "$GEMINI_CONFIG_DIR" ]; then RUNTIME="gemini"
elif [ -n "$OPENCODE_CONFIG_DIR" ] || [ -n "$OPENCODE_CONFIG" ]; then RUNTIME="opencode"
else RUNTIME="claude"; fi
```

Defina a variável do arquivo de instruções:
```bash
if [ "$RUNTIME" = "codex" ]; then INSTRUCTION_FILE="AGENTS.md"; else INSTRUCTION_FILE="CLAUDE.md"; fi
```

Todas as referências subsequentes ao arquivo de instruções do projeto usam `$INSTRUCTION_FILE`.

**Se `project_exists` for verdadeiro:** Erro — projeto já inicializado. Use `/gsd-progress`.

**Se `has_git` for falso:** Inicialize o git:

```bash
git init
```

## 2. Oferta Brownfield

**Se modo automático:** Pule para o Passo 4 (assuma greenfield, sintetize PROJECT.md a partir do documento fornecido).

**Se `needs_codebase_map` for verdadeiro** (detectado pelo init — código existente encontrado mas sem mapa de codebase):


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU se `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário que digite o número de sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Use AskUserQuestion:

- header: "Codebase"
- question: "Detectei código existente neste diretório. Gostaria de mapear o codebase primeiro?"
- options:
  - "Mapear codebase primeiro" — Execute /gsd-map-codebase para entender a arquitetura existente (Recomendado)
  - "Pular mapeamento" — Prossiga com a inicialização do projeto

**Se "Mapear codebase primeiro":**

```
Execute `/gsd-map-codebase` primeiro, depois retorne a `/gsd-new-project`
```

Encerre o comando.

**Se "Pular mapeamento" OU `needs_codebase_map` for falso:** Continue para o Passo 3.

## 2a. Config do Modo Automático (somente modo automático)

**Se modo automático:** Colete as configurações antecipadamente antes de processar o documento de ideia.

O modo YOLO é implícito (auto = YOLO). Faça as perguntas de config restantes:

**Rodada 1 — Configurações principais (3 perguntas, sem pergunta de Modo):**

```
AskUserQuestion([
  {
    header: "Granularity",
    question: "Com que nível de detalhe o escopo deve ser dividido em fases?",
    multiSelect: false,
    options: [
      { label: "Coarse (Recommended)", description: "Menos fases, mais amplas (3-5 fases, 1-3 planos cada)" },
      { label: "Standard", description: "Tamanho de fase equilibrado (5-8 fases, 3-5 planos cada)" },
      { label: "Fine", description: "Muitas fases focadas (8-12 fases, 5-10 planos cada)" }
    ]
  },
  {
    header: "Execution",
    question: "Executar planos em paralelo?",
    multiSelect: false,
    options: [
      { label: "Parallel (Recommended)", description: "Planos independentes executam simultaneamente" },
      { label: "Sequential", description: "Um plano por vez" }
    ]
  },
  {
    header: "Git Tracking",
    question: "Commitar documentos de planejamento no git?",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Documentos de planejamento rastreados no controle de versão" },
      { label: "No", description: "Manter .planning/ somente local (adicionar ao .gitignore)" }
    ]
  }
])
```

**Rodada 2 — Agentes de workflow (mesmo que o Passo 5):**

```
AskUserQuestion([
  {
    header: "Research",
    question: "Pesquisar antes de planejar cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Investigar domínio, encontrar padrões, revelar armadilhas" },
      { label: "No", description: "Planejar diretamente a partir dos requisitos" }
    ]
  },
  {
    header: "Plan Check",
    question: "Verificar se os planos atingirão seus objetivos? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Identificar lacunas antes da execução começar" },
      { label: "No", description: "Executar planos sem verificação" }
    ]
  },
  {
    header: "Verifier",
    question: "Verificar se o trabalho satisfaz os requisitos após cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Confirmar que as entregas correspondem aos objetivos da fase" },
      { label: "No", description: "Confiar na execução, pular verificação" }
    ]
  },
  {
    header: "AI Models",
    question: "Quais modelos de IA para os agentes de planejamento?",
    multiSelect: false,
    options: [
      { label: "Balanced (Recommended)", description: "Sonnet para a maioria dos agentes — boa relação qualidade/custo" },
      { label: "Quality", description: "Opus para pesquisa/roadmap — maior custo, análise mais profunda" },
      { label: "Budget", description: "Haiku onde possível — mais rápido, menor custo" },
      { label: "Inherit", description: "Usar o modelo da sessão atual para todos os agentes (OpenCode /model)" }
    ]
  }
])
```

Crie `.planning/config.json` com todas as configurações (a CLI preenche os padrões restantes automaticamente):

```bash
mkdir -p .planning
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-new-project '{"mode":"yolo","granularity":"[selected]","parallelization":true|false,"commit_docs":true|false,"model_profile":"quality|balanced|budget|inherit","workflow":{"research":true|false,"plan_check":true|false,"verifier":true|false,"nyquist_validation":true|false,"auto_advance":true}}'
```

**Se commit_docs = No:** Adicione `.planning/` ao `.gitignore`.

**Commitar config.json:**

```bash
mkdir -p .planning
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: add project config" --files .planning/config.json
```

**Persista a flag de encadeamento automático na config (sobrevive à compactação de contexto):**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

Prossiga para o Passo 4 (pule os Passos 3 e 5).

## 3. Questionamento Profundo

**Se modo automático:** Pule (já tratado no Passo 2a). Extraia o contexto do projeto a partir do documento fornecido e prossiga para o Passo 4.

**Exiba o banner de estágio:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► QUESTIONING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Abra a conversa:**

Pergunte inline (forma livre, NÃO use AskUserQuestion):

"O que você quer construir?"

Aguarde a resposta. Isso fornece o contexto necessário para fazer perguntas de acompanhamento inteligentes.

**Modo pesquisa-antes-das-perguntas:** Verifique se `workflow.research_before_questions` está habilitado em `.planning/config.json` (ou na config do contexto de init). Quando habilitado, antes de fazer perguntas de acompanhamento sobre uma área temática:

1. Faça uma busca rápida na web por melhores práticas relacionadas ao que o usuário descreveu
2. Mencione as principais descobertas naturalmente ao fazer as perguntas (ex: "A maioria dos projetos assim usa X — é isso que você está pensando, ou algo diferente?")
3. Isso torna as perguntas mais informadas sem alterar o fluxo conversacional

Quando desabilitado (padrão), faça as perguntas diretamente como antes.

**Siga o fio da conversa:**

Com base no que foi dito, faça perguntas de acompanhamento que aprofundem a resposta. Use AskUserQuestion com opções que sondem o que foi mencionado — interpretações, esclarecimentos, exemplos concretos.

Continue seguindo os fios. Cada resposta abre novos fios para explorar. Pergunte sobre:

- O que os animou
- Qual problema gerou isso
- O que eles querem dizer com termos vagos
- Como isso ficaria na prática
- O que já está decidido

Consulte `questioning.md` para técnicas:

- Desafie a vagueza
- Torne o abstrato concreto
- Revele premissas
- Encontre as bordas
- Descubra a motivação

**Verifique o contexto (em segundo plano, sem verbalizar):**

À medida que avança, verifique mentalmente o checklist de contexto de `questioning.md`. Se houver lacunas, faça as perguntas naturalmente. Não mude repentinamente para o modo checklist.

**Portão de decisão:**

Quando você puder escrever um PROJECT.md claro, use AskUserQuestion:

- header: "Pronto?"
- question: "Acho que entendi o que você busca. Pronto para criar o PROJECT.md?"
- options:
  - "Criar PROJECT.md" — Vamos em frente
  - "Continuar explorando" — Quero compartilhar mais / me faça mais perguntas

Se "Continuar explorando" — pergunte o que querem acrescentar, ou identifique lacunas e sonde naturalmente.

Repita até que "Criar PROJECT.md" seja selecionado.

## 4. Escrever PROJECT.md

**Se modo automático:** Sintetize a partir do documento fornecido. Nenhum portão "Pronto?" foi exibido — prossiga diretamente para o commit.

Sintetize todo o contexto em `.planning/PROJECT.md` usando o template de `templates/project.md`.

**Para projetos greenfield:**

Inicialize os requisitos como hipóteses:

```markdown
## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] [Requirement 1]
- [ ] [Requirement 2]
- [ ] [Requirement 3]

### Out of Scope

- [Exclusion 1] — [why]
- [Exclusion 2] — [why]
```

Todos os requisitos Ativos são hipóteses até serem entregues e validados.

**Para projetos brownfield (mapa de codebase existe):**

Infira os requisitos Validados a partir do código existente:

1. Leia `.planning/codebase/ARCHITECTURE.md` e `STACK.md`
2. Identifique o que o codebase já faz
3. Estes se tornam o conjunto inicial de Validados

```markdown
## Requirements

### Validated

- ✓ [Existing capability 1] — existing
- ✓ [Existing capability 2] — existing
- ✓ [Existing capability 3] — existing

### Active

- [ ] [New requirement 1]
- [ ] [New requirement 2]

### Out of Scope

- [Exclusion 1] — [why]
```

**Decisões-Chave:**

Inicialize com quaisquer decisões tomadas durante o questionamento:

```markdown
## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| [Choice from questioning] | [Why] | — Pending |
```

**Rodapé de última atualização:**

```markdown
---
*Last updated: [date] after initialization*
```

**Seção de Evolução** (inclua ao final do PROJECT.md, antes do rodapé):

```markdown
## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state
```

Não comprima. Capture tudo que foi coletado.

**Commitar PROJECT.md:**

```bash
mkdir -p .planning
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: initialize project" --files .planning/PROJECT.md
```

## 5. Preferências de Workflow

**Se modo automático:** Pule — a config foi coletada no Passo 2a. Prossiga para o Passo 5.5.

**Verifique os padrões globais** em `~/.gsd/defaults.json`. Se o arquivo existir, ofereça usar os padrões salvos:

```
AskUserQuestion([
  {
    question: "Usar suas configurações padrão salvas? (de ~/.gsd/defaults.json)",
    header: "Defaults",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Usar padrões salvos, pular perguntas de configuração" },
      { label: "No", description: "Configurar manualmente" }
    ]
  }
])
```

Se "Yes": leia `~/.gsd/defaults.json`, use esses valores para config.json e vá direto para **Commitar config.json** abaixo.

Se "No" ou `~/.gsd/defaults.json` não existir: prossiga com as perguntas abaixo.

**Rodada 1 — Configurações principais do workflow (4 perguntas):**

```
questions: [
  {
    header: "Mode",
    question: "Como você quer trabalhar?",
    multiSelect: false,
    options: [
      { label: "YOLO (Recommended)", description: "Aprovação automática, apenas execute" },
      { label: "Interactive", description: "Confirmar a cada etapa" }
    ]
  },
  {
    header: "Granularity",
    question: "Com que nível de detalhe o escopo deve ser dividido em fases?",
    multiSelect: false,
    options: [
      { label: "Coarse", description: "Menos fases, mais amplas (3-5 fases, 1-3 planos cada)" },
      { label: "Standard", description: "Tamanho de fase equilibrado (5-8 fases, 3-5 planos cada)" },
      { label: "Fine", description: "Muitas fases focadas (8-12 fases, 5-10 planos cada)" }
    ]
  },
  {
    header: "Execution",
    question: "Executar planos em paralelo?",
    multiSelect: false,
    options: [
      { label: "Parallel (Recommended)", description: "Planos independentes executam simultaneamente" },
      { label: "Sequential", description: "Um plano por vez" }
    ]
  },
  {
    header: "Git Tracking",
    question: "Commitar documentos de planejamento no git?",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Documentos de planejamento rastreados no controle de versão" },
      { label: "No", description: "Manter .planning/ somente local (adicionar ao .gitignore)" }
    ]
  }
]
```

**Rodada 2 — Agentes de workflow:**

Esses agentes são invocados durante o planejamento/execução. Eles adicionam tokens e tempo, mas melhoram a qualidade.

| Agente | Quando executa | O que faz |
|--------|----------------|-----------|
| **Researcher** | Antes de planejar cada fase | Investiga o domínio, encontra padrões, revela armadilhas |
| **Plan Checker** | Após a criação do plano | Verifica se o plano realmente atinge o objetivo da fase |
| **Verifier** | Após a execução da fase | Confirma que os itens obrigatórios foram entregues |

Todos recomendados para projetos importantes. Pule para experimentos rápidos.

```
questions: [
  {
    header: "Research",
    question: "Pesquisar antes de planejar cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Investigar domínio, encontrar padrões, revelar armadilhas" },
      { label: "No", description: "Planejar diretamente a partir dos requisitos" }
    ]
  },
  {
    header: "Plan Check",
    question: "Verificar se os planos atingirão seus objetivos? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Identificar lacunas antes da execução começar" },
      { label: "No", description: "Executar planos sem verificação" }
    ]
  },
  {
    header: "Verifier",
    question: "Verificar se o trabalho satisfaz os requisitos após cada fase? (adiciona tokens/tempo)",
    multiSelect: false,
    options: [
      { label: "Yes (Recommended)", description: "Confirmar que as entregas correspondem aos objetivos da fase" },
      { label: "No", description: "Confiar na execução, pular verificação" }
    ]
  },
  {
    header: "AI Models",
    question: "Quais modelos de IA para os agentes de planejamento?",
    multiSelect: false,
    options: [
      { label: "Balanced (Recommended)", description: "Sonnet para a maioria dos agentes — boa relação qualidade/custo" },
      { label: "Quality", description: "Opus para pesquisa/roadmap — maior custo, análise mais profunda" },
      { label: "Budget", description: "Haiku onde possível — mais rápido, menor custo" },
      { label: "Inherit", description: "Usar o modelo da sessão atual para todos os agentes (OpenCode /model)" }
    ]
  }
]
```

Crie `.planning/config.json` com todas as configurações (a CLI preenche os padrões restantes automaticamente):

```bash
mkdir -p .planning
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-new-project '{"mode":"[yolo|interactive]","granularity":"[selected]","parallelization":true|false,"commit_docs":true|false,"model_profile":"quality|balanced|budget|inherit","workflow":{"research":true|false,"plan_check":true|false,"verifier":true|false,"nyquist_validation":[false if granularity=coarse, true otherwise]}}'
```

**Nota:** Execute `/gsd-settings` a qualquer momento para atualizar perfil de modelo, agentes de workflow, estratégia de branching e outras preferências.

**Se commit_docs = No:**

- Defina `commit_docs: false` no config.json
- Adicione `.planning/` ao `.gitignore` (crie se necessário)

**Se commit_docs = Yes:**

- Nenhuma entrada adicional no gitignore é necessária

**Commitar config.json:**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: add project config" --files .planning/config.json
```

## 5.1. Detecção de Sub-Repositórios

**Detecte workspace multi-repositório:**

Verifique se há diretórios com suas próprias pastas `.git` (repositórios separados dentro do workspace):

```bash
find . -maxdepth 1 -type d -not -name ".*" -not -name "node_modules" -exec test -d "{}/.git" \; -print
```

**Se sub-repositórios forem encontrados:**

Remova o prefixo `./` para obter os nomes dos diretórios (ex: `./backend` → `backend`).

Use AskUserQuestion:

- header: "Multi-Repo Workspace"
- question: "Detectei repositórios git separados neste workspace. Quais diretórios contêm código que o GSD deve commitar?"
- multiSelect: true
- options: uma opção por diretório detectado
  - "[nome do diretório]" — Repositório git separado

**Se o usuário selecionar um ou mais diretórios:**

- Defina `planning.sub_repos` no config.json com o array de nomes dos diretórios selecionados (ex: `["backend", "frontend"]`)
- Defina automaticamente `planning.commit_docs` como `false` (documentos de planejamento ficam locais em workspaces multi-repositório)
- Adicione `.planning/` ao `.gitignore` se ainda não estiver presente

As alterações de config são salvas localmente — nenhum commit é necessário já que `commit_docs` é `false` em modo multi-repositório.

**Se nenhum sub-repositório for encontrado ou o usuário não selecionar nenhum:** Continue sem alterações na config.

## 5.5. Resolver Perfil de Modelos

Use os modelos do init: `researcher_model`, `synthesizer_model`, `roadmapper_model`.

## 6. Decisão de Pesquisa

**Se modo automático:** Padrão para "Pesquisar primeiro" sem perguntar.

Use AskUserQuestion:

- header: "Research"
- question: "Pesquisar o ecossistema do domínio antes de definir os requisitos?"
- options:
  - "Pesquisar primeiro (Recomendado)" — Descobrir stacks padrão, funcionalidades esperadas, padrões de arquitetura
  - "Pular pesquisa" — Conheço bem este domínio, ir direto para os requisitos

**Se "Pesquisar primeiro":**

Exiba o banner de estágio:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCHING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pesquisando ecossistema de [domain]...
```

Crie o diretório de pesquisa:

```bash
mkdir -p .planning/research
```

**Determine o contexto do milestone:**

Verifique se é greenfield ou um milestone subsequente:

- Se não há requisitos "Validated" no PROJECT.md → Greenfield (construindo do zero)
- Se há requisitos "Validated" → Milestone subsequente (adicionando a um app existente)

Exiba o indicador de inicialização:

```
◆ Iniciando 4 pesquisadores em paralelo...
  → Pesquisa de Stack
  → Pesquisa de Funcionalidades
  → Pesquisa de Arquitetura
  → Pesquisa de Armadilhas
```

Inicie 4 agentes gsd-project-researcher paralelos com referências de caminho:

```
Task(prompt="<research_type>
Project Research — Stack dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: Research the standard stack for building [domain] from scratch.
Subsequent: Research what's needed to add [target features] to an existing [domain] app. Don't re-research the existing system.
</milestone_context>

<question>
What's the standard 2025 stack for [domain]?
</question>

<files_to_read>
- {project_path} (Project context and goals)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<downstream_consumer>
Your STACK.md feeds into roadmap creation. Be prescriptive:
- Specific libraries with versions
- Clear rationale for each choice
- What NOT to use and why
</downstream_consumer>

<quality_gate>
- [ ] Versions are current (verify with Context7/official docs, not training data)
- [ ] Rationale explains WHY, not just WHAT
- [ ] Confidence levels assigned to each recommendation
</quality_gate>

<output>
Write to: .planning/research/STACK.md
Use template: $HOME/.claude/get-shit-done/templates/research-project/STACK.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Stack research")

Task(prompt="<research_type>
Project Research — Features dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: What features do [domain] products have? What's table stakes vs differentiating?
Subsequent: How do [target features] typically work? What's expected behavior?
</milestone_context>

<question>
What features do [domain] products have? What's table stakes vs differentiating?
</question>

<files_to_read>
- {project_path} (Project context)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<downstream_consumer>
Your FEATURES.md feeds into requirements definition. Categorize clearly:
- Table stakes (must have or users leave)
- Differentiators (competitive advantage)
- Anti-features (things to deliberately NOT build)
</downstream_consumer>

<quality_gate>
- [ ] Categories are clear (table stakes vs differentiators vs anti-features)
- [ ] Complexity noted for each feature
- [ ] Dependencies between features identified
</quality_gate>

<output>
Write to: .planning/research/FEATURES.md
Use template: $HOME/.claude/get-shit-done/templates/research-project/FEATURES.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Features research")

Task(prompt="<research_type>
Project Research — Architecture dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: How are [domain] systems typically structured? What are major components?
Subsequent: How do [target features] integrate with existing [domain] architecture?
</milestone_context>

<question>
How are [domain] systems typically structured? What are major components?
</question>

<files_to_read>
- {project_path} (Project context)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<downstream_consumer>
Your ARCHITECTURE.md informs phase structure in roadmap. Include:
- Component boundaries (what talks to what)
- Data flow (how information moves)
- Suggested build order (dependencies between components)
</downstream_consumer>

<quality_gate>
- [ ] Components clearly defined with boundaries
- [ ] Data flow direction explicit
- [ ] Build order implications noted
</quality_gate>

<output>
Write to: .planning/research/ARCHITECTURE.md
Use template: $HOME/.claude/get-shit-done/templates/research-project/ARCHITECTURE.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Architecture research")

Task(prompt="<research_type>
Project Research — Pitfalls dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: What do [domain] projects commonly get wrong? Critical mistakes?
Subsequent: What are common mistakes when adding [target features] to [domain]?
</milestone_context>

<question>
What do [domain] projects commonly get wrong? Critical mistakes?
</question>

<files_to_read>
- {project_path} (Project context)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<downstream_consumer>
Your PITFALLS.md prevents mistakes in roadmap/planning. For each pitfall:
- Warning signs (how to detect early)
- Prevention strategy (how to avoid)
- Which phase should address it
</downstream_consumer>

<quality_gate>
- [ ] Pitfalls are specific to this domain (not generic advice)
- [ ] Prevention strategies are actionable
- [ ] Phase mapping included where relevant
</quality_gate>

<output>
Write to: .planning/research/PITFALLS.md
Use template: $HOME/.claude/get-shit-done/templates/research-project/PITFALLS.md
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="Pitfalls research")
```

Após os 4 agentes concluírem, inicie o sintetizador para criar SUMMARY.md:

```
Task(prompt="
<task>
Synthesize research outputs into SUMMARY.md.
</task>

<files_to_read>
- .planning/research/STACK.md
- .planning/research/FEATURES.md
- .planning/research/ARCHITECTURE.md
- .planning/research/PITFALLS.md
</files_to_read>

${AGENT_SKILLS_SYNTHESIZER}

<output>
Write to: .planning/research/SUMMARY.md
Use template: $HOME/.claude/get-shit-done/templates/research-project/SUMMARY.md
Commit after writing.
</output>
", subagent_type="gsd-research-synthesizer", model="{synthesizer_model}", description="Synthesize research")
```

Exiba o banner de pesquisa concluída e as principais descobertas:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESEARCH COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Principais Descobertas

**Stack:** [do SUMMARY.md]
**Table Stakes:** [do SUMMARY.md]
**Fique Atento a:** [do SUMMARY.md]

Arquivos: `.planning/research/`
```

**Se "Pular pesquisa":** Continue para o Passo 7.

## 7. Definir Requisitos

Exiba o banner de estágio:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DEFINING REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Carregue o contexto:**

Leia o PROJECT.md e extraia:

- Valor central (a UMA coisa que deve funcionar)
- Restrições declaradas (orçamento, prazo, limitações técnicas)
- Quaisquer limites de escopo explícitos

**Se a pesquisa existir:** Leia research/FEATURES.md e extraia as categorias de funcionalidades.

**Se modo automático:**

- Inclua automaticamente todos os recursos table stakes (os usuários esperam por eles)
- Inclua funcionalidades mencionadas explicitamente no documento fornecido
- Adie automaticamente diferenciadores não mencionados no documento
- Pule os loops de AskUserQuestion por categoria
- Pule a pergunta "Alguma adição?"
- Pule o portão de aprovação de requisitos
- Gere REQUIREMENTS.md e commite diretamente

**Apresente as funcionalidades por categoria (somente modo interativo):**

```
Aqui estão as funcionalidades para [domain]:

## Autenticação
**Table stakes:**
- Cadastro com e-mail/senha
- Verificação de e-mail
- Redefinição de senha
- Gerenciamento de sessão

**Diferenciadores:**
- Login por magic link
- OAuth (Google, GitHub)
- 2FA

**Notas de pesquisa:** [quaisquer notas relevantes]

---

## [Próxima Categoria]
...
```

**Se não houver pesquisa:** Levante requisitos por meio da conversa.

Pergunte: "Quais são as principais coisas que os usuários precisam conseguir fazer?"

Para cada capacidade mencionada:

- Faça perguntas de esclarecimento para torná-la específica
- Sonde capacidades relacionadas
- Agrupe em categorias

**Defina o escopo de cada categoria:**

Para cada categoria, use AskUserQuestion:

- header: "[Categoria]" (máx 12 chars)
- question: "Quais funcionalidades de [categoria] entram na v1?"
- multiSelect: true
- options:
  - "[Funcionalidade 1]" — [breve descrição]
  - "[Funcionalidade 2]" — [breve descrição]
  - "[Funcionalidade 3]" — [breve descrição]
  - "Nenhuma para a v1" — Adiar toda a categoria

Registre as respostas:

- Funcionalidades selecionadas → requisitos da v1
- Table stakes não selecionados → v2 (os usuários esperam por eles)
- Diferenciadores não selecionados → fora do escopo

**Identifique lacunas:**

Use AskUserQuestion:

- header: "Adições"
- question: "Algum requisito que a pesquisa deixou passar? (Funcionalidades específicas da sua visão)"
- options:
  - "Não, a pesquisa cobriu tudo" — Prosseguir
  - "Sim, deixa eu adicionar alguns" — Capturar adições

**Valide o valor central:**

Verifique os requisitos em relação ao Valor Central do PROJECT.md. Se forem detectadas lacunas, sinalize-as.

**Gere REQUIREMENTS.md:**

Crie `.planning/REQUIREMENTS.md` com:

- Requisitos da v1 agrupados por categoria (checkboxes, REQ-IDs)
- Requisitos da v2 (adiados)
- Fora do Escopo (exclusões explícitas com justificativa)
- Seção de rastreabilidade (vazia, preenchida pelo roadmap)

**Formato de REQ-ID:** `[CATEGORIA]-[NÚMERO]` (AUTH-01, CONTENT-02)

**Critérios de qualidade dos requisitos:**

Bons requisitos são:

- **Específicos e testáveis:** "Usuário pode redefinir senha via link por e-mail" (não "Tratar redefinição de senha")
- **Centrados no usuário:** "Usuário pode X" (não "Sistema faz Y")
- **Atômicos:** Uma capacidade por requisito (não "Usuário pode fazer login e gerenciar perfil")
- **Independentes:** Dependências mínimas de outros requisitos

Rejeite requisitos vagos. Exija especificidade:

- "Tratar autenticação" → "Usuário pode fazer login com e-mail/senha e permanecer logado entre sessões"
- "Suportar compartilhamento" → "Usuário pode compartilhar post via link que abre no navegador do destinatário"

**Apresente a lista completa de requisitos (somente modo interativo):**

Mostre cada requisito (não contagens) para confirmação do usuário:

```
## Requisitos da v1

### Autenticação
- [ ] **AUTH-01**: Usuário pode criar conta com e-mail/senha
- [ ] **AUTH-02**: Usuário pode fazer login e permanecer logado entre sessões
- [ ] **AUTH-03**: Usuário pode fazer logout de qualquer página

### Conteúdo
- [ ] **CONT-01**: Usuário pode criar posts com texto
- [ ] **CONT-02**: Usuário pode editar seus próprios posts

[... lista completa ...]

---

Isso captura o que você está construindo? (sim / ajustar)
```

Se "ajustar": Retorne ao escopo.

**Commitar os requisitos:**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: define v1 requirements" --files .planning/REQUIREMENTS.md
```

## 8. Criar Roadmap

Exiba o banner de estágio:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CREATING ROADMAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando roadmapper...
```

Inicie o agente gsd-roadmapper com referências de caminho:

```
Task(prompt="
<planning_context>

<files_to_read>
- .planning/PROJECT.md (Project context)
- .planning/REQUIREMENTS.md (v1 Requirements)
- .planning/research/SUMMARY.md (Research findings - if exists)
- .planning/config.json (Granularity and mode settings)
</files_to_read>

${AGENT_SKILLS_ROADMAPPER}

</planning_context>

<instructions>
Create roadmap:
1. Derive phases from requirements (don't impose structure)
2. Map every v1 requirement to exactly one phase
3. Derive 2-5 success criteria per phase (observable user behaviors)
4. Validate 100% coverage
5. Write files immediately (ROADMAP.md, STATE.md, update REQUIREMENTS.md traceability)
6. Return ROADMAP CREATED with summary

Write files first, then return. This ensures artifacts persist even if context is lost.
</instructions>
", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Create roadmap")
```

**Trate o retorno do roadmapper:**

**Se `## ROADMAP BLOCKED`:**

- Apresente as informações do bloqueio
- Trabalhe com o usuário para resolver
- Reinicie quando resolvido

**Se `## ROADMAP CREATED`:**

Leia o ROADMAP.md criado e apresente-o de forma clara inline:

```
---

## Roadmap Proposto

**[N] fases** | **[X] requisitos mapeados** | Todos os requisitos da v1 cobertos ✓

| # | Fase | Objetivo | Requisitos | Critérios de Sucesso |
|---|------|----------|------------|----------------------|
| 1 | [Nome] | [Objetivo] | [REQ-IDs] | [quantidade] |
| 2 | [Nome] | [Objetivo] | [REQ-IDs] | [quantidade] |
| 3 | [Nome] | [Objetivo] | [REQ-IDs] | [quantidade] |
...

### Detalhes das Fases

**Fase 1: [Nome]**
Objetivo: [objetivo]
Requisitos: [REQ-IDs]
Critérios de sucesso:
1. [critério]
2. [critério]
3. [critério]

**Fase 2: [Nome]**
Objetivo: [objetivo]
Requisitos: [REQ-IDs]
Critérios de sucesso:
1. [critério]
2. [critério]

[... continuar para todas as fases ...]

---
```

**Se modo automático:** Pule o portão de aprovação — aprove automaticamente e commite diretamente.

**CRÍTICO: Solicite aprovação antes de commitar (somente modo interativo):**

Use AskUserQuestion:

- header: "Roadmap"
- question: "Esta estrutura de roadmap funciona para você?"
- options:
  - "Aprovar" — Commitar e continuar
  - "Ajustar fases" — Diga-me o que mudar
  - "Revisar arquivo completo" — Mostrar o ROADMAP.md bruto

**Se "Aprovar":** Continue para o commit.

**Se "Ajustar fases":**

- Obtenha as notas de ajuste do usuário
- Reinicie o roadmapper com contexto de revisão:

  ```
  Task(prompt="
  <revision>
  User feedback on roadmap:
  [user's notes]

  <files_to_read>
  - .planning/ROADMAP.md (Current roadmap to revise)
  </files_to_read>

  ${AGENT_SKILLS_ROADMAPPER}

  Update the roadmap based on feedback. Edit files in place.
  Return ROADMAP REVISED with changes made.
  </revision>
  ", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Revise roadmap")
  ```

- Apresente o roadmap revisado
- Repita até o usuário aprovar

**Se "Revisar arquivo completo":** Exiba o `cat .planning/ROADMAP.md` bruto, depois pergunte novamente.

**Gere ou atualize o arquivo de instruções do projeto antes do commit final:**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" generate-claude-md --output "$INSTRUCTION_FILE"
```

Isso garante que novos projetos recebam as orientações padrão de aplicação do workflow GSD e o contexto atual do projeto em `$INSTRUCTION_FILE`.

**Commitar o roadmap (após aprovação ou em modo automático):**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: create roadmap ([N] phases)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md "$INSTRUCTION_FILE"
```

## 9. Concluído

Apresente o resumo de conclusão:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PROJECT INITIALIZED ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**[Nome do Projeto]**

| Artefato        | Localização                 |
|-----------------|-----------------------------|
| Project         | `.planning/PROJECT.md`      |
| Config          | `.planning/config.json`     |
| Research        | `.planning/research/`       |
| Requirements    | `.planning/REQUIREMENTS.md` |
| Roadmap         | `.planning/ROADMAP.md`      |
| Guia do projeto | `$INSTRUCTION_FILE`         |

**[N] fases** | **[X] requisitos** | Pronto para construir ✓
```

**Se modo automático:**

```
╔══════════════════════════════════════════╗
║  AUTO-ADVANCING → DISCUSS PHASE 1        ║
╚══════════════════════════════════════════╝
```

Encerre a skill e invoque SlashCommand("/gsd-discuss-phase 1 --auto")

**Se modo interativo:**

Verifique se a Fase 1 tem indicadores de UI (procure por `**UI hint**: yes` na seção de detalhes da Fase 1 do ROADMAP.md):

```bash
PHASE1_SECTION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase 1 2>/dev/null)
PHASE1_HAS_UI=$(echo "$PHASE1_SECTION" | grep -qi "UI hint.*yes" && echo "true" || echo "false")
```

**Se a Fase 1 tem UI (`PHASE1_HAS_UI` é `true`):**

```
───────────────────────────────────────────────────────────────

## ▶ Próxima Etapa

**Fase 1: [Nome da Fase]** — [Objetivo do ROADMAP.md]

/clear e então:

/gsd-discuss-phase 1 — reunir contexto e esclarecer abordagem

---

**Também disponível:**
- /gsd-ui-phase 1 — gerar contrato de design de UI (recomendado para fases de frontend)
- /gsd-plan-phase 1 — pular a discussão, planejar diretamente

───────────────────────────────────────────────────────────────
```

**Se a Fase 1 não tem UI:**

```
───────────────────────────────────────────────────────────────

## ▶ Próxima Etapa

**Fase 1: [Nome da Fase]** — [Objetivo do ROADMAP.md]

/clear e então:

/gsd-discuss-phase 1 — reunir contexto e esclarecer abordagem

---

**Também disponível:**
- /gsd-plan-phase 1 — pular a discussão, planejar diretamente

───────────────────────────────────────────────────────────────
```

</process>

<output>

- `.planning/PROJECT.md`
- `.planning/config.json`
- `.planning/research/` (se pesquisa for selecionada)
  - `STACK.md`
  - `FEATURES.md`
  - `ARCHITECTURE.md`
  - `PITFALLS.md`
  - `SUMMARY.md`
- `.planning/REQUIREMENTS.md`
- `.planning/ROADMAP.md`
- `.planning/STATE.md`
- `$INSTRUCTION_FILE` (`AGENTS.md` para Codex, `CLAUDE.md` para todos os outros runtimes)

</output>

<success_criteria>

- [ ] Diretório .planning/ criado
- [ ] Repositório git inicializado
- [ ] Detecção brownfield concluída
- [ ] Questionamento profundo concluído (fios seguidos, sem pressa)
- [ ] PROJECT.md captura o contexto completo → **commitado**
- [ ] config.json tem modo de workflow, granularidade, paralelização → **commitado**
- [ ] Pesquisa concluída (se selecionada) — 4 agentes paralelos iniciados → **commitada**
- [ ] Requisitos levantados (da pesquisa ou da conversa)
- [ ] Usuário definiu o escopo de cada categoria (v1/v2/fora do escopo)
- [ ] REQUIREMENTS.md criado com REQ-IDs → **commitado**
- [ ] gsd-roadmapper iniciado com contexto
- [ ] Arquivos do roadmap escritos imediatamente (não em rascunho)
- [ ] Feedback do usuário incorporado (se houver)
- [ ] ROADMAP.md criado com fases, mapeamentos de requisitos, critérios de sucesso
- [ ] STATE.md inicializado
- [ ] Rastreabilidade do REQUIREMENTS.md atualizada
- [ ] `$INSTRUCTION_FILE` gerado com orientações do workflow GSD (AGENTS.md para Codex, CLAUDE.md caso contrário)
- [ ] Usuário sabe que o próximo passo é `/gsd-discuss-phase 1`

**Commits atômicos:** Cada fase commita seus artefatos imediatamente. Se o contexto for perdido, os artefatos persistem.

</success_criteria>
