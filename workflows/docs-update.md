<purpose>
Gerar, atualizar e verificar toda a documentação do projeto — tanto os tipos canônicos quanto documentos existentes escritos à mão. O orquestrador detecta a estrutura de documentação do projeto, monta um manifesto de trabalho rastreando cada item, despacha agentes paralelos de escrita e verificação de documentação em ondas, revisa documentos existentes quanto à precisão, identifica lacunas de documentação e corrige imprecisões via um loop limitado de correções. Todo o estado é persistido em um manifesto de trabalho para que nenhum item de trabalho seja perdido entre os passos. Saída: Documentação completa, ciente da estrutura, verificada contra o código-base ativo.
</purpose>

<available_agent_types>
Tipos válidos de subagentes GSD (use os nomes exatos — não recorra a 'general-purpose'):
- gsd-doc-writer — Escreve e atualiza arquivos de documentação do projeto
- gsd-doc-verifier — Verifica afirmações factuais em documentos contra o código-base ativo
</available_agent_types>

<process>

<step name="init_context" priority="first">
Carregue o contexto de docs-update:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" docs-init)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-doc-writer 2>/dev/null)
```

Extraia do JSON init:
- `doc_writer_model` — string do modelo para passar a cada agente gerado (nunca codifique um nome de modelo)
- `commit_docs` — se deve fazer commit dos arquivos gerados ao terminar
- `existing_docs` — array de objetos `{path, has_gsd_marker}` para arquivos Markdown existentes
- `project_type` — objeto com sinais booleanos: `has_package_json`, `has_api_routes`, `has_cli_bin`, `is_open_source`, `has_deploy_config`, `is_monorepo`, `has_tests`
- `doc_tooling` — objeto com booleanos: `docusaurus`, `vitepress`, `mkdocs`, `storybook`
- `monorepo_workspaces` — array de padrões glob de workspace (vazio se não for monorepo)
- `project_root` — caminho absoluto para a raiz do projeto
</step>

<step name="classify_project">
Mapeie os sinais booleanos `project_type` do JSON init para um rótulo de tipo primário e colete sinais condicionais de documentação.

**Classificação de tipo primário (a primeira correspondência vence):**

| Condição | primary_type |
|----------|-------------|
| `is_monorepo` é true | `"monorepo"` |
| `has_cli_bin` é true E `has_api_routes` é false | `"cli-tool"` |
| `has_api_routes` é true E `is_open_source` é false | `"saas"` |
| `is_open_source` é true E `has_api_routes` é false | `"open-source-library"` |
| (nenhum dos acima) | `"generic"` |

**Sinais condicionais de documentação (regra D-02 de união — verifique independentemente após a classificação primária):**

Após determinar o primary_type, verifique cada sinal independentemente independentemente do tipo primário. Uma ferramenta CLI que também é open source com rotas de API ainda recebe os três documentos condicionais.

| Sinal | Doc Condicional |
|-------|----------------|
| `has_api_routes` é true | Enfileire API.md |
| `is_open_source` é true | Enfileire CONTRIBUTING.md |
| `has_deploy_config` é true | Enfileire DEPLOYMENT.md |

Apresente o resultado da classificação:
```
Tipo do projeto: {primary_type}
Documentos condicionais enfileirados: {lista ou "nenhum"}
```
</step>

<step name="build_doc_queue">
Monte a fila de documentação completa a partir dos documentos sempre ativos mais os condicionais de classify_project.

**Documentos sempre ativos (enfileirados para cada projeto, sem exceções):**
1. README
2. ARCHITECTURE
3. GETTING-STARTED
4. DEVELOPMENT
5. TESTING
6. CONFIGURATION

**Documentos condicionais (adicione apenas se o sinal corresponder em classify_project):**
- API (se `has_api_routes`)
- CONTRIBUTING (se `is_open_source`)
- DEPLOYMENT (se `has_deploy_config`)

**IMPORTANTE: CHANGELOG.md NUNCA é enfileirado. A fila de documentação é construída exclusivamente dos 9 tipos de documentos conhecidos listados acima. Não derive a fila de `existing_docs` diretamente — existing_docs é usado apenas no próximo passo para determinar o modo criar vs atualizar.**

**Limite da fila de documentação:** Máximo de 9 documentos. Sempre ativos (6) + até 3 condicionais = no máximo 9.

**Confirmação CONTRIBUTING.md (somente para novo arquivo):**

Se CONTRIBUTING.md estiver na fila condicional E NÃO aparecer no array `existing_docs` do JSON init:

1. Se `--force` estiver presente em `$ARGUMENTS`: pule esta verificação, inclua CONTRIBUTING.md na fila.

**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON init for `true`. Quando TEXT_MODE estiver ativo, substitua toda chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
2. Caso contrário, use AskUserQuestion para confirmar:

```
AskUserQuestion([{
  question: "Este projeto parece ser open source (arquivo LICENSE detectado). CONTRIBUTING.md ainda não existe. Gostaria de criar um?",
  header: "Contribuição",
  multiSelect: false,
  options: [
    { label: "Sim, criar", description: "Gerar CONTRIBUTING.md com as diretrizes do projeto" },
    { label: "Não, pular", description: "Este projeto não precisa de um CONTRIBUTING.md" }
  ]
}])
```

Se o usuário selecionar "Não, pular": remova CONTRIBUTING.md da fila de documentação.
Se CONTRIBUTING.md já existir em `existing_docs`: pule este prompt completamente, inclua-o para atualização.

**Documentos não canônicos existentes (fila de revisão):**

Após montar a fila de documentos canônicos acima, analise o array `existing_docs` do JSON init em busca de arquivos que NÃO correspondam a nenhum caminho canônico na fila (nem caminho primário nem de fallback da tabela resolve_modes). Estes são documentos escritos à mão como `docs/api/endpoint-map.md` ou `docs/frontend/pages/not-found.md`.

Para cada documento não canônico existente encontrado:
- Adicione a uma `review_queue` separada
- Estes serão passados ao gsd-doc-verifier no passo verify_docs para verificação de precisão
- Se imprecisões forem encontradas, serão despachadas ao gsd-doc-writer no modo `fix` para correções cirúrgicas

Se documentos não canônicos forem encontrados, exiba-os na apresentação da fila:

```
Documentos existentes enfileirados para revisão de precisão:
  - docs/api/endpoint-map.md (escrito à mão)
  - docs/api/README.md (escrito à mão)
  - docs/frontend/pages/not-found.md (escrito à mão)
