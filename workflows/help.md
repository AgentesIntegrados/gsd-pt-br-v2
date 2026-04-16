<purpose>
Exibe a referência completa de comandos GSD. Exiba APENAS o conteúdo de referência. NÃO adicione análise específica do projeto, status do git, sugestões de próximos passos ou qualquer comentário além da referência.
</purpose>

<reference>
# Referência de Comandos GSD

**GSD** (Get Shit Done) cria planos hierárquicos de projeto otimizados para desenvolvimento agêntico solo com Claude Code.

## Início Rápido

1. `/gsd-new-project` - Inicializa o projeto (inclui pesquisa, requisitos, roadmap)
2. `/gsd-plan-phase 1` - Cria plano detalhado para a primeira fase
3. `/gsd-execute-phase 1` - Executa a fase

## Mantendo-se Atualizado

O GSD evolui rapidamente. Atualize periodicamente:

```bash
npx get-shit-done-cc@latest
```

## Fluxo Principal

```
/gsd-new-project → /gsd-plan-phase → /gsd-execute-phase → repetir
```

### Inicialização do Projeto

**`/gsd-new-project`**
Inicializa um novo projeto através de um fluxo unificado.

Um comando leva você da ideia ao pronto-para-planejamento:
- Questionamento profundo para entender o que você está construindo
- Pesquisa de domínio opcional (spawna 4 agentes pesquisadores em paralelo)
- Definição de requisitos com escopo v1/v2/fora-do-escopo
- Criação de roadmap com detalhamento de fases e critérios de sucesso

Cria todos os artefatos `.planning/`:
- `PROJECT.md` — visão e requisitos
- `config.json` — modo de fluxo (interactive/yolo)
- `research/` — pesquisa de domínio (se selecionada)
- `REQUIREMENTS.md` — requisitos com escopo e REQ-IDs
- `ROADMAP.md` — fases mapeadas para os requisitos
- `STATE.md` — memória do projeto

Uso: `/gsd-new-project`

**`/gsd-map-codebase`**
Mapeia um codebase existente para projetos brownfield.

- Analisa o codebase com agentes Explore em paralelo
- Cria `.planning/codebase/` com 7 documentos focados
- Cobre stack, arquitetura, estrutura, convenções, testes, integrações, preocupações
- Use antes de `/gsd-new-project` em codebases existentes

Uso: `/gsd-map-codebase`

### Planejamento de Fase

**`/gsd-discuss-phase <number>`**
Ajuda a articular sua visão para uma fase antes do planejamento.

- Captura como você imagina esta fase funcionando
- Cria CONTEXT.md com sua visão, essenciais e limites
- Use quando você tem ideias sobre como algo deve parecer/funcionar
- Opcional `--batch` faz 2-5 perguntas relacionadas de uma vez em vez de uma por uma

Uso: `/gsd-discuss-phase 2`
Uso: `/gsd-discuss-phase 2 --batch`
Uso: `/gsd-discuss-phase 2 --batch=3`

**`/gsd-research-phase <number>`**
Pesquisa abrangente do ecossistema para domínios nichados/complexos.

- Descobre stack padrão, padrões arquiteturais, armadilhas
- Cria RESEARCH.md com conhecimento de "como especialistas constroem isso"
- Use para 3D, games, áudio, shaders, ML e outros domínios especializados
- Vai além de "qual biblioteca" para conhecimento do ecossistema

Uso: `/gsd-research-phase 3`

**`/gsd-list-phase-assumptions <number>`**
Veja o que Claude planeja fazer antes de começar.

- Mostra a abordagem pretendida do Claude para uma fase
- Permite corrigir mal-entendidos se Claude interpretou mal sua visão
- Nenhum arquivo criado — saída apenas conversacional

Uso: `/gsd-list-phase-assumptions 3`

**`/gsd-plan-phase <number>`**
Cria plano de execução detalhado para uma fase específica.

- Gera `.planning/phases/XX-phase-name/XX-YY-PLAN.md`
- Divide a fase em tarefas concretas e acionáveis
- Inclui critérios de verificação e medidas de sucesso
- Múltiplos planos por fase suportados (XX-01, XX-02, etc.)

