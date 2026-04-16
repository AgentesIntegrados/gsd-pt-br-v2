<purpose>
Executar tarefas ad-hoc pequenas com garantias GSD (commits atômicos, rastreamento em STATE.md). O modo Quick spawna gsd-planner (modo quick) + gsd-executor(s), rastreia tarefas em `.planning/quick/` e atualiza a tabela "Quick Tasks Completed" de STATE.md.

Com a flag `--full`: habilita o pipeline completo de qualidade — discussão + pesquisa + verificação de planos + verificação. Uma flag para tudo.

Com a flag `--validate`: habilita verificação de planos (máximo 2 iterações) e verificação pós-execução apenas. Use quando quiser garantias de qualidade sem discussão ou pesquisa.

Com a flag `--discuss`: fase de discussão leve antes do planejamento. Apresenta suposições, esclarece áreas cinzas, captura decisões em CONTEXT.md para que o planejador as trate como bloqueadas.

Com a flag `--research`: spawna um agente de pesquisa focado antes do planejamento. Investiga abordagens de implementação, opções de biblioteca e armadilhas. Use quando não tiver certeza de como abordar uma tarefa.

Flags granulares são combináveis: `--discuss --research --validate` dá o mesmo resultado que `--full`.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<available_agent_types>
Tipos de subagente GSD válidos (use nomes exatos — não recorra a 'general-purpose'):
- gsd-phase-researcher — Pesquisa abordagens técnicas para uma fase
- gsd-planner — Cria planos detalhados a partir do escopo da fase
- gsd-plan-checker — Revisa a qualidade do plano antes da execução
- gsd-executor — Executa tarefas do plano, commita, cria SUMMARY.md
- gsd-verifier — Verifica a conclusão da fase, verifica gates de qualidade
- gsd-code-reviewer — Revisa arquivos fonte em busca de bugs, problemas de segurança e qualidade de código
</available_agent_types>

<process>
**Etapa 1: Analisar argumentos e obter descrição da tarefa**

Analise `$ARGUMENTS` para:
- Flag `--full` → armazene `$FULL_MODE=true`, `$DISCUSS_MODE=true`, `$RESEARCH_MODE=true`, `$VALIDATE_MODE=true`
- Flag `--validate` → armazene `$VALIDATE_MODE=true`
- Flag `--discuss` → armazene `$DISCUSS_MODE=true`
- Flag `--research` → armazene `$RESEARCH_MODE=true`
- Texto restante → use como `$DESCRIPTION` se não vazio

Se `$DESCRIPTION` estiver vazio após a análise, solicite ao usuário interativamente:


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.

```
AskUserQuestion(
  header: "Tarefa Rápida",
  question: "O que você quer fazer?",
  followUp: null
)
```

Armazene a resposta como `$DESCRIPTION`.

Se ainda vazio, solicite novamente: "Por favor, forneça uma descrição da tarefa."

Exiba banner com base nas flags ativas:

Se `$FULL_MODE` (todas as fases habilitadas — `--full` ou todas as flags granulares):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (COMPLETA)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + pesquisa + verificação de planos + verificação habilitadas
```

Se `$DISCUSS_MODE` e `$RESEARCH_MODE` e `$VALIDATE_MODE` (sem `$FULL_MODE` — composto granularmente):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSS + RESEARCH + VALIDATE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + pesquisa + verificação de planos + verificação habilitadas
```

Se `$DISCUSS_MODE` e `$VALIDATE_MODE` (sem pesquisa):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSS + VALIDATE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + verificação de planos + verificação habilitadas
```

Se `$DISCUSS_MODE` e `$RESEARCH_MODE` (sem validate):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSS + RESEARCH)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Discussão + pesquisa habilitadas
```

