<purpose>
Executar todos os planos de uma fase usando execução paralela baseada em ondas. O orquestrador permanece enxuto — delega a execução dos planos para subagentes.
</purpose>

<core_principle>
O orquestrador coordena, não executa. Cada subagente carrega o contexto completo de execute-plan. Orquestrador: descobrir planos → analisar dependências → agrupar ondas → iniciar agentes → tratar pontos de verificação → coletar resultados.
</core_principle>

<runtime_compatibility>
**A geração de subagentes é específica do runtime:**
- **Claude Code:** Usa `Task(subagent_type="gsd-executor", ...)` — bloqueia até completar, retorna resultado
- **Copilot:** A geração de subagentes não retorna sinais de conclusão de forma confiável. **Padrão para
  execução sequencial inline**: leia e siga execute-plan.md diretamente para cada plano
  em vez de gerar agentes paralelos. Tente geração paralela apenas se o usuário
  solicitar explicitamente — e nesse caso, dependa do fallback de verificação pontual no passo 3
  para detectar a conclusão.
- **Outros runtimes:** Se a ferramenta `Task`/`task` estiver indisponível, use execução sequencial inline como
  fallback. Verifique a disponibilidade da ferramenta em runtime em vez de assumir baseado no nome do runtime.

**Regra de fallback:** Se um agente gerado conclui seu trabalho (commits visíveis, SUMMARY.md existe) mas
o orquestrador nunca recebe o sinal de conclusão, trate como bem-sucedido com base em verificações pontuais
e continue para a próxima onda/plano. Nunca bloqueie indefinidamente aguardando um sinal — sempre verifique
via sistema de arquivos e estado git.
</runtime_compatibility>

<required_reading>
Leia STATE.md antes de qualquer operação para carregar o contexto do projeto.

@$HOME/.claude/get-shit-done/references/agent-contracts.md
@$HOME/.claude/get-shit-done/references/context-budget.md
@$HOME/.claude/get-shit-done/references/gates.md
</required_reading>

<available_agent_types>
Estes são os tipos válidos de subagentes GSD registrados em .claude/agents/ (ou equivalente para o seu runtime).
Sempre use o nome exato desta lista — não recorra a 'general-purpose' ou outros tipos embutidos:

- gsd-executor — Executa tarefas de plano, faz commits, cria SUMMARY.md
- gsd-verifier — Verifica a conclusão da fase, verifica gates de qualidade
- gsd-planner — Cria planos detalhados a partir do escopo da fase
- gsd-phase-researcher — Pesquisa abordagens técnicas para uma fase
- gsd-plan-checker — Revisa a qualidade do plano antes da execução
- gsd-debugger — Diagnostica e corrige problemas
- gsd-codebase-mapper — Mapeia a estrutura do projeto e dependências
- gsd-integration-checker — Verifica a integração entre fases
- gsd-nyquist-auditor — Valida a cobertura de verificação
- gsd-ui-researcher — Pesquisa abordagens de UI/UX
- gsd-ui-checker — Revisa a qualidade da implementação de UI
- gsd-ui-auditor — Audita a UI contra os requisitos de design
</available_agent_types>

<process>

<step name="parse_args" priority="first">
Analise `$ARGUMENTS` antes de carregar qualquer contexto:

- Primeiro token posicional → `PHASE_ARG`
- `--wave N` opcional → `WAVE_FILTER`
- `--gaps-only` opcional mantém seu significado atual
- `--cross-ai` opcional → `CROSS_AI_FORCE=true` (force todos os planos pela execução cross-AI)
- `--no-cross-ai` opcional → `CROSS_AI_DISABLED=true` (desabilita cross-AI para esta execução, substitui configuração e frontmatter)

Se `--wave` estiver ausente, preserve o comportamento atual de executar todas as ondas incompletas na fase.
</step>

<step name="initialize" priority="first">
Carregue todo o contexto em uma chamada:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-executor 2>/dev/null)
```

Analise o JSON para: `executor_model`, `verifier_model`, `commit_docs`, `parallelization`, `branching_strategy`, `branch_name`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `plans`, `incomplete_plans`, `plan_count`, `incomplete_count`, `state_exists`, `roadmap_exists`, `phase_req_ids`, `response_language`.

**Se `response_language` estiver definido:** Inclua `response_language: {value}` em todos os prompts de subagentes gerados para que qualquer saída voltada ao usuário fique no idioma configurado.

Leia a configuração do worktree:

```bash
USE_WORKTREES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.use_worktrees 2>/dev/null || echo "true")
```

Se o projeto usar submódulos git, o isolamento de worktree é ignorado independentemente da configuração `workflow.use_worktrees` — o protocolo de commit do executor não consegue tratar corretamente commits de submódulos dentro de worktrees isoladas. A execução sequencial trata submódulos de forma transparente.

```bash
if [ -f .gitmodules ]; then
  echo "[worktree] Projeto com submódulo detectado (.gitmodules existe) — revertendo para execução sequencial"
  USE_WORKTREES=false
fi
```

Quando `USE_WORKTREES` for `false`, todos os agentes executor rodam sem `isolation="worktree"` — eles executam sequencialmente na árvore de trabalho principal em vez de em worktrees paralelas.

Leia o tamanho da janela de contexto para enriquecimento adaptativo de prompt:

```bash
CONTEXT_WINDOW=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get context_window 2>/dev/null || echo "200000")
```

Quando `CONTEXT_WINDOW >= 500000` (modelos de classe 1M), prompts de subagentes incluem contexto mais rico:
- Agentes executor recebem arquivos SUMMARY.md de ondas anteriores e CONTEXT.md/RESEARCH.md da fase
- Agentes verifier recebem todos os arquivos PLAN.md, SUMMARY.md, CONTEXT.md mais REQUIREMENTS.md
- Isso habilita consciência entre fases e verificação com histórico

Quando `CONTEXT_WINDOW < 200000` (modelos sub-200K), prompts de subagentes são reduzidos para diminuir a sobrecarga estática:
- Agentes executor omitem exemplos estendidos de regras de desvio e exemplos de ponto de verificação do prompt inline — carregue sob demanda via @$HOME/.claude/get-shit-done/references/executor-examples.md
- Agentes planner omitem listas estendidas de anti-padrões e exemplos de especificidade do prompt inline — carregue sob demanda via @$HOME/.claude/get-shit-done/references/planner-antipatterns.md
- Regras centrais e lógica de decisão permanecem inline; apenas exemplos detalhados e listas de casos extremos são extraídos
- Isso reduz a sobrecarga estática do executor em ~40% preservando a corretude comportamental

**Se `phase_found` for false:** Erro — diretório da fase não encontrado.
**Se `plan_count` for 0:** Erro — nenhum plano encontrado na fase.
**Se `state_exists` for false mas `.planning/` existir:** Ofereça reconstrução ou continuação.

Quando `parallelization` for false, planos dentro de uma onda executam sequencialmente.

**Detecção de runtime para Copilot:**
Verifique se o runtime atual é Copilot testando o padrão de agente `@gsd-executor`
ou ausência da API de subagente `Task()`. Se rodando no Copilot, force execução sequencial inline
independentemente da configuração `parallelization` — os sinais de conclusão de subagentes do Copilot
são não confiáveis (veja `<runtime_compatibility>`). Defina `COPILOT_SEQUENTIAL=true`
internamente e pule o passo `execute_waves` em favor do caminho inline de `check_interactive_mode`
para cada plano.

**OBRIGATÓRIO — Sincronize a flag chain com a intenção.** Se o usuário invocou manualmente (sem `--auto`), limpe a flag chain efêmera de qualquer cadeia `--auto` interrompida anteriormente. Isso evita que `_auto_chain_active: true` obsoleto cause avanço automático indesejado. Isso NÃO toca `workflow.auto_advance` (a preferência persistente de configurações do usuário). Você DEVE executar este bloco bash antes de qualquer leitura de configuração:
```bash
# OBRIGATÓRIO: evita cadeia auto obsoleta de execuções anteriores com --auto
if [[ ! "$ARGUMENTS" =~ --auto ]]; then
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
fi
```
</step>

<step name="check_blocking_antipatterns" priority="first">
**OBRIGATÓRIO — Verifique anti-padrões bloqueantes antes de qualquer outro trabalho.**

Procure por `.continue-here.md` no diretório da fase atual:

```bash
ls ${phase_dir}/.continue-here.md 2>/dev/null || true
```

Se `.continue-here.md` existir, analise sua tabela de "Critical Anti-Patterns" para linhas com `severity` = `blocking`.

**Se um ou mais anti-padrões `blocking` forem encontrados:**

Este passo não pode ser pulado. Antes de prosseguir para `check_interactive_mode` ou qualquer outro passo, o agente deve demonstrar compreensão de cada anti-padrão bloqueante respondendo às três perguntas para cada um:

1. **O que é este anti-padrão?** — Descreva com suas próprias palavras, sem citar o handoff.
2. **Como ele se manifestou?** — Explique a falha específica que fez com que fosse registrado.
3. **Qual mecanismo estrutural (não um reconhecimento) o previne?** — Nomeie o passo concreto, item de checklist ou mecanismo de aplicação que impede a recorrência.

Escreva essas respostas inline antes de continuar. Se um anti-padrão bloqueante não puder ser respondido a partir do contexto em `.continue-here.md`, pare e peça ao usuário esclarecimentos.

**Se `.continue-here.md` não existir, ou nenhuma linha `blocking` for encontrada:** Prossiga diretamente para `check_interactive_mode`.
</step>

<step name="check_interactive_mode">
**Analise a flag `--interactive` de $ARGUMENTS.**

**Se a flag `--interactive` estiver presente:** Alterne para o modo de execução interativo.

O modo interativo executa planos sequencialmente **inline** (sem geração de subagentes) com
pontos de verificação do usuário entre tarefas. O usuário pode revisar, modificar ou redirecionar o trabalho a qualquer momento.

**Fluxo de execução interativa:**

1. Carregue o inventário de planos normalmente (discover_and_group_plans)
2. Para cada plano (sequencialmente, ignorando o agrupamento por ondas):

   a. **Apresente o plano ao usuário:**
      ```
      ## Plano {plan_id}: {plan_name}

      Objetivo: {do arquivo de plano}
      Tarefas: {task_count}

      Opções:
      - Executar (prosseguir com todas as tarefas)
      - Revisar primeiro (mostrar detalhamento de tarefas antes de começar)
      - Pular (ir para o próximo plano)
      - Parar (encerrar a execução, salvar progresso)
      ```

   b. **Se "Revisar primeiro":** Leia e exiba o arquivo completo do plano. Pergunte novamente: Executar, Modificar, Pular.

   c. **Se "Executar":** Leia e siga `$HOME/.claude/get-shit-done/workflows/execute-plan.md` **inline**
      (NÃO gere um subagente). Execute tarefas uma de cada vez.

   d. **Após cada tarefa:** Pause brevemente. Se o usuário intervier (digitar qualquer coisa), pare e trate
      o feedback antes de continuar. Caso contrário, prossiga para a próxima tarefa.

   e. **Após o plano completo:** Mostre os resultados, faça commit, crie SUMMARY.md, depois apresente o próximo plano.

3. Após todos os planos: prossiga para verificação (igual ao modo normal).

**Benefícios do modo interativo:**
- Sem sobrecarga de subagentes — uso de tokens dramaticamente menor
- O usuário detecta erros cedo — economiza ciclos custosos de verificação
- Mantém a estrutura de rastreamento/planejamento do GSD
- Melhor para: fases pequenas, correções de bugs, lacunas de verificação, aprender GSD

**Pule para o passo handle_branching** (planos interativos executam inline após o agrupamento).
</step>

<step name="handle_branching">
Verifique `branching_strategy` do init:

**"none":** Pule, continue no branch atual.

**"phase" ou "milestone":** Use o `branch_name` pré-computado do init:
```bash
git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
```

Todos os commits subsequentes vão para este branch. O usuário cuida do merge.
</step>

<step name="validate_phase">
Do JSON init: `phase_dir`, `plan_count`, `incomplete_count`.

Relate: "Encontrados {plan_count} planos em {phase_dir} ({incomplete_count} incompletos)"

**Atualize STATE.md para início da fase:**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state begin-phase --phase "${PHASE_NUMBER}" --name "${PHASE_NAME}" --plans "${PLAN_COUNT}"
```
Isso atualiza Status, Última Atividade, Foco atual, Posição atual e contagens de planos em STATE.md para que o frontmatter e o texto do corpo reflitam a fase ativa imediatamente.
</step>