Uso: `/gsd-plan-phase 1`
Resultado: Cria `.planning/phases/01-foundation/01-01-PLAN.md`

**Caminho Expresso com PRD:** Passe `--prd path/to/requirements.md` para pular a fase de discussão completamente. Seu PRD torna-se decisões bloqueadas no CONTEXT.md. Útil quando você já tem critérios de aceitação claros.

### Execução

**`/gsd-execute-phase <phase-number>`**
Executa todos os planos em uma fase, ou executa uma wave específica.

- Agrupa planos por wave (do frontmatter), executa waves sequencialmente
- Planos dentro de cada wave executam em paralelo via ferramenta Task
- Flag opcional `--wave N` executa apenas a Wave `N` e para a menos que a fase esteja agora totalmente completa
- Verifica o objetivo da fase após todos os planos concluírem
- Atualiza REQUIREMENTS.md, ROADMAP.md, STATE.md

Uso: `/gsd-execute-phase 5`
Uso: `/gsd-execute-phase 5 --wave 2`

### Roteador Inteligente

**`/gsd-do <description>`**
Roteia texto livre para o comando GSD correto automaticamente.

- Analisa input em linguagem natural para encontrar o melhor comando GSD correspondente
- Age como despachante — nunca faz o trabalho diretamente
- Resolve ambiguidade pedindo que você escolha entre as melhores opções
- Use quando você sabe o que quer mas não sabe qual comando `/gsd-*` usar

Uso: `/gsd-do corrigir o botão de login`
Uso: `/gsd-do refatorar o sistema de auth`
Uso: `/gsd-do quero começar um novo marco`

### Modo Rápido

**`/gsd-quick [--full] [--validate] [--discuss] [--research]`**
Executa tarefas pequenas e ad-hoc com garantias GSD mas pula agentes opcionais.

O modo rápido usa o mesmo sistema com um caminho mais curto:
- Spawna planner + executor (pula researcher, checker, verifier por padrão)
- Tarefas rápidas vivem em `.planning/quick/` separadas das fases planejadas
- Atualiza rastreamento no STATE.md (não no ROADMAP.md)

Flags habilitam etapas adicionais de qualidade:
- `--full` — Pipeline completo de qualidade: discussão + pesquisa + verificação de plano + verificação
- `--validate` — Verificação de plano (máx 2 iterações) e verificação pós-execução apenas
- `--discuss` — Discussão leve para identificar áreas cinzentas antes do planejamento
- `--research` — Agente de pesquisa focado investiga abordagens antes do planejamento

Flags granulares são combináveis: `--discuss --research --validate` é o mesmo que `--full`.

Uso: `/gsd-quick`
Uso: `/gsd-quick --full`
Uso: `/gsd-quick --research --validate`
Resultado: Cria `.planning/quick/NNN-slug/PLAN.md`, `.planning/quick/NNN-slug/SUMMARY.md`

---

**`/gsd-fast [description]`**
Executa uma tarefa trivial inline — sem subagentes, sem arquivos de planejamento, sem overhead.

Para tarefas pequenas demais para justificar planejamento: correções de typo, mudanças de config, commits esquecidos, adições simples. Executa no contexto atual, faz a mudança, commita e registra no STATE.md.

- Nenhum PLAN.md ou SUMMARY.md criado
- Nenhum subagente spawned (executa inline)
- ≤ 3 edições de arquivo — redireciona para `/gsd-quick` se a tarefa for não-trivial
- Commit atômico com mensagem convencional

Uso: `/gsd-fast "corrigir o typo no README"`
Uso: `/gsd-fast "adicionar .env ao gitignore"`

### Gerenciamento de Roadmap

**`/gsd-add-phase <description>`**
Adiciona nova fase ao final do marco atual.

- Anexa ao ROADMAP.md
- Usa o próximo número sequencial
- Atualiza a estrutura do diretório de fases

Uso: `/gsd-add-phase "Adicionar painel administrativo"`

