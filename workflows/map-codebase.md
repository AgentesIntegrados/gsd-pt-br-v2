<purpose>
Orquestrar agentes mapeadores de codebase em paralelo para analisar o projeto e produzir documentos estruturados em .planning/codebase/

Cada agente tem contexto próprio, explora uma área específica e **escreve os documentos diretamente**. O orquestrador só recebe confirmação + contagem de linhas, depois escreve um resumo.

Saída: pasta .planning/codebase/ com 7 documentos estruturados sobre o estado da codebase.
</purpose>

<available_agent_types>
Tipos válidos de subagentes GSD (use os nomes exatos — não use 'general-purpose' como alternativa):
- gsd-codebase-mapper — Mapeia estrutura e dependências do projeto
</available_agent_types>

<philosophy>
**Por que usar agentes mapeadores dedicados:**
- Contexto isolado por domínio (sem contaminação de tokens)
- Agentes escrevem os documentos diretamente (sem transferência de contexto ao orquestrador)
- Orquestrador apenas resume o que foi criado (uso mínimo de contexto)
- Execução mais rápida (agentes rodam simultaneamente)

**Qualidade do documento acima do tamanho:**
Inclua detalhes suficientes para que o documento seja útil como referência. Priorize exemplos práticos (especialmente padrões de código) em vez de brevidade arbitrária.

**Sempre inclua caminhos de arquivo:**
Os documentos são material de referência para o Claude ao planejar/executar. Sempre inclua caminhos de arquivo reais formatados com backticks: `src/services/user.ts`.
</philosophy>

<process>

<step name="init_context" priority="first">
Carregue o contexto de mapeamento da codebase:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init map-codebase)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_MAPPER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-codebase-mapper 2>/dev/null)
```

Extraia do JSON de init: `mapper_model`, `commit_docs`, `codebase_dir`, `existing_maps`, `has_maps`, `codebase_dir_exists`, `subagent_timeout`.
</step>

<step name="check_existing">
Verifique se .planning/codebase/ já existe usando `has_maps` do contexto de init.

Se `codebase_dir_exists` for true:
```bash
ls -la .planning/codebase/
```

**Se existir:**

```
.planning/codebase/ já existe com estes documentos:
[Lista de arquivos encontrados]

O que fazer agora?
1. Atualizar completamente - Excluir existente e remapear a codebase
2. Atualizar parcialmente - Manter existente, atualizar apenas documentos específicos
3. Manter - Usar o mapa de codebase existente como está
```

Aguarde a resposta do usuário.

Se "Atualizar completamente": Exclua .planning/codebase/, continue para create_structure
Se "Atualizar parcialmente": Pergunte quais documentos atualizar, continue para spawn_agents (filtrado)
Se "Manter": Encerre o workflow

**Se não existir:**
Continue para create_structure.
</step>

<step name="create_structure">
Crie o diretório .planning/codebase/:

```bash
mkdir -p .planning/codebase
```

**Arquivos de saída esperados:**
- STACK.md (do mapeador de tecnologia)
- INTEGRATIONS.md (do mapeador de tecnologia)
- ARCHITECTURE.md (do mapeador de arquitetura)
- STRUCTURE.md (do mapeador de arquitetura)
- CONVENTIONS.md (do mapeador de qualidade)
- TESTING.md (do mapeador de qualidade)
- CONCERNS.md (do mapeador de preocupações)

Continue para spawn_agents.
</step>

<step name="detect_runtime_capabilities">
Antes de iniciar os agentes, detecte se o runtime atual suporta a ferramenta `Task` para delegação a subagentes.

**Como detectar:** Verifique se você tem acesso a uma ferramenta `Task` (pode ser `Task` ou `task` dependendo do runtime). Se você NÃO tiver uma ferramenta `Task`/`task` (ou só tiver ferramentas como `browser_subagent`, que é para navegação web, NÃO para análise de código):

→ **Pule `spawn_agents` e `collect_confirmations`** — vá diretamente para `sequential_mapping`.

**CRÍTICO:** Nunca use `browser_subagent` ou `Explore` como substituto de `Task`. A ferramenta `browser_subagent` é exclusivamente para interação com páginas web e falhará para análise de codebase. Se `Task` estiver indisponível, realize o mapeamento sequencialmente no contexto atual.
</step>

<step name="spawn_agents" condition="A ferramenta Task está disponível">
Inicie 4 agentes gsd-codebase-mapper em paralelo.

Use a ferramenta Task com `subagent_type="gsd-codebase-mapper"`, `model="{mapper_model}"`, e `run_in_background=true` para execução paralela.

**CRÍTICO:** Use o agente dedicado `gsd-codebase-mapper`, NÃO `Explore` ou `browser_subagent`. O agente mapeador escreve os documentos diretamente.

**Agente 1: Foco em Tecnologia**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase tech stack",
  prompt="Focus: tech

Analyze this codebase for technology stack and external integrations.

Write these documents to .planning/codebase/:
- STACK.md - Languages, runtime, frameworks, dependencies, configuration
- INTEGRATIONS.md - External APIs, databases, auth providers, webhooks

Explore thoroughly. Write documents directly using templates. Return confirmation only.
${AGENT_SKILLS_MAPPER}"
)
```

