# Template de Prompt de Fase

> **Nota:** A metodologia de planejamento está em `agents/gsd-planner.md`.
> Este template define o formato de saída do PLAN.md produzido pelo agente.

Template para `.planning/phases/XX-name/{phase}-{plan}-PLAN.md` - planos de fase executáveis otimizados para execução paralela.

**Nomenclatura:** Use o formato `{phase}-{plan}-PLAN.md` (ex.: `01-02-PLAN.md` para a Fase 1, Plano 2)

---

## Template do Arquivo

```markdown
---
phase: XX-name
plan: NN
type: execute
wave: N                     # Onda de execução (1, 2, 3...). Pré-calculada no momento do planejamento.
depends_on: []              # IDs de planos que este plano requer (ex.: ["01-01"]).
files_modified: []          # Arquivos que este plano modifica.
autonomous: true            # false se o plano possui checkpoints que exigem interação do usuário
requirements: []            # OBRIGATÓRIO — IDs de Requisitos do ROADMAP que este plano atende. NÃO PODE estar vazio.
user_setup: []              # Configuração exigida por humanos que Claude não pode automatizar (veja abaixo)

# Verificação retroativa a partir do objetivo (derivada durante o planejamento, verificada após a execução)
must_haves:
  truths: []                # Comportamentos observáveis que devem ser verdadeiros para o objetivo ser atingido
  artifacts: []             # Arquivos que devem existir com implementação real
  key_links: []             # Conexões críticas entre artefatos
---

<objective>
[O que este plano realiza]

Propósito: [Por que isso é importante para o projeto]
Saída: [Quais artefatos serão criados]
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
[Se o plano contiver tarefas de checkpoint (type="checkpoint:*"), adicione:]
@$HOME/.claude/get-shit-done/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# Referencie SUMMARYs de planos anteriores apenas quando genuinamente necessário:
# - Este plano usa tipos/exports de um plano anterior
# - Um plano anterior tomou uma decisão que afeta este plano
# NÃO encadeie reflexivamente: Plano 02 referencia 01, Plano 03 referencia 02...

[Arquivos-fonte relevantes:]
@src/path/to/relevant.ts
</context>

<tasks>

<task type="auto">
  <name>Tarefa 1: [Nome orientado à ação]</name>
  <files>path/to/file.ext, another/file.ext</files>
  <read_first>path/to/reference.ext, path/to/source-of-truth.ext</read_first>
  <action>[Implementação específica - o que fazer, como fazer, o que evitar e POR QUÊ. Inclua valores CONCRETOS: identificadores exatos, parâmetros, saídas esperadas, caminhos de arquivo, argumentos de comando. Nunca diga "alinhe X com Y" sem especificar o estado-alvo exato.]</action>
  <verify>[Comando ou verificação para provar que funcionou]</verify>
  <acceptance_criteria>
    - [Condição verificável com grep: "file.ext contém 'string exata'"]
    - [Condição mensurável: "output.ext usa 'valor-esperado', NÃO 'valor-errado'"]
  </acceptance_criteria>
  <done>[Critérios de aceitação mensuráveis]</done>
</task>

<task type="auto">
  <name>Tarefa 2: [Nome orientado à ação]</name>
  <files>path/to/file.ext</files>
  <read_first>path/to/reference.ext</read_first>
  <action>[Implementação específica com valores concretos]</action>
  <verify>[Comando ou verificação]</verify>
  <acceptance_criteria>
    - [Condição verificável com grep]
  </acceptance_criteria>
  <done>[Critérios de aceitação]</done>
</task>

<!-- Para exemplos e padrões de tarefas de checkpoint, veja @$HOME/.claude/get-shit-done/references/checkpoints.md -->

<task type="checkpoint:decision" gate="blocking">
  <decision>[O que precisa ser decidido]</decision>
  <context>[Por que esta decisão importa]</context>
  <options>
    <option id="option-a"><name>[Nome]</name><pros>[Benefícios]</pros><cons>[Compensações]</cons></option>
    <option id="option-b"><name>[Nome]</name><pros>[Benefícios]</pros><cons>[Compensações]</cons></option>
  </options>
  <resume-signal>Selecione: option-a ou option-b</resume-signal>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[O que Claude construiu] - servidor rodando em [URL]</what-built>
  <how-to-verify>Acesse [URL] e verifique: [apenas verificações visuais, SEM comandos CLI]</how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva os problemas</resume-signal>
</task>

</tasks>

<verification>
Antes de declarar o plano concluído:
- [ ] [Comando de teste específico]
- [ ] [Build/verificação de tipos passa]
- [ ] [Verificação de comportamento]
</verification>

<success_criteria>

- Todas as tarefas concluídas
- Todas as verificações passam
- Nenhum erro ou aviso introduzido
- [Critérios específicos do plano]
  </success_criteria>

<output>
Após a conclusão, crie `.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md`
</output>
```

