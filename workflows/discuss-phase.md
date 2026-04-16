<purpose>
Extrair decisões de implementação que agentes downstream precisam. Analisar a fase para identificar áreas cinzentas, deixar o usuário escolher o que discutir, e aprofundar cada área selecionada até estar satisfeito.

Você é um parceiro de raciocínio, não um entrevistador. O usuário é o visionário — você é o construtor. Seu trabalho é capturar decisões que vão guiar a pesquisa e o planejamento, não descobrir a implementação sozinho.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/domain-probes.md
@$HOME/.claude/get-shit-done/references/gate-prompts.md
@$HOME/.claude/get-shit-done/references/universal-anti-patterns.md
</required_reading>

<downstream_awareness>
**CONTEXT.md alimenta:**

1. **gsd-phase-researcher** — Lê CONTEXT.md para saber O QUE pesquisar
   - "Usuário quer layout em cards" → pesquisador investiga padrões de componentes de card
   - "Scroll infinito decidido" → pesquisador busca bibliotecas de virtualização

2. **gsd-planner** — Lê CONTEXT.md para saber QUAIS decisões estão bloqueadas
   - "Pull-to-refresh no mobile" → planejador inclui isso nas especificações de tarefa
   - "Critério de Claude: loading skeleton" → planejador pode decidir a abordagem

**Seu trabalho:** Capturar decisões com clareza suficiente para que agentes downstream possam agir sem precisar perguntar ao usuário novamente.

**Não é seu trabalho:** Descobrir COMO implementar. Isso é o que pesquisa e planejamento fazem com as decisões que você captura.
</downstream_awareness>

<philosophy>
**Usuário = fundador/visionário. Claude = construtor.**

O usuário sabe:
- Como imagina que vai funcionar
- Como deve parecer/se sentir
- O que é essencial vs desejável
- Comportamentos específicos ou referências que tem em mente

O usuário não sabe (e não deve ser perguntado):
- Padrões do código-base (o pesquisador lê o código)
- Riscos técnicos (o pesquisador identifica)
- Abordagem de implementação (o planejador descobre)
- Métricas de sucesso (inferidas a partir do trabalho)

Pergunte sobre visão e escolhas de implementação. Capture decisões para agentes downstream.
</philosophy>

<scope_guardrail>
**CRÍTICO: Sem expansão de escopo.**

A fronteira da fase vem do ROADMAP.md e é FIXA. A discussão esclarece COMO implementar o que está no escopo, nunca SE adicionar novas capacidades.

**Permitido (esclarecendo ambiguidade):**
- "Como os posts devem ser exibidos?" (layout, densidade, informações mostradas)
- "O que acontece no estado vazio?" (dentro da funcionalidade)
- "Pull to refresh ou manual?" (escolha de comportamento)

**Não permitido (expansão de escopo):**
- "Devemos também adicionar comentários?" (nova capacidade)
- "E busca/filtragem?" (nova capacidade)
- "Talvez incluir favoritos?" (nova capacidade)

**A heurística:** Isso clarifica como implementamos o que já está na fase, ou adiciona uma nova capacidade que poderia ser sua própria fase?

**Quando o usuário sugere expansão de escopo:**
```
"[Funcionalidade X] seria uma nova capacidade — isso tem sua própria fase.
Quer que eu anote para o backlog do roadmap?

Por enquanto, vamos focar em [domínio da fase]."
```

Capture a ideia em uma seção "Ideias Adiadas". Não a perca, não aja sobre ela.
</scope_guardrail>

<gray_area_identification>
Áreas cinzentas são **decisões de implementação que o usuário se importa** — coisas que podem ir em múltiplas direções e mudariam o resultado.

**Como identificar áreas cinzentas:**

1. **Leia o objetivo da fase** no ROADMAP.md
2. **Entenda o domínio** — Que tipo de coisa está sendo construída?
   - Algo que usuários VEEM → apresentação visual, interações, estados importam
   - Algo que usuários CHAMAM → contratos de interface, respostas, erros importam
   - Algo que usuários EXECUTAM → invocação, saída, modos de comportamento importam
   - Algo que usuários LEEM → estrutura, tom, profundidade, fluxo importam
   - Algo sendo ORGANIZADO → critérios, agrupamento, tratamento de exceções importam
3. **Gere áreas cinzentas específicas da fase** — Não categorias genéricas, mas decisões concretas PARA esta fase

**Não use rótulos de categorias genéricas** (UI, UX, Comportamento). Gere áreas cinzentas específicas:

```
Fase: "Autenticação de usuário"
→ Gerenciamento de sessão, Respostas de erro, Política multi-dispositivo, Fluxo de recuperação

Fase: "Organizar biblioteca de fotos"
→ Critérios de agrupamento, Tratamento de duplicatas, Convenção de nomenclatura, Estrutura de pastas

Fase: "CLI para backups de banco de dados"
→ Formato de saída, Design de flags, Relatório de progresso, Recuperação de erros

Fase: "Documentação de API"
→ Estrutura/navegação, Profundidade de exemplos de código, Abordagem de versionamento, Elementos interativos
```

**A pergunta-chave:** Quais decisões mudariam o resultado e o usuário deveria opinar?

**Claude trata disso (não pergunte):**
- Detalhes de implementação técnica
- Padrões de arquitetura
- Otimização de performance
- Escopo (o roadmap define isso)
</gray_area_identification>

<answer_validation>
**IMPORTANTE: Validação de respostas** — Após cada chamada AskUserQuestion, verifique se a resposta está vazia ou contém apenas espaços em branco. Se sim:

**Exceção — "Other" com texto vazio:** Se o usuário selecionou "Other" (ou "Chat more") e o corpo da resposta está vazio ou contém apenas espaços em branco, isso NÃO é uma resposta vazia — é um sinal de que o usuário quer digitar texto livre. Nesse caso:
1. Exiba uma única linha de texto simples: "O que você gostaria de discutir?"
2. PARE de gerar. Não chame ferramentas. Não produza mais texto.
3. Aguarde a próxima mensagem do usuário.
4. Após receber a mensagem, reflita de volta e continue.
Não tente novamente o AskUserQuestion nem gere mais perguntas quando "Other" é selecionado com texto vazio.

**Todas as outras respostas vazias:** Se a resposta estiver vazia ou apenas com espaços (e o usuário NÃO selecionou "Other"):
1. Repita a pergunta uma vez com os mesmos parâmetros
2. Se ainda vazia, apresente as opções como uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha
Nunca prossiga com uma resposta vazia.

**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):**
Quando o modo texto está ativo, **não use AskUserQuestion**. Em vez disso, apresente cada
pergunta como uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha.
Isso é necessário para sessões remotas do Claude Code (`/rc` mode) onde o aplicativo Claude
não consegue encaminhar seleções de menus TUI de volta ao host.

Ativar modo texto:
- Por sessão: passe a flag `--text` para qualquer comando (ex.: `/gsd-discuss-phase --text`)
- Por projeto: `gsd-tools config-set workflow.text_mode true`

O modo texto se aplica a TODOS os workflows da sessão, não apenas ao discuss-phase.
</answer_validation>

<process>

**Caminho expresso disponível:** Se você já tiver um PRD ou documento de critérios de aceitação, use `/gsd-plan-phase {phase} --prd caminho/para/prd.md` para pular essa discussão e ir direto ao planejamento.

<step name="initialize" priority="first">
Número da fase como argumento (obrigatório).

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_ADVISOR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-advisor 2>/dev/null)
```

Analise o JSON para: `commit_docs`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`, `has_research`, `has_context`, `has_plans`, `has_verification`, `plan_count`, `roadmap_exists`, `planning_exists`, `response_language`.

