<trigger>
Use este fluxo quando:
- Iniciando uma nova sessão em um projeto existente
- O usuário diz "continuar", "o que vem depois", "onde paramos", "retomar"
- Qualquer operação de planejamento quando .planning/ já existir
- O usuário retorna após um tempo longe do projeto
</trigger>

<purpose>
Restaurar imediatamente o contexto completo do projeto para que "Onde paramos?" tenha uma resposta imediata e completa.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/continuation-format.md
</required_reading>

<process>

<step name="initialize">
Carregar todo o contexto em uma única chamada:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init resume)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON: `state_exists`, `roadmap_exists`, `project_exists`, `planning_exists`, `has_interrupted_agent`, `interrupted_agent_id`, `commit_docs`.

**Se `state_exists` for true:** Ir para load_state
**Se `state_exists` for false mas `roadmap_exists` ou `project_exists` for true:** Oferecer reconstruir STATE.md
**Se `planning_exists` for false:** Este é um novo projeto — redirecionar para /gsd-new-project
</step>

<step name="load_state">

Ler e analisar STATE.md e depois PROJECT.md:

```bash
cat .planning/STATE.md
cat .planning/PROJECT.md
```

**Extrair do STATE.md:**

- **Referência do Projeto**: Valor central e foco atual
- **Posição Atual**: Fase X de Y, Plano A de B, Status
- **Progresso**: Barra de progresso visual
- **Decisões Recentes**: Decisões-chave que afetam o trabalho atual
- **Tarefas Pendentes**: Ideias capturadas durante as sessões
- **Bloqueios/Preocupações**: Questões em aberto
- **Continuidade da Sessão**: Onde paramos, arquivos de retomada

**Extrair do PROJECT.md:**

- **O que É Isso**: Descrição atual e precisa
- **Requisitos**: Validados, Ativos, Fora do Escopo
- **Decisões-Chave**: Registro completo de decisões com resultados
- **Restrições**: Limites rígidos de implementação

</step>

<step name="check_incomplete_work">
Verificar trabalho incompleto que precisa de atenção:

```bash
# Verificar handoff estruturado (preferido — legível por máquina)
cat .planning/HANDOFF.json 2>/dev/null || true

# Verificar arquivos continue-here (retomada no meio do plano)
ls .planning/phases/*/.continue-here*.md 2>/dev/null || true

# Verificar planos sem resumos (execução incompleta)
for plan in .planning/phases/*/*-PLAN.md; do
  [ -e "$plan" ] || continue
  summary="${plan/PLAN/SUMMARY}"
  [ ! -f "$summary" ] && echo "Incomplete: $plan"
done 2>/dev/null || true

# Verificar agentes interrompidos (usar has_interrupted_agent e interrupted_agent_id do init)
if [ "$has_interrupted_agent" = "true" ]; then
  echo "Interrupted agent: $interrupted_agent_id"
fi
```

**Se HANDOFF.json existir:**

- Esta é a fonte principal de retomada — dados estruturados do `/gsd-pause-work`
- Analisar `status`, `phase`, `plan`, `task`, `total_tasks`, `next_action`
- Verificar `blockers` e `human_actions_pending` — exibir imediatamente
- Verificar `completed_tasks` para itens `in_progress` — estes precisam de atenção primeiro
- Validar `uncommitted_files` contra `git status` — sinalizar divergência
- Usar `context_notes` para restaurar o modelo mental
- Sinalizar: "Handoff estruturado encontrado — retomando da tarefa {task}/{total_tasks}"
- **Após retomada bem-sucedida, excluir HANDOFF.json** (é um artefato de uso único)

**Se arquivo .continue-here existir (fallback):**

- Este é um ponto de retomada no meio do plano
- Ler o arquivo para contexto específico de retomada
- Sinalizar: "Checkpoint de meio de plano encontrado"

**Se PLAN sem SUMMARY existir:**

- A execução foi iniciada mas não concluída
- Sinalizar: "Execução de plano incompleto encontrada"

**Se agente interrompido encontrado:**

- Subagente foi instanciado mas a sessão terminou antes da conclusão
- Ler agent-history.json para detalhes da tarefa
- Sinalizar: "Agente interrompido encontrado"
  </step>

<step name="present_status">
Apresentar o status completo do projeto ao usuário:

```
╔══════════════════════════════════════════════════════════════╗
║  STATUS DO PROJETO                                            ║
╠══════════════════════════════════════════════════════════════╣
║  Construindo: [descrição em uma linha do PROJECT.md]          ║
║                                                               ║
║  Fase: [X] de [Y] - [Nome da fase]                           ║
║  Plano:  [A] de [B] - [Status]                               ║
║  Progresso: [██████░░░░] XX%                                 ║
║                                                               ║
║  Última atividade: [data] - [o que aconteceu]                ║
╚══════════════════════════════════════════════════════════════╝

[Se trabalho incompleto encontrado:]
⚠️  Trabalho incompleto detectado:
    - [arquivo .continue-here ou plano incompleto]

[Se agente interrompido encontrado:]
⚠️  Agente interrompido detectado:
    ID do agente: [id]
    Tarefa: [descrição da tarefa do agent-history.json]
    Interrompido em: [timestamp]

    Retomar com: ferramenta Task (parâmetro resume com o ID do agente)

[Se tarefas pendentes existirem:]
📋 [N] tarefas pendentes — /gsd-check-todos para revisar

[Se bloqueios existirem:]
⚠️  Preocupações em aberto:
    - [bloqueio 1]
    - [bloqueio 2]

[Se alinhamento não for ✓:]
⚠️  Alinhamento resumido: [status] - [avaliação]
```

</step>

<step name="determine_next_action">
Com base no estado do projeto, determinar a próxima ação mais lógica:

**Se agente interrompido existir:**
→ Principal: Retomar agente interrompido (ferramenta Task com parâmetro resume)
→ Opção: Começar do zero (abandonar trabalho do agente)

**Se HANDOFF.json existir:**
→ Principal: Retomar do handoff estruturado (prioridade máxima — contexto específico de tarefa/bloqueio)
→ Opção: Descartar handoff e reavaliara partir dos arquivos

**Se arquivo .continue-here existir:**
→ Alternativa: Retomar do checkpoint
→ Opção: Começar do zero no plano atual

**Se plano incompleto (PLAN sem SUMMARY):**
→ Principal: Completar o plano incompleto
→ Opção: Abandonar e seguir em frente

**Se fase em andamento, todos os planos completos:**
→ Principal: Avançar para a próxima fase (via fluxo de transição interno)
→ Opção: Revisar trabalho concluído

**Se fase pronta para planejar:**
→ Verificar se CONTEXT.md existe para esta fase:

- Se CONTEXT.md ausente:
  → Principal: Discutir a visão da fase (como o usuário imagina que vai funcionar)
  → Secundário: Planejar diretamente (pular coleta de contexto)
- Se CONTEXT.md existir:
  → Principal: Planejar a fase
  → Opção: Revisar roadmap

**Se fase pronta para executar:**
→ Principal: Executar o próximo plano
→ Opção: Revisar o plano primeiro
</step>

<step name="offer_options">
Apresentar opções contextuais com base no estado do projeto:

```
O que você gostaria de fazer?

[Ação principal baseada no estado — ex.:]
1. Retomar agente interrompido [se agente interrompido encontrado]
   OU
1. Executar fase (/gsd-execute-phase {phase} ${GSD_WS})
   OU
1. Discutir contexto da Fase 3 (/gsd-discuss-phase 3 ${GSD_WS}) [se CONTEXT.md ausente]
   OU
1. Planejar Fase 3 (/gsd-plan-phase 3 ${GSD_WS}) [se CONTEXT.md existir ou opção de discussão recusada]

[Opções secundárias:]
2. Revisar status da fase atual
3. Verificar tarefas pendentes ([N] pendentes)
4. Revisar alinhamento resumido
5. Outra coisa
```

**Nota:** Ao oferecer planejamento de fase, verificar primeiro a existência do CONTEXT.md:

```bash
ls .planning/phases/XX-name/*-CONTEXT.md 2>/dev/null || true
```

Se ausente, sugerir discuss-phase antes do plano. Se existir, oferecer o plano diretamente.

Aguardar seleção do usuário.
</step>

<step name="route_to_workflow">
Com base na seleção do usuário, direcionar para o fluxo apropriado:

- **Executar plano** → Mostrar comando para o usuário executar após limpar:
  ```
  ---

  ## ▶ Próximo Passo

  **{phase}-{plan}: [Nome do Plano]** — [objetivo do PLAN.md]

  `/clear` e então:

  `/gsd-execute-phase {phase} ${GSD_WS}`

  ---
  ```
- **Planejar fase** → Mostrar comando para o usuário executar após limpar:
  ```
  ---

  ## ▶ Próximo Passo

  **Fase [N]: [Nome]** — [Objetivo do ROADMAP.md]

  `/clear` e então:

  `/gsd-plan-phase [phase-number] ${GSD_WS}`

  ---

  **Também disponível:**
  - `/gsd-discuss-phase [N] ${GSD_WS}` — coletar contexto primeiro
  - `/gsd-research-phase [N] ${GSD_WS}` — investigar incógnitas

  ---
  ```
- **Avançar para próxima fase** → ./transition.md (fluxo interno, invocado inline — NÃO é um comando do usuário)
- **Verificar tarefas** → Ler .planning/todos/pending/, apresentar resumo
- **Revisar alinhamento** → Ler PROJECT.md, comparar com estado atual
- **Outra coisa** → Perguntar o que precisam
</step>

<step name="update_session">
Antes de prosseguir para o fluxo redirecionado, atualizar continuidade da sessão:

Atualizar STATE.md:

```markdown
## Continuidade da Sessão

Última sessão: [agora]
Parou em: Sessão retomada, prosseguindo para [ação]
Arquivo de retomada: [atualizado se aplicável]
```

Isso garante que se a sessão terminar inesperadamente, a próxima retomada saiba o estado.
</step>

</process>

<reconstruction>
Se STATE.md estiver ausente mas outros artefatos existirem:

"STATE.md ausente. Reconstruindo a partir dos artefatos..."

1. Ler PROJECT.md → Extrair "O que É Isso" e Valor Central
2. Ler ROADMAP.md → Determinar fases, encontrar posição atual
3. Varrer arquivos \*-SUMMARY.md → Extrair decisões, preocupações
4. Contar tarefas pendentes em .planning/todos/pending/
5. Verificar arquivos .continue-here → Continuidade da sessão

Reconstruir e escrever STATE.md, depois prosseguir normalmente.

Isso trata os casos onde:

- O projeto é anterior à introdução do STATE.md
- O arquivo foi acidentalmente excluído
- Clonagem do repositório sem estado completo de .planning/
  </reconstruction>

<quick_resume>
Se o usuário disser "continuar" ou "vamos":
- Carregar estado silenciosamente
- Determinar ação principal
- Executar imediatamente sem apresentar opções

"Continuando de [estado]... [ação]"
</quick_resume>

<success_criteria>
A retomada está completa quando:

- [ ] STATE.md carregado (ou reconstruído)
- [ ] Trabalho incompleto detectado e sinalizado
- [ ] Status claro apresentado ao usuário
- [ ] Próximas ações contextuais oferecidas
- [ ] Usuário sabe exatamente onde o projeto está
- [ ] Continuidade da sessão atualizada
      </success_criteria>