<step name="discover_and_group_plans">
Carregue o inventário de planos com agrupamento por ondas em uma chamada:

```bash
PLAN_INDEX=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase-plan-index "${PHASE_NUMBER}")
```

Analise o JSON para: `phase`, `plans[]` (cada um com `id`, `wave`, `autonomous`, `objective`, `files_modified`, `task_count`, `has_summary`), `waves` (mapa de número de onda → IDs de plano), `incomplete`, `has_checkpoints`.

**Filtragem:** Pule planos onde `has_summary: true`. Se `--gaps-only`: também pule planos não gap_closure. Se `WAVE_FILTER` estiver definido: também pule planos cujo `wave` não seja igual a `WAVE_FILTER`.

**Verificação de segurança de onda:** Se `WAVE_FILTER` estiver definido e ainda houver planos incompletos em qualquer onda inferior que correspondam ao modo de execução atual, PARE e diga ao usuário para terminar as ondas anteriores primeiro. Não deixe a Onda 2+ executar enquanto planos de ondas pré-requisito anteriores estejam incompletos.

Se todos filtrados: "Nenhum plano incompleto correspondente" → saia.

Relate:
```
## Plano de Execução

**Fase {X}: {Nome}** — {total_plans} planos correspondentes em {wave_count} onda(s)

{Se WAVE_FILTER estiver definido: `Filtro de onda ativo: executando apenas a Onda {WAVE_FILTER}`.}

| Onda | Planos | O que constrói |
|------|--------|----------------|
| 1 | 01-01, 01-02 | {dos objetivos do plano, 3-8 palavras} |
| 2 | 01-03 | ... |
```
</step>

<step name="cross_ai_delegation">
**Passo opcional 2.5 — Delegar planos para um runtime externo de IA.**

Este passo executa após a descoberta de planos e antes da execução normal por ondas. Ele identifica planos
que devem ser delegados a um comando externo de IA e os executa via entrega de prompt por stdin.
Planos tratados aqui são removidos da lista de planos do execute_waves para que o executor normal os pule.

**Lógica de ativação:**

1. Se `CROSS_AI_DISABLED` for true (flag `--no-cross-ai`): pule este passo completamente.
2. Se `CROSS_AI_FORCE` for true (flag `--cross-ai`): marque TODOS os planos incompletos para execução cross-AI.
3. Caso contrário: verifique o frontmatter de cada plano por `cross_ai: true` E verifique se a configuração
   `workflow.cross_ai_execution` é `true`. Planos correspondendo a ambas as condições são marcados para cross-AI.

```bash
CROSS_AI_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.cross_ai_execution --default false 2>/dev/null)
CROSS_AI_CMD=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.cross_ai_command --default "" 2>/dev/null)
CROSS_AI_TIMEOUT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.cross_ai_timeout --default 300 2>/dev/null)
```

**Se nenhum plano for marcado para cross-AI:** Pule para execute_waves.

**Se planos forem marcados mas `cross_ai_command` estiver vazio:** Erro — diga ao usuário para definir
`workflow.cross_ai_command` via `gsd-tools.cjs config-set workflow.cross_ai_command "<comando>"`.

**Para cada plano cross-AI (sequencialmente):**

1. **Construa o prompt de tarefa** a partir do arquivo de plano:
   - Extraia as seções `<objective>` e `<tasks>` do PLAN.md
   - Adicione contexto do PROJECT.md (nome do projeto, descrição, stack tecnológico)
   - Formate como um prompt de execução autocontido

2. **Verifique a árvore de trabalho suja antes da execução:**
   ```bash
   if ! git diff --quiet HEAD 2>/dev/null; then
     echo "AVISO: árvore de trabalho suja detectada — o comando externo de IA pode produzir mudanças não commitadas que conflitem com modificações existentes"
   fi
   ```

3. **Execute o comando externo** a partir da raiz do projeto, escrevendo o prompt para stdin.
   Nunca interpole o prompt no shell — sempre passe via stdin para evitar injeção:
   ```bash
   echo "$TASK_PROMPT" | timeout "${CROSS_AI_TIMEOUT}s" ${CROSS_AI_CMD} > "$CANDIDATE_SUMMARY" 2>"$ERROR_LOG"
   EXIT_CODE=$?
   ```

4. **Avalie o resultado:**

   **Sucesso (exit 0 + resumo válido):**
   - Leia `$CANDIDATE_SUMMARY` e valide que contém conteúdo significativo
     (não vazio, tem pelo menos um título e descrição — uma estrutura válida de SUMMARY.md)
   - Escreva como o arquivo SUMMARY.md do plano
   - Atualize o status do plano em STATE.md para completo
   - Atualize o progresso no ROADMAP.md
   - Marque o plano como tratado — pule-o em execute_waves

   **Falha (exit não-zero ou resumo inválido):**
   - Exiba a saída de erro e o código de saída
   - Avise: "O comando externo pode ter deixado mudanças não commitadas ou edições parciais
     na árvore de trabalho. Revise `git status` e `git diff` antes de prosseguir."
   - Ofereça três escolhas:
     - **retry** — execute o mesmo plano pelo cross-AI novamente
     - **skip** — volte para o executor normal para este plano (re-adicione à lista do execute_waves)
     - **abort** — pare a execução completamente, preserve o estado para retomada

5. **Após todos os planos cross-AI processados:** Remova os planos tratados com sucesso da lista de planos incompletos para que execute_waves os pule. Quaisquer planos ignorados-para-fallback permanecem na lista para processamento normal pelo executor.
</step>

<step name="execute_waves">
Execute cada onda selecionada em sequência. Dentro de uma onda: paralelo se `PARALLELIZATION=true`, sequencial se `false`.

**Para cada onda:**