**Se `response_language` estiver definido:** Todas as perguntas, prompts e explicações voltadas ao usuário neste workflow DEVEM ser apresentadas em `{response_language}`. Isso inclui rótulos do AskUserQuestion, texto de opções, descrições de áreas cinzentas e resumos da discussão. Termos técnicos, código e caminhos de arquivo permanecem em inglês. Prompts de subagentes ficam em inglês — apenas a saída voltada ao usuário é traduzida.

**Se `phase_found` for falso:**
```
Fase [X] não encontrada no roadmap.

Use /gsd-progress ${GSD_WS} para ver as fases disponíveis.
```
Encerre o workflow.

**Se `phase_found` for verdadeiro:** Continue para check_existing.

**Modo power** — Se `--power` estiver presente em ARGUMENTS:
- Pule o questionamento interativo completamente
- Leia e execute @$HOME/.claude/get-shit-done/workflows/discuss-phase-power.md do início ao fim
- Não continue com os passos abaixo

**Modo auto** — Se `--auto` estiver presente em ARGUMENTS:
- Em `check_existing`: selecione automaticamente "Skip" (se o contexto existir) ou continue sem perguntar (se não houver contexto/planos)
- Em `present_gray_areas`: selecione TODAS as áreas cinzentas automaticamente sem perguntar ao usuário
- Em `discuss_areas`: para cada pergunta de discussão, escolha a opção recomendada (primeira opção, ou a marcada como "recomendada") sem usar AskUserQuestion
- Registre cada escolha automática inline para que o usuário possa revisar as decisões no arquivo de contexto
- Após a conclusão da discussão, avance automaticamente para plan-phase (comportamento existente)

**Modo chain** — Se `--chain` estiver presente em ARGUMENTS:
- A discussão é totalmente interativa (perguntas, seleção de área cinzenta — igual ao modo padrão)
- Após a conclusão da discussão, avance automaticamente para plan-phase → execute-phase (igual ao `--auto`)
- Este é o meio-termo: o usuário controla as decisões de discussão, depois plan+execute rodam autonomamente
</step>

<step name="check_blocking_antipatterns" priority="first">
**OBRIGATÓRIO — Verifique anti-padrões bloqueantes antes de qualquer outro trabalho.**

Procure por `.continue-here.md` no diretório da fase atual:

```bash
ls ${phase_dir}/.continue-here.md 2>/dev/null || true
```

Se `.continue-here.md` existir, analise sua tabela de "Critical Anti-Patterns" para linhas com `severity` = `blocking`.

**Se um ou mais anti-padrões `blocking` forem encontrados:**

Este passo não pode ser pulado. Antes de prosseguir para `check_existing` ou qualquer outro passo, o agente deve demonstrar compreensão de cada anti-padrão bloqueante respondendo às três perguntas para cada um:

1. **O que é este anti-padrão?** — Descreva com suas próprias palavras, sem citar o handoff.
2. **Como ele se manifestou?** — Explique a falha específica que fez com que fosse registrado.
3. **Qual mecanismo estrutural (não um reconhecimento) o previne?** — Nomeie o passo concreto, item de checklist ou mecanismo de aplicação que impede a recorrência.

Escreva essas respostas inline antes de continuar. Se um anti-padrão bloqueante não puder ser respondido a partir do contexto em `.continue-here.md`, pare e peça ao usuário esclarecimentos.

**Se `.continue-here.md` não existir, ou nenhuma linha `blocking` for encontrada:** Prossiga diretamente para `check_existing`.
</step>

<step name="check_existing">
Verifique se CONTEXT.md já existe usando `has_context` do init.

```bash
ls ${phase_dir}/*-CONTEXT.md 2>/dev/null || true
```

**Se existir:**

**Se `--auto`:** Selecione automaticamente "Update it" — carregue o contexto existente e continue para analyze_phase. Registre: `[auto] Contexto existe — atualizando com decisões selecionadas automaticamente.`

**Caso contrário:** Use AskUserQuestion:
- header: "Contexto"
- question: "A Fase [X] já tem contexto. O que você quer fazer?"
- options:
  - "Update it" — Revisar e atualizar o contexto existente
  - "View it" — Me mostre o que está lá
  - "Skip" — Usar o contexto existente como está

Se "Update": Carregue o existente, continue para analyze_phase
Se "View": Exiba CONTEXT.md, depois ofereça update/skip
Se "Skip": Encerre o workflow

**Se não existir:**

**Verifique por ponto de verificação de discussão interrompida:**

```bash
ls ${phase_dir}/*-DISCUSS-CHECKPOINT.json 2>/dev/null || true
```

Se um arquivo de ponto de verificação existir (sessão anterior foi interrompida antes de CONTEXT.md ser escrito):

**Se `--auto`:** Selecione automaticamente "Resume" — carregue o ponto de verificação e continue a partir da última área concluída.

**Caso contrário:** Use AskUserQuestion:
- header: "Retomar"
- question: "Encontrado ponto de verificação de discussão interrompida ({N} áreas concluídas de {M}). Retomar de onde parou?"
- options:
  - "Resume" — Carregar ponto de verificação, pular áreas concluídas, continuar discussão
  - "Start fresh" — Excluir ponto de verificação, começar discussão do zero

Se "Resume": Analise o JSON do ponto de verificação. Carregue `decisions` no acumulador interno. Defina `areas_completed` para pular essas áreas. Continue para `present_gray_areas` apenas com as áreas restantes.
Se "Start fresh": Exclua o arquivo de ponto de verificação. Continue como se nenhum ponto de verificação existisse.

Verifique `has_plans` e `plan_count` do init. **Se `has_plans` for verdadeiro:**

**Se `--auto`:** Selecione automaticamente "Continue and replan after". Registre: `[auto] Planos existem — continuando com captura de contexto, replanejará depois.`

**Caso contrário:** Use AskUserQuestion:
- header: "Planos existem"
- question: "A Fase [X] já tem {plan_count} plano(s) criado(s) sem contexto do usuário. Suas decisões aqui não afetarão os planos existentes a menos que você replaneje."
- options:
  - "Continue and replan after" — Capturar contexto, depois executar /gsd-plan-phase {X} ${GSD_WS} para replanejar
  - "View existing plans" — Mostrar planos antes de decidir
  - "Cancel" — Pular o discuss-phase

Se "Continue and replan after": Continue para analyze_phase.
Se "View existing plans": Exiba os arquivos de plano, depois ofereça "Continue" / "Cancel".
Se "Cancel": Encerre o workflow.

**Se `has_plans` for falso:** Continue para load_prior_context.
</step>

<step name="load_prior_context">
Leia o contexto de nível de projeto e de fases anteriores para evitar repetir perguntas já decididas e manter consistência.

**Passo 1: Leia os arquivos de nível de projeto**
```bash
# Arquivos principais do projeto
cat .planning/PROJECT.md 2>/dev/null || true
cat .planning/REQUIREMENTS.md 2>/dev/null || true
cat .planning/STATE.md 2>/dev/null || true
```

Extraia destes:
- **PROJECT.md** — Visão, princípios, itens inegociáveis, preferências do usuário
- **REQUIREMENTS.md** — Critérios de aceitação, restrições, obrigatórios vs desejáveis
- **STATE.md** — Progresso atual, quaisquer flags ou notas de sessão

**Passo 2: Leia todos os arquivos CONTEXT.md anteriores**
```bash
# Encontre todos os arquivos CONTEXT.md de fases anteriores à atual
(find .planning/phases -name "*-CONTEXT.md" 2>/dev/null || true) | sort
```

Para cada CONTEXT.md onde o número da fase < fase atual:
- Leia a seção `<decisions>` — estas são preferências bloqueadas
- Leia `<specifics>` — referências particulares ou momentos "quero como X"
- Observe padrões (ex.: "usuário prefere UI mínima", "usuário rejeitou atalhos de tecla única")

**Passo 3: Construa o contexto interno `<prior_decisions>`**

