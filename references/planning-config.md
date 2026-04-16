<planning_config>

Opções de configuração para o comportamento do diretório `.planning/`.

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"git": {
  "branching_strategy": "none",
  "base_branch": null,
  "phase_branch_template": "gsd/phase-{phase}-{slug}",
  "milestone_branch_template": "gsd/{milestone}-{slug}",
  "quick_branch_template": null
},
"manager": {
  "flags": {
    "discuss": "",
    "plan": "",
    "execute": ""
  }
}
```

| Opção | Padrão | Descrição |
|-------|--------|-----------|
| `commit_docs` | `true` | Se deve commitar artifacts de planejamento no git |
| `search_gitignored` | `false` | Adiciona `--no-ignore` em buscas amplas com rg |
| `git.branching_strategy` | `"none"` | Abordagem de branching do git: `"none"`, `"phase"` ou `"milestone"` |
| `git.base_branch` | `null` (detecção automática) | Branch de destino para PRs e merges (ex.: `"master"`, `"develop"`). Quando `null`, detecta automaticamente via `git symbolic-ref refs/remotes/origin/HEAD`, com fallback para `"main"`. |
| `git.phase_branch_template` | `"gsd/phase-{phase}-{slug}"` | Template de branch para a estratégia phase |
| `git.milestone_branch_template` | `"gsd/{milestone}-{slug}"` | Template de branch para a estratégia milestone |
| `git.quick_branch_template` | `null` | Template de branch opcional para execuções de quick-task |
| `workflow.use_worktrees` | `true` | Se os agentes executor devem rodar em git worktrees isolados. Defina como `false` para desabilitar worktrees — os agentes executam sequencialmente na árvore de trabalho principal. Recomendado para desenvolvedores solo ou quando merges de worktree causam problemas. |
| `workflow.subagent_timeout` | `300000` | Timeout em milissegundos para tarefas de subagente paralelas (ex.: mapeamento de codebase). Aumente para codebases grandes ou modelos mais lentos. Padrão: 300000 (5 minutos). |
| `workflow.inline_plan_threshold` | `2` | Planos com esta quantidade de tarefas ou menos executam inline (Padrão C) em vez de criar um subagente. Evita o overhead de ~14K tokens de spawn para planos pequenos. Defina como `0` para sempre criar subagentes. |
| `manager.flags.discuss` | `""` | Flags passadas ao `/gsd-discuss-phase` quando despachado pelo manager (ex.: `"--auto --analyze"`) |
| `manager.flags.plan` | `""` | Flags passadas ao workflow de plan quando despachado pelo manager |
| `manager.flags.execute` | `""` | Flags passadas ao workflow de execute quando despachado pelo manager |
| `response_language` | `null` | Idioma para perguntas e prompts voltados ao usuário em todas as fases/subagentes (ex.: `"Portuguese"`, `"Japanese"`, `"Spanish"`). Quando definido, todos os agentes criados incluem uma diretiva para responder nesse idioma. |
</config_schema>

<commit_docs_behavior>

**Quando `commit_docs: true` (padrão):**
- Arquivos de planejamento commitados normalmente
- SUMMARY.md, STATE.md, ROADMAP.md rastreados no git
- Histórico completo das decisões de planejamento preservado

**Quando `commit_docs: false`:**
- Pular todos os `git add`/`git commit` para arquivos `.planning/`
- O usuário deve adicionar `.planning/` ao `.gitignore`
- Útil para: contribuições OSS, projetos de clientes, manter o planejamento privado

**Usando gsd-tools.cjs (preferido):**

```bash
# Commit com verificações automáticas de commit_docs + gitignore:
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: update state" --files .planning/STATE.md

# Carregar config via state load (retorna JSON):
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docs está disponível na saída JSON