**Agente 2: Foco em Arquitetura**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase architecture",
  prompt="Focus: arch

Analyze this codebase architecture and directory structure.

Write these documents to .planning/codebase/:
- ARCHITECTURE.md - Pattern, layers, data flow, abstractions, entry points
- STRUCTURE.md - Directory layout, key locations, naming conventions

Explore thoroughly. Write documents directly using templates. Return confirmation only.
${AGENT_SKILLS_MAPPER}"
)
```

**Agente 3: Foco em Qualidade**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase conventions",
  prompt="Focus: quality

Analyze this codebase for coding conventions and testing patterns.

Write these documents to .planning/codebase/:
- CONVENTIONS.md - Code style, naming, patterns, error handling
- TESTING.md - Framework, structure, mocking, coverage

Explore thoroughly. Write documents directly using templates. Return confirmation only.
${AGENT_SKILLS_MAPPER}"
)
```

**Agente 4: Foco em Preocupações**

```
Task(
  subagent_type="gsd-codebase-mapper",
  model="{mapper_model}",
  run_in_background=true,
  description="Map codebase concerns",
  prompt="Focus: concerns

Analyze this codebase for technical debt, known issues, and areas of concern.

Write this document to .planning/codebase/:
- CONCERNS.md - Tech debt, bugs, security, performance, fragile areas

Explore thoroughly. Write document directly using template. Return confirmation only.
${AGENT_SKILLS_MAPPER}"
)
```

Continue para collect_confirmations.
</step>

<step name="collect_confirmations">
Aguarde a conclusão de todos os 4 agentes usando a ferramenta TaskOutput.

**Para cada task_id retornado pelas chamadas à ferramenta Agent acima:**
```
Ferramenta TaskOutput:
  task_id: "{task_id do resultado do Agent}"
  block: true
  timeout: {subagent_timeout do contexto de init, padrão 300000}
```

> O timeout é configurável via `workflow.subagent_timeout` em `.planning/config.json` (milissegundos). Padrão: 300000 (5 minutos). Aumente para codebases grandes ou modelos mais lentos.

Chame TaskOutput para todos os 4 agentes em paralelo (uma única mensagem com 4 chamadas TaskOutput).

Assim que todas as chamadas TaskOutput retornarem, leia o arquivo de saída de cada agente para coletar as confirmações.

**Formato de confirmação esperado de cada agente:**
```
## Mapping Complete

**Focus:** {focus}
**Documents written:**
- `.planning/codebase/{DOC1}.md` ({N} lines)
- `.planning/codebase/{DOC2}.md` ({N} lines)

Ready for orchestrator summary.
```

**O que você recebe:** Apenas caminhos de arquivo e contagem de linhas. NÃO o conteúdo dos documentos.

Se algum agente falhar, registre a falha e continue com os documentos bem-sucedidos.

Continue para verify_output.
</step>

<step name="sequential_mapping" condition="A ferramenta Task NÃO está disponível (ex: Antigravity, Gemini CLI, Codex)">
Quando a ferramenta `Task` estiver indisponível, realize o mapeamento da codebase sequencialmente no contexto atual. Isso substitui `spawn_agents` e `collect_confirmations`.

**IMPORTANTE:** NÃO use `browser_subagent`, `Explore` ou qualquer ferramenta baseada em navegador. Use apenas ferramentas do sistema de arquivos (Read, Bash, Write, Grep, Glob, list_dir, view_file, grep_search ou ferramentas equivalentes disponíveis no seu runtime).

Execute todas as 4 passagens de mapeamento sequencialmente:

**Passagem 1: Foco em Tecnologia**
- Explore package.json/Cargo.toml/go.mod/requirements.txt, arquivos de config, árvores de dependências
- Escreva `.planning/codebase/STACK.md` — Linguagens, runtime, frameworks, dependências, configuração
- Escreva `.planning/codebase/INTEGRATIONS.md` — APIs externas, bancos de dados, provedores de auth, webhooks

**Passagem 2: Foco em Arquitetura**
- Explore estrutura de diretórios, pontos de entrada, limites de módulos, fluxo de dados
- Escreva `.planning/codebase/ARCHITECTURE.md` — Padrão, camadas, fluxo de dados, abstrações, pontos de entrada
- Escreva `.planning/codebase/STRUCTURE.md` — Layout de diretórios, localizações-chave, convenções de nomenclatura