Estruture as informações extraídas:
```
<prior_decisions>
## Nível de Projeto
- [Princípio ou restrição principal do PROJECT.md]
- [Requisito que afeta esta fase do REQUIREMENTS.md]

## De Fases Anteriores
### Fase N: [Nome]
- [Decisão que pode ser relevante para a fase atual]
- [Preferência que estabelece um padrão]

### Fase M: [Nome]
- [Outra decisão relevante]
</prior_decisions>
```

**Uso nos passos subsequentes:**
- `analyze_phase`: Pule áreas cinzentas já decididas em fases anteriores
- `present_gray_areas`: Anote opções com decisões anteriores ("Você escolheu X na Fase 5")
- `discuss_areas`: Pré-preencha respostas ou sinalize conflitos ("Isso contradiz a Fase 3 — igual aqui ou diferente?")

**Se não existir contexto anterior:** Continue sem — isso é esperado para fases iniciais.
</step>

<step name="cross_reference_todos">
Verifique se alguma tarefa pendente é relevante para o escopo desta fase. Traz itens de backlog que poderiam ser perdidos.

**Carregue e combine tarefas:**
```bash
TODO_MATCHES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" todo match-phase "${PHASE_NUMBER}")
```

Analise o JSON para: `todo_count`, `matches[]` (cada um com `file`, `title`, `area`, `score`, `reasons`).

**Se `todo_count` for 0 ou `matches` estiver vazio:** Pule silenciosamente — sem atrasar o workflow.

**Se houver correspondências:**

Apresente as tarefas correspondentes ao usuário. Mostre cada correspondência com seu título, área e por que correspondeu:

```
📋 Encontradas {N} tarefa(s) pendente(s) que podem ser relevantes para a Fase {X}:

{Para cada correspondência:}
- **{title}** (área: {area}, relevância: {score}) — correspondeu em {reasons}
```

Use AskUserQuestion (multiSelect) perguntando quais tarefas incluir no escopo desta fase:

```
Quais dessas tarefas devem ser incluídas no escopo da Fase {X}?
(Selecione quantas quiser, ou nenhuma para pular)
```

**Para tarefas selecionadas (incluídas):**
- Armazene internamente como `<folded_todos>` para inclusão na seção `<decisions>` do CONTEXT.md
- Estes se tornam itens de escopo adicionais que agentes downstream (pesquisador, planejador) verão

**Para tarefas não selecionadas (revisadas mas não incluídas):**
- Armazene internamente como `<reviewed_todos>` para inclusão na seção `<deferred>` do CONTEXT.md
- Isso evita que fases futuras resurfaçam as mesmas tarefas como "perdidas"

**Modo auto (`--auto`):** Inclua automaticamente todas as tarefas com score >= 0.4. Registre a seleção.
</step>

<step name="scout_codebase">
Varredura leve do código existente para informar a identificação de áreas cinzentas e a discussão. Usa ~10% do contexto — aceitável para uma sessão interativa.

**Passo 1: Verifique mapas de código-base existentes**
```bash
ls .planning/codebase/*.md 2>/dev/null || true
```

**Se mapas de código-base existirem:** Leia os mais relevantes (CONVENTIONS.md, STRUCTURE.md, STACK.md com base no tipo de fase). Extraia:
- Componentes/hooks/utilitários reutilizáveis
- Padrões estabelecidos (gerenciamento de estado, estilização, busca de dados)
- Pontos de integração (onde o novo código se conectaria)

Pule para o Passo 3 abaixo.

**Passo 2: Se não houver mapas de código-base, faça grep direcionado**

Extraia termos-chave do objetivo da fase (ex.: "feed" → "post", "card", "list"; "auth" → "login", "session", "token").

```bash
# Encontre arquivos relacionados aos termos do objetivo da fase
grep -rl "{term1}\|{term2}" src/ app/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" 2>/dev/null | head -10 || true

# Encontre componentes/hooks existentes
ls src/components/ 2>/dev/null || true
ls src/hooks/ 2>/dev/null || true
ls src/lib/ src/utils/ 2>/dev/null || true
```

Leia os 3-5 arquivos mais relevantes para entender os padrões existentes.

**Passo 3: Construa o contexto interno codebase_context**

A partir da varredura, identifique:
- **Ativos reutilizáveis** — componentes, hooks, utilitários existentes que poderiam ser usados nesta fase
- **Padrões estabelecidos** — como o código-base faz gerenciamento de estado, estilização, busca de dados
- **Pontos de integração** — onde o novo código se conectaria (rotas, nav, providers)
- **Opções criativas** — abordagens que a arquitetura existente habilita ou restringe

Armazene como `<codebase_context>` interno para uso em analyze_phase e present_gray_areas. Isso NÃO é gravado em arquivo — é usado apenas dentro desta sessão.
</step>

<step name="analyze_phase">
Analise a fase para identificar áreas cinzentas que valem ser discutidas. **Use tanto `prior_decisions` quanto `codebase_context` para fundamentar a análise.**

**Leia a descrição da fase no ROADMAP.md e determine:**

1. **Fronteira de domínio** — Qual capacidade esta fase está entregando? Declare claramente.

1b. **Inicialize o acumulador de refs canônicas** — Comece a construir a lista `<canonical_refs>` para CONTEXT.md. Isso se acumula ao longo de toda a discussão, não apenas neste passo.

   **Fonte 1 (agora):** Copie `Canonical refs:` do ROADMAP.md para esta fase. Expanda cada um para um caminho relativo completo.
   **Fonte 2 (agora):** Verifique REQUIREMENTS.md e PROJECT.md para quaisquer specs/ADRs referenciados para esta fase.
   **Fonte 3 (scout_codebase):** Se o código existente referenciar documentos (ex.: comentários citando ADRs), adicione esses.
   **Fonte 4 (discuss_areas):** Quando o usuário disser "leia X", "verifique Y", ou referenciar qualquer doc/spec/ADR durante a discussão — adicione imediatamente. Essas são frequentemente as refs MAIS importantes porque representam documentos que o usuário especificamente quer que sejam seguidos.

   Esta lista é OBRIGATÓRIA no CONTEXT.md. Cada ref deve ter um caminho relativo completo para que agentes downstream possam lê-la diretamente. Se nenhum documento externo existir, anote isso explicitamente.

2. **Verifique decisões anteriores** — Antes de gerar áreas cinzentas, verifique se alguma já foi decidida:
   - Analise `<prior_decisions>` para escolhas relevantes (ex.: "Apenas Ctrl+C, sem atalhos de tecla única")
   - Estas estão **pré-respondidas** — não pergunte novamente a menos que esta fase tenha necessidades conflitantes
   - Anote as decisões anteriores aplicáveis para uso na apresentação

3. **Áreas cinzentas por categoria** — Para cada categoria relevante (UI, UX, Comportamento, Estados Vazios, Conteúdo), identifique 1-2 ambiguidades específicas que mudariam a implementação. **Anote com contexto de código onde relevante** (ex.: "Você já tem um componente Card" ou "Nenhum padrão existente para isso").

4. **Avaliação de pulo** — Se não existirem áreas cinzentas significativas (infraestrutura pura, implementação clara, ou tudo já decidido em fases anteriores), a fase pode não precisar de discussão.

**Detecção do Modo Advisor:**

Verifique se o modo advisor deve ser ativado:

1. Verifique USER-PROFILE.md:
   ```bash
   PROFILE_PATH="$HOME/.claude/get-shit-done/USER-PROFILE.md"
   ```
   ADVISOR_MODE = arquivo existe em PROFILE_PATH → true, caso contrário → false

2. Se ADVISOR_MODE for true, resolva o nível de calibração vendor_philosophy:
   - Prioridade 1: Leia config.json > preferences.vendor_philosophy (override de nível de projeto)
   - Prioridade 2: Leia a classificação Vendor Choices/Philosophy do USER-PROFILE.md (global)
   - Prioridade 3: Padrão para "standard" se nenhum tiver valor ou valor for UNSCORED

   Mapeie para nível de calibração:
   - conservative OR thorough-evaluator → full_maturity
   - opinionated → minimal_decisive
   - pragmatic-fast OR qualquer outro valor OR vazio → standard