# Ou use comandos init que incluem commit_docs:
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# commit_docs está incluído em todas as saídas de comandos init
```

**Detecção automática:** Se `.planning/` estiver no gitignore, `commit_docs` é automaticamente `false` independentemente do config.json. Isso evita erros do git quando usuários têm `.planning/` no `.gitignore`.

**Commit via CLI (lida com as verificações automaticamente):**

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: update state" --files .planning/STATE.md
```

A CLI verifica internamente a configuração `commit_docs` e o status do gitignore — sem condicionais manuais necessários.

</commit_docs_behavior>

<search_behavior>

**Quando `search_gitignored: false` (padrão):**
- Comportamento padrão do rg (respeita .gitignore)
- Buscas por caminho direto funcionam: `rg "pattern" .planning/` encontra arquivos
- Buscas amplas ignoram arquivos gitignored: `rg "pattern"` pula `.planning/`

**Quando `search_gitignored: true`:**
- Adicione `--no-ignore` em buscas amplas com rg que devem incluir `.planning/`
- Necessário apenas ao pesquisar o repositório inteiro esperando correspondências em `.planning/`

**Nota:** A maioria das operações do GSD usa leituras diretas de arquivos ou caminhos explícitos, que funcionam independentemente do status do gitignore.

</search_behavior>

<setup_uncommitted_mode>

Para usar o modo sem commits:

1. **Definir config:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **Adicionar ao .gitignore:**
   ```
   .planning/
   ```

3. **Arquivos já rastreados:** Se `.planning/` estava rastreado anteriormente:
   ```bash
   git rm -r --cached .planning/
   git commit -m "chore: stop tracking planning docs"
   ```

4. **Merges de branches:** Ao usar `branching_strategy: phase` ou `milestone`, o workflow `complete-milestone` remove automaticamente os arquivos `.planning/` do staging antes dos commits de merge quando `commit_docs: false`.

</setup_uncommitted_mode>

<branching_strategy_behavior>

**Estratégias de Branching:**

| Estratégia | Quando o branch é criado | Escopo do branch | Ponto de merge |
|------------|--------------------------|------------------|----------------|
| `none` | Nunca | N/A | N/A |
| `phase` | No início do `execute-phase` | Fase única | Usuário faz merge após a fase |
| `milestone` | No primeiro `execute-phase` do milestone | Milestone inteiro | Em `complete-milestone` |

**Quando `git.branching_strategy: "none"` (padrão):**
- Todo trabalho é commitado na branch atual
- Comportamento padrão do GSD

**Quando `git.branching_strategy: "phase"`:**
- `execute-phase` cria/alterna para uma branch antes da execução
- Nome da branch a partir de `phase_branch_template` (ex.: `gsd/phase-03-authentication`)
- Todos os commits do plano vão para aquela branch
- Usuário faz merge das branches manualmente após a conclusão da fase
- `complete-milestone` oferece fazer merge de todas as branches de fase

**Quando `git.branching_strategy: "milestone"`:**
- O primeiro `execute-phase` do milestone cria a branch do milestone
- Nome da branch a partir de `milestone_branch_template` (ex.: `gsd/v1.0-mvp`)
- Todas as fases do milestone commitam na mesma branch
- `complete-milestone` oferece fazer merge da branch do milestone para main

**Variáveis de template:**

| Variável | Disponível em | Descrição |
|----------|---------------|-----------|
| `{phase}` | phase_branch_template | Número da fase com zero à esquerda (ex.: "03") |
| `{slug}` | Ambas | Nome em minúsculas com hifens |
| `{milestone}` | milestone_branch_template | Versão do milestone (ex.: "v1.0") |

**Verificando a configuração:**

Use `init execute-phase` que retorna toda a config como JSON:
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "1")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# A saída JSON inclui: branching_strategy, phase_branch_template, milestone_branch_template
```

Ou use `state load` para os valores de configuração:
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
# Extraia branching_strategy, phase_branch_template, milestone_branch_template do JSON
```

**Criação de branch:**