```

Se nenhum for encontrado, omita esta seção da apresentação da fila.

**Detecção de lacunas de documentação (documentos não canônicos ausentes):**

Após montar as filas canônica e de revisão, analise o código-base para identificar áreas que deveriam ter documentação mas não têm. Isso garante que o comando crie documentação completa do projeto, não apenas os 9 tipos canônicos.

1. **Analise o código-base em busca de áreas não documentadas:**
   - Use Glob/Grep para descobrir diretórios de código-fonte significativos (ex.: `src/components/`, `src/pages/`, `src/services/`, `src/api/`, `lib/`, `routes/`)
   - Compare com os documentos existentes: para cada diretório fonte principal, verifique se a documentação correspondente existe na árvore de docs
   - Observe a estrutura de documentação existente do projeto em busca de padrões — se o projeto tiver `docs/frontend/components/`, `docs/services/`, etc., isso indica as convenções de documentação do projeto

2. **Identifique lacunas com base nas convenções do projeto:**
   - Se o projeto tiver um diretório `docs/` com subdiretórios agrupados, cada área do módulo de código-fonte que tenha um subdiretório de docs correspondente mas esteja sem arquivos de documentação representa uma lacuna
   - Se o projeto tiver componentes/páginas frontend mas sem documentação de componentes, sinalize isso
   - Se o projeto tiver módulos de serviço mas sem documentação de serviços, sinalize isso
   - Pule áreas já cobertas por documentos canônicos (ex.: não sinalize falta de docs de API se `docs/API.md` já está na fila canônica)

3. **Apresente lacunas descobertas ao usuário:**

```
AskUserQuestion([{
  question: "Encontradas {N} lacunas de documentação no código-base. Quais devem ser criadas?",
  header: "Lacunas de docs",
  multiSelect: true,
  options: [
    { label: "{área}", description: "{por que precisa de docs — ex.: '5 componentes em src/components/ sem docs'}" },
    ...até 4 opções (agrupe lacunas relacionadas se houver mais de 4)
  ]
}])
```

4. Para cada lacuna selecionada pelo usuário:
   - Adicione à fila de geração com mode = `"create"`
   - Defina o caminho de saída para corresponder à estrutura de diretório de docs existente do projeto
   - O gsd-doc-writer receberá um `doc_assignment` com `type: "custom"` e uma descrição do que documentar, usando os arquivos de código-fonte do projeto como alvos de descoberta de conteúdo

Se nenhuma lacuna for detectada, omita esta seção completamente.

Apresente a fila montada ao usuário antes de prosseguir:

Apresente a tabela de resolução de modos de resolve_modes (mostrada acima), seguida de:

```
{Se documentos não canônicos forem encontrados, mostre como tabela:}

Documentos existentes enfileirados para revisão de precisão:

| Caminho | Tipo |
|---------|------|
| {caminho} | escrito à mão |
| ... | ... |

CHANGELOG.md: excluído (fora do escopo)
```

A tabela de resolução de modos É a apresentação da fila — ela mostra cada documento com seu caminho resolvido, modo e fonte. Não duplique a lista em outro formato.

Então confirme com AskUserQuestion:

```
AskUserQuestion([{
  question: "Fila de documentação montada ({N} documentos). Prosseguir com a geração?",
  header: "Fila de docs",
  multiSelect: false,
  options: [
    { label: "Prosseguir", description: "Gerar todos os {N} documentos na fila" },
    { label: "Cancelar", description: "Cancelar a geração de documentação" }
  ]
}])
```

Se o usuário selecionar "Cancelar": encerre o workflow. Caso contrário, continue para resolve_modes.
</step>

<step name="resolve_modes">
Para cada documento na fila montada, determine se deve criar (novo arquivo) ou atualizar (arquivo existente).

**Mapeamento de tipo de documento para caminho canônico (padrões):**

| Tipo | Caminho Padrão | Caminho de Fallback |
|------|---------------|---------------------|
| `readme` | `README.md` | — |
| `architecture` | `docs/ARCHITECTURE.md` | `ARCHITECTURE.md` |
| `getting_started` | `docs/GETTING-STARTED.md` | `GETTING-STARTED.md` |
| `development` | `docs/DEVELOPMENT.md` | `DEVELOPMENT.md` |
| `testing` | `docs/TESTING.md` | `TESTING.md` |
| `api` | `docs/API.md` | `API.md` |
| `configuration` | `docs/CONFIGURATION.md` | `CONFIGURATION.md` |
| `deployment` | `docs/DEPLOYMENT.md` | `DEPLOYMENT.md` |
| `contributing` | `CONTRIBUTING.md` | — |

**Resolução de caminho com consciência de estrutura:**

Antes de aplicar a tabela de caminho padrão, inspecione a estrutura de diretório de docs existente do projeto para detectar se o projeto usa **subdiretórios agrupados** ou **arquivos planos**. Isso determina como TODOS os novos documentos são colocados.

**Passo 1: Detecte o padrão de organização de docs do projeto.**

Liste subdiretórios em `docs/` a partir dos caminhos `existing_docs`. Se o projeto tiver 2+ subdiretórios (ex.: `docs/architecture/`, `docs/api/`, `docs/guides/`, `docs/frontend/`), o projeto usa uma **estrutura agrupada**. Se os docs forem apenas arquivos planos diretamente em `docs/` (ex.: `docs/ARCHITECTURE.md`), usa uma **estrutura plana**.

**Passo 2: Resolva caminhos com base no padrão detectado.**

**Se estrutura AGRUPADA detectada:**

Cada tipo de documento DEVE ser colocado em um subdiretório apropriado — nenhum documento deve ficar plano em `docs/` quando o projeto organiza em grupos. Use a seguinte lógica de resolução:

| Tipo | Resolução de subdiretório (em ordem de prioridade) |
|------|----------------------------------------------------|
| `architecture` | `docs/architecture/` existente → criar `docs/architecture/` se ausente |
| `getting_started` | `docs/guides/` existente → `docs/getting-started/` existente → criar `docs/guides/` |
| `development` | `docs/guides/` existente → `docs/development/` existente → criar `docs/guides/` |
| `testing` | `docs/testing/` existente → `docs/guides/` existente → criar `docs/testing/` |
| `api` | `docs/api/` existente → criar `docs/api/` se ausente |
| `configuration` | `docs/configuration/` existente → `docs/guides/` existente → criar `docs/configuration/` |
| `deployment` | `docs/deployment/` existente → `docs/guides/` existente → criar `docs/deployment/` |

Para cada tipo, verifique a cadeia de resolução da esquerda para a direita. Use o primeiro subdiretório existente. Se nenhum existir, crie a opção mais à direita.

O nome do arquivo dentro do subdiretório deve ser contextual — ex.: `docs/guides/getting-started.md`, `docs/architecture/overview.md`, `docs/api/reference.md` — em vez de `docs/architecture/ARCHITECTURE.md`. Corresponda o estilo de nomenclatura dos arquivos existentes naquele subdiretório (lowercase-kebab, UPPERCASE, etc.).

**Se estrutura PLANA detectada (ou sem diretório docs/):**

Use a tabela de caminho padrão acima como está (ex.: `docs/ARCHITECTURE.md`, `docs/TESTING.md`).

**Passo 3: Armazene cada caminho resolvido e crie diretórios.**

Para cada tipo de documento, armazene o caminho resolvido como `resolved_path`. Então crie todos os diretórios necessários:
```bash
mkdir -p {cada diretório único dos caminhos resolvidos}
```

**Lógica de resolução de modo:**

Para cada tipo de documento na fila:
1. Verifique se o `resolved_path` aparece no array `existing_docs` do JSON init
2. Se não encontrado no caminho resolvido, verifique os caminhos padrão e de fallback da tabela
3. Se encontrado em qualquer caminho: mode = `"update"` — use a ferramenta Read para carregar o conteúdo atual do arquivo (será passado como `existing_content` no bloco doc_assignment). Use o caminho encontrado como caminho de saída (não mova documentos existentes).
4. Se não encontrado: mode = `"create"` — sem conteúdo existente para carregar. Use o `resolved_path`.

**Garanta que o diretório docs/ exista:**
Antes de prosseguir para o próximo passo, crie o diretório `docs/` e quaisquer subdiretórios resolvidos se não existirem:
```bash
mkdir -p docs/
```

**Apresente uma tabela de resolução de modos:**

Apresente uma tabela mostrando o caminho resolvido, modo e fonte para cada documento na fila:

```
Resolução de modos:

| Documento | Caminho Resolvido | Modo | Fonte |
|-----------|-------------------|------|-------|
| readme | README.md | update | encontrado em README.md |
| architecture | docs/architecture/overview.md | create | novo diretório |
| getting_started | docs/guides/getting-started.md | update | encontrado, escrito à mão |
| development | docs/guides/development.md | create | correspondeu docs/guides/ |
| testing | docs/guides/testing.md | create | correspondeu docs/guides/ |
| configuration | docs/guides/configuration.md | create | correspondeu docs/guides/ |
| api | docs/api/reference.md | create | novo diretório |
| deployment | docs/guides/deployment.md | update | encontrado, escrito à mão |
```

Esta tabela DEVE ser mostrada ao usuário — é a confirmação principal de onde os arquivos serão escritos e se arquivos existentes serão atualizados. Aparece como parte da apresentação da fila ANTES da confirmação do AskUserQuestion.

Rastreie o modo resolvido e o caminho de arquivo para cada documento enfileirado. Para documentos em modo update, armazene o conteúdo do arquivo carregado — será passado ao agente nos próximos passos.

**CRÍTICO: Persista o manifesto de trabalho.**

Após resolve_modes completar, escreva TODOS os itens de trabalho em `.planning/tmp/docs-work-manifest.json`. Esta é a única fonte de verdade para cada passo subsequente — o orquestrador DEVE ler este arquivo em cada passo em vez de depender da memória.

```bash
mkdir -p .planning/tmp
```

Escreva o manifesto usando a ferramenta Write:

```json
{
  "canonical_queue": [
    {
      "type": "readme",
      "resolved_path": "README.md",
      "mode": "create|update|supplement",
      "preservation_mode": null,
      "wave": 1,
      "status": "pending"
    }
  ],
  "review_queue": [
    {
      "path": "docs/frontend/components/button.md",
      "type": "hand-written",
      "status": "pending_review"
    }
  ],
  "gap_queue": [
    {
      "description": "Componentes frontend em src/components/",
      "output_path": "docs/frontend/components/overview.md",
      "status": "pending"
    }
  ],
  "created_at": "{ISO timestamp}"
}
```

Cada passo subsequente (dispatch, collect, verify, fix_loop, report) DEVE começar lendo `.planning/tmp/docs-work-manifest.json` e atualizar o campo `status` para os itens que processa. Isso evita que o orquestrador "esqueça" qualquer item de trabalho ao longo do workflow multi-passo.
</step>

<step name="preservation_check">
Verifique os documentos escritos à mão na fila e colete as decisões do usuário antes do despacho.

**Condições de pulo (verifique em ordem):**

1. Se `--force` estiver presente em `$ARGUMENTS`: trate todos os documentos como mode: regenerate, pule para detect_runtime_capabilities.
2. Se `--verify-only` estiver presente em `$ARGUMENTS`: pule para verify_only_report (não continue para detect_runtime_capabilities).
3. Se nenhum documento na fila tiver `has_gsd_marker: false` no array `existing_docs`: pule para detect_runtime_capabilities.

**Para cada documento enfileirado onde `has_gsd_marker` for false (documento escrito à mão detectado):**

Apresente a seguinte escolha usando `AskUserQuestion` se disponível, ou prompt inline caso contrário:

```
{filename} parece ter sido escrito à mão (nenhum marcador GSD encontrado).

Como este arquivo deve ser tratado?
  [1] preserve    -- Pular completamente. Deixar sem alterações.
  [2] supplement  -- Adicionar apenas seções ausentes. Conteúdo existente intocado.
  [3] regenerate  -- Sobrescrever com um documento GSD gerado do zero.