**Passagem 3: Foco em Qualidade**
- Explore estilo de código, padrões de tratamento de erros, arquivos de teste, config de CI
- Escreva `.planning/codebase/CONVENTIONS.md` — Estilo de código, nomenclatura, padrões, tratamento de erros
- Escreva `.planning/codebase/TESTING.md` — Framework, estrutura, mocking, cobertura

**Passagem 4: Foco em Preocupações**
- Explore TODOs, problemas conhecidos, áreas frágeis, padrões de segurança
- Escreva `.planning/codebase/CONCERNS.md` — Dívida técnica, bugs, segurança, performance, áreas frágeis

Use os mesmos templates de documento que o agente `gsd-codebase-mapper`. Inclua caminhos de arquivo reais formatados com backticks.

Continue para verify_output.
</step>

<step name="verify_output">
Verifique se todos os documentos foram criados com sucesso:

```bash
ls -la .planning/codebase/
wc -l .planning/codebase/*.md
```

**Lista de verificação:**
- Todos os 7 documentos existem
- Nenhum documento vazio (cada um deve ter >20 linhas)

Se algum documento estiver faltando ou vazio, registre qual agente pode ter falhado.

Continue para scan_for_secrets.
</step>

<step name="scan_for_secrets">
**VERIFICAÇÃO DE SEGURANÇA CRÍTICA:** Escaneie os arquivos de saída em busca de segredos vazados acidentalmente antes de fazer o commit.

Execute a detecção de padrões de segredos:

```bash
# Check for common API key patterns in generated docs
grep -E '(sk-[a-zA-Z0-9]{20,}|sk_live_[a-zA-Z0-9]+|sk_test_[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|glpat-[a-zA-Z0-9_-]+|AKIA[A-Z0-9]{16}|xox[baprs]-[a-zA-Z0-9-]+|-----BEGIN.*PRIVATE KEY|eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.)' .planning/codebase/*.md 2>/dev/null && SECRETS_FOUND=true || SECRETS_FOUND=false
```

**Se SECRETS_FOUND=true:**

```
⚠️  ALERTA DE SEGURANÇA: Possíveis segredos detectados nos documentos da codebase!

Padrões semelhantes a chaves de API ou tokens encontrados em:
[mostrar saída do grep]

Isso exporia credenciais se for feito o commit.

**Ação necessária:**
1. Revise o conteúdo sinalizado acima
2. Se forem segredos reais, eles devem ser removidos antes do commit
3. Considere adicionar arquivos sensíveis às permissões "Deny" do Claude Code

Pausando antes do commit. Responda "seguro para prosseguir" se o conteúdo sinalizado não for realmente sensível, ou edite os arquivos primeiro.
```

Aguarde a confirmação do usuário antes de continuar para commit_codebase_map.

**Se SECRETS_FOUND=false:**

Continue para commit_codebase_map.
</step>

<step name="commit_codebase_map">
Faça o commit do mapa da codebase:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: map existing codebase" --files .planning/codebase/*.md
```

Continue para offer_next.
</step>

<step name="offer_next">
Apresente o resumo de conclusão e os próximos passos.

**Obtenha a contagem de linhas:**
```bash
wc -l .planning/codebase/*.md
```

**Formato de saída:**

```
Mapeamento da codebase concluído.

Criado em .planning/codebase/:
- STACK.md ([N] linhas) - Tecnologias e dependências
- ARCHITECTURE.md ([N] linhas) - Design do sistema e padrões
- STRUCTURE.md ([N] linhas) - Layout de diretórios e organização
- CONVENTIONS.md ([N] linhas) - Estilo de código e padrões
- TESTING.md ([N] linhas) - Estrutura de testes e práticas
- INTEGRATIONS.md ([N] linhas) - Serviços externos e APIs
- CONCERNS.md ([N] linhas) - Dívida técnica e problemas


---

## ▶ Próximo Passo

**Inicializar projeto** — usar contexto da codebase para planejamento

`/clear` e depois:

`/gsd-new-project`

---

**Também disponível:**
- Refazer o mapeamento: `/gsd-map-codebase`
- Revisar arquivo específico: `cat .planning/codebase/STACK.md`
- Editar qualquer documento antes de prosseguir

---
```

Encerre o workflow.
</step>

</process>

<success_criteria>
- Diretório .planning/codebase/ criado
- Se a ferramenta Task estiver disponível: 4 agentes gsd-codebase-mapper paralelos iniciados com run_in_background=true
- Se a ferramenta Task NÃO estiver disponível: 4 passagens de mapeamento sequenciais realizadas inline (nunca usando browser_subagent)
- Todos os 7 documentos da codebase existem
- Nenhum documento vazio (cada um deve ter >20 linhas)
- Resumo claro de conclusão com contagem de linhas
- Usuário recebe próximos passos claros no estilo GSD
</success_criteria>