Se `$RESEARCH_MODE` e `$VALIDATE_MODE` (sem discuss):
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (RESEARCH + VALIDATE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Pesquisa + verificação de planos + verificação habilitadas
```

Se apenas `$DISCUSS_MODE`:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (DISCUSS)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Fase de discussão habilitada — apresentando áreas cinzas antes do planejamento
```

Se apenas `$RESEARCH_MODE`:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (RESEARCH)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Fase de pesquisa habilitada — investigando abordagens antes do planejamento
```

Se apenas `$VALIDATE_MODE`:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TAREFA RÁPIDA (VALIDATE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Verificação de planos + verificação habilitadas
```

---

**Etapa 2: Inicializar**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init quick "$DESCRIPTION")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_PLANNER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-planner 2>/dev/null)
AGENT_SKILLS_EXECUTOR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-executor 2>/dev/null)
AGENT_SKILLS_CHECKER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-checker 2>/dev/null)
AGENT_SKILLS_VERIFIER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-verifier 2>/dev/null)
```

Analise JSON para: `planner_model`, `executor_model`, `checker_model`, `verifier_model`, `commit_docs`, `branch_name`, `quick_id`, `slug`, `date`, `timestamp`, `quick_dir`, `task_dir`, `roadmap_exists`, `planning_exists`.

```bash
USE_WORKTREES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.use_worktrees 2>/dev/null || echo "true")
```

Se o projeto usa submódulos git, o isolamento de worktree é ignorado:

```bash
if [ -f .gitmodules ]; then
  echo "[worktree] Projeto com submódulos detectado (.gitmodules existe) — revertendo para execução sequencial"
  USE_WORKTREES=false
fi
```

**Se `roadmap_exists` for false:** Erro — O modo Quick requer um projeto ativo com ROADMAP.md. Execute `/gsd-new-project` primeiro.

Tarefas rápidas podem ser executadas no meio de uma fase — a validação verifica apenas se ROADMAP.md existe, não o status da fase.

---

**Etapa 2.5: Lidar com branching de tarefa rápida**

**Se `branch_name` estiver vazio/null:** Pule e continue no branch atual.

**Se `branch_name` estiver definido:** Faça checkout do branch de tarefa rápida antes de qualquer commit de planejamento:

```bash
git checkout -b "$branch_name" 2>/dev/null || git checkout "$branch_name"
```

Todos os commits de tarefa rápida para esta execução ficam naquele branch. O usuário lida com merge/rebase depois.

---

**Etapa 3: Criar diretório de tarefa**

```bash
mkdir -p "${task_dir}"
```

---

**Etapa 4: Criar diretório de tarefa rápida**

Crie o diretório para esta tarefa rápida:

```bash
QUICK_DIR=".planning/quick/${quick_id}-${slug}"
mkdir -p "$QUICK_DIR"
```

Informe ao usuário:
```
Criando tarefa rápida ${quick_id}: ${DESCRIPTION}
Diretório: ${QUICK_DIR}
```

Armazene `$QUICK_DIR` para uso na orquestração.

---

**Etapa 4.5: Fase de discussão (apenas quando `$DISCUSS_MODE`)**

Pule esta etapa inteiramente se NÃO for `$DISCUSS_MODE`.

Exiba banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DISCUTINDO TAREFA RÁPIDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Apresentando áreas cinzas para: ${DESCRIPTION}
```

**4.5a. Identificar áreas cinzas**

Analise `$DESCRIPTION` para identificar 2-4 áreas cinzas — decisões de implementação que mudariam o resultado e nas quais o usuário deveria pesar.

Use a heurística ciente de domínio para gerar áreas cinzas específicas de fase (não genéricas):
- Algo que os usuários **VÊM** → layout, densidade, interações, estados
- Algo que os usuários **CHAMAM** → respostas, erros, auth, versionamento
- Algo que os usuários **EXECUTAM** → formato de saída, flags, modos, tratamento de erros
- Algo que os usuários **LÊM** → estrutura, tom, profundidade, fluxo
- Algo sendo **ORGANIZADO** → critérios, agrupamento, nomenclatura, exceções