---

## Campos do Frontmatter

| Campo | Obrigatório | Propósito |
|-------|-------------|-----------|
| `phase` | Sim | Identificador da fase (ex.: `01-foundation`) |
| `plan` | Sim | Número do plano dentro da fase (ex.: `01`, `02`) |
| `type` | Sim | Sempre `execute` para planos padrão, `tdd` para planos TDD |
| `wave` | Sim | Número da onda de execução (1, 2, 3...). Pré-calculado no momento do planejamento. |
| `depends_on` | Sim | Array de IDs de planos que este plano requer. |
| `files_modified` | Sim | Arquivos que este plano toca. |
| `autonomous` | Sim | `true` se não houver checkpoints, `false` se houver checkpoints |
| `requirements` | Sim | **DEVE** listar IDs de requisitos do ROADMAP. Todo requisito do roadmap DEVE aparecer em pelo menos um plano. |
| `user_setup` | Não | Array de itens de configuração exigidos por humanos (serviços externos) |
| `must_haves` | Sim | Critérios de verificação retroativa a partir do objetivo (veja abaixo) |

**Wave é pré-calculado:** Os números de wave são atribuídos durante `/gsd-plan-phase`. O execute-phase lê o `wave` diretamente do frontmatter e agrupa os planos por número de wave. Nenhuma análise de dependência em tempo de execução é necessária.

**Must-haves habilitam verificação:** O campo `must_haves` carrega os requisitos retroativos do objetivo do planejamento para a execução. Após todos os planos serem concluídos, o execute-phase spawna um subagente de verificação que verifica esses critérios em relação ao codebase real.

---

## Paralelo vs Sequencial

<parallel_examples>

**Candidatos para Wave 1 (paralelo):**

```yaml
# Plano 01 - Funcionalidade de Usuário
wave: 1
depends_on: []
files_modified: [src/models/user.ts, src/api/users.ts]
autonomous: true

# Plano 02 - Funcionalidade de Produto (sem sobreposição com o Plano 01)
wave: 1
depends_on: []
files_modified: [src/models/product.ts, src/api/products.ts]
autonomous: true

# Plano 03 - Funcionalidade de Pedido (sem sobreposição)
wave: 1
depends_on: []
files_modified: [src/models/order.ts, src/api/orders.ts]
autonomous: true
```

Todos os três rodam em paralelo (Wave 1) - sem dependências, sem conflitos de arquivo.

**Sequencial (dependência genuína):**

```yaml
# Plano 01 - Fundação de Auth
wave: 1
depends_on: []
files_modified: [src/lib/auth.ts, src/middleware/auth.ts]
autonomous: true

# Plano 02 - Funcionalidades protegidas (precisa de auth)
wave: 2
depends_on: ["01"]
files_modified: [src/features/dashboard.ts]
autonomous: true
```

O Plano 02 na Wave 2 aguarda o Plano 01 na Wave 1 - dependência genuína dos tipos/middleware de auth.

**Plano com checkpoint:**

```yaml
# Plano 03 - UI com verificação
wave: 3
depends_on: ["01", "02"]
files_modified: [src/components/Dashboard.tsx]
autonomous: false  # Possui checkpoint:human-verify
```

A Wave 3 roda após as Waves 1 e 2. Pausa no checkpoint, o orquestrador apresenta ao usuário, retoma após aprovação.

</parallel_examples>

---

## Seção de Contexto

**Contexto ciente de paralelismo:**

```markdown
<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# Inclua referências a SUMMARYs apenas se genuinamente necessário:
# - Este plano importa tipos de um plano anterior
# - Um plano anterior tomou uma decisão que afeta este plano
# - A saída de um plano anterior é entrada para este plano
#
# Planos independentes NÃO precisam de referências a SUMMARYs anteriores.
# NÃO encadeie reflexivamente: 02 referencia 01, 03 referencia 02...

@src/relevant/source.ts
</context>
```