```

Registre cada decisão. Atualize a fila de documentos:
- Decisões `preserve`: remova o documento da fila completamente
- Decisões `supplement`: defina mode como `supplement` no bloco doc_assignment; inclua `existing_content` (conteúdo completo do arquivo)
- Decisões `regenerate`: defina mode como `create` (trate como escrita do zero)

**Fallback quando AskUserQuestion estiver indisponível:** Padrão todos os documentos escritos à mão para `preserve` (padrão mais seguro). Exiba a mensagem:

```
AskUserQuestion indisponível — documentos escritos à mão preservados por padrão.
Use --force para regenerar todos os documentos, ou re-execute no Claude Code para obter prompts por arquivo.
```

Após todas as decisões registradas, continue para detect_runtime_capabilities.
</step>

<!-- Se a ferramenta Task não estiver disponível em runtime, pule as ondas de dispatch/collect e use sequential_generation em vez disso. -->

<step name="dispatch_wave_1" condition="Ferramenta Task disponível">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — use itens `canonical_queue` com `wave: 1` para este passo.

Inicie 3 agentes gsd-doc-writer em paralelo para documentos da Onda 1: README, ARCHITECTURE, CONFIGURATION.

Estes são documentos fundamentais sem referências cruzadas necessárias, tornando-os ideais para geração em paralelo.

Use `run_in_background=true` para todos os três para habilitar execução em paralelo.

**Agente 1: README**

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar README.md para o projeto alvo",
  prompt="<doc_assignment>
type: readme
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente 2: ARCHITECTURE**

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar ARCHITECTURE.md para o projeto alvo",
  prompt="<doc_assignment>
type: architecture
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente 3: CONFIGURATION**

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar CONFIGURATION.md para o projeto alvo",
  prompt="<doc_assignment>
type: configuration
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
note: Aplique marcadores VERIFY a qualquer afirmação de infraestrutura não descobrível a partir do repositório.
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**CRÍTICO:** Os prompts dos agentes devem conter APENAS o bloco `<doc_assignment>`, a variável `${AGENT_SKILLS}` e a instrução de retorno. Não inclua contexto de planejamento do projeto, prosa de workflow ou quaisquer referências a ferramentas internas nos prompts dos agentes.

Continue para collect_wave_1.
</step>

<step name="collect_wave_1">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — atualize `status` para `"completed"` ou `"failed"` para cada item da Onda 1 após a coleta. Escreva o manifesto atualizado de volta ao disco.

Aguarde todos os 3 agentes da Onda 1 completarem usando a ferramenta TaskOutput.

Chame TaskOutput para todos os 3 agentes em paralelo (única mensagem com 3 chamadas TaskOutput):

```
Ferramenta TaskOutput:
  task_id: "{task_id do resultado do agente README}"
  block: true
  timeout: 300000

Ferramenta TaskOutput:
  task_id: "{task_id do resultado do agente ARCHITECTURE}"
  block: true
  timeout: 300000

Ferramenta TaskOutput:
  task_id: "{task_id do resultado do agente CONFIGURATION}"
  block: true
  timeout: 300000
```

**Formato de confirmação esperado de cada agente:**
```
## Geração de Documentação Completa
**Tipo:** {type}
**Modo:** {mode}
**Arquivo escrito:** `{path}` ({N} linhas)
Pronto para resumo do orquestrador.
```

**Após a coleta, verifique se os arquivos da Onda 1 existem no disco** usando o `resolved_path` de cada entrada do manifesto:
```bash
ls -la {resolved_path_1} {resolved_path_2} {resolved_path_3} 2>/dev/null
```

Se algum agente falhar ou seu arquivo estiver ausente:
- Anote a falha
- Continue com os documentos bem-sucedidos (NÃO pause a Onda 2 por uma única falha)
- O documento ausente será anotado no relatório final

Continue para dispatch_wave_2.
</step>

<step name="dispatch_wave_2" condition="Ferramenta Task disponível">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — use itens `canonical_queue` com `wave: 2` para este passo.

Inicie agentes para todos os documentos da Onda 2 enfileirados: GETTING-STARTED, DEVELOPMENT, TESTING, e quaisquer documentos condicionais (API, DEPLOYMENT, CONTRIBUTING) que foram enfileirados em build_doc_queue.

Os agentes da Onda 2 podem referenciar saídas da Onda 1 para referências cruzadas — inclua o campo `wave_1_outputs` em cada bloco doc_assignment.

Use `run_in_background=true` para todos os agentes da Onda 2 para habilitar execução em paralelo dentro da onda.

**Agente: GETTING-STARTED**

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar GETTING-STARTED.md para o projeto alvo",
  prompt="<doc_assignment>
type: getting_started
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
wave_1_outputs:
  - README.md
  - docs/ARCHITECTURE.md
  - docs/CONFIGURATION.md
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente: DEVELOPMENT**

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar DEVELOPMENT.md para o projeto alvo",
  prompt="<doc_assignment>
type: development
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
wave_1_outputs:
  - README.md
  - docs/ARCHITECTURE.md
  - docs/CONFIGURATION.md
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente: TESTING**

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar TESTING.md para o projeto alvo",
  prompt="<doc_assignment>
type: testing
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
wave_1_outputs:
  - README.md
  - docs/ARCHITECTURE.md
  - docs/CONFIGURATION.md
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente Condicional: API** (apenas se `has_api_routes` for true — inicie apenas se API.md foi enfileirado)

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar API.md para o projeto alvo",
  prompt="<doc_assignment>
type: api
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
wave_1_outputs:
  - README.md
  - docs/ARCHITECTURE.md
  - docs/CONFIGURATION.md
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente Condicional: DEPLOYMENT** (apenas se `has_deploy_config` for true — inicie apenas se DEPLOYMENT.md foi enfileirado)

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar DEPLOYMENT.md para o projeto alvo",
  prompt="<doc_assignment>
type: deployment
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
note: Aplique marcadores VERIFY a qualquer afirmação de infraestrutura não descobrível a partir do repositório.
wave_1_outputs:
  - README.md
  - docs/ARCHITECTURE.md
  - docs/CONFIGURATION.md
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**Agente Condicional: CONTRIBUTING** (apenas se `is_open_source` for true — inicie apenas se CONTRIBUTING.md foi enfileirado)

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar CONTRIBUTING.md para o projeto alvo",
  prompt="<doc_assignment>
type: contributing
mode: {create|update|supplement}
preservation_mode: {preserve|supplement|regenerate|null}
project_context: {INIT JSON}
{existing_content: | (inclua o conteúdo completo do arquivo aqui se mode for update ou supplement, senão omita esta linha)}
wave_1_outputs:
  - README.md
  - docs/ARCHITECTURE.md
  - docs/CONFIGURATION.md
</doc_assignment>

{AGENT_SKILLS}

Escreva o arquivo doc diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

**CRÍTICO:** Os prompts dos agentes devem conter APENAS o bloco `<doc_assignment>`, a variável `${AGENT_SKILLS}` e a instrução de retorno. Não inclua contexto de planejamento do projeto, prosa de workflow ou quaisquer referências a ferramentas internas nos prompts dos agentes.

Continue para collect_wave_2.
</step>

<step name="collect_wave_2">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — atualize `status` para `"completed"` ou `"failed"` para cada item da Onda 2 após a coleta. Escreva o manifesto atualizado de volta ao disco.

Aguarde todos os agentes da Onda 2 completarem usando a ferramenta TaskOutput.

Chame TaskOutput para todos os agentes da Onda 2 em paralelo (única mensagem com N chamadas TaskOutput — uma por agente da Onda 2 iniciado):

```
Ferramenta TaskOutput:
  task_id: "{task_id do resultado do agente GETTING-STARTED}"
  block: true
  timeout: 300000

