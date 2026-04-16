<purpose>

Iniciar um novo ciclo de milestone para um projeto existente. Carrega o contexto do projeto, coleta os objetivos do milestone (de MILESTONE-CONTEXT.md ou da conversa), atualiza PROJECT.md e STATE.md, opcionalmente executa pesquisa paralela, define requisitos com escopo e REQ-IDs, inicia o roadmapper para criar um plano de execução em fases e faz o commit de todos os artefatos. Equivalente brownfield de new-project.

</purpose>

<required_reading>

Leia todos os arquivos referenciados pelo execution_context do prompt de invocação antes de começar.

</required_reading>

<available_agent_types>
Tipos válidos de subagentes GSD (use os nomes exatos — não use 'general-purpose' como alternativa):
- gsd-project-researcher — Pesquisa decisões técnicas no nível do projeto
- gsd-research-synthesizer — Sintetiza descobertas de agentes de pesquisa paralelos
- gsd-roadmapper — Cria roadmaps de execução em fases
</available_agent_types>

<process>

## 1. Carregar Contexto

Analise `$ARGUMENTS` antes de fazer qualquer coisa:
- Flag `--reset-phase-numbers` → opte por reiniciar a numeração das fases do roadmap em `1`
- texto restante → use como nome do milestone, se presente

Se a flag estiver ausente, mantenha o comportamento atual de continuar a numeração de fases do milestone anterior.

- Leia PROJECT.md (projeto existente, requisitos validados, decisões)
- Leia MILESTONES.md (o que foi entregue anteriormente)
- Leia STATE.md (todos pendentes, bloqueios)
- Verifique se existe MILESTONE-CONTEXT.md (de /gsd-discuss-milestone)

## 2. Coletar Objetivos do Milestone

**Se MILESTONE-CONTEXT.md existir:**
- Use funcionalidades e escopo do discuss-milestone
- Apresente resumo para confirmação

**Se não houver arquivo de contexto:**
- Apresente o que foi entregue no último milestone

**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da opção. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
- Pergunte diretamente (livre, NÃO AskUserQuestion): "O que você quer construir a seguir?"
- Aguarde a resposta, depois use AskUserQuestion para aprofundar os detalhes
- Se o usuário selecionar "Outro" para fornecer entrada livre, faça o acompanhamento em texto simples — não em outro AskUserQuestion

## 2.5. Escanear Seeds Plantadas

Verifique `.planning/seeds/` para arquivos de seed que correspondam aos objetivos do milestone coletados no passo 2.

```bash
ls .planning/seeds/SEED-*.md 2>/dev/null
```

**Se não houver arquivos de seed:** Pule este passo silenciosamente — não exiba nenhuma mensagem ou prompt.

**Se houver arquivos de seed:** Leia cada arquivo `SEED-*.md` e extraia do frontmatter e do corpo:
- **Ideia** — o título da seed (cabeçalho após o frontmatter, ex: `# SEED-001: <ideia>`)
- **Condições de gatilho** — o campo `trigger_when` do frontmatter e a lista de pontos da seção "When to Surface"
- **Plantada durante** — o campo `planted_during` do frontmatter (para contexto)

Compare as condições de gatilho de cada seed com os objetivos do milestone do passo 2. Uma seed corresponde quando suas condições de gatilho são relevantes para qualquer uma das funcionalidades ou objetivos-alvo do milestone.

**Se nenhuma seed corresponder:** Pule silenciosamente — não promova o usuário.

**Se seeds correspondentes forem encontradas:**

**Modo `--auto`:** Selecione automaticamente TODAS as seeds correspondentes. Registre: `[auto] Selecionada(s) N seed(s) correspondente(s): [lista de nomes de seeds]`

**Modo texto (`TEXT_MODE=true`):** Apresente seeds correspondentes como lista numerada em texto simples:
```
Seeds que correspondem aos seus objetivos do milestone:
1. SEED-001: <ideia> (gatilho: <trigger_when>)
2. SEED-003: <ideia> (gatilho: <trigger_when>)

Digite os números a incluir (separados por vírgula) ou "nenhum" para pular:
```

**Modo normal:** Apresente via AskUserQuestion:
```
AskUserQuestion(
  header: "Seeds",
  question: "Estas seeds plantadas correspondem aos seus objetivos do milestone. Incluir alguma no escopo deste milestone?",
  multiSelect: true,
  options: [
    { label: "SEED-001: <ideia>", description: "Gatilho: <trigger_when> | Plantada durante: <planted_during>" },
    ...
  ]
)
```