Cada área cinza deve ser um ponto de decisão concreto, não uma categoria vaga. Exemplo: "Comportamento de carregamento" não "UX".

**4.5b. Apresentar áreas cinzas**

```
AskUserQuestion(
  header: "Áreas Cinzas",
  question: "Quais áreas precisam de esclarecimento antes do planejamento?",
  options: [
    { label: "${area_1}", description: "${por_que_importa_1}" },
    { label: "${area_2}", description: "${por_que_importa_2}" },
    { label: "${area_3}", description: "${por_que_importa_3}" },
    { label: "Tudo claro", description: "Pular discussão — já sei o que quero" }
  ],
  multiSelect: true
)
```

Se o usuário selecionar "Tudo claro" → pule para a Etapa 5 (sem CONTEXT.md escrito).

**4.5c. Discutir áreas selecionadas**

Para cada área selecionada, faça 1-2 perguntas focadas via AskUserQuestion:

```
AskUserQuestion(
  header: "${area_name}",
  question: "${pergunta_específica_sobre_esta_área}",
  options: [
    { label: "${escolha_concreta_1}", description: "${o_que_isso_significa}" },
    { label: "${escolha_concreta_2}", description: "${o_que_isso_significa}" },
    { label: "${escolha_concreta_3}", description: "${o_que_isso_significa}" },
    { label: "Você decide", description: "A critério do Claude" }
  ],
  multiSelect: false
)
```

Regras:
- As opções devem ser escolhas concretas, não categorias abstratas
- Destaque a escolha recomendada onde você tem uma opinião clara
- Se o usuário selecionar "Outro" com texto livre, mude para acompanhamento em texto simples (conforme a regra freeform de questioning.md)
- Se o usuário selecionar "Você decide", capture como Critério do Claude em CONTEXT.md
- Máximo de 2 perguntas por área — isso é leve, não um mergulho profundo

Colete todas as decisões em `$DECISIONS`.

**4.5d. Escrever CONTEXT.md**

Escreva `${QUICK_DIR}/${quick_id}-CONTEXT.md` usando a estrutura padrão de template de contexto:

```markdown
# Tarefa Rápida ${quick_id}: ${DESCRIPTION} - Contexto

**Reunido:** ${date}
**Status:** Pronto para planejamento

<domain>
## Limite da Tarefa

${DESCRIPTION}

</domain>

<decisions>
## Decisões de Implementação

### ${area_1_name}
- ${decisão_da_discussão}

### ${area_2_name}
- ${decisão_da_discussão}

### Critério do Claude
${áreas_onde_usuário_disse_você_decide_ou_áreas_não_discutidas}

</decisions>

<specifics>
## Ideias Específicas

${referências_específicas_ou_exemplos_da_discussão}

[Se nenhuma: "Sem requisitos específicos — aberto a abordagens padrão"]

</specifics>

<canonical_refs>
## Referências Canônicas

${quaisquer_specs_adrs_ou_docs_referenciados_durante_a_discussão}

[Se nenhuma: "Sem specs externos — requisitos totalmente capturados nas decisões acima"]

</canonical_refs>
```

Nota: O CONTEXT.md de tarefa rápida omite seções `<code_context>` e `<deferred>` (sem exploração de código-fonte, sem escopo de fase para diferir). Mantenha-o enxuto. A seção `<canonical_refs>` é incluída quando docs externos foram referenciados — omita-a apenas se nenhum doc externo se aplicar.

Informe: `Contexto capturado: ${QUICK_DIR}/${quick_id}-CONTEXT.md`

---

**Etapa 4.75: Fase de pesquisa (apenas quando `$RESEARCH_MODE`)**

Pule esta etapa inteiramente se NÃO for `$RESEARCH_MODE`.