**`/gsd-insert-phase <after> <description>`**
Insere trabalho urgente como fase decimal entre fases existentes.

- Cria fase intermediária (ex.: 7.1 entre 7 e 8)
- Útil para trabalho descoberto que deve acontecer no meio do marco
- Mantém a ordenação das fases

Uso: `/gsd-insert-phase 7 "Corrigir bug crítico de auth"`
Resultado: Cria a Fase 7.1

**`/gsd-remove-phase <number>`**
Remove uma fase futura e renumera as fases subsequentes.

- Apaga o diretório da fase e todas as referências
- Renumera todas as fases subsequentes para fechar a lacuna
- Funciona apenas em fases futuras (não iniciadas)
- Commit git preserva o registro histórico

Uso: `/gsd-remove-phase 17`
Resultado: Fase 17 apagada, fases 18-20 tornam-se 17-19

### Gerenciamento de Marco

**`/gsd-new-milestone <name>`**
Inicia um novo marco através do fluxo unificado.

- Questionamento profundo para entender o que você está construindo a seguir
- Pesquisa de domínio opcional (spawna 4 agentes pesquisadores em paralelo)
- Definição de requisitos com escopo
- Criação de roadmap com detalhamento de fases
- Flag opcional `--reset-phase-numbers` reinicia a numeração na Fase 1 e arquiva os diretórios de fase antigos primeiro por segurança

Espelha o fluxo `/gsd-new-project` para projetos brownfield (PROJECT.md existente).

Uso: `/gsd-new-milestone "Funcionalidades v2.0"`
Uso: `/gsd-new-milestone --reset-phase-numbers "Funcionalidades v2.0"`

**`/gsd-complete-milestone <version>`**
Arquiva o marco concluído e prepara para a próxima versão.

- Cria entrada no MILESTONES.md com estatísticas
- Arquiva detalhes completos no diretório milestones/
- Cria tag git para o release
- Prepara o workspace para a próxima versão

Uso: `/gsd-complete-milestone 1.0.0`

### Rastreamento de Progresso

**`/gsd-progress`**
Verifica o status do projeto e roteia inteligentemente para a próxima ação.

- Mostra barra de progresso visual e percentual de conclusão
- Resume trabalho recente dos arquivos SUMMARY
- Exibe posição atual e o que vem a seguir
- Lista decisões principais e issues abertas
- Oferece executar o próximo plano ou criá-lo se estiver ausente
- Detecta 100% de conclusão de marco

Uso: `/gsd-progress`

### Gerenciamento de Sessão

**`/gsd-resume-work`**
Retoma o trabalho da sessão anterior com restauração completa de contexto.

- Lê STATE.md para contexto do projeto
- Mostra posição atual e progresso recente
- Oferece próximas ações com base no estado do projeto

Uso: `/gsd-resume-work`

**`/gsd-pause-work`**
Cria handoff de contexto ao pausar trabalho no meio de uma fase.

- Cria arquivo .continue-here com o estado atual
- Atualiza a seção de continuidade de sessão do STATE.md
- Captura o contexto de trabalho em progresso

Uso: `/gsd-pause-work`

### Depuração

**`/gsd-debug [issue description]`**
Depuração sistemática com estado persistente entre resets de contexto.

- Coleta sintomas através de questionamento adaptativo
- Cria `.planning/debug/[slug].md` para rastrear a investigação
- Investiga usando o método científico (evidência → hipótese → teste)
- Sobrevive ao `/clear` — execute `/gsd-debug` sem args para retomar
- Arquiva issues resolvidas em `.planning/debug/resolved/`

Uso: `/gsd-debug "o botão de login não funciona"`
Uso: `/gsd-debug` (retoma sessão ativa)

### Notas Rápidas

**`/gsd-note <text>`**
Captura de ideia com zero atrito — um comando, salvo instantaneamente, sem perguntas.

- Salva nota com timestamp em `.planning/notes/` (ou `$HOME/.claude/notes/` globalmente)
- Três subcomandos: append (padrão), list, promote
- Promote converte uma nota em um todo estruturado
- Funciona sem um projeto (recorre ao escopo global)