**Após a seleção:**
- Seeds selecionadas se tornam contexto adicional para a definição de requisitos no passo 9. Armazene-as em um acumulador (ex: `$SELECTED_SEEDS`) para que o passo 9 possa referenciar as ideias e suas seções "Why This Matters" ao definir requisitos.
- Seeds não selecionadas permanecem intocadas em `.planning/seeds/` — nunca exclua ou modifique arquivos de seed durante este workflow.

## 3. Determinar Versão do Milestone

- Analise a última versão de MILESTONES.md
- Sugira a próxima versão (v1.0 → v1.1, ou v2.0 para maior)
- Confirme com o usuário

## 3.5. Verificar Entendimento do Milestone

Antes de escrever qualquer arquivo, apresente um resumo do que foi coletado e peça confirmação.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► RESUMO DO MILESTONE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Milestone v[X.Y]: [Nome]**

**Objetivo:** [Uma frase]

**Funcionalidades-alvo:**
- [Funcionalidade 1]
- [Funcionalidade 2]
- [Funcionalidade 3]

**Contexto principal:** [Quaisquer restrições, decisões ou notas importantes do questionamento]
```

AskUserQuestion:
- header: "Confirmar?"
- question: "Isso captura o que você quer construir neste milestone?"
- options:
  - "Parece bom" — Prosseguir para escrever PROJECT.md
  - "Ajustar" — Deixe-me corrigir ou adicionar detalhes

**Se "Ajustar":** Pergunte o que precisa ser alterado (texto simples, NÃO AskUserQuestion). Incorpore as mudanças, reapresente o resumo. Repita até que "Parece bom" seja selecionado.

**Se "Parece bom":** Prossiga para o Passo 4.

## 4. Atualizar PROJECT.md

Adicione/atualize:

```markdown
## Milestone Atual: v[X.Y] [Nome]

**Objetivo:** [Uma frase descrevendo o foco do milestone]

**Funcionalidades-alvo:**
- [Funcionalidade 1]
- [Funcionalidade 2]
- [Funcionalidade 3]
```

Atualize a seção de requisitos Ativos e o rodapé "Last updated".

Certifique-se de que a seção `## Evolution` existe em PROJECT.md. Se estiver faltando (projetos criados antes deste recurso), adicione-a antes do rodapé:

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

## 5. Atualizar STATE.md

```markdown
## Current Position

Phase: Not started (defining requirements)
Plan: —
Status: Definindo requisitos
Last activity: [hoje] — Milestone v[X.Y] iniciado
```

Mantenha a seção Accumulated Context do milestone anterior.

## 6. Limpeza e Commit

Exclua MILESTONE-CONTEXT.md se existir (consumido).

Limpe os diretórios de fase remanescentes do milestone anterior:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases clear --confirm
```

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: start milestone v[X.Y] [Name]" --files .planning/PROJECT.md .planning/STATE.md
```