Ferramenta TaskOutput:
  task_id: "{task_id do resultado do agente DEVELOPMENT}"
  block: true
  timeout: 300000

Ferramenta TaskOutput:
  task_id: "{task_id do resultado do agente TESTING}"
  block: true
  timeout: 300000

# Adicione uma chamada TaskOutput por agente condicional iniciado (API, DEPLOYMENT, CONTRIBUTING)
```

**Após a coleta, verifique se todos os arquivos da Onda 2 existem no disco** usando o `resolved_path` de cada entrada do manifesto:
```bash
ls -la {resolved_path para cada item da onda 2} 2>/dev/null
```

Se algum agente falhar ou seu arquivo estiver ausente, anote a falha e continue. Documentos ausentes serão relatados no relatório final.

Continue para dispatch_monorepo_packages (se monorepo_workspaces não estiver vazio) ou commit_docs.
</step>

<step name="dispatch_monorepo_packages" condition="monorepo_workspaces não está vazio">
Após a coleta da Onda 2, gere READMEs por pacote para cada workspace do monorepo.

**Condição:** Execute este passo apenas se `monorepo_workspaces` do JSON init não estiver vazio.

**Resolva pacotes de workspace a partir de padrões glob:**

```bash
# Expanda globs de workspace para diretórios de pacotes reais
for pattern in {monorepo_workspaces}; do
  ls -d $pattern 2>/dev/null