Exiba banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISANDO TAREFA RÁPIDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Investigando abordagens para: ${DESCRIPTION}
```

Spawne um único pesquisador focado (não 4 pesquisadores paralelos como fases completas — tarefas rápidas precisam de pesquisa direcionada, não levantamentos amplos de domínio):

```
Task(
  prompt="
<research_context>

**Modo:** quick-task
**Tarefa:** ${DESCRIPTION}
**Saída:** ${QUICK_DIR}/${quick_id}-RESEARCH.md

<files_to_read>
- .planning/STATE.md (Estado do projeto — o que já foi construído)
- .planning/PROJECT.md (Contexto do projeto)
- ./CLAUDE.md (se existir — diretrizes específicas do projeto)
${DISCUSS_MODE ? '- ' + QUICK_DIR + '/' + quick_id + '-CONTEXT.md (Decisões do usuário — pesquisa deve alinhar com estas)' : ''}
</files_to_read>

${AGENT_SKILLS_PLANNER}

</research_context>

<focus>
Esta é uma tarefa rápida, não uma fase completa. A pesquisa deve ser concisa e direcionada:
1. Melhores bibliotecas/padrões para esta tarefa específica
2. Armadilhas comuns e como evitá-las
3. Pontos de integração com o código-fonte existente
4. Quaisquer restrições ou problemas a saber antes do planejamento

NÃO produza um levantamento completo do domínio. Mire em 1-2 páginas de descobertas acionáveis.
</focus>

<output>
Escreva a pesquisa em: ${QUICK_DIR}/${quick_id}-RESEARCH.md
Use o formato padrão de pesquisa mas mantenha enxuto — pule seções que não se aplicam.
Retorne: ## RESEARCH COMPLETE com caminho do arquivo
</output>
",
  subagent_type="gsd-phase-researcher",
  model="{planner_model}",
  description="Pesquisar: ${DESCRIPTION}"
)
```

Após o retorno do pesquisador:
1. Verifique se a pesquisa existe em `${QUICK_DIR}/${quick_id}-RESEARCH.md`
2. Informe: "Pesquisa concluída: ${QUICK_DIR}/${quick_id}-RESEARCH.md"

Se o arquivo de pesquisa não for encontrado, avise mas continue: "O agente de pesquisa não produziu saída — prosseguindo para o planejamento sem pesquisa."

---

**Etapa 5: Spawnar planejador (modo quick)**

**Se `$VALIDATE_MODE`:** Use o modo `quick-full` com restrições mais rígidas.

**Se NÃO for `$VALIDATE_MODE`:** Use o modo `quick` padrão.

```
Task(
  prompt="
<planning_context>

**Modo:** ${VALIDATE_MODE ? 'quick-full' : 'quick'}
**Diretório:** ${QUICK_DIR}
**Descrição:** ${DESCRIPTION}

<files_to_read>
- .planning/STATE.md (Estado do Projeto)
- ./CLAUDE.md (se existir — siga as diretrizes específicas do projeto)
${DISCUSS_MODE ? '- ' + QUICK_DIR + '/' + quick_id + '-CONTEXT.md (Decisões do usuário — bloqueadas, não revisite)' : ''}
${RESEARCH_MODE ? '- ' + QUICK_DIR + '/' + quick_id + '-RESEARCH.md (Descobertas de pesquisa — use para informar escolhas de implementação)' : ''}
</files_to_read>

${AGENT_SKILLS_PLANNER}

**Habilidades do projeto:** Verifique o diretório .claude/skills/ ou .agents/skills/ (se qualquer um existir) — leia arquivos SKILL.md, os planos devem considerar as regras de habilidades do projeto

</planning_context>

<constraints>
- Crie um ÚNICO plano com 1-3 tarefas focadas
- Tarefas rápidas devem ser atômicas e autocontidas
${RESEARCH_MODE ? '- Descobertas de pesquisa disponíveis — use-as para informar escolhas de biblioteca/padrão' : '- Sem fase de pesquisa'}
${VALIDATE_MODE ? '- Mire em ~40% de uso de contexto (estruturado para verificação)' : '- Mire em ~30% de uso de contexto (simples, focado)'}
${VALIDATE_MODE ? '- DEVE gerar `must_haves` no frontmatter do plano (truths, artifacts, key_links)' : ''}
${VALIDATE_MODE ? '- Cada tarefa DEVE ter campos `files`, `action`, `verify`, `done`' : ''}
</constraints>

<output>
Escreva o plano em: ${QUICK_DIR}/${quick_id}-PLAN.md
Retorne: ## PLANNING COMPLETE com caminho do plano
</output>
",
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Plano rápido: ${DESCRIPTION}"
)
```

Após o retorno do planejador:
1. Verifique se o plano existe em `${QUICK_DIR}/${quick_id}-PLAN.md`
2. Extraia a contagem de planos (tipicamente 1 para tarefas rápidas)
3. Informe: "Plano criado: ${QUICK_DIR}/${quick_id}-PLAN.md"

Se o plano não for encontrado, erro: "O planejador falhou ao criar ${quick_id}-PLAN.md"

---

**Etapa 5.5: Loop de verificação de plano (apenas quando `$VALIDATE_MODE`)**

Pule esta etapa inteiramente se NÃO for `$VALIDATE_MODE`.

Exiba banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO PLANO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning verificador de plano...
```