1. **Verificação de sobreposição de files_modified dentro da onda (ANTES de gerar):**

   Antes de gerar quaisquer agentes para esta onda, inspecione a lista `files_modified` de todos os planos
   na onda. Verifique cada par de planos na onda — se quaisquer dois planos compartilharem até mesmo um arquivo
   em suas listas `files_modified`, esses planos têm uma dependência implícita e NÃO DEVEM rodar
   em paralelo.

   **Algoritmo de detecção (pseudocódigo):**
   ```
   seen_files = {}
   overlapping_plans = []
   para cada plano em wave_plans:
     para cada arquivo em plan.files_modified:
       se arquivo em seen_files:
         overlapping_plans.add(plan, seen_files[file])  # ambos os planos sobrepõem neste arquivo
       senão:
         seen_files[file] = plan
   ```

   **Se sobreposição for detectada:**
   - Avise o usuário:
     ```
     ⚠ Sobreposição de files_modified dentro da onda detectada na Onda {N}:
       Plano {A} e Plano {B} ambos modificam {file}
       Executando estes planos sequencialmente para evitar conflitos de worktree paralela.
     ```
   - Substitua `PARALLELIZATION` por `false` apenas para esta onda — execute todos os planos na onda
     sequencialmente independentemente da configuração global de paralelização.
   - Este é uma rede de segurança para planos incorretamente atribuídos à mesma onda.
     O planejador deveria ter detectado isso; sinalize-o como um defeito de planejamento para que o usuário possa
     replanejar a fase se desejar.

   **Se sem sobreposição:** prossiga normalmente (paralelo se `PARALLELIZATION=true`).

2. **Descreva o que está sendo construído (ANTES de gerar):**

   Leia o `<objective>` de cada plano. Extraia o que está sendo construído e por quê.

   ```
   ---
   ## Onda {N}

   **{ID do Plano}: {Nome do Plano}**
   {2-3 frases: o que isso constrói, abordagem técnica, por que importa}

   Gerando {count} agente(s)...
   ---
   ```

   - Ruim: "Executando plano de geração de terreno"
   - Bom: "Gerador de terreno procedural usando ruído Perlin — cria mapas de altura, zonas de bioma e malhas de colisão. Necessário antes que a física de veículo possa interagir com o chão."