done
```

**Para cada diretório resolvido que contém um `package.json`:**

Determine o modo:
- Se `{package_dir}/README.md` existir: mode = `update`, leia o conteúdo existente
- Caso contrário: mode = `create`

Inicie um agente `gsd-doc-writer` com `run_in_background=true`:

```
Task(
  subagent_type="gsd-doc-writer",
  model="{doc_writer_model}",
  run_in_background=true,
  description="Gerar README por pacote para {package_dir}",
  prompt="<doc_assignment>
type: readme
mode: {create|update}
scope: per_package
package_dir: {caminho absoluto para o diretório do pacote}
project_context: {INIT JSON com project_root definido para o diretório do pacote}
{existing_content: | (inclua o conteúdo completo do README.md aqui se mode for update, senão omita)}
</doc_assignment>

{AGENT_SKILLS}

Escreva {package_dir}/README.md diretamente. Retorne apenas confirmação — não retorne o conteúdo do doc."
)
```

Colete confirmações via TaskOutput para todos os agentes de pacote. Anote falhas no relatório final.

**Fallback quando a ferramenta Task estiver indisponível:** Gere READMEs por pacote sequencialmente inline após o passo `sequential_generation`. Para cada diretório de pacote com um `package.json`, construa o bloco `doc_assignment` equivalente e gere o README seguindo as instruções do gsd-doc-writer.

Continue para commit_docs.
</step>

<step name="sequential_generation" condition="Ferramenta Task NÃO disponível (ex.: Antigravity, Gemini CLI, Codex, Copilot)">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — use itens `canonical_queue` para a ordem de geração. Atualize `status` após cada documento ser gerado. Escreva o manifesto atualizado de volta ao disco após todos os documentos estarem completos.

Quando a ferramenta `Task` estiver indisponível, gere documentos sequencialmente no contexto atual. Este passo substitui dispatch_wave_1, collect_wave_1, dispatch_wave_2 e collect_wave_2.

**IMPORTANTE:** NÃO use `browser_subagent`, `Explore` ou qualquer ferramenta baseada em navegador. Use apenas ferramentas de sistema de arquivos (Read, Bash, Write, Grep, Glob ou ferramentas equivalentes disponíveis no seu runtime).

Leia as instruções de `agents/gsd-doc-writer.md` uma vez antes de começar. Siga as instruções de create_mode ou update_mode desse agente para cada documento, usando os mesmos campos doc_assignment do caminho paralelo.

**Onda 1 (sequencial — complete todos os três antes de iniciar a Onda 2):**

Para cada documento da Onda 1, construa o bloco doc_assignment equivalente e gere o arquivo inline:

1. **README** — mode de resolve_modes; para mode update/supplement, inclua existing_content
   - Construa doc_assignment: `type: readme`, `mode: {create|update|supplement}`, `preservation_mode: {value|null}`, `project_context: {INIT JSON}`, `existing_content:` (se update/supplement)
   - Explore o código-base (Read, Grep, Glob, Bash) seguindo as instruções create_mode / update_mode do gsd-doc-writer
   - Escreva o arquivo no caminho resolvido (README.md)

2. **ARCHITECTURE** — mode de resolve_modes; para mode update/supplement, inclua existing_content
   - Construa doc_assignment: `type: architecture`, `mode: {create|update|supplement}`, `preservation_mode: {value|null}`, `project_context: {INIT JSON}`, `existing_content:` (se update/supplement)
   - Explore o código-base seguindo as instruções do gsd-doc-writer
   - Escreva o arquivo no caminho resolvido (docs/ARCHITECTURE.md, ou ARCHITECTURE.md se encontrado na raiz como fallback)

3. **CONFIGURATION** — mode de resolve_modes; para mode update/supplement, inclua existing_content
   - Construa doc_assignment: `type: configuration`, `mode: {create|update|supplement}`, `preservation_mode: {value|null}`, `project_context: {INIT JSON}`, `existing_content:` (se update/supplement)
   - Aplique marcadores VERIFY a qualquer afirmação de infraestrutura não descobrível a partir do repositório
   - Explore o código-base seguindo as instruções do gsd-doc-writer
   - Escreva o arquivo no caminho resolvido (docs/CONFIGURATION.md, ou CONFIGURATION.md se encontrado na raiz como fallback)

**Onda 2 (sequencial — comece apenas após todos os documentos da Onda 1 estarem escritos):**

Os documentos da Onda 2 podem referenciar saídas da Onda 1 já que estão escritas. Inclua `wave_1_outputs` em cada doc_assignment.

4. **GETTING-STARTED** — mode de resolve_modes; inclua wave_1_outputs: [README.md, docs/ARCHITECTURE.md, docs/CONFIGURATION.md]
5. **DEVELOPMENT** — mode de resolve_modes; inclua wave_1_outputs
6. **TESTING** — mode de resolve_modes; inclua wave_1_outputs
7. **API** (apenas se enfileirado) — mode de resolve_modes; inclua wave_1_outputs
8. **DEPLOYMENT** (apenas se enfileirado) — Aplique marcadores VERIFY a qualquer afirmação de infraestrutura não descobrível a partir do repositório; inclua wave_1_outputs
9. **CONTRIBUTING** (apenas se enfileirado) — mode de resolve_modes; inclua wave_1_outputs

**READMEs por pacote do monorepo (apenas se `monorepo_workspaces` não estiver vazio):**

Após todos os 9 documentos de nível raiz estarem escritos, gere READMEs por pacote sequencialmente:

Para cada diretório de pacote resolvido (da expansão glob de workspace) que contém um `package.json`:
- Determine o mode: se `{package_dir}/README.md` existir, mode = `update`; caso contrário mode = `create`
- Construa doc_assignment: `type: readme`, `mode: {create|update}`, `scope: per_package`, `package_dir: {caminho absoluto}`, `project_context: {INIT JSON com project_root definido para o diretório do pacote}`, `existing_content:` (se update)
- Siga as instruções do gsd-doc-writer para escopo per_package
- Escreva o arquivo em `{package_dir}/README.md`

Continue para verify_docs.
</step>

<step name="verify_docs">
Verifique afirmações factuais em TODOS os documentos — tanto canônicos (gerados) quanto não canônicos (existentes escritos à mão) — contra o código-base ativo.

**CRÍTICO: Leia o manifesto de trabalho primeiro.**

```
Read .planning/tmp/docs-work-manifest.json
```

Extraia `canonical_queue` (itens com `status: "completed"`) e `review_queue` (itens com `status: "pending_review"`). Ambas as filas são verificadas neste passo.

**Condição de pulo:** Se `--verify-only` estiver presente em `$ARGUMENTS`, este passo já foi tratado por `verify_only_report` (saída antecipada). Pule.

**Fase 1: Verifique documentos canônicos (docs gerados/atualizados)**

Para cada documento em `canonical_queue` que foi escrito com sucesso no disco:

1. Inicie o agente `gsd-doc-verifier` (ou invoque sequencialmente se a ferramenta Task estiver indisponível) com um bloco `<verify_assignment>`:
   ```xml
   <verify_assignment>
   doc_path: {caminho relativo para o arquivo doc, ex.: README.md}
   project_root: {project_root do JSON init}
   </verify_assignment>
   ```

2. Após o verificador completar, leia o JSON de resultado de `.planning/tmp/verify-{doc_filename}.json`.

3. Atualize o manifesto: defina `status: "verified"` para cada documento canônico processado.

**Fase 2: Verifique documentos não canônicos (documentos existentes escritos à mão)**

Isto NÃO é opcional. Cada documento em `review_queue` DEVE ser verificado.

Para cada documento em `review_queue` do manifesto:

1. Inicie o agente `gsd-doc-verifier` com o mesmo bloco `<verify_assignment>` acima.
2. Leia o JSON de resultado de `.planning/tmp/verify-{doc_filename}.json`.
3. Atualize o manifesto: defina `status: "verified"` para cada documento da review_queue processado.

Documentos não canônicos com falhas SÃO elegíveis para o fix_loop. Quando um documento não canônico tem `claims_failed > 0`, despache-o ao gsd-doc-writer no modo `fix` com o array de falhas — o escritor em modo fix faz correções cirúrgicas em linhas específicas independentemente do tipo de documento (sem necessidade de template). O escritor NÃO DEVE reestruturar, reformular ou reformatar nenhum conteúdo além das afirmações falhando.

**Fase 3: Apresente o resumo combinado de verificação**

Colete TODOS os resultados (canônicos + não canônicos) em um único array `verification_results`:

```
Resultados de verificação:

Documentos canônicos (gerados):

| Documento | Afirmações | Aprovadas | Reprovadas |
|-----------|------------|-----------|------------|
| README.md | 12 | 10 | 2 |
| docs/architecture/overview.md | 8 | 8 | 0 |

Documentos existentes (revisados):

| Documento | Afirmações | Aprovadas | Reprovadas |
|-----------|------------|-----------|------------|
| docs/frontend/components/button.md | 5 | 4 | 1 |
| docs/services/api.md | 8 | 8 | 0 |