Prompt do verificador:

```markdown
<verification_context>
**Modo:** quick-full
**Descrição da Tarefa:** ${DESCRIPTION}

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano a verificar)
</files_to_read>

${AGENT_SKILLS_CHECKER}

**Escopo:** Esta é uma tarefa rápida, não uma fase completa. Pule verificações que requerem um objetivo de fase do ROADMAP.
</verification_context>

<check_dimensions>
- Cobertura de requisito: O plano endereça a descrição da tarefa?
- Completude da tarefa: As tarefas têm campos files, action, verify, done?
- Links chave: Os arquivos referenciados são reais?
- Sanidade de escopo: Está devidamente dimensionado para uma tarefa rápida (1-3 tarefas)?
- Derivação de must_haves: Os must_haves são rastreáveis à descrição da tarefa?

Pule: dependências entre planos (plano único), alinhamento com ROADMAP
${DISCUSS_MODE ? '- Conformidade com contexto: O plano honra as decisões bloqueadas do CONTEXT.md?' : '- Pule: conformidade com contexto (sem CONTEXT.md)'}
</check_dimensions>

<expected_output>
- ## VERIFICATION PASSED — todas as verificações passam
- ## ISSUES FOUND — lista estruturada de problemas
</expected_output>
```

```
Task(
  prompt=checker_prompt,
  subagent_type="gsd-plan-checker",
  model="{checker_model}",
  description="Verificar plano rápido: ${DESCRIPTION}"
)
```

**Tratar retorno do verificador:**

- **`## VERIFICATION PASSED`:** Exibir confirmação, prosseguir para a etapa 6.
- **`## ISSUES FOUND`:** Exibir problemas, verificar contagem de iterações, entrar no loop de revisão.

**Loop de revisão (máximo 2 iterações):**

Rastreie `iteration_count` (começa em 1 após o plano inicial + verificação).

**Se iteration_count < 2:**

Exiba: `Enviando de volta ao planejador para revisão... (iteração ${N}/2)`

Prompt de revisão:

```markdown
<revision_context>
**Modo:** quick-full (revisão)

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano existente)
</files_to_read>

${AGENT_SKILLS_PLANNER}

**Problemas do verificador:** ${structured_issues_from_checker}

</revision_context>

<instructions>
Faça atualizações direcionadas para resolver os problemas do verificador.
NÃO replaneje do zero a menos que os problemas sejam fundamentais.
Retorne o que mudou.
</instructions>
```

```
Task(
  prompt=revision_prompt,
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Revisar plano rápido: ${DESCRIPTION}"
)
```