Uso: `/gsd-note refatorar o sistema de hook`
Uso: `/gsd-note list`
Uso: `/gsd-note promote 3`
Uso: `/gsd-note --global ideia cross-projeto`

### Gerenciamento de Todo

**`/gsd-add-todo [description]`**
Captura ideia ou tarefa como todo a partir da conversa atual.

- Extrai contexto da conversa (ou usa a descrição fornecida)
- Cria arquivo de todo estruturado em `.planning/todos/pending/`
- Infere área a partir dos caminhos de arquivo para agrupamento
- Verifica duplicatas antes de criar
- Atualiza contagem de todos no STATE.md

Uso: `/gsd-add-todo` (infere da conversa)
Uso: `/gsd-add-todo Adicionar refresh de token de auth`

**`/gsd-check-todos [area]`**
Lista todos pendentes e seleciona um para trabalhar.

- Lista todos os todos pendentes com título, área, idade
- Filtro de área opcional (ex.: `/gsd-check-todos api`)
- Carrega contexto completo para o todo selecionado
- Roteia para ação apropriada (trabalhar agora, adicionar à fase, brainstorming)
- Move todo para done/ quando o trabalho começa

Uso: `/gsd-check-todos`
Uso: `/gsd-check-todos api`

### Testes de Aceitação do Usuário

**`/gsd-verify-work [phase]`**
Valida funcionalidades construídas através de UAT conversacional.

- Extrai entregáveis testáveis dos arquivos SUMMARY.md
- Apresenta testes um por um (respostas sim/não)
- Diagnostica falhas automaticamente e cria planos de correção
- Pronto para re-execução se problemas forem encontrados

Uso: `/gsd-verify-work 3`

### Publicar Trabalho

**`/gsd-ship [phase]`**
Cria um PR a partir do trabalho de fase concluído com um body gerado automaticamente.

- Faz push do branch para o remote
- Cria PR com resumo do SUMMARY.md, VERIFICATION.md, REQUIREMENTS.md
- Opcionalmente solicita revisão de código
- Atualiza STATE.md com status de publicação

Pré-requisitos: Fase verificada, CLI `gh` instalado e autenticado.

Uso: `/gsd-ship 4` ou `/gsd-ship 4 --draft`

---

**`/gsd-review --phase N [--gemini] [--claude] [--codex] [--coderabbit] [--opencode] [--qwen] [--cursor] [--all]`**
Revisão por pares cross-AI — invoca CLIs de IA externas para revisar independentemente os planos de fase.

- Detecta CLIs disponíveis (gemini, claude, codex, coderabbit)
- Cada CLI revisa os planos independentemente com o mesmo prompt estruturado
- CodeRabbit revisa o diff git atual (não um prompt) — pode levar até 5 minutos
- Produz REVIEWS.md com feedback por revisor e resumo de consenso
- Alimente as revisões de volta ao planejamento: `/gsd-plan-phase N --reviews`

Uso: `/gsd-review --phase 3 --all`

---

**`/gsd-pr-branch [target]`**
Cria um branch limpo para pull requests filtrando commits do .planning/.

- Classifica commits: somente código (incluir), somente planejamento (excluir), misto (incluir sem .planning/)
- Cherry-pica commits de código em um branch limpo
- Revisores veem apenas mudanças de código, sem artefatos GSD

Uso: `/gsd-pr-branch` ou `/gsd-pr-branch main`

---

**`/gsd-plant-seed [idea]`**
Captura uma ideia prospectiva com condições de gatilho para surfacing automático.

- Seeds preservam O QUÊ, QUANDO surfacing e trilhas para código relacionado
- Surfacing automático durante `/gsd-new-milestone` quando as condições de gatilho correspondem
- Melhor que itens adiados — gatilhos são verificados, não esquecidos

Uso: `/gsd-plant-seed "adicionar notificações em tempo real quando construirmos o sistema de eventos"`

---