Total: {total_checked} afirmações verificadas, {total_failed} falhas
```

Escreva o manifesto atualizado de volta ao disco.

Se todos os documentos tiverem `claims_failed === 0`: pule fix_loop, continue para scan_for_secrets.
Se algum documento (canônico OU não canônico) tiver `claims_failed > 0`: continue para fix_loop.
</step>

<step name="fix_loop">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — identifique TODOS os documentos (canônicos E não canônicos) com `claims_failed > 0` dos resultados de verificação em `.planning/tmp/verify-*.json`. Ambas as filas são elegíveis para correções.

Corrija imprecisões sinalizadas reenviando documentos com falhas ao doc-writer no modo fix. Conforme D-06, máximo de 2 iterações. Conforme D-05, pare imediatamente em caso de regressão.

**Condição de pulo:** Se todos os documentos passaram na verificação (sem falhas), pule este passo.

**Rastreamento de iteração:**
- `MAX_FIX_ITERATIONS = 2`
- `iteration = 0`
- `previous_passed_docs` = conjunto de doc_paths onde claims_failed === 0 após a verificação inicial

**Para cada iteração (enquanto iteration < MAX_FIX_ITERATIONS e houver documentos com falhas):**

1. Para cada documento com `claims_failed > 0` nos últimos verification_results:
   a. Leia o conteúdo atual do arquivo no disco.
   b. Inicie agente `gsd-doc-writer` (ou invoque sequencialmente) com uma atribuição de correção:
      ```xml
      <doc_assignment>
      type: {tipo de documento original da fila, ex.: readme}
      mode: fix
      doc_path: {caminho relativo}
      project_context: {INIT JSON}
      existing_content: {conteúdo atual do arquivo lido do disco}
      failures:
        - line: {linha}
          claim: "{afirmação}"
          expected: "{esperado}"
          actual: "{atual}"
      </doc_assignment>
      ```
   c. Um agente por documento com falhas. Não agrupe múltiplos documentos em uma única inicialização.

2. Após todos os agentes de correção completarem, re-verifique TODOS os documentos (não apenas os que foram corrigidos):
   - Re-execute o mesmo processo de verificação que o passo verify_docs.
   - Leia os JSONs de resultado atualizados de `.planning/tmp/verify-{doc_filename}.json`.

3. **Detecção de regressão (D-05):**
   Para cada documento nos novos verification_results:
   - Se este documento estava em `previous_passed_docs` (passou na rodada anterior) E agora tem `claims_failed > 0`, isso é uma REGRESSÃO.
   - Se regressão detectada: PARE o loop imediatamente. Apresente:
     ```
     REGRESSÃO DETECTADA — parando loop de correção.

     {doc_path} passou anteriormente na verificação mas agora tem {claims_failed} falhas após a iteração de correção {iteration + 1}.

     Isso significa que a correção introduziu novos erros. As falhas restantes requerem revisão manual.
     ```
     Continue para scan_for_secrets (não tente mais correções).

4. Atualize `previous_passed_docs` com documentos que agora passam.
5. Incremente `iteration`.

**Após esgotamento do loop (iteration === MAX_FIX_ITERATIONS e ainda há falhas):**

Apresente as falhas restantes:
```
Loop de correção concluído ({MAX_FIX_ITERATIONS} iterações). Falhas restantes:

| Documento | Afirmações com Falha |
|-----------|---------------------|
| {doc_path} | {count} |

Essas falhas requerem correção manual. Revise a saída de verificação em .planning/tmp/verify-*.json para detalhes.
```

Continue para scan_for_secrets.
</step>

<step name="verify_only_report">
**Atingido quando `--verify-only` estiver presente em `$ARGUMENTS`.** Este é um passo de saída antecipada — não prossiga para passos de despacho, geração, commit ou relatório após este passo.

Invoque o agente gsd-doc-verifier em modo somente leitura para cada arquivo em `existing_docs` do JSON init:

1. Para cada documento em `existing_docs`:
   a. Inicie `gsd-doc-verifier` (ou invoque sequencialmente se a ferramenta Task estiver indisponível) com:
      ```xml
      <verify_assignment>
      doc_path: {doc.path}
      project_root: {project_root do JSON init}
      </verify_assignment>
      ```
   b. Leia o JSON de resultado de `.planning/tmp/verify-{doc_filename}.json`.

2. Também conte marcadores VERIFY em cada documento: pesquise `<!-- VERIFY:` no conteúdo do arquivo.

Apresente uma tabela de resumo combinada:

```
Auditoria --verify-only:

| Arquivo | Afirmações Verificadas | Aprovadas | Reprovadas | Marcadores VERIFY |
|---------|----------------------|-----------|------------|-------------------|
| README.md | 12 | 10 | 2 | 0 |
| docs/ARCHITECTURE.md | 8 | 8 | 0 | 0 |
| docs/CONFIGURATION.md | 5 | 3 | 2 | 5 |
| ... | ... | ... | ... | ... |

Total: {total_checked} afirmações verificadas, {total_failed} falhas, {total_markers} marcadores VERIFY requerendo revisão manual
```

Se houver falhas, mostre os detalhes:
```
Afirmações com falha:
  README.md:34 - "src/cli/index.ts" (esperado: arquivo existe, atual: arquivo não encontrado)
  docs/CONFIGURATION.md:12 - "npm run deploy" (esperado: script em package.json, atual: script não encontrado)
```

Exiba nota:
```
Para corrigir falhas automaticamente: /gsd-docs-update (executa geração + loop de correção)
Para regenerar todos os documentos do zero: /gsd-docs-update --force
```

Limpe arquivos temporários: remova os arquivos `.planning/tmp/verify-*.json`.

Encerre o workflow — não prossiga para passos de despacho, commit ou relatório.
</step>

<step name="scan_for_secrets">
VERIFICAÇÃO DE SEGURANÇA CRÍTICA: Analise todos os arquivos de documentação gerados/atualizados em busca de segredos vazados acidentalmente antes do commit. Conforme D-07, isso executa uma vez após o loop de correção ser concluído, antes de commit_docs.

Construa a lista de arquivos a partir da fila de geração — inclua todos os documentos que foram escritos no disco (criados, atualizados, suplementados ou corrigidos). Não codifique uma lista estática; use a lista real de arquivos que foram gerados ou modificados.

Execute a detecção de padrões de segredo:

```bash
# Verifique padrões comuns de chave de API em documentos gerados
grep -E '(sk-[a-zA-Z0-9]{20,}|sk_live_[a-zA-Z0-9]+|sk_test_[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|glpat-[a-zA-Z0-9_-]+|AKIA[A-Z0-9]{16}|xox[baprs]-[a-zA-Z0-9-]+|-----BEGIN.*PRIVATE KEY|eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.)' \
  {lista de arquivos de documentação gerados separada por espaço} 2>/dev/null \
  && SECRETS_FOUND=true || SECRETS_FOUND=false