## 7. Carregar Contexto e Resolver Modelos

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init new-milestone)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_RESEARCHER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-project-researcher 2>/dev/null)
AGENT_SKILLS_SYNTHESIZER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-synthesizer 2>/dev/null)
AGENT_SKILLS_ROADMAPPER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-roadmapper 2>/dev/null)
```

Extraia do JSON de init: `researcher_model`, `synthesizer_model`, `roadmapper_model`, `commit_docs`, `research_enabled`, `current_milestone`, `project_exists`, `roadmap_exists`, `latest_completed_milestone`, `phase_dir_count`, `phase_archive_path`.

## 7.5 Segurança de reset de fase (somente quando `--reset-phase-numbers`)

Se `--reset-phase-numbers` estiver ativo:

1. Defina o número de fase inicial como `1` para o próximo roadmap.
2. Se `phase_dir_count > 0`, arquive os diretórios de fase antigos antes do roadmapping para que novos diretórios `01-*` / `02-*` não colidam com diretórios de milestone obsoletos.

Se `phase_dir_count > 0` e `phase_archive_path` estiver disponível:

```bash
mkdir -p "${phase_archive_path}"
find .planning/phases -mindepth 1 -maxdepth 1 -type d -exec mv {} "${phase_archive_path}/" \;
```

Em seguida, verifique se `.planning/phases/` não contém mais diretórios de milestone antigos antes de continuar.

Se `phase_dir_count > 0` mas `phase_archive_path` estiver faltando:
- Pare e explique que redefinir a numeração é inseguro sem um destino de arquivamento do milestone concluído.
- Diga ao usuário para concluir/arquivar o milestone anterior primeiro, depois execute novamente `/gsd-new-milestone --reset-phase-numbers ${GSD_WS}`.

## 8. Decisão de Pesquisa

Verifique `research_enabled` do JSON de init (carregado da config).

**Se `research_enabled` for `true`:**

AskUserQuestion: "Pesquisar o ecossistema do domínio para as novas funcionalidades antes de definir os requisitos?"
- "Pesquisar primeiro (Recomendado)" — Descubra padrões, funcionalidades, arquitetura para NOVAS capacidades
- "Pular pesquisa para este milestone" — Vá direto para os requisitos (não altera seu padrão)

**Se `research_enabled` for `false`:**

AskUserQuestion: "Pesquisar o ecossistema do domínio para as novas funcionalidades antes de definir os requisitos?"
- "Pular pesquisa (padrão atual)" — Vá direto para os requisitos
- "Pesquisar primeiro" — Descubra padrões, funcionalidades, arquitetura para NOVAS capacidades

**IMPORTANTE:** NÃO persista esta escolha em config.json. A configuração `workflow.research` é uma preferência persistente do usuário que controla o comportamento de plan-phase em todo o projeto. Alterá-la aqui mudaria silenciosamente o comportamento futuro de `/gsd-plan-phase`. Para alterar o padrão, use `/gsd-settings`.

**Se o usuário escolheu "Pesquisar primeiro":**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISANDO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando 4 pesquisadores em paralelo...
  → Stack, Funcionalidades, Arquitetura, Armadilhas
```

```bash
mkdir -p .planning/research
```

Inicie 4 agentes gsd-project-researcher em paralelo. Cada um usa este template com campos específicos de dimensão:

**Estrutura comum para todos os 4 pesquisadores:**
```
Task(prompt="
<research_type>Project Research — {DIMENSION} for [new features].</research_type>

<milestone_context>
SUBSEQUENT MILESTONE — Adding [target features] to existing app.
{EXISTING_CONTEXT}
Focus ONLY on what's needed for the NEW features.
</milestone_context>

<question>{QUESTION}</question>

<files_to_read>
- .planning/PROJECT.md (Project context)
</files_to_read>

${AGENT_SKILLS_RESEARCHER}

<downstream_consumer>{CONSUMER}</downstream_consumer>

<quality_gate>{GATES}</quality_gate>

<output>
Write to: .planning/research/{FILE}
Use template: $HOME/.claude/get-shit-done/templates/research-project/{FILE}
</output>
", subagent_type="gsd-project-researcher", model="{researcher_model}", description="{DIMENSION} research")
```

**Campos específicos por dimensão:**

| Campo | Stack | Funcionalidades | Arquitetura | Armadilhas |
|-------|-------|-----------------|-------------|------------|
| EXISTING_CONTEXT | Capacidades validadas existentes (NÃO pesquise novamente): [de PROJECT.md] | Funcionalidades existentes (já construídas): [de PROJECT.md] | Arquitetura existente: [de PROJECT.md ou mapa da codebase] | Foque em erros comuns ao ADICIONAR essas funcionalidades ao sistema existente |
| QUESTION | Quais adições/mudanças de stack são necessárias para [novas funcionalidades]? | Como [funcionalidades-alvo] normalmente funcionam? Comportamento esperado? | Como [funcionalidades-alvo] se integram à arquitetura existente? | Erros comuns ao adicionar [funcionalidades-alvo] a [domínio]? |
| CONSUMER | Bibliotecas específicas com versões para NOVAS capacidades, pontos de integração, o que NÃO adicionar | Table stakes vs diferenciadores vs anti-funcionalidades, complexidade anotada, dependências de existentes | Pontos de integração, novos componentes, mudanças no fluxo de dados, ordem de construção sugerida | Sinais de alerta, estratégia de prevenção, qual fase deve tratar isso |
| GATES | Versões atuais (verifique com Context7), justificativa explica POR QUÊ, integração considerada | Categorias claras, complexidade anotada, dependências identificadas | Pontos de integração identificados, novo vs modificado explícito, ordem de construção considera dependências | Armadilhas específicas de adicionar essas funcionalidades, armadilhas de integração cobertas, prevenção acionável |
| FILE | STACK.md | FEATURES.md | ARCHITECTURE.md | PITFALLS.md |