3. Resolva o modelo para agentes advisor:
   ```bash
   ADVISOR_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-advisor-researcher --raw)
   ```

Se ADVISOR_MODE for false, pule todos os passos específicos do advisor — o workflow prossegue com o fluxo conversacional existente sem alterações.

**Detecção de Idioma do Perfil do Usuário:**

Verifique USER-PROFILE.md para preferências de comunicação que indiquem um dono de produto não técnico:

```bash
PROFILE_CONTENT=$(cat "$HOME/.claude/get-shit-done/USER-PROFILE.md" 2>/dev/null || true)
```

Defina NON_TECHNICAL_OWNER = true se QUALQUER um dos seguintes estiver presente em USER-PROFILE.md:
- `learning_style: guided`
- A palavra `jargon` aparece em uma seção `frustration_triggers`
- `explanation_depth: practical-detailed` (sem modificador técnico)
- `explanation_depth: high-level`

NON_TECHNICAL_OWNER = false se USER-PROFILE.md não existir ou nenhum dos sinais acima estiver presente.

Quando NON_TECHNICAL_OWNER for true, reformule rótulos e descrições de áreas cinzentas em linguagem de resultado de produto antes de apresentá-las ao usuário. Preserve a mesma decisão subjacente — apenas mude o enquadramento:
- Termo técnico de implementação → resultado que o usuário vai experimentar
  - "Arquitetura de token" → "Sistema de cores: qual abordagem evita que o tema escuro pisque branco ao abrir"
  - "Estratégia de variáveis CSS" → "Cores do tema: como suas cores de marca ficam consistentes nos modos claro e escuro"
  - "Superfície de API de componente" → "Como os blocos de construção se conectam: quão acopladas essas partes devem ser"
  - "Estratégia de cache: SWR vs React Query" → "Velocidade de carregamento: as telas devem mostrar dados salvos imediatamente ou aguardar dados frescos"
- Todas as decisões permanecem as mesmas. Apenas a linguagem das perguntas se adapta.

Este reformulamento se aplica a:
1. Rótulos e descrições de áreas cinzentas em `present_gray_areas`
2. Reescritas do raciocínio de pesquisa do advisor em síntese `advisor_research`

**Produza sua análise internamente, depois apresente ao usuário.**

Exemplo de análise para a fase "Feed de Posts" (com contexto de código e contexto anterior):
```
Domínio: Exibir posts de usuários seguidos
Existente: Componente Card (src/components/ui/Card.tsx), hook useInfiniteQuery, Tailwind CSS
Decisões anteriores: "UI mínima preferida" (Fase 2), "Sem paginação — sempre scroll infinito" (Fase 4)
Áreas cinzentas:
- UI: Estilo de layout (cards vs timeline vs grid) — Componente Card existe com variantes shadow/rounded
- UI: Densidade de informação (posts completos vs préviews) — sem padrões de densidade existentes
- Comportamento: Padrão de carregamento — JÁ DECIDIDO: scroll infinito (Fase 4)
- Estado Vazio: O que exibir quando não há posts — Componente EmptyState existe em ui/
- Conteúdo: Quais metadados exibir (hora, autor, contagem de reações)
```
</step>

<step name="present_gray_areas">
Apresente a fronteira de domínio, decisões anteriores e áreas cinzentas ao usuário.

**Primeiro, declare a fronteira e quaisquer decisões anteriores que se aplicam:**
```
Fase [X]: [Nome]
Domínio: [O que esta fase entrega — da sua análise]

Vamos esclarecer COMO implementar isso.
(Novas capacidades pertencem a outras fases.)

[Se decisões anteriores se aplicarem:]
**Carregando de fases anteriores:**
- [Decisão da Fase N que se aplica aqui]
- [Decisão da Fase M que se aplica aqui]
```

**Se `--auto`:** Selecione TODAS as áreas cinzentas automaticamente. Registre: `[auto] Todas as áreas cinzentas selecionadas: [lista de nomes das áreas].` Pule o AskUserQuestion abaixo e continue diretamente para discuss_areas com todas as áreas selecionadas.

**Caso contrário, use AskUserQuestion (multiSelect: true):**
- header: "Discutir"
- question: "Quais áreas você quer discutir para [nome da fase]?"
- options: Gere 3-4 áreas cinzentas específicas da fase, cada uma com:
  - "[Área específica]" (rótulo) — concreto, não genérico
  - [1-2 perguntas que isso cobre + anotação de contexto de código] (descrição)
  - **Destaque a escolha recomendada com breve explicação do motivo**

**Anotações de decisões anteriores:** Quando uma área cinzenta já foi decidida em uma fase anterior, anote-a:
```
☐ Atalhos de saída — Como os usuários devem sair?
  (Você decidiu "Apenas Ctrl+C, sem atalhos de tecla única" na Fase 5 — revisitar ou manter?)
```

**Anotações de contexto de código:** Quando o scout encontrou código relevante existente, anote a descrição da área cinzenta:
```
☐ Estilo de layout — Cards vs lista vs timeline?
  (Você já tem um componente Card com variantes shadow/rounded. Reutilizá-lo mantém o app consistente.)
```

**Combinando ambos:** Quando tanto decisões anteriores quanto contexto de código se aplicam:
```
☐ Comportamento de carregamento — Scroll infinito ou paginação?
  (Você escolheu scroll infinito na Fase 4. Hook useInfiniteQuery já configurado.)
```

**NÃO inclua uma opção "skip" ou "você decide".** O usuário executou este comando para discutir — dê a eles escolhas reais.

**Exemplos por domínio (com contexto de código):**

Para "Feed de Posts" (funcionalidade visual):
```
☐ Estilo de layout — Cards vs lista vs timeline? (Componente Card existe com variantes)
☐ Comportamento de carregamento — Scroll infinito ou paginação? (Hook useInfiniteQuery disponível)
☐ Ordenação de conteúdo — Cronológica, algorítmica ou escolha do usuário?
☐ Metadados de post — Quais informações por post? Timestamps, reações, autor?
```

Para "CLI de backup de banco de dados" (ferramenta de linha de comando):
```
☐ Formato de saída — JSON, tabela ou texto simples? Níveis de verbosidade?
☐ Design de flags — Flags curtas, longas ou ambas? Obrigatórias vs opcionais?
☐ Relatório de progresso — Silencioso, barra de progresso ou log detalhado?
☐ Recuperação de erros — Falhar rapidamente, tentar novamente ou perguntar ao usuário?
```

Para "Organizar biblioteca de fotos" (tarefa de organização):
```
☐ Critérios de agrupamento — Por data, localização, rostos ou eventos?
☐ Tratamento de duplicatas — Manter a melhor, manter todas ou perguntar cada vez?
☐ Convenção de nomenclatura — Nomes originais, datas ou descritivos?
☐ Estrutura de pastas — Plana, aninhada por ano ou por categoria?
```

Continue para discuss_areas com as áreas selecionadas (ou advisor_research se ADVISOR_MODE for true).
</step>

<step name="advisor_research">
**Pesquisa do Advisor** (apenas quando ADVISOR_MODE for true)

Após o usuário selecionar áreas cinzentas em present_gray_areas, inicie agentes de pesquisa em paralelo.

1. Exiba status breve: "Pesquisando {N} área(s)..."