3. **Gere agentes executor:**

   Passe apenas caminhos — executores leem os arquivos eles mesmos com sua janela de contexto fresca.
   Para modelos de 200K, isso mantém o contexto do orquestrador enxuto (~10-15%).
   Para modelos de 1M+ (Opus 4.6, Sonnet 4.6), contexto mais rico pode ser passado diretamente.

   **Modo worktree** (`USE_WORKTREES` não é `false`):

   Antes de gerar, capture o HEAD atual:
   ```bash
   EXPECTED_BASE=$(git rev-parse HEAD)
   ```

   **Despacho sequencial para execução paralela (ondas com 2+ agentes):**
   Ao gerar múltiplos agentes em uma onda, despache cada chamada `Task()` **uma de cada vez
   com `run_in_background: true`** — NÃO envie todas as chamadas Task em uma única mensagem.
   `git worktree add` adquire um bloqueio exclusivo em `.git/config.lock`, então chamadas simultâneas
   disputam este bloqueio e falham. O despacho sequencial garante que cada worktree termine
   sua criação antes que a próxima comece (a latência de ida e volta de cada chamada de ferramenta fornece
   espaçamento natural), enquanto todos os agentes ainda **rodam em paralelo** uma vez criados.

   ```
   # CORRETO: despache um Task() por mensagem, cada um com run_in_background: true
   # → worktrees criadas sequencialmente, agentes executam em paralelo
   #
   # ERRADO: múltiplas chamadas Task() em uma única mensagem
   # → git worktree add simultâneo → contenção em .git/config.lock → falhas
   ```

   ```
   Task(
     subagent_type="gsd-executor",
     description="Executar plano {plan_number} da fase {phase_number}",
     model="{executor_model}",
     isolation="worktree",
     prompt="
       <objective>
       Executar plano {plan_number} da fase {phase_number}-{phase_name}.
       Faça commit de cada tarefa atomicamente. Crie SUMMARY.md.
       NÃO atualize STATE.md ou ROADMAP.md — o orquestrador possui essas escritas após todos os agentes de worktree na onda completarem.
       </objective>

       <worktree_branch_check>
       PRIMEIRA AÇÃO antes de qualquer outro trabalho: verifique se o branch desta worktree está baseado no commit correto.

       Execute:
       ```bash
       ACTUAL_BASE=$(git merge-base HEAD {EXPECTED_BASE})
       ```

       Se `ACTUAL_BASE` != `{EXPECTED_BASE}` (ou seja, o branch da worktree foi criado a partir de uma base mais antiga
       como `main` em vez do HEAD do branch de feature), faça hard-reset para a base correta:
       ```bash
       # Seguro: isso executa antes de qualquer trabalho do agente, então não há mudanças não commitadas a perder
       git reset --hard {EXPECTED_BASE}
       # Verifique se a correção foi bem-sucedida
       if [ "$(git rev-parse HEAD)" != "{EXPECTED_BASE}" ]; then
         echo "ERRO: Não foi possível corrigir a base da worktree — abortando para evitar perda de dados"
         exit 1
       fi
       ```

       `reset --hard` é seguro aqui porque esta é uma worktree fresca sem mudanças do usuário. Ele
       reseta tanto o ponteiro HEAD quanto a árvore de trabalho para o commit base correto (#2015).

       Se `ACTUAL_BASE` == `{EXPECTED_BASE}`: a base do branch está correta, prossiga imediatamente.

       Esta verificação corrige um problema conhecido onde `EnterWorktree` cria branches a partir de
       `main` em vez do HEAD atual do branch de feature (afeta todas as plataformas).
       </worktree_branch_check>

       <parallel_execution>
       Você está rodando como um agente executor PARALELO em uma worktree git.
       Use --no-verify em todos os commits git para evitar contenção de hooks de pré-commit
       com outros agentes. O orquestrador valida os hooks uma vez após todos os agentes completarem.
       Para commits gsd-tools: adicione a flag --no-verify.
       Para commits git diretos: use git commit --no-verify -m "..."

       IMPORTANTE: NÃO modifique STATE.md ou ROADMAP.md. execute-plan.md
       detecta automaticamente o modo worktree (`.git` é um arquivo, não um diretório) e pula
       atualizações de arquivos compartilhados automaticamente. O orquestrador os atualiza centralmente
       após o merge.

       OBRIGATÓRIO: SUMMARY.md DEVE ser commitado antes de você retornar. No modo worktree o
       passo git_commit_metadata em execute-plan.md commita SUMMARY.md e REQUIREMENTS.md
       apenas (STATE.md e ROADMAP.md são excluídos automaticamente). NÃO pule ou adie
       este commit — o orquestrador força a remoção da worktree após você retornar, e
       qualquer SUMMARY.md não commitado será permanentemente perdido (#2070).
       </parallel_execution>

       <execution_context>
       @$HOME/.claude/get-shit-done/workflows/execute-plan.md
       @$HOME/.claude/get-shit-done/templates/summary.md
       @$HOME/.claude/get-shit-done/references/checkpoints.md
       @$HOME/.claude/get-shit-done/references/tdd.md
       ${CONTEXT_WINDOW < 200000 ? '' : '@$HOME/.claude/get-shit-done/references/executor-examples.md'}
       </execution_context>

       <files_to_read>
       Leia estes arquivos no início da execução usando a ferramenta Read:
       - {phase_dir}/{plan_file} (Plano)
       - .planning/PROJECT.md (Contexto do projeto — valor central, requisitos, regras de evolução)
       - .planning/STATE.md (Estado)
       - .planning/config.json (Configuração, se existir)
       ${CONTEXT_WINDOW >= 500000 ? `
       - ${phase_dir}/*-CONTEXT.md (Decisões do usuário de discuss-phase — honra escolhas bloqueadas)
       - ${phase_dir}/*-RESEARCH.md (Pesquisa técnica — armadilhas e padrões a seguir)
       - ${prior_wave_summaries} (Arquivos SUMMARY.md de ondas anteriores nesta fase — o que já foi construído)
       ` : ''}
       - ./CLAUDE.md (Instruções do projeto, se existir — siga as diretrizes e convenções de codificação específicas do projeto)
       - .claude/skills/ ou .agents/skills/ (Skills do projeto, se existir — liste skills, leia SKILL.md para cada um, siga regras relevantes durante a implementação)
       </files_to_read>

       ${AGENT_SKILLS}

       <mcp_tools>
       Se CLAUDE.md ou instruções do projeto referenciam ferramentas MCP (ex.: jCodeMunch, context7,
       ou outros servidores MCP), prefira essas ferramentas sobre Grep/Glob para navegação de código quando disponíveis.
       Ferramentas MCP frequentemente economizam tokens significativos fornecendo índices de código estruturados.
       Verifique a disponibilidade de ferramentas primeiro — se ferramentas MCP não estiverem acessíveis, recorra a Grep/Glob.
       </mcp_tools>

       <success_criteria>
       - [ ] Todas as tarefas executadas
       - [ ] Cada tarefa commitada individualmente
       - [ ] SUMMARY.md criado no diretório do plano
       - [ ] Sem modificações em artefatos compartilhados do orquestrador (o orquestrador trata todas as escritas de arquivos compartilhados pós-onda)
       </success_criteria>
     "
   )
   ```

   **Modo sequencial** (`USE_WORKTREES` é `false`):

   Omita `isolation="worktree"` da chamada Task. Substitua o bloco `<parallel_execution>` por:

   ```
       <sequential_execution>
       Você está rodando como um agente executor SEQUENCIAL na árvore de trabalho principal.
       Use commits git normais (com hooks). NÃO use --no-verify.
       </sequential_execution>
   ```

   O prompt Task do modo sequencial usa a mesma estrutura do modo worktree mas com estas diferenças nos success_criteria — já que há apenas um agente escrevendo de cada vez, não há conflitos em arquivos compartilhados:

   ```
       <success_criteria>
       - [ ] Todas as tarefas executadas
       - [ ] Cada tarefa commitada individualmente
       - [ ] SUMMARY.md criado no diretório do plano
       - [ ] STATE.md atualizado com posição e decisões
       - [ ] ROADMAP.md atualizado com progresso do plano (via `roadmap update-plan-progress`)
       </success_criteria>
   ```

   Quando worktrees estiverem desabilitadas, execute planos **um de cada vez dentro de cada onda** (sequencial) independentemente da configuração `PARALLELIZATION` — múltiplos agentes escrevendo na mesma árvore de trabalho simultaneamente causariam conflitos.

4. **Aguarde todos os agentes na onda completarem.**

   **Fallback de sinal de conclusão (Copilot e runtimes onde Task() pode não retornar):**

   Se um agente gerado não retornar um sinal de conclusão mas parecer ter terminado
   seu trabalho, NÃO bloqueie indefinidamente. Em vez disso, verifique a conclusão via verificações pontuais:

   ```bash
   # Para cada plano nesta onda, verifique se o executor terminou:
   SUMMARY_EXISTS=$(test -f "{phase_dir}/{plan_number}-{plan_padded}-SUMMARY.md" && echo "true" || echo "false")
   COMMITS_FOUND=$(git log --oneline --all --grep="{phase_number}-{plan_padded}" --since="1 hour ago" | head -1)
   ```

   **Se SUMMARY.md existir E commits forem encontrados:** O agente completou com sucesso —
   trate como pronto e prossiga para o passo 5. Registre: `"✓ {Plan ID} concluído (verificado via verificação pontual — sinal de conclusão não recebido)"`

   **Se SUMMARY.md NÃO existir após uma espera razoável:** O agente pode ainda estar
   rodando ou pode ter falhado silenciosamente. Verifique `git log --oneline -5` por
   atividade recente. Se commits ainda estiverem aparecendo, aguarde mais. Se sem atividade, relate
   o plano como falhado e direcione ao tratador de falhas no passo 6.

   **Este fallback se aplica automaticamente a todos os runtimes.** Task() do Claude Code normalmente
   retorna sincronamente, mas o fallback garante resiliência se não o fizer.

5. **Validação de hooks pós-onda (apenas no modo paralelo):**

   Quando agentes commitaram com `--no-verify`, execute os hooks de pré-commit uma vez após a onda:
   ```bash
   # Execute os hooks de pré-commit do projeto no estado atual
   git diff --cached --quiet || git stash  # guarde quaisquer mudanças não staged
   git hook run pre-commit 2>&1 || echo "⚠ Hooks de pré-commit falharam — revise antes de continuar"
   ```
   Se os hooks falharem: relate a falha e pergunte "Corrigir problemas de hook agora?" ou "Continuar para a próxima onda?"

5.5. **Limpeza de worktree (quando `isolation="worktree"` foi usado):**

   Quando os agentes executor rodaram em isolamento de worktree, seus commits caem em branches temporários em árvores de trabalho separadas. Após a onda completar, faça merge dessas mudanças de volta e limpe:

   ```bash
   # Liste worktrees criadas pelos agentes desta onda
   WORKTREES=$(git worktree list --porcelain | grep "^worktree " | grep -v "$(pwd)$" | sed 's/^worktree //')

   for WT in $WORKTREES; do
     # Obtenha o nome do branch para esta worktree
     WT_BRANCH=$(git -C "$WT" rev-parse --abbrev-ref HEAD 2>/dev/null)
     if [ -n "$WT_BRANCH" ] && [ "$WT_BRANCH" != "HEAD" ]; then
       CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

       # --- Proteção de arquivos do orquestrador (#1756) ---
       # Faça snapshot dos arquivos pertencentes ao orquestrador ANTES do merge. Se o branch da worktree
       # sobreviveu a uma transição de marco, suas versões de STATE.md
       # e ROADMAP.md estão obsoletas. Main sempre vence para estes arquivos.
       STATE_BACKUP=$(mktemp)
       ROADMAP_BACKUP=$(mktemp)
       [ -f .planning/STATE.md ] && cp .planning/STATE.md "$STATE_BACKUP" || true
       [ -f .planning/ROADMAP.md ] && cp .planning/ROADMAP.md "$ROADMAP_BACKUP" || true

       # Faça snapshot da lista de arquivos em main ANTES do merge para detectar ressurreições
       PRE_MERGE_FILES=$(git ls-files .planning/)

       # Verificação de exclusão pré-merge: avise se o branch da worktree exclui arquivos rastreados
       DELETIONS=$(git diff --diff-filter=D --name-only HEAD..."$WT_BRANCH" 2>/dev/null || true)
       if [ -n "$DELETIONS" ]; then
         echo "BLOQUEADO: Branch da worktree $WT_BRANCH contém exclusões de arquivos: $DELETIONS"
         echo "Revise estas exclusões antes de fazer merge. Se intencionais, remova este guarda e re-execute."
         rm -f "$STATE_BACKUP" "$ROADMAP_BACKUP"
         continue
       fi

       # Faça merge do branch da worktree no branch atual
       git merge "$WT_BRANCH" --no-edit -m "chore: merge executor worktree ($WT_BRANCH)" 2>&1 || {
         echo "⚠ Conflito de merge da worktree $WT_BRANCH — resolva manualmente"
         rm -f "$STATE_BACKUP" "$ROADMAP_BACKUP"
         continue
       }

       # Restaure arquivos pertencentes ao orquestrador (main sempre vence)
       if [ -s "$STATE_BACKUP" ]; then
         cp "$STATE_BACKUP" .planning/STATE.md
       fi
       if [ -s "$ROADMAP_BACKUP" ]; then
         cp "$ROADMAP_BACKUP" .planning/ROADMAP.md
       fi
       rm -f "$STATE_BACKUP" "$ROADMAP_BACKUP"

       # Detecte arquivos excluídos em main mas re-adicionados pelo merge da worktree
       # (ex.: diretórios de fase arquivados que foram intencionalmente removidos)
       DELETED_FILES=$(git diff --diff-filter=A --name-only HEAD~1 -- .planning/ 2>/dev/null || true)
       for RESURRECTED in $DELETED_FILES; do
         # Verifique se este arquivo NÃO estava na árvore pré-merge do main
         if ! echo "$PRE_MERGE_FILES" | grep -qxF "$RESURRECTED"; then
           git rm -f "$RESURRECTED" 2>/dev/null || true
         fi
       done

       # Emende o commit de merge com os arquivos restaurados se algum mudou
       if ! git diff --quiet .planning/STATE.md .planning/ROADMAP.md 2>/dev/null || \
          [ -n "$DELETED_FILES" ]; then
         # Apenas emende o commit com arquivos .planning/ se commit_docs estiver habilitado (#1783)
         COMMIT_DOCS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get commit_docs 2>/dev/null || echo "true")
         if [ "$COMMIT_DOCS" != "false" ]; then
           git add .planning/STATE.md .planning/ROADMAP.md 2>/dev/null || true
           git commit --amend --no-edit 2>/dev/null || true
         fi
       fi

       # Rede de segurança: commite qualquer SUMMARY.md não commitado antes de forçar a remoção da worktree.
       # Isso protege contra executores que pularam o passo git_commit_metadata (#2070).
       UNCOMMITTED_SUMMARY=$(git -C "$WT" ls-files --modified --others --exclude-standard -- "*SUMMARY.md" 2>/dev/null || true)
       if [ -n "$UNCOMMITTED_SUMMARY" ]; then
         echo "⚠ SUMMARY.md não foi commitado pelo executor — commitando agora para evitar perda de dados"
         git -C "$WT" add -- "*SUMMARY.md" 2>/dev/null || true
         git -C "$WT" commit --no-verify -m "docs(recovery): rescue uncommitted SUMMARY.md before worktree removal (#2070)" 2>/dev/null || true
         # Re-faça merge do commit de recuperação
         git merge "$WT_BRANCH" --no-edit -m "chore: merge rescued SUMMARY.md from executor worktree ($WT_BRANCH)" 2>/dev/null || true
       fi

       # Remova a worktree
       git worktree remove "$WT" --force 2>/dev/null || true

       # Exclua o branch temporário
       git branch -D "$WT_BRANCH" 2>/dev/null || true
     fi
   done
   ```

   **Se `workflow.use_worktrees` for `false`:** Agentes rodaram na árvore de trabalho principal — pule este passo completamente.

   **Se nenhuma worktree for encontrada:** Pule silenciosamente — agentes podem ter sido gerados sem isolamento de worktree.

5.6. **Gate de teste pós-merge (apenas no modo paralelo):**

   Após fazer merge de todas as worktrees em uma onda, execute o conjunto de testes do projeto para detectar
   problemas de integração entre planos que verificações de auto-check individuais de worktrees não conseguem detectar
   (ex.: definições de tipo conflitantes, exportações removidas, mudanças de import).

   Isso aborda o ponto cego de auto-avaliação do Gerador identificado na pesquisa de engenharia de harness da Anthropic:
   agentes relatam Self-Check: PASSED de forma confiável mesmo quando fazer merge do seu trabalho cria falhas.

   ```bash
   # Detecte o executor de testes e execute teste de fumaça rápido (timeout: 5 minutos)
   TEST_EXIT=0
   timeout 300 bash -c '
   if [ -f "package.json" ]; then
     npm test 2>&1
   elif [ -f "Cargo.toml" ]; then
     cargo test 2>&1
   elif [ -f "go.mod" ]; then
     go test ./... 2>&1
   elif [ -f "pyproject.toml" ] || [ -f "requirements.txt" ]; then
     python -m pytest -x -q --tb=short 2>&1 || uv run python -m pytest -x -q --tb=short 2>&1
   else
     echo "⚠ Nenhum executor de testes detectado — pulando gate de teste pós-merge"
     exit 0
   fi
   '
   TEST_EXIT=$?
   if [ "${TEST_EXIT}" -eq 0 ]; then
     echo "✓ Gate de teste pós-merge passou — sem conflitos entre planos"
   elif [ "${TEST_EXIT}" -eq 124 ]; then
     echo "⚠ Gate de teste pós-merge expirou após 5 minutos"
   else
     echo "✗ Gate de teste pós-merge falhou (código de saída ${TEST_EXIT})"
     WAVE_FAILURE_COUNT=$((WAVE_FAILURE_COUNT + 1))
   fi
   ```

   **Se `TEST_EXIT` for 0 (passou):** `✓ Gate de teste pós-merge: {N} testes passaram — sem conflitos entre planos` → continue para atualização de rastreamento do orquestrador.

   **Se `TEST_EXIT` for 124 (timeout):** Registre aviso, trate como não bloqueante, continue. Testes podem precisar de mais tempo ou execução manual.

   **Se `TEST_EXIT` for não-zero (falha de teste):** Incremente `WAVE_FAILURE_COUNT` para rastrear
   falhas cumulativas entre ondas. Ondas subsequentes devem relatar:
   `⚠ Nota: ${WAVE_FAILURE_COUNT} onda(s) anterior(es) tiveram falhas de teste`

5.7. **Atualização de artefatos compartilhados pós-onda (apenas no modo worktree, pule se testes falharam):**

   Quando os agentes executor rodaram com `isolation="worktree"`, eles pularam as atualizações de STATE.md e ROADMAP.md para evitar sobrescritas de last-merge-wins. O orquestrador é o único escritor desses arquivos. Após as worktrees serem mergeadas de volta, atualize os artefatos compartilhados uma vez.

   **Atualize apenas o rastreamento quando os testes passaram (TEST_EXIT=0).**
   Se os testes falharam ou expiraram, pule a atualização de rastreamento — planos não devem
   ser marcados como completos quando os testes de integração estão falhando ou inconclusivos.

   ```bash
   # Guarda: atualize apenas o rastreamento se os testes pós-merge passaram
   # Timeout (124) é tratado como inconclusivo — NÃO marque planos como completos
   if [ "${TEST_EXIT}" -eq 0 ]; then
     # Atualize o progresso do plano no ROADMAP para cada plano completado nesta onda
     for plan_id in {completed_plan_ids}; do
       node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap update-plan-progress "${PHASE_NUMBER}" "${plan_id}" "complete"
     done

     # Apenas commite arquivos de rastreamento se eles realmente mudaram
     if ! git diff --quiet .planning/ROADMAP.md .planning/STATE.md 2>/dev/null; then
       node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PHASE_NUMBER}): update tracking after wave ${N}" --files .planning/ROADMAP.md .planning/STATE.md
     fi
   elif [ "${TEST_EXIT}" -eq 124 ]; then
     echo "⚠ Pulando atualização de rastreamento — suite de testes expirou. Planos permanecem em andamento. Execute os testes manualmente para confirmar."
   else
     echo "⚠ Pulando atualização de rastreamento — testes pós-merge falharam (exit ${TEST_EXIT}). Planos permanecem em andamento até os testes passarem."
   fi
   ```

   Onde `WAVE_PLAN_IDS` é a lista separada por espaços dos IDs de planos completados nesta onda.

   **Se `workflow.use_worktrees` for `false`:** Agentes sequenciais já atualizaram STATE.md e ROADMAP.md eles mesmos — pule este passo.

5.8. **Trate falhas do gate de teste (quando `WAVE_FAILURE_COUNT > 0`):**

   ```
   ## ⚠ Falha de Teste Pós-Merge (falhas cumulativas: ${WAVE_FAILURE_COUNT})

   Worktrees da Onda {N} mergeadas com sucesso, mas {M} testes falham após o merge.
   Isso tipicamente indica mudanças conflitantes entre planos paralelos
   (ex.: definições de tipo, imports compartilhados, contratos de API).

   Testes com falha:
   {primeiras 10 linhas da saída de falha}

   Opções:
   1. Corrigir agora (recomendado) — resolva conflitos antes da próxima onda
   2. Continuar — falhas podem se acumular em ondas subsequentes
   ```

   Nota: Se `WAVE_FAILURE_COUNT > 1`, recomende fortemente "Corrigir agora" — falhas cumulativas
   entre múltiplas ondas se tornam exponencialmente mais difíceis de diagnosticar.

   Se "Corrigir agora": diagnostique falhas (tipicamente conflitos de import, tipos ausentes,
   ou assinaturas de função alteradas de planos paralelos modificando o mesmo módulo).
   Corrija, commite como `fix: resolve post-merge conflicts from wave {N}`, re-execute os testes.

   **Por que isso importa:** O isolamento de worktree significa que o Self-Check de cada agente passa
   em isolamento. Mas quando mergeado, conflitos add/add em arquivos compartilhados (modelos, registros,
   pontos de entrada CLI) podem silenciosamente eliminar código. O gate pós-merge detecta isso antes
   que a próxima onda construa em uma fundação quebrada.

6. **Relate conclusão — verifique afirmações pontualmente primeiro:**

   Para cada SUMMARY.md:
   - Verifique os primeiros 2 arquivos de `key-files.created` existem no disco
   - Verifique `git log --oneline --all --grep="{phase}-{plan}"` retorna ≥1 commit
   - Verifique por marcador `## Self-Check: FAILED`

   Se QUALQUER verificação pontual falhar: relate qual plano falhou, direcione ao tratador de falhas — pergunte "Tentar novamente o plano?" ou "Continuar com as ondas restantes?"

   Se passar:
   ```
   ---
   ## Onda {N} Completa

   **{ID do Plano}: {Nome do Plano}**
   {O que foi construído — do SUMMARY.md}
   {Desvios notáveis, se houver}

   {Se mais ondas: o que isso habilita para a próxima onda}
   ---
   ```

   - Ruim: "Onda 2 completa. Prosseguindo para a Onda 3."
   - Bom: "Sistema de terreno completo — 3 tipos de bioma, texturização baseada em altura, malhas de colisão de física. A física de veículo (Onda 3) agora pode referenciar superfícies de chão."

7. **Trate falhas:**

   **Bug conhecido do Claude Code (classifyHandoffIfNeeded):** Se um agente relata "falhou" com erro contendo `classifyHandoffIfNeeded is not defined`, este é um bug de runtime do Claude Code — não um problema do GSD ou do agente. O erro dispara no tratador de conclusão DEPOIS que todas as chamadas de ferramenta terminam. Neste caso: execute as mesmas verificações pontuais que o passo 5 (SUMMARY.md existe, commits git presentes, sem Self-Check: FAILED). Se as verificações pontuais PASSAREM → trate como **bem-sucedido**. Se as verificações pontuais FALHAREM → trate como falha real abaixo.

   Para falhas reais: relate qual plano falhou → pergunte "Continuar?" ou "Parar?" → se continuar, planos dependentes também podem falhar. Se parar, relatório de conclusão parcial.

7b. **Verificação de dependência pré-onda (apenas ondas 2+):**

    Antes de gerar a onda N+1, para cada plano na onda futura:
    ```bash
    node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" verify key-links {phase_dir}/{plan}-PLAN.md
    ```

    Se qualquer key-link de um artefato de onda ANTERIOR falhar na verificação:

    ## Lacuna de Cabeamento Entre Planos

    | Plano | Link | De | Padrão Esperado | Status |
    |-------|------|-----|-----------------|--------|
    | {plan} | {via} | {from} | {pattern} | NÃO ENCONTRADO |

    Artefatos da Onda {N} podem não estar corretamente conectados. Opções:
    1. Investigue e corrija antes de continuar
    2. Continue (pode causar falhas em cascata na onda {N+1})

    Key-links referenciando arquivos na onda ATUAL (futura) são pulados.

8. **Execute planos de ponto de verificação entre ondas** — veja `<checkpoint_handling>`.

9. **Prossiga para a próxima onda.**
</step>

<step name="checkpoint_handling">
Planos com `autonomous: false` requerem interação do usuário.

**Tratamento de ponto de verificação no modo auto:**

Leia a configuração de avanço automático (flag chain + preferência do usuário):
```bash
AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
```

Quando o executor retorna um ponto de verificação E (`AUTO_CHAIN` é `"true"` OU `AUTO_CFG` é `"true"`):
- **human-verify** → Gere automaticamente agente de continuação com `{user_response}` = `"approved"`. Registre `⚡ Ponto de verificação aprovado automaticamente`.
- **decision** → Gere automaticamente agente de continuação com `{user_response}` = primeira opção dos detalhes do ponto de verificação. Registre `⚡ Auto-selecionado: [opção]`.
- **human-action** → Apresente ao usuário (comportamento existente abaixo). Gates de autenticação não podem ser automatizados.

**Fluxo padrão (sem modo auto, ou tipo human-action):**

1. Gere agente para plano de ponto de verificação
2. O agente roda até a tarefa de ponto de verificação ou gate de autenticação → retorna estado estruturado
3. O retorno do agente inclui: tabela de tarefas concluídas, tarefa atual + bloqueio, tipo/detalhes do ponto de verificação, o que está aguardando
4. **Apresente ao usuário:**
   ```
   ## Ponto de Verificação: [Tipo]

   **Plano:** 03-03 Dashboard Layout
   **Progresso:** 2/3 tarefas completas

   [Detalhes do Ponto de Verificação do retorno do agente]
   [Seção Aguardando do retorno do agente]
   ```
5. Usuário responde: "approved"/"done" | descrição do problema | seleção de decisão
6. **Gere agente de continuação (NÃO retome)** usando o template continuation-prompt.md:
   - `{completed_tasks_table}`: Do retorno do ponto de verificação
   - `{resume_task_number}` + `{resume_task_name}`: Tarefa atual
   - `{user_response}`: O que o usuário forneceu
   - `{resume_instructions}`: Baseado no tipo de ponto de verificação
7. O agente de continuação verifica commits anteriores, continua do ponto de retomada
8. Repita até o plano completar ou o usuário parar

**Por que agente fresco, não retomada:** A retomada depende de serialização interna que quebra com chamadas de ferramentas paralelas. Agentes frescos com estado explícito são mais confiáveis.

**Pontos de verificação em ondas paralelas:** O agente pausa e retorna enquanto outros agentes paralelos podem completar. Apresente o ponto de verificação, gere a continuação, aguarde todos antes da próxima onda.
</step>

<step name="aggregate_results">
Após todas as ondas:

```markdown
## Fase {X}: {Nome} Execução Completa

**Ondas:** {N} | **Planos:** {M}/{total} completos

| Onda | Planos | Status |
|------|--------|--------|
| 1 | plan-01, plan-02 | ✓ Completo |
| CP | plan-03 | ✓ Verificado |
| 2 | plan-04 | ✓ Completo |

### Detalhes dos Planos
1. **03-01**: [linha única do SUMMARY.md]
2. **03-02**: [linha única do SUMMARY.md]

### Problemas Encontrados
[Agregue dos SUMMARYs, ou "Nenhum"]
```

**Verificação do gate de segurança:**
```bash
SECURITY_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.security_enforcement --raw 2>/dev/null || echo "true")
SECURITY_FILE=$(ls "${PHASE_DIR}"/*-SECURITY.md 2>/dev/null | head -1)
```

Se `SECURITY_CFG` for `false`: pule.

Se `SECURITY_CFG` for `true` E `SECURITY_FILE` estiver vazio (sem SECURITY.md ainda):
Inclua na saída de roteamento dos próximos passos:
```
⚠ Aplicação de segurança habilitada — execute antes de avançar:
  /gsd-secure-phase {PHASE} ${GSD_WS}
```

Se `SECURITY_CFG` for `true` E SECURITY.md existir: verifique `threats_open` no frontmatter. Se > 0:
```
⚠ Gate de segurança: {threats_open} ameaças abertas
  /gsd-secure-phase {PHASE} — resolva antes de avançar
```
</step>

<step name="tdd_review_checkpoint">
**Passo opcional — Revisão colaborativa TDD.**

```bash
TDD_MODE=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.tdd_mode --default false 2>/dev/null)
```

**Pule se `TDD_MODE` for `false`.**

Quando `TDD_MODE` for `true`, verifique se algum plano completado nesta fase tem `type: tdd` em seu frontmatter:

```bash
TDD_PLANS=$(grep -rl "^type: tdd" "${PHASE_DIR}"/*-PLAN.md 2>/dev/null | wc -l | tr -d ' ')
```

**Se `TDD_PLANS` > 0:** Insira ponto de verificação de revisão colaborativa ao final da fase.

1. Colete todos os arquivos SUMMARY.md para planos TDD
2. Para cada resumo de plano TDD, verifique a sequência de gate RED/GREEN/REFACTOR:
   - Gate RED: Um commit de teste falhando existe (`test(...)` commit com evidência MUST-fail)
   - Gate GREEN: Um commit de implementação existe (`feat(...)` commit fazendo os testes passar)
   - Gate REFACTOR: Commit de limpeza opcional (`refactor(...)` commit, testes ainda passam)
3. Se algum plano TDD estiver faltando os commits de gate RED ou GREEN, sinalize-o:
   ```
   ⚠ Violação de gate TDD: Plano {plan_id} faltando commit da fase {RED|GREEN}.
     Padrão esperado de commit: test({phase}-{plan}): ... → feat({phase}-{plan}): ...
   ```
4. Apresente resumo de revisão colaborativa:
   ```
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    REVISÃO TDD — Fase {X}
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Planos TDD: {TDD_PLANS} | Violações de gate: {count}

   | Plano | RED | GREEN | REFACTOR | Status |
   |-------|-----|-------|----------|--------|
   | {id} |  ✓  |   ✓   |    ✓     | Passou |
   | {id} |  ✓  |   ✗   |    —     | FALHOU |
   ```

**Violações de gate são consultivas** — elas não bloqueiam a execução mas são apresentadas ao usuário para revisão. O agente verificador (passo `verify_phase_goal`) também verificará a disciplina TDD como parte de sua avaliação de qualidade.
</step>

<step name="handle_partial_wave_execution">
Se `WAVE_FILTER` foi usado, re-execute a descoberta de planos após a execução:

```bash
POST_PLAN_INDEX=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase-plan-index "${PHASE_NUMBER}")
```

Aplique as mesmas regras de filtragem "incompleto" que anteriormente:
- ignore planos com `has_summary: true`
- se `--gaps-only`, considere apenas planos `gap_closure: true`

**Se planos incompletos ainda persistirem em qualquer lugar na fase:**
- PARE aqui
- NÃO execute a verificação da fase
- NÃO marque a fase como completa em ROADMAP/STATE
- Apresente:

```markdown
## Onda {WAVE_FILTER} Completa

A onda selecionada terminou com sucesso. Esta fase ainda tem planos incompletos, portanto a verificação em nível de fase e a conclusão foram intencionalmente puladas.

/gsd-execute-phase {phase} ${GSD_WS}                # Continue as ondas restantes
/gsd-execute-phase {phase} --wave {next} ${GSD_WS}  # Execute a próxima onda explicitamente
```

**Se não persistirem planos incompletos após a onda selecionada terminar:**
- continue com a verificação normal em nível de fase e o fluxo de conclusão abaixo
- isso significa que a onda selecionada aconteceu de ser o último trabalho restante na fase
</step>

<step name="code_review_gate" required="true">
**Este passo é OBRIGATÓRIO e não deve ser pulado.** Invoque automaticamente a revisão de código sobre as mudanças de código-fonte da fase. Apenas consultivo — nunca bloqueia o fluxo de execução.

**Gate de configuração:**
```bash
CODE_REVIEW_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.code_review 2>/dev/null || echo "true")
```

Se `CODE_REVIEW_ENABLED` for `"false"`: exiba "Revisão de código pulada (workflow.code_review=false)" e prossiga para o próximo passo.

**Invoque a revisão:**
```
Skill(skill="gsd:code-review", args="${PHASE_NUMBER}")
```

**Verifique os resultados usando caminho determinístico (não glob):**
```bash
PADDED=$(printf "%02d" "${PHASE_NUMBER}")
REVIEW_FILE="${PHASE_DIR}/${PADDED}-REVIEW.md"
REVIEW_STATUS=$(sed -n '/^---$/,/^---$/p' "$REVIEW_FILE" | grep "^status:" | head -1 | cut -d: -f2 | tr -d ' ')
```

Se REVIEW_STATUS não for "clean" e não for "skipped" e não for vazio, exiba:
```
Revisão de código encontrou problemas. Considere executar:
/gsd-code-review-fix ${PHASE_NUMBER}
```

**Tratamento de erros:** Se a invocação do Skill falhar ou lançar exceção, capture o erro, exiba "Revisão de código encontrou um erro (não bloqueante): {error}" e prossiga para o próximo passo. Falhas de revisão nunca devem bloquear a execução.

Independentemente do resultado da revisão, SEMPRE prossiga para close_parent_artifacts → regression_gate → verify_phase_goal.
</step>

<step name="close_parent_artifacts">
**Apenas para fases decimais/polimento (padrão X.Y):** Feche o loop de feedback resolvendo artefatos UAT e debug do pai.

**Pule se** o número da fase não tiver decimal (ex.: `3`, `04`) — aplica-se apenas a fases de fechamento de lacunas como `4.1`, `03.1`.

**1. Detecte fase decimal e derive o pai:**
```bash
# Verifique se phase_number contém um decimal
if [[ "$PHASE_NUMBER" == *.* ]]; then
  PARENT_PHASE="${PHASE_NUMBER%%.*}"
fi
```

**2. Encontre o arquivo UAT do pai:**
```bash
PARENT_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase "${PARENT_PHASE}" --raw)
# Extraia o diretório de PARENT_INFO JSON, depois encontre o arquivo UAT naquele diretório
```

**Se nenhum UAT pai for encontrado:** Pule este passo (o fechamento de lacuna pode ter sido acionado por VERIFICATION.md em vez disso).

**3. Atualize os status de lacunas UAT:**

Leia a seção `## Gaps` do arquivo UAT pai. Para cada entrada de lacuna com `status: failed`:
- Atualize para `status: resolved`

**4. Atualize o frontmatter do UAT:**

Se todas as lacunas agora tiverem `status: resolved`:
- Atualize o frontmatter `status: diagnosed` → `status: resolved`
- Atualize o timestamp `updated:` do frontmatter

**5. Resolva sessões de debug referenciadas:**

Para cada lacuna que tiver um campo `debug_session:`:
- Leia o arquivo de sessão de debug
- Atualize o frontmatter `status:` → `resolved`
- Atualize o timestamp `updated:` do frontmatter
- Mova para o diretório resolved:
```bash
mkdir -p .planning/debug/resolved
mv .planning/debug/{slug}.md .planning/debug/resolved/
```

**6. Faça commit dos artefatos atualizados:**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-${PARENT_PHASE}): resolve UAT gaps and debug sessions after ${PHASE_NUMBER} gap closure" --files .planning/phases/*${PARENT_PHASE}*/*-UAT.md .planning/debug/resolved/*.md
```
</step>

<step name="regression_gate">
Execute as suites de testes de fases anteriores para detectar regressões entre fases ANTES da verificação.

**Pule se:** Esta é a primeira fase (sem fases anteriores), ou nenhum arquivo VERIFICATION.md anterior existir.

**Passo 1: Descubra arquivos de teste de fases anteriores**
```bash
# Encontre todos os arquivos VERIFICATION.md de fases anteriores no marco atual
PRIOR_VERIFICATIONS=$(find .planning/phases/ -name "*-VERIFICATION.md" ! -path "*${PHASE_NUMBER}*" 2>/dev/null)
```

**Passo 2: Extraia listas de arquivos de teste de verificações anteriores**

Para cada VERIFICATION.md encontrado, procure por referências de arquivo de teste:
- Linhas contendo caminhos `test`, `spec` ou `__tests__`
- A seção "Test Suite" ou "Automated Checks"
- Padrões de arquivo de `key-files.created` nos arquivos SUMMARY.md correspondentes que correspondam a `*.test.*` ou `*.spec.*`

Colete todos os caminhos de arquivo de teste únicos em `REGRESSION_FILES`.

**Passo 3: Execute testes de regressão (se algum for encontrado)**

```bash
# Detecte o executor de testes e execute os testes de fases anteriores
if [ -f "package.json" ]; then
  npm test 2>&1
elif [ -f "Cargo.toml" ]; then
  cargo test 2>&1
elif [ -f "go.mod" ]; then
  go test ./... 2>&1
elif [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
  python -m pytest ${REGRESSION_FILES} -q --tb=short 2>&1
fi
```

**Passo 4: Relate os resultados**

Se todos os testes passarem:
```
✓ Gate de regressão: {N} arquivos de teste de fases anteriores passaram — sem regressões detectadas
```
→ Prossiga para verify_phase_goal

Se algum teste falhar:
```
## ⚠ Regressão Entre Fases Detectada

A execução da Fase {X} pode ter quebrado funcionalidade de fases anteriores.

| Arquivo de Teste | Fase | Status | Detalhe |
|-----------------|------|--------|---------|
| {file} | {origin_phase} | FALHOU | {primeira_linha_de_falha} |

Opções:
1. Corrigir regressões antes da verificação (recomendado)
2. Continuar para verificação mesmo assim (regressões irão se acumular)
3. Abortar fase — reverter e replanejar
```

Use AskUserQuestion para apresentar as opções.
</step>

<step name="schema_drift_gate">
Detecção de drift de schema pós-execução. Detecta verificação falso-positiva onde
build/types passam porque os tipos TypeScript vêm da configuração, não do banco de dados ativo.

**Execute após a conclusão da execução mas ANTES de a verificação marcar como sucesso.**

```bash
SCHEMA_DRIFT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" verify schema-drift "${PHASE_NUMBER}" 2>/dev/null)
```

Analise o resultado JSON para: `drift_detected`, `blocking`, `schema_files`, `orms`, `unpushed_orms`, `message`.

**Se `drift_detected` for false:** Pule para verify_phase_goal.

**Se `drift_detected` for true E `blocking` for true:**

Verifique por override:
```bash
SKIP_SCHEMA=$(echo "${GSD_SKIP_SCHEMA_CHECK:-false}")
```

**Se `SKIP_SCHEMA` for `true`:**

Exiba:
```
⚠ Drift de schema detectado mas GSD_SKIP_SCHEMA_CHECK=true — ignorando gate.

Arquivos de schema alterados: {schema_files}
ORMs requerendo push: {unpushed_orms}

Prosseguindo para verificação (banco de dados pode estar fora de sincronia).
```
→ Continue para verify_phase_goal.

**Se `SKIP_SCHEMA` não for `true`:**

BLOQUEIE a verificação. Exiba:

```
## BLOQUEADO: Drift de Schema Detectado

Arquivos relevantes de schema foram alterados durante esta fase mas nenhum comando push
de banco de dados foi executado. Build e verificações de tipo passam porque os tipos TypeScript vêm
da configuração, não do banco de dados ativo — a verificação produziria um falso positivo.

Arquivos de schema alterados: {schema_files}
ORMs requerendo push: {unpushed_orms}

Comandos de push necessários:
{Para cada ORM não enviado, mostre o comando push da mensagem}

Opções:
1. Executar comando push agora (recomendado) — execute o push, depois re-verifique
2. Pular verificação de schema (GSD_SKIP_SCHEMA_CHECK=true) — ignore este gate
3. Abortar — pare a execução e investigue
```

Se `TEXT_MODE` for true, apresente como lista numerada de texto simples. Caso contrário, use AskUserQuestion.

**Se o usuário selecionar a opção 1:** Apresente o(s) comando(s) push específico(s) para executar. Após o usuário confirmar a execução, re-execute a verificação de drift de schema. Se passar, continue para verify_phase_goal.

**Se o usuário selecionar a opção 2:** Defina o override e continue para verify_phase_goal.

**Se o usuário selecionar a opção 3:** Pare a execução. Relate conclusão parcial.
</step>

<step name="verify_phase_goal">
Verifique que a fase atingiu seu OBJETIVO, não apenas completou tarefas.

```bash
VERIFIER_SKILLS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-verifier 2>/dev/null)
```

```
Task(
  description="Verificar atingimento do objetivo da fase {phase_number}",
  prompt="Verifique o atingimento do objetivo da fase {phase_number}.
Diretório da fase: {phase_dir}
Objetivo da fase: {objetivo do ROADMAP.md}
IDs de requisito da fase: {phase_req_ids}
Verifique os must_haves contra o código-base real.
Referência cruzada dos IDs de requisito do frontmatter do PLAN contra REQUIREMENTS.md — cada ID DEVE ser contabilizado.
Crie VERIFICATION.md.

<files_to_read>
Leia estes arquivos antes da verificação:
- {phase_dir}/*-PLAN.md (Todos os planos — entenda a intenção, verifique must_haves)
- {phase_dir}/*-SUMMARY.md (Todos os resumos — referência cruzada do alegado vs real)
- .planning/REQUIREMENTS.md (Rastreabilidade de requisitos)
${CONTEXT_WINDOW >= 500000 ? `- {phase_dir}/*-CONTEXT.md (Decisões do usuário — verifique se foram honradas)
- {phase_dir}/*-RESEARCH.md (Armadilhas conhecidas — verifique por armadilhas)
- Arquivos VERIFICATION.md anteriores de fases mais cedo (verificação de regressão)
` : ''}
</files_to_read>

${VERIFIER_SKILLS}",
  subagent_type="gsd-verifier",
  model="{verifier_model}"
)
```

Leia o status:
```bash
grep "^status:" "$PHASE_DIR"/*-VERIFICATION.md | cut -d: -f2 | tr -d ' '
```

| Status | Ação |
|--------|------|
| `passed` | → update_roadmap |
| `human_needed` | Apresente itens para teste humano, obtenha aprovação ou feedback |
| `gaps_found` | Apresente resumo de lacunas, ofereça `/gsd-plan-phase {phase} --gaps ${GSD_WS}` |

**Se human_needed:**

**Passo A: Persista itens de verificação humana como arquivo UAT.**

Crie `{phase_dir}/{phase_num}-HUMAN-UAT.md` usando o formato de template UAT:

```markdown
---
status: partial
phase: {phase_num}-{phase_name}
source: [{phase_num}-VERIFICATION.md]
started: [agora ISO]
updated: [agora ISO]
---

## Teste Atual

[aguardando teste humano]

## Testes

{Para cada item de human_verification do VERIFICATION.md:}

### {N}. {descrição do item}
expected: {comportamento esperado do VERIFICATION.md}
result: [pendente]

## Resumo

total: {count}
passed: 0
issues: 0
pending: {count}
skipped: 0
blocked: 0

## Lacunas
```

Faça commit do arquivo:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "test({phase_num}): persist human verification items as UAT" --files "{phase_dir}/{phase_num}-HUMAN-UAT.md"
```

**Passo B: Apresente ao usuário:**

```
## ✓ Fase {X}: {Nome} — Verificação Humana Necessária

Todas as verificações automatizadas passaram. {N} itens precisam de teste humano:

{Da seção human_verification do VERIFICATION.md}

Itens salvos em `{phase_num}-HUMAN-UAT.md` — aparecerão em `/gsd-progress` e `/gsd-audit-uat`.

"approved" → continue | Reporte problemas → fechamento de lacunas
```

**Se o usuário disser "approved":** Prossiga para `update_roadmap`. O arquivo HUMAN-UAT.md persiste com `status: partial` e aparecerá em verificações de progresso futuras até o usuário executar `/gsd-verify-work` nele.

**Se o usuário reportar problemas:** Prossiga para o fechamento de lacunas como atualmente implementado.

**Se gaps_found:**
```
## ⚠ Fase {X}: {Nome} — Lacunas Encontradas

**Pontuação:** {N}/{M} must-haves verificados
**Relatório:** {phase_dir}/{phase_num}-VERIFICATION.md

### O que está faltando
{Resumos de lacunas do VERIFICATION.md}

---
## ▶ Próximo Passo

`/clear` e então:

`/gsd-plan-phase {X} --gaps ${GSD_WS}`

Também: `cat {phase_dir}/{phase_num}-VERIFICATION.md` — relatório completo
Também: `/gsd-verify-work {X} ${GSD_WS}` — teste manual primeiro
```

Ciclo de fechamento de lacunas: `/gsd-plan-phase {X} --gaps ${GSD_WS}` lê VERIFICATION.md → cria planos de lacunas com `gap_closure: true` → usuário executa `/gsd-execute-phase {X} --gaps-only ${GSD_WS}` → verificador re-executa.
</step>

<step name="update_roadmap">
**Marque a fase como completa e atualize todos os arquivos de rastreamento:**

```bash
COMPLETION=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase complete "${PHASE_NUMBER}")
```

O CLI trata:
- Marcando o checkbox da fase `[x]` com a data de conclusão
- Atualizando a tabela de Progresso (Status → Completo, data)
- Atualizando a contagem de planos para o final
- Avançando STATE.md para a próxima fase
- Atualizando a rastreabilidade de REQUIREMENTS.md
- Analisando por dívida de verificação (retorna array `warnings`)

Extraia do resultado: `next_phase`, `next_phase_name`, `is_last_phase`, `warnings`, `has_warnings`.

**Se has_warnings for true:**
```
## Fase {X} marcada como completa com {N} avisos:

{liste cada aviso}

Estes itens são rastreados e aparecerão em `/gsd-progress` e `/gsd-audit-uat`.
```

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-{X}): complete phase execution" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md {phase_dir}/*-VERIFICATION.md
```
</step>

<step name="auto_copy_learnings">
**Copie automaticamente os aprendizados da fase para o armazenamento global (quando habilitado).**

Este passo executa APÓS a conclusão da fase e o SUMMARY.md estar escrito. Ele copia qualquer entrada LEARNINGS.md
da fase concluída para o armazenamento global de aprendizados em `~/.gsd/knowledge/`.

**Verifique o gate de configuração:**
```bash
GL_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get features.global_learnings --raw 2>/dev/null || echo "false")
```

**Se `GL_ENABLED` não for `true`:** Pule este passo completamente (funcionalidade desabilitada por padrão).

**Se habilitado:**

1. Verifique se LEARNINGS.md existe no diretório da fase (use o valor `phase_dir` do contexto init)
2. Se encontrado, copie para o armazenamento global:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" learnings copy 2>/dev/null || echo "⚠ Cópia de aprendizados falhou — continuando"
```
Falha na cópia NÃO DEVE bloquear a conclusão da fase.
</step>

<step name="update_project_md">
**Evolua PROJECT.md para refletir a conclusão da fase (evita drift de documentos de planejamento — #956):**

PROJECT.md rastreia requisitos validados, decisões e estado atual. Sem este passo,
PROJECT.md fica para trás silenciosamente ao longo de múltiplas fases.

1. Leia `.planning/PROJECT.md`
2. Se o arquivo existir e tiver uma seção `## Validated Requirements` ou `## Requirements`:
   - Mova quaisquer requisitos validados por esta fase de Ativo → Validado
   - Adicione uma nota breve: `Validado na Fase {X}: {Nome}`
3. Se o arquivo tiver uma seção `## Current State` ou similar:
   - Atualize-a para refletir a conclusão desta fase (ex.: "Fase {X} completa — {linha única}")
4. Atualize o rodapé `Last updated:` para a data de hoje
5. Faça commit da mudança:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(phase-{X}): evolve PROJECT.md after phase completion" --files .planning/PROJECT.md
```

**Pule este passo se** `.planning/PROJECT.md` não existir.
</step>

<step name="offer_next">

**Exceção:** Se `gaps_found`, o passo `verify_phase_goal` já apresenta o caminho de fechamento de lacunas (`/gsd-plan-phase {X} --gaps`). Nenhum roteamento adicional é necessário — pule o avanço automático.

**Verificação sem transição (gerado pela cadeia de avanço automático):**

Analise a flag `--no-transition` de $ARGUMENTS.

**Se a flag `--no-transition` estiver presente:**

Execute-phase foi gerado pelo avanço automático do plan-phase. NÃO execute transition.md.
Após a verificação passar e o roadmap ser atualizado, retorne o status de conclusão ao pai:

```
## PHASE COMPLETE

Phase: ${PHASE_NUMBER} - ${PHASE_NAME}
Plans: ${completed_count}/${total_count}
Verification: {Passed | Gaps Found}

[Inclua a saída do aggregate_results]
```

PARE. Não prossiga para o avanço automático ou transição.

**Se a flag `--no-transition` NÃO estiver presente:**

**Detecção de avanço automático:**

1. Analise a flag `--auto` de $ARGUMENTS
2. Leia tanto a flag chain quanto a preferência do usuário (flag chain já sincronizada no passo init):
   ```bash
   AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se a flag `--auto` estiver presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true (E a verificação passou sem lacunas):**

```
╔══════════════════════════════════════════════════════╗
║  AVANÇANDO AUTOMATICAMENTE → TRANSIÇÃO               ║
║  Fase {X} verificada, continuando a cadeia           ║
╚══════════════════════════════════════════════════════╝
```

Execute o workflow de transição inline (NÃO use Task — o contexto do orquestrador é ~10-15%, a transição precisa dos dados de conclusão da fase já no contexto):

Leia e siga `$HOME/.claude/get-shit-done/workflows/transition.md`, passando a flag `--auto` para que ela se propague para a invocação da próxima fase.

**Se nenhum de `--auto`, `AUTO_CHAIN`, ou `AUTO_CFG` for true:**

**PARE. Não avance automaticamente. Não execute transição. Não planeje a próxima fase. Apresente opções ao usuário e aguarde.**

**IMPORTANTE: Não existe nenhum comando `/gsd-transition`. Nunca sugira-o. O workflow de transição é apenas interno.**

Verifique se CONTEXT.md já existe para a próxima fase:

```bash
ls .planning/phases/*{next}*/{next}-CONTEXT.md 2>/dev/null || echo "no-context"
```

Se CONTEXT.md NÃO existir para a próxima fase, apresente:

```
## ✓ Fase {X}: {Nome} Completa