**Padrão ruim (cria dependências falsas):**
```markdown
<context>
@.planning/phases/03-features/03-01-SUMMARY.md  # Apenas porque vem antes do 02
@.planning/phases/03-features/03-02-SUMMARY.md  # Encadeamento reflexivo
</context>
```

---

## Orientações de Escopo

**Dimensionamento de planos:**

- 2-3 tarefas por plano
- Uso máximo de ~50% do contexto
- Fases complexas: Múltiplos planos focados, não um plano grande

**Quando dividir:**

- Subsistemas diferentes (auth vs API vs UI)
- Mais de 3 tarefas
- Risco de overflow de contexto
- Candidatos TDD - planos separados

**Fatias verticais preferidas:**

```
PREFIRA: Plano 01 = Usuário (model + API + UI)
         Plano 02 = Produto (model + API + UI)

EVITE:   Plano 01 = Todos os models
         Plano 02 = Todas as APIs
         Plano 03 = Todas as UIs
```

---

## Planos TDD

Funcionalidades TDD recebem planos dedicados com `type: tdd`.

**Heurística:** Você consegue escrever `expect(fn(input)).toBe(output)` antes de escrever `fn`?
→ Sim: Crie um plano TDD
→ Não: Tarefa padrão em plano padrão

Veja `$HOME/.claude/get-shit-done/references/tdd.md` para a estrutura de planos TDD.

---

## Tipos de Tarefas

| Tipo | Uso | Autonomia |
|------|-----|-----------|
| `auto` | Tudo que Claude pode fazer independentemente | Totalmente autônomo |
| `checkpoint:human-verify` | Verificação visual/funcional | Pausa, retorna ao orquestrador |
| `checkpoint:decision` | Escolhas de implementação | Pausa, retorna ao orquestrador |
| `checkpoint:human-action` | Etapas manuais verdadeiramente inevitáveis (raro) | Pausa, retorna ao orquestrador |

**Comportamento de checkpoint na execução paralela:**
- O plano roda até o checkpoint
- O agente retorna com detalhes do checkpoint + agent_id
- O orquestrador apresenta ao usuário
- O usuário responde
- O orquestrador retoma o agente com `resume: agent_id`

---

## Exemplos

**Plano paralelo autônomo:**

```markdown
---
phase: 03-features
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: [src/features/user/model.ts, src/features/user/api.ts, src/features/user/UserList.tsx]
autonomous: true
---

<objective>
Implementar a funcionalidade de Usuário completa como fatia vertical.

Propósito: Gerenciamento de usuários autocontido que pode rodar em paralelo com outras funcionalidades.
Saída: Model de usuário, endpoints de API e componentes de UI.
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
</context>

<tasks>
<task type="auto">
  <name>Tarefa 1: Criar model de Usuário</name>
  <files>src/features/user/model.ts</files>
  <action>Definir o tipo User com id, email, name, createdAt. Exportar interface TypeScript.</action>
  <verify>tsc --noEmit passa</verify>
  <done>Tipo User exportado e utilizável</done>
</task>

<task type="auto">
  <name>Tarefa 2: Criar endpoints de API de Usuário</name>
  <files>src/features/user/api.ts</files>
  <action>GET /users (lista), GET /users/:id (individual), POST /users (criação). Usar tipo User do model.</action>
  <verify>testes fetch passam para todos os endpoints</verify>
  <done>Todas as operações CRUD funcionam</done>
</task>
</tasks>

<verification>
- [ ] npm run build bem-sucedido
- [ ] Endpoints de API respondem corretamente
</verification>

<success_criteria>
- Todas as tarefas concluídas
- Funcionalidade de Usuário funciona de ponta a ponta
</success_criteria>

<output>
Após a conclusão, crie `.planning/phases/03-features/03-01-SUMMARY.md`
</output>
```

**Plano com checkpoint (não autônomo):**