2. Para CADA área cinzenta selecionada pelo usuário, inicie um Task() em paralelo:

   Task(
     prompt="Primeiro, leia @$HOME/.claude/agents/gsd-advisor-researcher.md para seu papel e instruções.

     <gray_area>{area_name}: {area_description from gray area identification}</gray_area>
     <phase_context>{phase_goal and description from ROADMAP.md}</phase_context>
     <project_context>{project name and brief description from PROJECT.md}</project_context>
     <calibration_tier>{resolved calibration tier: full_maturity | standard | minimal_decisive}</calibration_tier>

     Pesquise esta área cinzenta e retorne uma tabela de comparação estruturada com raciocínio.
     ${AGENT_SKILLS_ADVISOR}",
     subagent_type="general-purpose",
     model="{ADVISOR_MODEL}",
     description="Pesquisa: {area_name}"
   )

   Todas as chamadas Task() são iniciadas simultaneamente — NÃO aguarde uma antes de iniciar a próxima.

3. Após TODOS os agentes retornarem, SINTETIZE os resultados antes de apresentar:
   Para o retorno de cada agente:
   a. Analise a tabela de comparação em markdown e o parágrafo de raciocínio
   b. Verifique se todas as 5 colunas estão presentes (Opção | Prós | Contras | Complexidade | Recomendação) — preencha colunas ausentes em vez de mostrar tabela quebrada
   c. Verifique se a contagem de opções corresponde ao nível de calibração:
      - full_maturity: 3-5 opções aceitáveis
      - standard: 2-4 opções aceitáveis
      - minimal_decisive: 1-2 opções aceitáveis
      Se o agente retornou muitas, remova as menos viáveis. Se poucas, aceite como está.
   d. Reescreva o parágrafo de raciocínio para incorporar o contexto do projeto e o contexto da discussão em andamento que o agente não tinha acesso
   e. Se o agente retornou apenas 1 opção, converta do formato de tabela para recomendação direta: "Abordagem padrão para {área}: {opção}. {raciocínio}"
   f. **Se NON_TECHNICAL_OWNER for true:** Após completar os passos a–e, aplique uma reescrita em linguagem simples ao parágrafo de raciocínio. Substitua termos técnicos por descrições de resultados que o usuário pode entender sem contexto técnico. Os nomes de opções na tabela também podem ser reescritos em linguagem simples se forem termos de implementação — o valor da coluna Recomendação e a estrutura da tabela permanecem intactos. Não remova detalhes; traduza-os.

4. Armazene as tabelas sintetizadas para uso em discuss_areas.

**Se ADVISOR_MODE for false:** Pule este passo completamente — prossiga diretamente de present_gray_areas para discuss_areas.
</step>

<step name="discuss_areas">
Discuta cada área selecionada com o usuário. O fluxo depende do modo advisor.

**Se ADVISOR_MODE for true:**

Fluxo de discussão com tabela primeiro — apresente tabelas de comparação baseadas em pesquisa, depois capture as escolhas do usuário.

**Para cada área selecionada:**

1. **Apresente a tabela de comparação sintetizada + parágrafo de raciocínio** (do passo advisor_research)

2. **Use AskUserQuestion:**
   - header: "{area_name}"
   - question: "Qual abordagem para {area_name}?"
   - options: Extraia da coluna Opção da tabela (AskUserQuestion adiciona "Other" automaticamente)

3. **Registre a seleção do usuário:**
   - Se o usuário escolher opções da tabela → registre como decisão bloqueada para aquela área
   - Se o usuário escolher "Other" → receba o input, reflita de volta para confirmação, registre

   **Parceiro de raciocínio (condicional):**
   Se `features.thinking_partner` estiver habilitado na configuração, verifique a resposta do usuário por sinais de trade-off
   (veja `references/thinking-partner.md` para a lista de sinais). Se trade-off detectado:

   ```
   Noto prioridades concorrentes aqui — {option_A} otimiza para {goal_A} enquanto {option_B} otimiza para {goal_B}.

   Quer que eu analise os trade-offs antes de bloquearmos isso?
   [Sim, analisar] / [Não, decisão tomada]
   ```

   Se sim: forneça análise de 3-5 pontos (o que cada um otimiza/sacrifica, alinhamento com objetivos do PROJECT.md, recomendação). Depois volte ao fluxo normal.
   Se não ou thinking_partner desabilitado: continue para a próxima área.

4. **Após registrar a escolha, Claude decide se perguntas de acompanhamento são necessárias:**
   - Se a escolha tiver ambiguidade que afetaria o planejamento downstream → faça 1-2 perguntas de acompanhamento direcionadas usando AskUserQuestion
   - Se a escolha for clara e autocontida → mova para a próxima área
   - NÃO faça as 4 perguntas padrão — a tabela já forneceu o contexto

5. **Após todas as áreas processadas:**
   - header: "Pronto"
   - question: "Isso cobre [lista de áreas]. Pronto para criar o contexto?"
   - options: "Create context" / "Revisit an area"

**Tratamento de expansão de escopo (modo advisor):**
Se o usuário mencionar algo fora do domínio da fase:
```
"[Funcionalidade] parece uma nova capacidade — pertence à sua própria fase.
Vou anotar como ideia adiada.

Voltando a [área atual]: [retorne à pergunta atual]"
```

Rastreie ideias adiadas internamente.

---

**Se ADVISOR_MODE for false:**

Para cada área selecionada, conduza um loop de discussão focado.

**Modo pesquisa-antes-de-perguntas:** Verifique se `workflow.research_before_questions` está habilitado na configuração (do contexto init ou `.planning/config.json`). Quando habilitado, antes de apresentar perguntas para cada área:
1. Faça uma busca rápida na web por melhores práticas relacionadas ao tópico da área
2. Resuma os principais resultados em 2-3 pontos
3. Apresente a pesquisa junto com a pergunta para que o usuário possa tomar uma decisão mais informada

Exemplo com pesquisa habilitada:
```
Vamos falar sobre [Estratégia de Autenticação].

📊 Pesquisa de melhores práticas:
• OAuth 2.0 + PKCE é o padrão atual para SPAs (substitui o fluxo implícito)
• Tokens de sessão com cookies httpOnly preferidos a localStorage para proteção XSS
• Considere suporte a passkey/WebAuthn — adoção está acelerando em 2025-2026

Com esse contexto: Como os usuários devem autenticar?
```

Quando desabilitado (padrão), pule a pesquisa e apresente as perguntas diretamente como antes.

**Suporte a modo texto:** Analise o `--text` opcional de `$ARGUMENTS`.
- Aceite a flag `--text` OU leia `workflow.text_mode` da configuração (do contexto init)
- Quando ativo, substitua TODAS as chamadas `AskUserQuestion` por listas numeradas em texto simples
- O usuário digita um número para selecionar, ou texto livre para "Other"
- Isso é necessário para sessões remotas do Claude Code (`/rc` mode) onde menus TUI
  não funcionam pelo aplicativo Claude

**Suporte a modo batch:** Analise o `--batch` opcional de `$ARGUMENTS`.
- Aceite `--batch`, `--batch=N`, ou `--batch N`

**Suporte a modo analyze:** Analise o `--analyze` opcional de `$ARGUMENTS`.
Quando `--analyze` estiver ativo, antes de apresentar cada pergunta (ou grupo de perguntas no modo batch), forneça uma breve **análise de trade-offs** para a decisão:
- 2-3 opções com prós/contras baseados em contexto de código e padrões comuns
- Uma abordagem recomendada com raciocínio
- Armadilhas conhecidas ou restrições de fases anteriores

Exemplo com `--analyze`:
```
**Análise de trade-offs: Estratégia de autenticação**

| Abordagem | Prós | Contras |
|-----------|------|---------|
| Cookies de sessão | Simples, httpOnly previne XSS | Requer proteção CSRF, sessões stickies |
| JWT (stateless) | Escalável, sem estado no servidor | Tamanho do token, complexidade de revogação |
| OAuth 2.0 + PKCE | Padrão da indústria para SPAs | Mais configuração, UX de fluxo de redirecionamento |

💡 Recomendado: OAuth 2.0 + PKCE — seu app tem login social nos requisitos (REQ-04) e isso se alinha com a configuração NextAuth existente em `src/lib/auth.ts`.

Como os usuários devem autenticar?
```