Após o retorno do planejador → spawne o verificador novamente, incremente iteration_count.

**Se iteration_count >= 2:**

Exiba: `Máximo de iterações atingido. ${N} problemas restam:` + lista de problemas

Ofereça: 1) Forçar prosseguimento, 2) Abandonar

---

**Etapa 6: Spawnar executor**

Capture o HEAD atual antes de spawnar (usado para verificação de branch de worktree):
```bash
EXPECTED_BASE=$(git rev-parse HEAD)
```

Spawne gsd-executor com referência ao plano:

```
Task(
  prompt="
Execute a tarefa rápida ${quick_id}.

${USE_WORKTREES !== "false" ? `
<worktree_branch_check>
PRIMEIRA AÇÃO antes de qualquer outro trabalho: verifique se este branch de worktree é baseado no commit correto.
Execute: git merge-base HEAD ${EXPECTED_BASE}
Se o resultado diferir de ${EXPECTED_BASE}, faça hard-reset para a base correta (seguro — executa antes de qualquer trabalho do agente):
  git reset --hard ${EXPECTED_BASE}
Em seguida, verifique: if [ "$(git rev-parse HEAD)" != "${EXPECTED_BASE}" ]; then echo "ERRO: Não foi possível corrigir a base do worktree"; exit 1; fi
Isso corrige um problema conhecido onde EnterWorktree cria branches a partir do main em vez do HEAD do branch de feature (afeta todas as plataformas).
</worktree_branch_check>
` : ''}

<files_to_read>
- ${QUICK_DIR}/${quick_id}-PLAN.md (Plano)
- .planning/STATE.md (Estado do projeto)
- ./CLAUDE.md (Instruções do projeto, se existir)
- .claude/skills/ ou .agents/skills/ (Habilidades do projeto, se qualquer um existir — liste habilidades, leia SKILL.md para cada, siga regras relevantes durante a implementação)
</files_to_read>

${AGENT_SKILLS_EXECUTOR}

<constraints>
- Execute todas as tarefas do plano
- Commite cada tarefa atomicamente (apenas alterações de código)
- Crie resumo em: ${QUICK_DIR}/${quick_id}-SUMMARY.md
- NÃO commite artefatos de docs (SUMMARY.md, STATE.md, PLAN.md) — o orquestrador lida com o commit de docs na Etapa 8
- NÃO atualize ROADMAP.md (tarefas rápidas são separadas das fases planejadas)
</constraints>
",
  subagent_type="gsd-executor",
  model="{executor_model}",
  ${USE_WORKTREES !== "false" ? 'isolation="worktree",' : ''}
  description="Executar: ${DESCRIPTION}"
)
```

Após o retorno do executor:
1. **Limpeza de worktree:** Se o executor foi executado com `isolation="worktree"`, mescle o branch do worktree de volta e limpe
2. Verifique se o resumo existe em `${QUICK_DIR}/${quick_id}-SUMMARY.md`
3. Extraia o hash de commit do output do executor
4. Informe o status de conclusão

**Bug conhecido do Claude Code (classifyHandoffIfNeeded):** Se o executor reportar "failed" com erro `classifyHandoffIfNeeded is not defined`, este é um bug de runtime do Claude Code — não uma falha real. Verifique se o arquivo de resumo existe e se o git log mostra commits. Se sim, trate como bem-sucedido.

Se o resumo não for encontrado, erro: "O executor falhou ao criar ${quick_id}-SUMMARY.md"

---

**Etapa 6.25: Revisão de código (automática)**

Pule esta etapa inteiramente se `$FULL_MODE` for false.

**Gate de configuração:**
```bash
CODE_REVIEW_ENABLED=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.code_review 2>/dev/null || echo "true")
```
Se `"false"`, pule com mensagem "Revisão de código ignorada (workflow.code_review=false)".

---