Após a conclusão de todos os 4, inicie o sintetizador:

```
Task(prompt="
Synthesize research outputs into SUMMARY.md.

<files_to_read>
- .planning/research/STACK.md
- .planning/research/FEATURES.md
- .planning/research/ARCHITECTURE.md
- .planning/research/PITFALLS.md
</files_to_read>

${AGENT_SKILLS_SYNTHESIZER}

Write to: .planning/research/SUMMARY.md
Use template: $HOME/.claude/get-shit-done/templates/research-project/SUMMARY.md
Commit after writing.
", subagent_type="gsd-research-synthesizer", model="{synthesizer_model}", description="Synthesize research")
```

Exiba as descobertas principais de SUMMARY.md:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PESQUISA CONCLUÍDA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Adições ao stack:** [de SUMMARY.md]
**Table stakes de funcionalidades:** [de SUMMARY.md]
**Atenção:** [de SUMMARY.md]
```

**Se "Pular pesquisa":** Continue para o Passo 9.

## 9. Definir Requisitos

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DEFININDO REQUISITOS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Leia PROJECT.md: valor central, objetivos do milestone atual, requisitos validados (o que existe).

**Se `$SELECTED_SEEDS` não for vazio (do passo 2.5):** Inclua as ideias de seeds selecionadas e suas seções "Why This Matters" como entrada adicional ao definir requisitos. Seeds fornecem ideias de funcionalidades validadas pelo usuário que devem ser incorporadas nas categorias de requisitos junto com descobertas de pesquisa ou funcionalidades coletadas na conversa.

**Se a pesquisa existir:** Leia FEATURES.md, extraia categorias de funcionalidades.

Apresente funcionalidades por categoria:
```
## [Categoria 1]
**Table stakes:** Funcionalidade A, Funcionalidade B
**Diferenciadores:** Funcionalidade C, Funcionalidade D
**Notas de pesquisa:** [quaisquer notas relevantes]
```

**Se não houver pesquisa:** Colete requisitos por conversa. Pergunte: "Quais são as principais coisas que os usuários precisam fazer com [novas funcionalidades]?" Esclareça, aprofunde as capacidades relacionadas, agrupe em categorias.

**Defina o escopo de cada categoria** via AskUserQuestion (multiSelect: true, header máximo 12 chars):
- "[Funcionalidade 1]" — [breve descrição]
- "[Funcionalidade 2]" — [breve descrição]
- "Nenhuma para este milestone" — Adiar toda a categoria

Rastreie: Selecionadas → este milestone. Não selecionadas table stakes → futuro. Não selecionadas diferenciadores → fora do escopo.

**Identifique lacunas** via AskUserQuestion:
- "Não, a pesquisa cobriu" — Prosseguir
- "Sim, vou adicionar algumas" — Capturar adições

**Gere REQUIREMENTS.md:**
- Requisitos v1 agrupados por categoria (caixas de seleção, REQ-IDs)
- Requisitos Futuros (adiados)
- Fora do Escopo (exclusões explícitas com justificativa)
- Seção de Rastreabilidade (vazia, preenchida pelo roadmap)

**Formato REQ-ID:** `[CATEGORIA]-[NÚMERO]` (AUTH-01, NOTIF-02). Continue a numeração a partir do existente.

**Critérios de qualidade de requisitos:**

Bons requisitos são:
- **Específicos e testáveis:** "Usuário pode redefinir senha via link de e-mail" (não "Tratar redefinição de senha")
- **Centrados no usuário:** "Usuário pode X" (não "Sistema faz Y")
- **Atômicos:** Uma capacidade por requisito (não "Usuário pode fazer login e gerenciar perfil")
- **Independentes:** Dependências mínimas de outros requisitos

Apresente a lista COMPLETA de requisitos para confirmação:

```
## Requisitos do Milestone v[X.Y]

### [Categoria 1]
- [ ] **CAT1-01**: Usuário pode fazer X
- [ ] **CAT1-02**: Usuário pode fazer Y

### [Categoria 2]
- [ ] **CAT2-01**: Usuário pode fazer Z

Isso captura o que você está construindo? (sim / ajustar)
```

Se "ajustar": Retorne ao escopo.

**Faça o commit dos requisitos:**
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: define milestone v[X.Y] requirements" --files .planning/REQUIREMENTS.md
```