```bash
# Para estratégia phase
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# Para estratégia milestone
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**Opções de merge em complete-milestone:**

| Opção | Comando git | Resultado |
|-------|-------------|-----------|
| Squash merge (recomendado) | `git merge --squash` | Um único commit limpo por branch |
| Merge com histórico | `git merge --no-ff` | Preserva todos os commits individuais |
| Deletar sem merge | `git branch -D` | Descarta o trabalho da branch |
| Manter branches | (nenhum) | Tratamento manual depois |

O squash merge é recomendado — mantém o histórico da branch principal limpo enquanto preserva o histórico completo de desenvolvimento na branch (até que seja deletada).

**Casos de uso:**

| Estratégia | Melhor para |
|------------|-------------|
| `none` | Desenvolvimento solo, projetos simples |
| `phase` | Revisão de código por fase, rollback granular, colaboração em equipe |
| `milestone` | Branches de release, ambientes de staging, PR por versão |

</branching_strategy_behavior>

<complete_field_reference>

## Referência Completa de Campos

Gerada a partir de `CONFIG_DEFAULTS` (core.cjs) e `VALID_CONFIG_KEYS` (config.cjs).

### Campos Principais

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `model_profile` | string | `"balanced"` | `"quality"`, `"balanced"`, `"budget"`, `"inherit"` | Preset de seleção de modelo para subagentes |
| `mode` | string | `"interactive"` | `"interactive"`, `"yolo"` | Modo de operação: `"interactive"` exibe gates e confirmações; `"yolo"` executa autonomamente sem prompts |
| `granularity` | string | (nenhum) | `"coarse"`, `"standard"`, `"fine"` | Profundidade de planejamento para planos de fase (migrado do `depth` depreciado) |
| `commit_docs` | boolean | `true` | `true`, `false` | Commitar artifacts de .planning/ no git (auto-false se .planning/ estiver no gitignore) |
| `search_gitignored` | boolean | `false` | `true`, `false` | Incluir caminhos gitignored em buscas amplas com rg via `--no-ignore` |
| `phase_naming` | string | `"sequential"` | `"sequential"`, `"custom"` | Numeração de fases: auto-incremento ou IDs de string arbitrários |
| `project_code` | string\|null | `null` | Qualquer string curta | Prefixo para diretórios de fase (ex.: `"CK"` produz `CK-01-foundation`) |
| `response_language` | string\|null | `null` | Qualquer nome de idioma | Idioma para prompts voltados ao usuário (ex.: `"Portuguese"`, `"Japanese"`) |
| `context_window` | number | `200000` | `200000`, `1000000` | Tamanho da janela de contexto; defina `1000000` para modelos com contexto de 1M |
| `resolve_model_ids` | boolean\|string | `false` | `false`, `true`, `"omit"` | Mapear aliases de modelo para IDs completos do Claude; `"omit"` retorna string vazia |
| `context` | string\|null | `null` | `"dev"`, `"research"`, `"review"` | Perfil de contexto de execução que ajusta o comportamento do agente: `"dev"` para tarefas de desenvolvimento, `"research"` para investigação/exploração, `"review"` para fluxos de revisão de código |
| `review.models.<cli>` | string\|null | `null` | Qualquer string de ID de modelo | Override de modelo por CLI para /gsd-review (ex.: `review.models.gemini`). Usa o padrão da CLI quando null. |

### Campos de Workflow

Definidos via namespace `workflow.*` no config.json (ex.: `"workflow": { "research": true }`).

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `workflow.research` | boolean | `true` | `true`, `false` | Executar agente de pesquisa antes do planejamento |
| `workflow.plan_check` | boolean | `true` | `true`, `false` | Executar agente plan-checker para validar planos. _Alias:_ `plan_checker` é a forma de chave plana usada em `CONFIG_DEFAULTS`; `workflow.plan_check` é a forma canônica com namespace. |
| `workflow.verifier` | boolean | `true` | `true`, `false` | Executar agente de verificação após a execução |
| `workflow.nyquist_validation` | boolean | `true` | `true`, `false` | Habilitar gates de validação inspirados no princípio de Nyquist |
| `workflow.auto_prune_state` | boolean | `false` | `true`, `false` | Podar automaticamente entradas antigas do STATE.md ao concluir uma fase (mantém as 3 fases mais recentes) |
| `workflow.auto_advance` | boolean | `false` | `true`, `false` | Avançar automaticamente para a próxima fase após a conclusão |
| `workflow.node_repair` | boolean | `true` | `true`, `false` | Tentar reparo automático de nós de plano com falha |
| `workflow.node_repair_budget` | number | `2` | Qualquer inteiro positivo | Máximo de tentativas de reparo por nó com falha |
| `workflow.ai_integration_phase` | boolean | `true` | `true`, `false` | Executar /gsd-ai-integration-phase antes de planejar fases de sistemas de IA |
| `workflow.ui_phase` | boolean | `true` | `true`, `false` | Gerar UI-SPEC.md para fases de frontend |
| `workflow.ui_safety_gate` | boolean | `true` | `true`, `false` | Exigir aprovação do safety gate para mudanças de UI |
| `workflow.text_mode` | boolean | `false` | `true`, `false` | Usar listas numeradas em texto simples em vez de menus AskUserQuestion |
| `workflow.research_before_questions` | boolean | `false` | `true`, `false` | Executar pesquisa antes das perguntas interativas na fase de discuss |
| `workflow.discuss_mode` | string | `"discuss"` | `"discuss"`, `"assumptions"` | Modo padrão para discuss-phase: `"discuss"` executa questionamento interativo; `"assumptions"` analisa a codebase e levanta suposições |
| `workflow.skip_discuss` | boolean | `false` | `true`, `false` | Pular a fase de discuss completamente |
| `workflow.use_worktrees` | boolean | `true` | `true`, `false` | Executar agentes executor em git worktrees isolados |
| `workflow.subagent_timeout` | number | `300000` | Qualquer inteiro positivo (ms) | Timeout para tarefas de subagente paralelas (padrão: 5 minutos) |
| `workflow.inline_plan_threshold` | number | `2` | `0`–`10` | Planos com ≤N tarefas executam inline em vez de criar um subagente |
| `workflow.code_review` | boolean | `true` | `true`, `false` | Habilitar etapa de revisão de código embutida no workflow de ship |
| `workflow.code_review_depth` | string | `"standard"` | `"light"`, `"standard"`, `"deep"` | Nível de profundidade para análise de revisão de código no workflow de ship |
| `workflow._auto_chain_active` | boolean | `false` | `true`, `false` | Interno: rastreia se o encadeamento autônomo está ativo |

### Campos Git

Definidos via namespace `git.*` (ex.: `"git": { "branching_strategy": "phase" }`).

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `git.branching_strategy` | string | `"none"` | `"none"`, `"phase"`, `"milestone"` | Abordagem de branching do git para isolamento de fase/milestone |
| `git.base_branch` | string\|null | `null` (detecção automática) | Qualquer nome de branch | Branch de destino para PRs e merges; detecta automaticamente via `origin/HEAD` quando `null` |
| `git.phase_branch_template` | string | `"gsd/phase-{phase}-{slug}"` | Template com `{phase}`, `{slug}` | Template de nomenclatura de branch para estratégia `phase` |
| `git.milestone_branch_template` | string | `"gsd/{milestone}-{slug}"` | Template com `{milestone}`, `{slug}` | Template de nomenclatura de branch para estratégia `milestone` |
| `git.quick_branch_template` | string\|null | `null` | Template com `{slug}` | Template de branch opcional para execuções de quick-task |

### Campos de Busca e API

Ativam integrações de busca externas. Detectados automaticamente na criação do projeto quando as chaves de API estão presentes.

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `brave_search` | boolean | `false` | `true`, `false` | Habilitar busca web Brave para agente de pesquisa (requer `BRAVE_API_KEY`) |
| `firecrawl` | boolean | `false` | `true`, `false` | Habilitar scraping de páginas com Firecrawl (requer `FIRECRAWL_API_KEY`) |
| `exa_search` | boolean | `false` | `true`, `false` | Habilitar busca semântica Exa (requer `EXA_API_KEY`) |

### Campos de Features

Definidos via namespace `features.*` (ex.: `"features": { "thinking_partner": true }`).

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `features.thinking_partner` | boolean | `false` | `true`, `false` | Habilitar raciocínio estendido condicional em pontos de decisão do workflow (usado por discuss-phase e plan-phase para análise de tradeoffs arquiteturais) |
| `features.global_learnings` | boolean | `false` | `true`, `false` | Habilitar injeção de aprendizados globais de `~/.gsd/learnings/` nos prompts dos agentes |

### Campos de Hooks

Definidos via namespace `hooks.*` (ex.: `"hooks": { "context_warnings": true }`).

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `hooks.context_warnings` | boolean | `true` | `true`, `false` | Exibir avisos quando o orçamento de contexto é excedido |

### Campos de Learnings

Definidos via namespace `learnings.*` (ex.: `"learnings": { "max_inject": 5 }`). Usados junto com `features.global_learnings`.

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `learnings.max_inject` | number | `10` | Qualquer inteiro positivo | Número máximo de entradas de aprendizado global a injetar nos prompts dos agentes por sessão |

### Campos de Intel

Definidos via namespace `intel.*` (ex.: `"intel": { "enabled": true }`). Controla o sistema de inteligência de codebase consultável pelo `/gsd-intel`.

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `intel.enabled` | boolean | `false` | `true`, `false` | Habilitar sistema de inteligência de codebase consultável. Quando `true`, comandos `/gsd-intel` constroem e consultam um índice JSON em `.planning/intel/`. |

### Campos de Manager

Definidos via namespace `manager.*` (ex.: `"manager": { "flags": { "discuss": "--auto" } }`).

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `manager.flags.discuss` | string | `""` | Qualquer string de flags CLI | Flags passadas ao `/gsd-discuss-phase` pelo manager (ex.: `"--auto --analyze"`) |
| `manager.flags.plan` | string | `""` | Qualquer string de flags CLI | Flags passadas ao workflow de plan pelo manager |
| `manager.flags.execute` | string | `""` | Qualquer string de flags CLI | Flags passadas ao workflow de execute pelo manager |

### Campos Avançados

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `parallelization` | boolean\|object | `true` | `true`, `false`, `{ "enabled": true }` | Habilitar execução paralela de waves; a forma de objeto permite sub-chaves adicionais |
| `model_overrides` | object\|null | `null` | `{ "<agent-type>": "<model-id>" }` | Override de seleção de modelo por tipo de agente |
| `agent_skills` | object | `{}` | `{ "<agent-type>": "<skill-set>" }` | Atribuir conjuntos de habilidades a tipos específicos de agente |
| `sub_repos` | array | `[]` | Array de strings de caminho relativo | Diretórios filhos com repositórios `.git` independentes (detectados automaticamente) |

### Campos de Planejamento

Podem ser definidos no nível superior ou aninhados em `planning.*` (ex.: `"planning": { "commit_docs": false }`). Ambas as formas são equivalentes; o nível superior tem precedência se ambos existirem.

| Chave | Tipo | Padrão | Valores Permitidos | Descrição |
|-------|------|--------|-------------------|-----------|
| `planning.commit_docs` | boolean | `true` | `true`, `false` | Alias para `commit_docs` de nível superior |
| `planning.search_gitignored` | boolean | `false` | `true`, `false` | Alias para `search_gitignored` de nível superior |

---

## Interações entre Campos

Vários campos de configuração afetam uns aos outros ou acionam comportamentos especiais:

1. **Detecção automática de `commit_docs`** — Quando nenhum valor explícito está definido no config.json e `.planning/` está no `.gitignore`, `commit_docs` resolve automaticamente para `false`. Um `true` ou `false` explícito no config sempre sobrepõe a detecção automática.

2. **`branching_strategy` controla templates de branch** — Os campos `phase_branch_template` e `milestone_branch_template` só são usados quando `branching_strategy` está definido como `"phase"` ou `"milestone"`, respectivamente. Quando `branching_strategy` é `"none"`, todos os campos de template são ignorados.

3. **Limiar de `context_window` aciona comportamentos** — Quando `context_window >= 500000`, os workflows habilitam enriquecimento adaptativo de contexto: leituras completas de SUMMARYs de fases anteriores, injeção de contexto entre fases no plan-phase, e maior profundidade de leitura para referências de anti-padrões. Abaixo de 500000, apenas frontmatter e resumos são lidos.

4. **Polimorfismo de `parallelization`** — Aceita tanto um boolean simples quanto um objeto com campo `enabled`. `loadConfig()` normaliza qualquer forma para um boolean. `{ "enabled": true }` é equivalente a `true`.

5. **Chaves de API de busca e flags** — `brave_search`, `firecrawl` e `exa_search` são automaticamente definidos como `true` durante a criação do projeto se a chave de API correspondente for detectada (variável de ambiente ou arquivo `~/.gsd/<name>_api_key`). Definir como `true` sem a chave de API não tem efeito.

6. **Equivalência entre `planning.*` e nível superior** — `planning.commit_docs` e `commit_docs` são equivalentes; `planning.search_gitignored` e `search_gitignored` são equivalentes. Se ambos estiverem definidos, o valor de nível superior tem precedência.

7. **Migração de `depth` para `granularity`** — A chave depreciada `depth` (`quick`/`standard`/`comprehensive`) é automaticamente migrada para `granularity` (`coarse`/`standard`/`fine`) no carregamento da config e persistida de volta ao disco.

8. **Auto-sincronização de `sub_repos`** — A cada carregamento de config, o GSD escaneia diretórios filhos com `.git` e atualiza o array `sub_repos` se o sistema de arquivos mudou. O legado `multiRepo: true` é automaticamente migrado para um array `sub_repos` detectado.

---

## Exemplos de Configuração

### Mínima — Desenvolvedor Solo

```json
{
  "model_profile": "balanced",
  "commit_docs": true,
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "use_worktrees": false
  }
}
```

### Projeto em Equipe com Branching

```json
{
  "model_profile": "quality",
  "commit_docs": true,
  "project_code": "APP",
  "git": {
    "branching_strategy": "phase",
    "base_branch": "develop",
    "phase_branch_template": "gsd/phase-{phase}-{slug}"
  },
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true,
    "use_worktrees": true,
    "discuss_mode": "discuss"
  },
  "manager": {
    "flags": {
      "discuss": "",
      "plan": "",
      "execute": ""
    }
  },
  "response_language": "English"
}
```

### Codebase Grande — Contexto de 1M com Timeouts Estendidos

```json
{
  "model_profile": "quality",
  "context_window": 1000000,
  "commit_docs": true,
  "project_code": "MEGA",
  "phase_naming": "sequential",
  "git": {
    "branching_strategy": "milestone",
    "milestone_branch_template": "gsd/{milestone}-{slug}"
  },
  "workflow": {
    "research": true,
    "plan_check": true,
    "verifier": true,
    "nyquist_validation": true,
    "subagent_timeout": 600000,
    "use_worktrees": true,
    "node_repair": true,
    "node_repair_budget": 3,
    "auto_advance": true
  },
  "brave_search": true,
  "hooks": {
    "context_warnings": true
  }
}
```

</complete_field_reference>

</planning_config>