```

**Se SECRETS_FOUND=true:**

```
ALERTA DE SEGURANÇA: Possíveis segredos detectados na documentação gerada!

Encontrados padrões que parecem chaves de API ou tokens em:
{mostrar saída do grep}

Isso exporia credenciais se for feito commit.

Ação necessária:
1. Revise as linhas sinalizadas acima
2. Remova quaisquer segredos reais dos arquivos de documentação
3. Re-execute /gsd-docs-update para regenerar documentação limpa
```

Então confirme com AskUserQuestion:

```
AskUserQuestion([{
  question: "Possíveis segredos detectados em documentos gerados. Como gostaria de prosseguir?",
  header: "Segurança",
  multiSelect: false,
  options: [
    { label: "Seguro para prosseguir", description: "Revisei as linhas sinalizadas — sem segredos reais, faça commit dos documentos" },
    { label: "Cancelar commit", description: "Pular o commit — vou limpar os documentos primeiro" }
  ]
}])
```

Se o usuário selecionar "Cancelar commit": pule commit_docs e continue para report. Se "Seguro para prosseguir": continue para commit_docs.

**Se SECRETS_FOUND=false:**

Continue para commit_docs.
</step>

<step name="commit_docs">
Execute este passo apenas se `commit_docs` for `true` no JSON init. Se `commit_docs` for false, pule para report.

Monte a lista de arquivos que foram realmente gerados (não inclua arquivos que falharam ou foram ignorados):

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: generate project documentation" \
  --files README.md docs/ARCHITECTURE.md docs/CONFIGURATION.md docs/GETTING-STARTED.md docs/DEVELOPMENT.md docs/TESTING.md
# Adicione quaisquer documentos condicionais que foram gerados:
# --files ... docs/API.md docs/DEPLOYMENT.md CONTRIBUTING.md
# Adicione READMEs por pacote se o despacho do monorepo executou:
# --files ... packages/core/README.md packages/cli/README.md
```

Inclua apenas os arquivos que foram escritos com sucesso no disco. Não inclua documentos com falha ou ignorados.

Continue para report.
</step>

<step name="report">
**Leia o manifesto de trabalho primeiro:** `Read .planning/tmp/docs-work-manifest.json` — use o manifesto para compilar o relatório completo cobrindo todos os documentos canônicos, resultados da review_queue e resultados da gap_queue. O manifesto é a fonte de verdade do que foi processado.

Apresente um resumo de conclusão ao usuário.

**Formato do resumo:**

```
Geração de documentação concluída.

Tipo do projeto: {primary_type}

Documentos gerados:
| Arquivo | Modo | Linhas |
|---------|------|--------|
| README.md | create | 87 |
| docs/ARCHITECTURE.md | update | 124 |
| docs/GETTING-STARTED.md | create | 63 |
| docs/DEVELOPMENT.md | create | 71 |
| docs/TESTING.md | create | 58 |
| docs/CONFIGURATION.md | create | 45 |
[documentos condicionais se gerados]

{Se READMEs por pacote do monorepo foram gerados:}
READMEs por pacote:
| Pacote | Modo | Linhas |
|--------|------|--------|
| packages/core | create | 42 |
| packages/cli | create | 38 |

{Se algum documento falhou ou foi ignorado:}
Ignorados / com falha:
  - docs/API.md: agente não completou

{Se preservation_check executou:}
Decisões de preservação:
  - {filename}: {preserve|supplement|regenerate}

{Se docs/DEPLOYMENT.md ou docs/CONFIGURATION.md foram gerados:}
Marcadores VERIFY: {N} marcadores colocados em docs/DEPLOYMENT.md e/ou docs/CONFIGURATION.md para afirmações de infraestrutura que requerem verificação manual.

{Se review_queue não estava vazia:}

Revisão de precisão de documentos existentes:

| Documento | Afirmações Verificadas | Aprovadas | Reprovadas | Corrigidas |
|-----------|----------------------|-----------|------------|------------|
| docs/api/endpoint-map.md | 5 | 4 | 1 | 1 |

{Para quaisquer falhas não corrigidas restantes após fix_loop:}
Imprecisões restantes não puderam ser corrigidas automaticamente — revisão manual recomendada para os itens sinalizados acima.

{Se commit_docs for true:}
Todos os arquivos gerados foram enviados para o repositório.
```

Lembre ao usuário que pode verificar os documentos gerados:

```
Execute `/gsd-docs-update --verify-only` para verificar os documentos gerados contra o código-base.
```

Encerre o workflow.
</step>

</process>

<success_criteria>
- [ ] JSON docs-init carregado e todos os campos extraídos
- [ ] Tipo de projeto corretamente classificado a partir dos sinais project_type
- [ ] Fila de documentos contém todos os documentos sempre ativos mais apenas os documentos condicionais correspondendo aos sinais do projeto
- [ ] CHANGELOG.md NÃO foi gerado ou enfileirado
- [ ] Cada documento foi gerado no modo correto (create para novo, update para existente)
- [ ] Documentos da Onda 1 (README, ARCHITECTURE, CONFIGURATION) completados antes da Onda 2 iniciar
- [ ] Documentos gerados contêm zero conteúdo da metodologia GSD
- [ ] docs/DEPLOYMENT.md e docs/CONFIGURATION.md usam marcadores VERIFY para afirmações não descobríveis (se gerados)
- [ ] Todos os arquivos gerados foram enviados para o repositório (se commit_docs for true)
- [ ] Documentos escritos à mão (sem marcador GSD) solicitaram preserve/supplement/regenerate antes do despacho (a menos que --force)
- [ ] Flag --force pulou os prompts de preservação e regenerou todos os documentos
- [ ] Flag --verify-only relatou o status dos documentos sem gerar arquivos
- [ ] READMEs por pacote gerados para workspaces do monorepo (se aplicável)
- [ ] Passo verify_docs verificou todos os documentos gerados contra o código-base ativo
- [ ] fix_loop executou no máximo 2 iterações e parou em regressão
- [ ] scan_for_secrets executou antes do commit e bloqueou em padrões detectados
- [ ] --verify-only invoca gsd-doc-verifier para verificação completa de fatos (não apenas contagem de marcadores VERIFY)
</success_criteria>