## 10. Criar Roadmap

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CRIANDO ROADMAP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Iniciando roadmapper...
```

**Número de fase inicial:**
- Se `--reset-phase-numbers` estiver ativo, comece na **Fase 1**
- Caso contrário, continue a partir do último número de fase do milestone anterior (v1.0 terminou na fase 5 → v1.1 começa na fase 6)

```
Task(prompt="
<planning_context>
<files_to_read>
- .planning/PROJECT.md
- .planning/REQUIREMENTS.md
- .planning/research/SUMMARY.md (if exists)
- .planning/config.json
- .planning/MILESTONES.md
</files_to_read>

${AGENT_SKILLS_ROADMAPPER}

</planning_context>

<instructions>
Create roadmap for milestone v[X.Y]:
1. Respect the selected numbering mode:
   - `--reset-phase-numbers` → start at Phase 1
   - default behavior → continue from the previous milestone's last phase number
2. Derive phases from THIS MILESTONE's requirements only
3. Map every requirement to exactly one phase
4. Derive 2-5 success criteria per phase (observable user behaviors)
5. Validate 100% coverage
6. Write files immediately (ROADMAP.md, STATE.md, update REQUIREMENTS.md traceability)
7. Return ROADMAP CREATED with summary

Write files first, then return.
</instructions>
", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Create roadmap")
```

**Tratar retorno:**

**Se `## ROADMAP BLOCKED`:** Apresente o bloqueio, trabalhe com o usuário, reinicie.

**Se `## ROADMAP CREATED`:** Leia ROADMAP.md, apresente inline:

```
## Roadmap Proposto

**[N] fases** | **[X] requisitos mapeados** | Todos cobertos ✓

| # | Fase | Objetivo | Requisitos | Critérios de Sucesso |
|---|------|----------|------------|---------------------|
| [N] | [Nome] | [Objetivo] | [REQ-IDs] | [contagem] |

### Detalhes das Fases

**Fase [N]: [Nome]**
Objetivo: [objetivo]
Requisitos: [REQ-IDs]
Critérios de sucesso:
1. [critério]
2. [critério]
```

**Solicite aprovação** via AskUserQuestion:
- "Aprovar" — Fazer commit e continuar
- "Ajustar fases" — Me diga o que mudar
- "Revisar arquivo completo" — Mostrar ROADMAP.md bruto

**Se "Ajustar":** Receba notas, reinicie o roadmapper com contexto de revisão, repita até aprovação.
**Se "Revisar":** Exiba ROADMAP.md bruto, pergunte novamente.

**Faça o commit do roadmap** (após aprovação):
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: create milestone v[X.Y] roadmap ([N] phases)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md
```

## 11. Concluído

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► MILESTONE INICIALIZADO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Milestone v[X.Y]: [Nome]**

| Artefato       | Localização                 |
|----------------|-----------------------------|
| Projeto        | `.planning/PROJECT.md`      |
| Pesquisa       | `.planning/research/`       |
| Requisitos     | `.planning/REQUIREMENTS.md` |
| Roadmap        | `.planning/ROADMAP.md`      |

**[N] fases** | **[X] requisitos** | Pronto para construir ✓

## ▶ Próximo Passo

**Fase [N]: [Nome da Fase]** — [Objetivo]

`/clear` e depois:

`/gsd-discuss-phase [N] ${GSD_WS}` — coletar contexto e esclarecer abordagem

Também: `/gsd-plan-phase [N] ${GSD_WS}` — pular discussão, planejar diretamente
```

</process>

<success_criteria>
- [ ] PROJECT.md atualizado com seção de Milestone Atual
- [ ] STATE.md reiniciado para novo milestone
- [ ] MILESTONE-CONTEXT.md consumido e excluído (se existia)
- [ ] Pesquisa concluída (se selecionada) — 4 agentes paralelos, ciente do milestone
- [ ] Requisitos coletados e com escopo definido por categoria
- [ ] REQUIREMENTS.md criado com REQ-IDs
- [ ] gsd-roadmapper iniciado com contexto de numeração de fases
- [ ] Arquivos do roadmap escritos imediatamente (não rascunho)
- [ ] Feedback do usuário incorporado (se houver)
- [ ] Modo de numeração de fases respeitado (continuado ou reiniciado)
- [ ] Todos os commits realizados (se documentos de planejamento forem commitados)
- [ ] Usuário sabe o próximo passo: `/gsd-discuss-phase [N] ${GSD_WS}`

**Commits atômicos:** Cada fase faz o commit de seus artefatos imediatamente.
</success_criteria>