Isso dá ao usuário contexto para tomar decisões informadas sem prompts extras. Quando `--analyze` estiver ausente, apresente as perguntas diretamente como antes.
- Aceite `--batch`, `--batch=N`, ou `--batch N`
- Padrão para 4 perguntas por batch quando nenhum número for fornecido
- Limite tamanhos explícitos a 2-5 para que um batch seja respondível
- Se `--batch` estiver ausente, mantenha o fluxo existente de uma pergunta por vez

**Filosofia:** seja adaptativo, mas deixe o usuário escolher o ritmo.
- Modo padrão: 4 turnos de pergunta única, depois verifique se deve continuar
- Modo `--batch`: 1 turno agrupado com 2-5 perguntas numeradas, depois verifique se deve continuar

Cada resposta (ou conjunto de respostas, no modo batch) deve revelar a próxima pergunta ou próximo batch.

**Modo auto (`--auto`):** Para cada área, Claude seleciona a opção recomendada (primeira opção, ou a marcada explicitamente como "recomendada") para cada pergunta sem usar AskUserQuestion. Registre cada escolha automática:
```
[auto] [Área] — P: "[texto da pergunta]" → Selecionado: "[opção escolhida]" (padrão recomendado)
```
Após todas as áreas serem resolvidas automaticamente, pule o prompt "Explore mais áreas cinzentas" e prossiga diretamente para write_context.

**CRÍTICO — Limite de passes no modo auto:**
No modo `--auto`, o passo de discussão DEVE ser concluído em uma **única passagem**. Após escrever CONTEXT.md uma vez, você terminou — prossiga imediatamente para write_context e depois auto_advance. NÃO releia seu próprio CONTEXT.md para encontrar "lacunas", "tipos indefinidos" ou "decisões ausentes" e execute passagens adicionais. Isso cria um loop auto-alimentado onde cada passagem gera referências que a próxima trata como lacunas, consumindo tempo e recursos ilimitados.

Verifique o limite de passes da configuração:
```bash
MAX_PASSES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.max_discuss_passes 2>/dev/null || echo "3")
```

Se você já escreveu e fez commit de CONTEXT.md, o passo de discussão está completo. Siga em frente.

**Modo interativo (sem `--auto`):**

**Para cada área:**

1. **Anuncie a área:**
   ```
   Vamos falar sobre [Área].
   ```

2. **Faça perguntas usando o ritmo selecionado:**

   **Padrão (sem `--batch`): Faça 4 perguntas usando AskUserQuestion**
   - header: "[Área]" (máximo 12 caracteres — abrevie se necessário)
   - question: Decisão específica para esta área
   - options: 2-3 escolhas concretas (AskUserQuestion adiciona "Other" automaticamente), com a escolha recomendada destacada e breve explicação do motivo
   - **Anote opções com contexto de código** quando relevante:
     ```
     "Como os posts devem ser exibidos?"
     - Cards (reutiliza o componente Card existente — consistente com Mensagens)
     - Lista (mais simples, seria um novo padrão)
     - Timeline (precisa de novo componente Timeline — nenhum existe ainda)
     ```
   - Inclua "You decide" como opção quando razoável — captura o critério de Claude
   - **Context7 para escolhas de biblioteca:** Quando uma área cinzenta envolve seleção de biblioteca (ex.: "magic links" → consulte docs do next-auth) ou decisões de abordagem de API, use ferramentas `mcp__context7__*` para buscar documentação atual e informar as opções. Não use Context7 para cada pergunta — apenas quando conhecimento específico de biblioteca melhora as opções.

   **Modo batch (`--batch`): Faça 2-5 perguntas numeradas em um único turno de texto simples**
   - Agrupe perguntas intimamente relacionadas para a área atual em uma única mensagem
   - Mantenha cada pergunta concreta e respondível em uma resposta
   - Quando opções forem úteis, inclua escolhas inline curtas por pergunta em vez de um AskUserQuestion separado para cada item
   - Após o usuário responder, reflita de volta as decisões capturadas, anote itens não respondidos e faça apenas o acompanhamento mínimo necessário antes de seguir em frente
   - Preserve a adaptabilidade entre batches: use o conjunto completo de respostas para decidir o próximo batch ou se a área está suficientemente clara

3. **Após o conjunto atual de perguntas, verifique:**
   - header: "[Área]" (máximo 12 caracteres)
   - question: "Mais perguntas sobre [área], ou mover para a próxima? (Restantes: [lista de outras áreas não visitadas])"
   - options: "More questions" / "Next area"

   Ao construir o texto da pergunta, liste as áreas restantes não visitadas para que o usuário saiba o que vem pela frente. Por exemplo: "Mais perguntas sobre Layout, ou mover para a próxima? (Restantes: Comportamento de carregamento, Ordenação de conteúdo)"

   Se "More questions" → faça mais 4 perguntas únicas, ou outro batch de 2-5 perguntas quando `--batch` estiver ativo, depois verifique novamente
   Se "Next area" → prossiga para a próxima área selecionada
   Se "Other" (texto livre) → interprete a intenção: frases de continuação ("chat more", "keep going", "yes", "more") mapeiam para "More questions"; frases de avanço ("done", "move on", "next", "skip") mapeiam para "Next area". Se ambíguo, pergunte: "Continuar com mais perguntas sobre [área], ou mover para a próxima área?"

4. **Após todas as áreas inicialmente selecionadas estarem completas:**
   - Resuma o que foi capturado na discussão até agora
   - AskUserQuestion:
     - header: "Pronto"
     - question: "Discutimos [lista de áreas]. Quais áreas cinzentas ainda não estão claras?"
     - options: "Explore more gray areas" / "I'm ready for context"
   - Se "Explore more gray areas":
     - Identifique 2-4 áreas cinzentas adicionais com base no que foi aprendido
     - Retorne à lógica de present_gray_areas com estas novas áreas
     - Loop: discuta novas áreas, depois pergunte novamente
   - Se "I'm ready for context": Prossiga para write_context

**Acumulação de refs canônicas durante a discussão:**
Quando o usuário referenciar um doc, spec ou ADR durante qualquer resposta — ex.: "leia o adr-014", "verifique a especificação MCP", "conforme browse-spec.md" — imediatamente:
1. Leia o doc referenciado (ou confirme que existe)
2. Adicione ao acumulador de refs canônicas com caminho relativo completo
3. Use o que aprendeu do doc para informar perguntas subsequentes

Esses docs referenciados pelo usuário são frequentemente MAIS importantes que os refs do ROADMAP.md porque representam documentos que o usuário especificamente quer que agentes downstream sigam. Nunca os descarte.

**Design das perguntas:**
- As opções devem ser concretas, não abstratas ("Cards" não "Opção A")
- Cada resposta deve informar a próxima pergunta ou o próximo batch
- Se o usuário escolher "Other" para fornecer input livre (ex.: "deixa eu descrever", "outra coisa", ou uma resposta aberta), faça seu acompanhamento como texto simples — NÃO outro AskUserQuestion. Aguarde que eles digitem no prompt normal, depois reflita o input deles de volta e confirme antes de retomar AskUserQuestion ou o próximo batch numerado.

**Tratamento de expansão de escopo:**
Se o usuário mencionar algo fora do domínio da fase:
```
"[Funcionalidade] parece uma nova capacidade — pertence à sua própria fase.
Vou anotar como ideia adiada.

Voltando a [área atual]: [retorne à pergunta atual]"
```

Rastreie ideias adiadas internamente.

**Ponto de verificação incremental — salve após cada área ser concluída:**

Após cada área ser resolvida (usuário diz "Next area" ou área é resolvida automaticamente no modo `--auto`), escreva imediatamente um arquivo de ponto de verificação com todas as decisões capturadas até agora. Isso previne perda de dados se a sessão for interrompida durante a discussão.