```markdown
---
phase: 03-features
plan: 03
type: execute
wave: 2
depends_on: ["03-01", "03-02"]
files_modified: [src/components/Dashboard.tsx]
autonomous: false
---

<objective>
Construir dashboard com verificação visual.

Propósito: Integrar funcionalidades de usuário e produto em uma visão unificada.
Saída: Componente de dashboard funcionando.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
@$HOME/.claude/get-shit-done/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/phases/03-features/03-01-SUMMARY.md
@.planning/phases/03-features/03-02-SUMMARY.md
</context>

<tasks>
<task type="auto">
  <name>Tarefa 1: Construir layout do Dashboard</name>
  <files>src/components/Dashboard.tsx</files>
  <action>Criar grade responsiva com componentes UserList e ProductList. Usar Tailwind para estilização.</action>
  <verify>npm run build bem-sucedido</verify>
  <done>Dashboard renderiza sem erros</done>
</task>

<!-- Padrão de checkpoint: Claude inicia o servidor, usuário acessa a URL. Veja checkpoints.md para padrões completos. -->
<task type="auto">
  <name>Iniciar servidor de desenvolvimento</name>
  <action>Execute `npm run dev` em segundo plano, aguarde o estado de pronto</action>
  <verify>fetch http://localhost:3000 retorna 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard - servidor em http://localhost:3000</what-built>
  <how-to-verify>Acesse localhost:3000/dashboard. Verifique: grade no desktop, empilhamento no mobile, sem problemas de scroll.</how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva os problemas</resume-signal>
</task>
</tasks>

<verification>
- [ ] npm run build bem-sucedido
- [ ] Verificação visual aprovada
</verification>

<success_criteria>
- Todas as tarefas concluídas
- Usuário aprovou o layout visual
</success_criteria>

<output>
Após a conclusão, crie `.planning/phases/03-features/03-03-SUMMARY.md`
</output>
```

---

## Anti-Padrões

**Ruim: Encadeamento reflexivo de dependências**
```yaml
depends_on: ["03-01"]  # Apenas porque 01 vem antes do 02
```

**Ruim: Agrupamento horizontal por camadas**
```
Plano 01: Todos os models
Plano 02: Todas as APIs (depende do 01)
Plano 03: Todas as UIs (depende do 02)
```

**Ruim: Flag de autonomia ausente**
```yaml
# Tem checkpoint mas sem autonomous: false
depends_on: []
files_modified: [...]
# autonomous: ???  <- Ausente!
```

**Ruim: Tarefas vagas**
```xml
<task type="auto">
  <name>Configurar autenticação</name>
  <action>Adicionar auth ao aplicativo</action>
</task>
```

**Ruim: read_first ausente (executor modifica arquivos que não leu)**
```xml
<task type="auto">
  <name>Atualizar configuração do banco de dados</name>
  <files>src/config/database.ts</files>
  <!-- Sem read_first! O executor não conhece o estado atual nem as convenções -->
  <action>Atualizar a configuração do banco de dados para corresponder às configurações de produção</action>
</task>
```

**Ruim: Critérios de aceitação vagos (não verificáveis)**
```xml
<acceptance_criteria>
  - Configuração está devidamente configurada
  - Conexão com o banco de dados funciona corretamente
</acceptance_criteria>
```

**Bom: Concreto com read_first + critérios verificáveis**
```xml
<task type="auto">
  <name>Atualizar configuração do banco de dados para pool de conexões</name>
  <files>src/config/database.ts</files>
  <read_first>src/config/database.ts, .env.example, docker-compose.yml</read_first>
  <action>Adicionar configuração de pool: min=2, max=20, idleTimeoutMs=30000. Adicionar configuração SSL: rejectUnauthorized=true quando NODE_ENV=production. Adicionar entrada em .env.example: DATABASE_POOL_MAX=20.</action>
  <acceptance_criteria>
    - database.ts contém "max: 20" e "idleTimeoutMillis: 30000"
    - database.ts contém SSL condicional ao NODE_ENV
    - .env.example contém DATABASE_POOL_MAX
  </acceptance_criteria>
</task>
```

---

## Diretrizes

- Sempre usar estrutura XML para o parse do Claude
- Incluir `wave`, `depends_on`, `files_modified`, `autonomous` em todo plano
- Preferir fatias verticais a camadas horizontais
- Referenciar SUMMARYs anteriores apenas quando genuinamente necessário
- Agrupar checkpoints com tarefas auto relacionadas no mesmo plano
- 2-3 tarefas por plano, máximo de ~50% do contexto

---

## Configuração do Usuário (Serviços Externos)

Quando um plano introduz serviços externos que requerem configuração humana, declare no frontmatter:

```yaml
user_setup:
  - service: stripe
    why: "Processamento de pagamentos requer chaves de API"
    env_vars:
      - name: STRIPE_SECRET_KEY
        source: "Painel Stripe → Desenvolvedores → Chaves de API → Chave secreta"
      - name: STRIPE_WEBHOOK_SECRET
        source: "Painel Stripe → Desenvolvedores → Webhooks → Segredo de assinatura"
    dashboard_config:
      - task: "Criar endpoint de webhook"
        location: "Painel Stripe → Desenvolvedores → Webhooks → Adicionar endpoint"
        details: "URL: https://[seu-domínio]/api/webhooks/stripe"
    local_dev:
      - "stripe listen --forward-to localhost:3000/api/webhooks/stripe"
```

**A regra de automação em primeiro lugar:** `user_setup` contém APENAS o que Claude literalmente não pode fazer:
- Criação de conta (requer cadastro humano)
- Recuperação de segredos (requer acesso ao painel)
- Configuração no painel (requer humano no navegador)

**NÃO incluído:** Instalações de pacotes, alterações de código, criação de arquivos, comandos CLI que Claude pode executar.

**Resultado:** O execute-plan gera `{phase}-USER-SETUP.md` com checklist para o usuário.

Veja `$HOME/.claude/get-shit-done/templates/user-setup.md` para esquema completo e exemplos

---

## Must-Haves (Verificação Retroativa a partir do Objetivo)

O campo `must_haves` define o que deve ser VERDADEIRO para que o objetivo da fase seja atingido. Derivado durante o planejamento, verificado após a execução.

**Estrutura:**

```yaml
must_haves:
  truths:
    - "Usuário consegue ver mensagens existentes"
    - "Usuário consegue enviar uma mensagem"
    - "Mensagens persistem após atualização da página"
  artifacts:
    - path: "src/components/Chat.tsx"
      provides: "Renderização da lista de mensagens"
      min_lines: 30
    - path: "src/app/api/chat/route.ts"
      provides: "Operações CRUD de mensagens"
      exports: ["GET", "POST"]
    - path: "prisma/schema.prisma"
      provides: "Model de mensagem"
      contains: "model Message"
  key_links:
    - from: "src/components/Chat.tsx"
      to: "/api/chat"
      via: "fetch no useEffect"
      pattern: "fetch.*api/chat"
    - from: "src/app/api/chat/route.ts"
      to: "prisma.message"
      via: "consulta ao banco de dados"
      pattern: "prisma\\.message\\.(find|create)"
```

**Descrição dos campos:**

| Campo | Propósito |
|-------|-----------|
| `truths` | Comportamentos observáveis da perspectiva do usuário. Cada um deve ser testável. |
| `artifacts` | Arquivos que devem existir com implementação real. |
| `artifacts[].path` | Caminho do arquivo relativo à raiz do projeto. |
| `artifacts[].provides` | O que este artefato entrega. |
| `artifacts[].min_lines` | Opcional. Número mínimo de linhas para ser considerado substantivo. |
| `artifacts[].exports` | Opcional. Exports esperados para verificação. |
| `artifacts[].contains` | Opcional. Padrão que deve existir no arquivo. |
| `key_links` | Conexões críticas entre artefatos. |
| `key_links[].from` | Artefato de origem. |
| `key_links[].to` | Artefato de destino ou endpoint. |
| `key_links[].via` | Como eles se conectam (descrição). |
| `key_links[].pattern` | Opcional. Regex para verificar se a conexão existe. |

**Por que isso importa:**

Conclusão de tarefa ≠ Atingimento do objetivo. Uma tarefa "criar componente de chat" pode ser concluída com a criação de um placeholder. O campo `must_haves` captura o que deve realmente funcionar, permitindo que a verificação identifique lacunas antes que elas se acumulem.

**Fluxo de verificação:**

1. O plan-phase deriva os must_haves a partir do objetivo da fase (retroativamente)
2. Os must_haves são escritos no frontmatter do PLAN.md
3. O execute-phase executa todos os planos
4. O subagente de verificação verifica os must_haves em relação ao codebase
5. Lacunas encontradas → planos de correção criados → execução → re-verificação
6. Todos os must_haves passam → fase concluída

Veja `$HOME/.claude/get-shit-done/workflows/verify-phase.md` para a lógica de verificação.