**`/gsd-audit-uat`**
Auditoria cross-fase de todos os itens pendentes de UAT e verificação.
- Varre cada fase em busca de itens pendentes, pulados, bloqueados e human_needed
- Faz referência cruzada com o codebase para detectar documentação obsoleta
- Produz plano de testes humanos priorizado agrupado por testabilidade
- Use antes de iniciar um novo marco para limpar dívida de verificação

Uso: `/gsd-audit-uat`

### Auditoria de Marco

**`/gsd-audit-milestone [version]`**
Audita a conclusão do marco em relação à intenção original.

- Lê todos os arquivos VERIFICATION.md das fases
- Verifica cobertura de requisitos
- Spawna verificador de integração para fiação cross-fase
- Cria MILESTONE-AUDIT.md com lacunas e dívida técnica

Uso: `/gsd-audit-milestone`

**`/gsd-plan-milestone-gaps`**
Cria fases para fechar lacunas identificadas pela auditoria.

- Lê MILESTONE-AUDIT.md e agrupa lacunas em fases
- Prioriza por prioridade de requisito (must/should/nice)
- Adiciona fases de fechamento de lacunas ao ROADMAP.md
- Pronto para `/gsd-plan-phase` nas novas fases

Uso: `/gsd-plan-milestone-gaps`

### Configuração

**`/gsd-settings`**
Configura toggles de fluxo de trabalho e perfil de modelo de forma interativa.

- Ativa/desativa agentes researcher, plan checker, verifier
- Seleciona perfil de modelo (quality/balanced/budget/inherit)
- Atualiza `.planning/config.json`

Uso: `/gsd-settings`

**`/gsd-set-profile <profile>`**
Troca rapidamente o perfil de modelo para agentes GSD.

- `quality` — Opus em todo lugar exceto verificação
- `balanced` — Opus para planejamento, Sonnet para execução (padrão)
- `budget` — Sonnet para escrita, Haiku para pesquisa/verificação
- `inherit` — Usa o modelo da sessão atual para todos os agentes (OpenCode `/model`)

Uso: `/gsd-set-profile budget`

### Comandos Utilitários

**`/gsd-cleanup`**
Arquiva diretórios de fase acumulados de marcos concluídos.

- Identifica fases de marcos concluídos ainda em `.planning/phases/`
- Mostra resumo de dry-run antes de mover qualquer coisa
- Move diretórios de fase para `.planning/milestones/v{X.Y}-phases/`
- Use após múltiplos marcos para reduzir a bagunça em `.planning/phases/`

Uso: `/gsd-cleanup`

**`/gsd-help`**
Mostra esta referência de comandos.

**`/gsd-update`**
Atualiza o GSD para a versão mais recente com prévia do changelog.

- Mostra comparação entre versão instalada e mais recente
- Exibe entradas do changelog para versões que você perdeu
- Destaca breaking changes
- Confirma antes de executar a instalação
- Melhor que o `npx get-shit-done-cc` puro

Uso: `/gsd-update`

**`/gsd-join-discord`**
Participe da comunidade GSD no Discord.

- Obtenha ajuda, compartilhe o que você está construindo, mantenha-se atualizado
- Conecte-se com outros usuários GSD

Uso: `/gsd-join-discord`

## Arquivos e Estrutura