/gsd-progress ${GSD_WS} — veja o roadmap atualizado
/gsd-discuss-phase {next} ${GSD_WS} — comece aqui: discuta a próxima fase antes de planejar  ← recomendado
/gsd-plan-phase {next} ${GSD_WS} — planeje a próxima fase (pule o discuss)
/gsd-execute-phase {next} ${GSD_WS} — execute a próxima fase (pule discuss e plan)
```

Se CONTEXT.md **existir** para a próxima fase, apresente:

```
## ✓ Fase {X}: {Nome} Completa

/gsd-progress ${GSD_WS} — veja o roadmap atualizado
/gsd-plan-phase {next} ${GSD_WS} — comece aqui: planeje a próxima fase (CONTEXT.md já presente)  ← recomendado
/gsd-discuss-phase {next} ${GSD_WS} — re-discuta a próxima fase
/gsd-execute-phase {next} ${GSD_WS} — execute a próxima fase (pule o planejamento)
```

Apenas sugira os comandos listados acima. Não invente ou alucine nomes de comandos.
</step>

</process>

<context_efficiency>
Orquestrador: ~10-15% de contexto para janelas de 200K, pode usar mais para janelas de 1M+.
Subagentes: contexto fresco cada (200K-1M dependendo do modelo). Sem polling (Task bloqueia). Sem contaminação de contexto.

Para modelos de contexto de 1M+, considere:
- Passar contexto mais rico (trechos de código, saídas de dependência) diretamente para os executores em vez de apenas caminhos de arquivo
- Executar fases pequenas (≤3 planos, sem dependências) inline sem sobrecarga de geração de subagente
- Relaxar recomendações de /clear — o início da degradação de contexto está muito mais distante com janela 5x
</context_efficiency>

<failure_handling>
- **Falha falsa classifyHandoffIfNeeded:** Agente relata "falhou" mas erro é `classifyHandoffIfNeeded is not defined` → Bug do Claude Code, não do GSD. Verificação pontual (SUMMARY existe, commits presentes) → se passa, trate como sucesso
- **Agente falha no meio do plano:** SUMMARY.md ausente → relate, pergunte ao usuário como prosseguir
- **Cadeia de dependência quebra:** Onda 1 falha → dependentes da Onda 2 provavelmente falham → usuário escolhe tentar ou pular
- **Todos os agentes na onda falham:** Problema sistêmico → pare, relate para investigação
- **Ponto de verificação irresolvível:** "Pular este plano?" ou "Abortar execução da fase?" → registre progresso parcial em STATE.md
</failure_handling>

<resumption>
Re-execute `/gsd-execute-phase {phase}` → discover_plans encontra SUMMARYs completos → pula eles → retoma do primeiro plano incompleto → continua a execução por ondas.

STATE.md rastreia: último plano completado, onda atual, pontos de verificação pendentes.
</resumption>