**Etapa 6.5: Verificação (apenas quando `$VALIDATE_MODE`)**

Pule esta etapa inteiramente se NÃO for `$VALIDATE_MODE`.

Exiba banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► VERIFICANDO RESULTADOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning verificador...
```

---

**Etapa 7: Atualizar STATE.md**

Atualize STATE.md com o registro de conclusão da tarefa rápida.

**7a. Verifique se a seção "Tarefas Rápidas Concluídas" existe:**

Leia STATE.md e verifique a seção `### Quick Tasks Completed`.

**7b. Se a seção não existir, crie-a:**

Insira após a seção `### Bloqueios/Preocupações`.

**7c. Adicione nova linha à tabela:**

Use `date` do init.

**7d. Atualize a linha "Última atividade":**

Use `date` do init:
```
Última atividade: ${date} - Tarefa rápida ${quick_id} concluída: ${DESCRIPTION}
```

Use a ferramenta Edit para fazer essas alterações atomicamente.

---

**Etapa 8: Commit final e conclusão**

Prepare e commite artefatos de tarefa rápida.

Exiba output de conclusão:

**Se `$VALIDATE_MODE`:**
```
---

GSD > TAREFA RÁPIDA CONCLUÍDA (VALIDADA)

Tarefa Rápida ${quick_id}: ${DESCRIPTION}

${RESEARCH_MODE ? 'Pesquisa: ' + QUICK_DIR + '/' + quick_id + '-RESEARCH.md' : ''}
Resumo: ${QUICK_DIR}/${quick_id}-SUMMARY.md
Verificação: ${QUICK_DIR}/${quick_id}-VERIFICATION.md (${VERIFICATION_STATUS})
Commit: ${commit_hash}

---

Pronto para a próxima tarefa: /gsd-quick ${GSD_WS}
```

**Se NÃO for `$VALIDATE_MODE`:**
```
---

GSD > TAREFA RÁPIDA CONCLUÍDA

Tarefa Rápida ${quick_id}: ${DESCRIPTION}

${RESEARCH_MODE ? 'Pesquisa: ' + QUICK_DIR + '/' + quick_id + '-RESEARCH.md' : ''}
Resumo: ${QUICK_DIR}/${quick_id}-SUMMARY.md
Commit: ${commit_hash}

---

Pronto para a próxima tarefa: /gsd-quick ${GSD_WS}
```

</process>

<success_criteria>
- [ ] Validação de ROADMAP.md passa
- [ ] Usuário fornece descrição da tarefa
- [ ] Flags `--full`, `--validate`, `--discuss` e `--research` analisadas dos argumentos quando presentes
- [ ] `--full` define todos os booleanos (`$FULL_MODE`, `$DISCUSS_MODE`, `$RESEARCH_MODE`, `$VALIDATE_MODE`)
- [ ] Slug gerado (minúsculas, hifens, máx. 40 chars)
- [ ] ID rápido gerado (formato YYMMDD-xxx, precisão Base36 de 2s)
- [ ] Diretório criado em `.planning/quick/YYMMDD-xxx-slug/`
- [ ] (--discuss) Áreas cinzas identificadas e apresentadas, decisões capturadas em `${quick_id}-CONTEXT.md`
- [ ] (--research) Agente de pesquisa spawned, `${quick_id}-RESEARCH.md` criado
- [ ] `${quick_id}-PLAN.md` criado pelo planejador (honra decisões de CONTEXT.md quando --discuss, usa descobertas de RESEARCH.md quando --research)
- [ ] (--validate) Verificador de plano valida plano, loop de revisão limitado a 2
- [ ] `${quick_id}-SUMMARY.md` criado pelo executor
- [ ] (--validate) `${quick_id}-VERIFICATION.md` criado pelo verificador
- [ ] STATE.md atualizado com linha de tarefa rápida (coluna Status quando --validate)
- [ ] Artefatos commitados
</success_criteria>