```
.planning/
├── PROJECT.md            # Visão do projeto
├── ROADMAP.md            # Detalhamento atual das fases
├── STATE.md              # Memória e contexto do projeto
├── RETROSPECTIVE.md      # Retrospectiva viva (atualizada por marco)
├── config.json           # Modo de fluxo e gates
├── todos/                # Ideias e tarefas capturadas
│   ├── pending/          # Todos aguardando trabalho
│   └── done/             # Todos concluídos
├── debug/                # Sessões de debug ativas
│   └── resolved/         # Issues resolvidas arquivadas
├── milestones/
│   ├── v1.0-ROADMAP.md       # Snapshot arquivado do roadmap
│   ├── v1.0-REQUIREMENTS.md  # Requisitos arquivados
│   └── v1.0-phases/          # Diretórios de fase arquivados (via /gsd-cleanup ou --archive-phases)
│       ├── 01-foundation/
│       └── 02-core-features/
├── codebase/             # Mapa do codebase (projetos brownfield)
│   ├── STACK.md          # Linguagens, frameworks, dependências
│   ├── ARCHITECTURE.md   # Padrões, camadas, fluxo de dados
│   ├── STRUCTURE.md      # Layout de diretórios, arquivos principais
│   ├── CONVENTIONS.md    # Padrões de código, nomenclatura
│   ├── TESTING.md        # Configuração de testes, padrões
│   ├── INTEGRATIONS.md   # Serviços externos, APIs
│   └── CONCERNS.md       # Dívida técnica, issues conhecidas
└── phases/
    ├── 01-foundation/
    │   ├── 01-01-PLAN.md
    │   └── 01-01-SUMMARY.md
    └── 02-core-features/
        ├── 02-01-PLAN.md
        └── 02-01-SUMMARY.md
```

## Modos de Fluxo

Definido durante `/gsd-new-project`:

**Modo Interativo**

- Confirma cada decisão importante
- Pausa nos checkpoints para aprovação
- Mais orientação ao longo do processo

**Modo YOLO**

- Aprova automaticamente a maioria das decisões
- Executa planos sem confirmação
- Para apenas em checkpoints críticos

Mude a qualquer momento editando `.planning/config.json`

## Configuração de Planejamento

Configure como os artefatos de planejamento são gerenciados em `.planning/config.json`:

**`planning.commit_docs`** (padrão: `true`)
- `true`: Artefatos de planejamento commitados no git (fluxo padrão)
- `false`: Artefatos de planejamento mantidos apenas localmente, não commitados

Quando `commit_docs: false`:
- Adicione `.planning/` ao seu `.gitignore`
- Útil para contribuições OSS, projetos de clientes ou manter o planejamento privado
- Todos os arquivos de planejamento ainda funcionam normalmente, apenas não rastreados no git

**`planning.search_gitignored`** (padrão: `false`)
- `true`: Adiciona `--no-ignore` a buscas amplas do ripgrep
- Necessário apenas quando `.planning/` está no gitignore e você quer que buscas no projeto inteiro o incluam

Exemplo de config:
```json
{
  "planning": {
    "commit_docs": false,
    "search_gitignored": true
  }
}
```

## Fluxos Comuns

**Iniciando um novo projeto:**

```
/gsd-new-project        # Fluxo unificado: questionamento → pesquisa → requisitos → roadmap
/clear
/gsd-plan-phase 1       # Crie planos para a primeira fase
/clear
/gsd-execute-phase 1    # Execute todos os planos na fase
```

**Retomando o trabalho após uma pausa:**

```
/gsd-progress  # Veja onde você parou e continue
```

**Adicionando trabalho urgente no meio do marco:**

```
/gsd-insert-phase 5 "Correção crítica de segurança"
/gsd-plan-phase 5.1
/gsd-execute-phase 5.1
```

**Concluindo um marco:**

```
/gsd-complete-milestone 1.0.0
/clear
/gsd-new-milestone  # Inicie o próximo marco (questionamento → pesquisa → requisitos → roadmap)
```

**Capturando ideias durante o trabalho:**

```
/gsd-add-todo                       # Capture a partir do contexto da conversa
/gsd-add-todo Corrigir z-index do modal  # Capture com descrição explícita
/gsd-check-todos                    # Revise e trabalhe nos todos
/gsd-check-todos api                # Filtre por área
```

**Depurando um problema:**

```
/gsd-debug "o envio do formulário falha silenciosamente"  # Inicie uma sessão de debug
# ... a investigação acontece, o contexto fica cheio ...
/clear
/gsd-debug                                                # Retome de onde parou
```

## Obtendo Ajuda

- Leia `.planning/PROJECT.md` para a visão do projeto
- Leia `.planning/STATE.md` para o contexto atual
- Verifique `.planning/ROADMAP.md` para o status das fases
- Execute `/gsd-progress` para verificar onde você está
</reference>