**Arquivo de ponto de verificação:** `${phase_dir}/${padded_phase}-DISCUSS-CHECKPOINT.json`

Escreva após cada área:
```json
{
  "phase": "{PHASE_NUM}",
  "phase_name": "{phase_name}",
  "timestamp": "{ISO timestamp}",
  "areas_completed": ["Área 1", "Área 2"],
  "areas_remaining": ["Área 3", "Área 4"],
  "decisions": {
    "Área 1": [
      {"question": "...", "answer": "...", "options_presented": ["..."]},
      {"question": "...", "answer": "...", "options_presented": ["..."]}
    ],
    "Área 2": [
      {"question": "...", "answer": "...", "options_presented": ["..."]}
    ]
  },
  "deferred_ideas": ["..."],
  "canonical_refs": ["..."]
}
```

Este é um ponto de verificação estruturado, não o CONTEXT.md final — o passo `write_context` ainda produz a saída canônica. Mas se a sessão morrer, a próxima invocação de `/gsd-discuss-phase` pode detectar este ponto de verificação e oferecer retomar a partir dele em vez de começar do zero.

**Na retomada de sessão:** No passo `check_existing`, também verifique `*-DISCUSS-CHECKPOINT.json`. Se encontrado e nenhum CONTEXT.md existir:
- Exiba: "Encontrado ponto de verificação de discussão interrompida ({N} áreas concluídas). Retomar a partir do ponto de verificação?"
- Opções: "Resume" / "Start fresh"
- Em "Resume": Carregue o ponto de verificação, pule áreas concluídas, continue de onde parou
- Em "Start fresh": Exclua o ponto de verificação, prossiga normalmente

**Após write_context completar com sucesso:** Exclua o arquivo de ponto de verificação — o CONTEXT.md canônico agora tem todas as decisões.

**Rastreie dados do log de discussão internamente:**
Para cada pergunta feita, acumule:
- Nome da área
- Todas as opções apresentadas (rótulo + descrição)
- Qual opção o usuário selecionou (ou sua resposta em texto livre)
- Quaisquer notas de acompanhamento ou esclarecimentos que o usuário forneceu
Esses dados são usados para gerar DISCUSSION-LOG.md no passo `write_context`.
</step>

<step name="write_context">
Crie CONTEXT.md capturando as decisões tomadas.

**Também gere DISCUSSION-LOG.md** — um registro completo de auditoria do Q&A do discuss-phase.
Este arquivo é apenas para referência humana (auditorias de software, revisões de conformidade). NÃO é
consumido por agentes downstream (pesquisador, planejador, executor).

**Encontre ou crie o diretório da fase:**

Use os valores do init: `phase_dir`, `phase_slug`, `padded_phase`.

Se `phase_dir` for nulo (fase existe no roadmap mas sem diretório):
```bash
mkdir -p ".planning/phases/${padded_phase}-${phase_slug}"
```

**Localização do arquivo:** `${phase_dir}/${padded_phase}-CONTEXT.md`

**Estruture o conteúdo pelo que foi discutido:**

```markdown
# Fase [X]: [Nome] - Contexto

**Coletado em:** [data]
**Status:** Pronto para planejamento

<domain>
## Fronteira da Fase

[Declaração clara do que esta fase entrega — a âncora do escopo]

</domain>

<decisions>
## Decisões de Implementação

### [Categoria 1 que foi discutida]
- **D-01:** [Decisão ou preferência capturada]
- **D-02:** [Outra decisão se aplicável]

### [Categoria 2 que foi discutida]
- **D-03:** [Decisão ou preferência capturada]

### Critério de Claude
[Áreas onde o usuário disse "você decide" — anote que Claude tem flexibilidade aqui]

### Tarefas Incluídas
[Se alguma tarefa foi incluída no escopo a partir do passo cross_reference_todos, liste-as aqui.
Cada entrada deve incluir o título da tarefa, o problema original e como se encaixa no escopo desta fase.
Se nenhuma tarefa foi incluída: omita esta subseção completamente.]

</decisions>

<canonical_refs>
## Referências Canônicas

**Agentes downstream DEVEM ler estas antes de planejar ou implementar.**

[Seção OBRIGATÓRIA. Escreva a lista COMPLETA de refs canônicas acumuladas aqui.
Fontes: refs do ROADMAP.md + refs do REQUIREMENTS.md + docs referenciados pelo usuário durante
a discussão + quaisquer docs descobertos durante a varredura do código. Agrupe por área temática.
Cada entrada precisa de um caminho relativo completo — não apenas um nome.]

### [Área temática 1]
- `caminho/para/adr-ou-spec.md` — [O que isso decide/define que é relevante]
- `caminho/para/doc.md` §N — [Referência de seção específica]

### [Área temática 2]
- `caminho/para/feature-doc.md` — [O que este doc define]

[Se não houver specs externas: "Nenhuma spec externa — requisitos totalmente capturados nas decisões acima"]

</canonical_refs>

<code_context>
## Insights do Código Existente

### Ativos Reutilizáveis
- [Componente/hook/utilitário]: [Como poderia ser usado nesta fase]

### Padrões Estabelecidos
- [Padrão]: [Como restringe/habilita esta fase]

### Pontos de Integração
- [Onde o novo código se conecta ao sistema existente]

</code_context>

<specifics>
## Ideias Específicas

[Quaisquer referências particulares, exemplos ou momentos "quero como X" da discussão]

[Se nenhum: "Sem requisitos específicos — aberto a abordagens padrão"]

</specifics>

<deferred>
## Ideias Adiadas

[Ideias que surgiram mas pertencem a outras fases. Não as perca.]

### Tarefas Revisadas (não incluídas)
[Se alguma tarefa foi revisada em cross_reference_todos mas não incluída no escopo,
liste-as aqui para que fases futuras saibam que foram consideradas.
Cada entrada: título da tarefa + motivo pelo qual foi adiada (fora do escopo, pertence à Fase Y, etc.)
Se nenhuma tarefa revisada mas não incluída: omita esta subseção completamente.]

[Se nenhuma: "Nenhuma — a discussão ficou dentro do escopo da fase"]

</deferred>

---

*Fase: XX-nome*
*Contexto coletado em: [data]*
```

Escreva o arquivo.
</step>

<step name="confirm_creation">
Apresente o resumo e os próximos passos:

```
Criado: .planning/phases/${PADDED_PHASE}-${SLUG}/${PADDED_PHASE}-CONTEXT.md

## Decisões Capturadas

### [Categoria]
- [Decisão principal]

### [Categoria]
- [Decisão principal]

[Se houver ideias adiadas:]
## Anotado para Depois
- [Ideia adiada] — fase futura

---

## ▶ Próximo Passo

**Fase ${PHASE}: [Nome]** — [Objetivo do ROADMAP.md]

`/clear` e então:

`/gsd-plan-phase ${PHASE} ${GSD_WS}`

---

**Também disponível:**
- `/gsd-discuss-phase ${PHASE} --chain ${GSD_WS}` — reexecute com plan+execute automático depois
- `/gsd-plan-phase ${PHASE} --skip-research ${GSD_WS}` — planeje sem pesquisa
- `/gsd-ui-phase ${PHASE} ${GSD_WS}` — gere contrato de design UI antes do planejamento (se a fase tiver trabalho frontend)
- Revise/edite CONTEXT.md antes de continuar

---
```
</step>

<step name="git_commit">
**Escreva DISCUSSION-LOG.md antes do commit:**

**Localização do arquivo:** `${phase_dir}/${padded_phase}-DISCUSSION-LOG.md`

```markdown
# Fase [X]: [Nome] - Log de Discussão

> **Apenas registro de auditoria.** Não use como input para agentes de planejamento, pesquisa ou execução.
> As decisões estão capturadas em CONTEXT.md — este log preserva as alternativas consideradas.

**Data:** [data ISO]
**Fase:** [número da fase]-[nome da fase]
**Áreas discutidas:** [lista separada por vírgulas]

---

[Para cada área cinzenta discutida:]

## [Nome da Área]

| Opção | Descrição | Selecionada |
|-------|-----------|-------------|
| [Opção 1] | [Descrição do AskUserQuestion] | |
| [Opção 2] | [Descrição] | ✓ |
| [Opção 3] | [Descrição] | |

**Escolha do usuário:** [Opção selecionada ou resposta em texto livre]
**Notas:** [Quaisquer esclarecimentos, contexto de acompanhamento ou raciocínio que o usuário forneceu]

---

[Repita para cada área]

## Critério de Claude

[Liste áreas onde o usuário disse "você decide" ou delegou para Claude]

## Ideias Adiadas

[Ideias mencionadas durante a discussão que foram anotadas para fases futuras]
```

Escreva o arquivo.

**Limpe o arquivo de ponto de verificação** — CONTEXT.md é agora o registro canônico:

```bash
rm -f "${phase_dir}/${padded_phase}-DISCUSS-CHECKPOINT.json"
```

Faça commit do contexto da fase e do log de discussão:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): capture phase context" --files "${phase_dir}/${padded_phase}-CONTEXT.md" "${phase_dir}/${padded_phase}-DISCUSSION-LOG.md"
```

Confirme: "Commit realizado: docs(${padded_phase}): capture phase context"
</step>

<step name="update_state">
Atualize STATE.md com informações da sessão:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state record-session \
  --stopped-at "Phase ${PHASE} context gathered" \
  --resume-file "${phase_dir}/${padded_phase}-CONTEXT.md"
```

Faça commit do STATE.md:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(state): record phase ${PHASE} context session" --files .planning/STATE.md
```
</step>

<step name="auto_advance">
Verifique o gatilho de avanço automático:

1. Analise as flags `--auto` e `--chain` de $ARGUMENTS
2. **Sincronize a flag chain com a intenção** — se o usuário invocou manualmente (sem `--auto` e sem `--chain`), limpe a flag chain efêmera de qualquer cadeia `--auto` interrompida anteriormente. Isso NÃO toca `workflow.auto_advance` (a preferência persistente de configurações do usuário):
   ```bash
   if [[ ! "$ARGUMENTS" =~ --auto ]] && [[ ! "$ARGUMENTS" =~ --chain ]]; then
     node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active false 2>/dev/null
   fi
   ```
3. Leia tanto a flag chain quanto a preferência do usuário:
   ```bash
   AUTO_CHAIN=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow._auto_chain_active 2>/dev/null || echo "false")
   AUTO_CFG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.auto_advance 2>/dev/null || echo "false")
   ```

**Se a flag `--auto` ou `--chain` estiver presente E `AUTO_CHAIN` não for true:** Persista a flag chain na configuração (lida com uso direto sem new-project):
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set workflow._auto_chain_active true
```

**Se a flag `--auto` estiver presente OU a flag `--chain` estiver presente OU `AUTO_CHAIN` for true OU `AUTO_CFG` for true:**

Exiba o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► AVANÇANDO AUTOMATICAMENTE PARA PLANEJAMENTO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Contexto capturado. Iniciando plan-phase...
```

Inicie o plan-phase usando a ferramenta Skill para evitar sessões Task aninhadas (que causam travamentos de runtime devido ao aninhamento profundo de agentes — veja #686):
```
Skill(skill="gsd-plan-phase", args="${PHASE} --auto ${GSD_WS}")
```

Isso mantém a cadeia de avanço automático plana — discuss, plan e execute rodam no mesmo nível de aninhamento em vez de gerar agentes Task cada vez mais profundos.

**Trate o retorno do plan-phase:**
- **PHASE COMPLETE** → Cadeia completa com sucesso. Exiba:
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   GSD ► FASE ${PHASE} COMPLETA
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Pipeline de avanço automático concluído: discuss → plan → execute

  /clear e então:

  Próximo: /gsd-discuss-phase ${NEXT_PHASE} ${WAS_CHAIN ? "--chain" : "--auto"} ${GSD_WS}
  ```
- **PLANNING COMPLETE** → Planejamento concluído, execução não completou:
  ```
  Avanço automático parcial: Planejamento completo, execução não terminou.
  Continue: /gsd-execute-phase ${PHASE} ${GSD_WS}
  ```
- **PLANNING INCONCLUSIVE / CHECKPOINT** → Pare a cadeia:
  ```
  Avanço automático parado: Planejamento precisa de input.
  Continue: /gsd-plan-phase ${PHASE} ${GSD_WS}
  ```
- **GAPS FOUND** → Pare a cadeia:
  ```
  Avanço automático parado: Lacunas encontradas durante a execução.
  Continue: /gsd-plan-phase ${PHASE} --gaps ${GSD_WS}
  ```

**Se nenhum de `--auto`, `--chain`, nem configuração estiver habilitado:**
Direcione para o passo `confirm_creation` (comportamento manual existente — mostre os próximos passos manuais).
</step>

</process>

<power_user_mode>
Quando a flag `--power` estiver presente em ARGUMENTS, pule o questionamento interativo e execute o workflow do usuário avançado.

O modo usuário avançado gera TODAS as perguntas antecipadamente em arquivos legíveis por máquina e amigáveis ao ser humano, depois aguarda o usuário responder no seu próprio ritmo antes de processar todas as respostas em uma única passagem.

**Instruções passo a passo completas:** @$HOME/.claude/get-shit-done/workflows/discuss-phase-power.md

**Resumo do fluxo:**
1. Execute a mesma análise de fase (identificação de áreas cinzentas) que o modo padrão
2. Escreva todas as perguntas em `{phase_dir}/{padded_phase}-QUESTIONS.json` e `{phase_dir}/{padded_phase}-QUESTIONS.html`
3. Notifique o usuário com os caminhos dos arquivos e aguarde um comando "refresh" ou "finalize"
4. Em "refresh": leia o JSON, processe perguntas respondidas, atualize estatísticas e HTML
5. Em "finalize": leia todas as respostas do JSON, gere CONTEXT.md no formato padrão
</power_user_mode>

<success_criteria>
- Fase validada contra o roadmap
- Contexto anterior carregado (PROJECT.md, REQUIREMENTS.md, STATE.md, arquivos CONTEXT.md anteriores)
- Perguntas já decididas não são repetidas (carregadas de fases anteriores)
- Código-base verificado para ativos reutilizáveis, padrões e pontos de integração
- Áreas cinzentas identificadas por análise inteligente com anotações de código e decisões anteriores
- Usuário selecionou quais áreas discutir
- Cada área selecionada explorada até o usuário estar satisfeito (com opções informadas por código e decisões anteriores)
- Expansão de escopo redirecionada para ideias adiadas
- CONTEXT.md captura decisões reais, não visão vaga
- CONTEXT.md inclui seção canonical_refs com caminhos completos de arquivos para cada spec/ADR/doc que agentes downstream precisam (OBRIGATÓRIO — nunca omita)
- CONTEXT.md inclui seção code_context com ativos reutilizáveis e padrões
- Ideias adiadas preservadas para fases futuras
- STATE.md atualizado com informações da sessão
- Usuário sabe os próximos passos
- Arquivo de ponto de verificação escrito após cada área concluída (salvamento incremental)
- Sessões interrompidas podem ser retomadas a partir do ponto de verificação (sem responder novamente áreas concluídas)
- Arquivo de ponto de verificação limpo após escrita bem-sucedida do CONTEXT.md
- `--chain` aciona discussão interativa seguida de plan+execute automático (sem respostas automáticas)
- `--chain` e `--auto` ambos persistem a flag chain e avançam automaticamente para plan-phase
</success_criteria>
