<overview>
Os planos são executados de forma autônoma. Os checkpoints formalizam pontos de interação onde verificação humana ou decisões são necessárias.

**Princípio fundamental:** Claude automatiza tudo via CLI/API. Checkpoints servem para verificação e decisões, não para trabalho manual.

**Regras de ouro:**
1. **Se Claude pode executar, Claude executa** - Nunca peça ao usuário para executar comandos CLI, iniciar servidores ou rodar builds
2. **Claude configura o ambiente de verificação** - Inicie servidores de desenvolvimento, popule bancos de dados, configure variáveis de ambiente
3. **O usuário só faz o que requer julgamento humano** - Verificações visuais, avaliação de UX, "isso parece certo?"
4. **Segredos vêm do usuário, automação vem de Claude** - Peça chaves de API, então Claude as usa via CLI
5. **Modo automático ignora checkpoints de verificação/decisão** — Quando `workflow._auto_chain_active` ou `workflow.auto_advance` é verdadeiro na configuração: human-verify aprova automaticamente, decision seleciona automaticamente a primeira opção, human-action ainda para (portões de autenticação não podem ser automatizados)
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify (Mais Comum - 90%)

**Quando:** Claude concluiu o trabalho automatizado, o humano confirma que está funcionando corretamente.

**Use para:**
- Verificações visuais de UI (layout, estilização, responsividade)
- Fluxos interativos (navegar por wizard, testar fluxos do usuário)
- Verificação funcional (funcionalidade opera como esperado)
- Qualidade de reprodução de áudio/vídeo
- Suavidade de animações
- Testes de acessibilidade

**Estrutura:**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[O que Claude automatizou e implantou/construiu]</what-built>
  <how-to-verify>
    [Passos exatos para testar - URLs, comandos, comportamento esperado]
  </how-to-verify>
  <resume-signal>[Como continuar - "approved", "yes", ou descrever problemas]</resume-signal>
</task>
```

**Exemplo: Componente de UI (mostra padrão-chave: Claude inicia o servidor ANTES do checkpoint)**
```xml
<task type="auto">
  <name>Build responsive dashboard layout</name>
  <files>src/components/Dashboard.tsx, src/app/dashboard/page.tsx</files>
  <action>Create dashboard with sidebar, header, and content area. Use Tailwind responsive classes for mobile.</action>
  <verify>npm run build succeeds, no TypeScript errors</verify>
  <done>Dashboard component builds without errors</done>
</task>

<task type="auto">
  <name>Start dev server for verification</name>
  <action>Run `npm run dev` in background, wait for "ready" message, capture port</action>
  <verify>fetch http://localhost:3000 returns 200</verify>
  <done>Dev server running at http://localhost:3000</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Responsive dashboard layout - dev server running at http://localhost:3000</what-built>
  <how-to-verify>
    Visit http://localhost:3000/dashboard and verify:
    1. Desktop (>1024px): Sidebar left, content right, header top
    2. Tablet (768px): Sidebar collapses to hamburger menu
    3. Mobile (375px): Single column layout, bottom nav appears
    4. No layout shift or horizontal scroll at any size
  </how-to-verify>
  <resume-signal>Type "approved" or describe layout issues</resume-signal>
</task>
```

**Exemplo: Build com Xcode**
```xml
<task type="auto">
  <name>Build macOS app with Xcode</name>
  <files>App.xcodeproj, Sources/</files>
  <action>Run `xcodebuild -project App.xcodeproj -scheme App build`. Check for compilation errors in output.</action>
  <verify>Build output contains "BUILD SUCCEEDED", no errors</verify>
  <done>App builds successfully</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Built macOS app at DerivedData/Build/Products/Debug/App.app</what-built>
  <how-to-verify>
    Open App.app and test:
    - App launches without crashes
    - Menu bar icon appears
    - Preferences window opens correctly
    - No visual glitches or layout issues
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```
</type>

<type name="decision">
## checkpoint:decision (9%)

**Quando:** O humano deve fazer uma escolha que afeta a direção da implementação.

**Use para:**
- Seleção de tecnologia (qual provedor de auth, qual banco de dados)
- Decisões de arquitetura (monorepo vs. repositórios separados)
- Escolhas de design (esquema de cores, abordagem de layout)
- Priorização de funcionalidades (qual variante construir)
- Decisões de modelo de dados (estrutura do esquema)

**Estrutura:**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>[O que está sendo decidido]</decision>
  <context>[Por que essa decisão importa]</context>
  <options>
    <option id="option-a">
      <name>[Nome da opção]</name>
      <pros>[Benefícios]</pros>
      <cons>[Trocas]</cons>
    </option>
    <option id="option-b">
      <name>[Nome da opção]</name>
      <pros>[Benefícios]</pros>
      <cons>[Trocas]</cons>
    </option>
  </options>
  <resume-signal>[Como indicar a escolha]</resume-signal>
</task>
```

**Exemplo: Seleção de Provedor de Auth**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Select authentication provider</decision>
  <context>
    Need user authentication for the app. Three solid options with different tradeoffs.
  </context>
  <options>
    <option id="supabase">
      <name>Supabase Auth</name>
      <pros>Built-in with Supabase DB we're using, generous free tier, row-level security integration</pros>
      <cons>Less customizable UI, tied to Supabase ecosystem</cons>
    </option>
    <option id="clerk">
      <name>Clerk</name>
      <pros>Beautiful pre-built UI, best developer experience, excellent docs</pros>
      <cons>Paid after 10k MAU, vendor lock-in</cons>
    </option>
    <option id="nextauth">
      <name>NextAuth.js</name>
      <pros>Free, self-hosted, maximum control, widely adopted</pros>
      <cons>More setup work, you manage security updates, UI is DIY</cons>
    </option>
  </options>
  <resume-signal>Select: supabase, clerk, or nextauth</resume-signal>
</task>
```

**Exemplo: Seleção de Banco de Dados**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Select database for user data</decision>
  <context>
    App needs persistent storage for users, sessions, and user-generated content.
    Expected scale: 10k users, 1M records first year.
  </context>
  <options>
    <option id="supabase">
      <name>Supabase (Postgres)</name>
      <pros>Full SQL, generous free tier, built-in auth, real-time subscriptions</pros>
      <cons>Vendor lock-in for real-time features, less flexible than raw Postgres</cons>
    </option>
    <option id="planetscale">
      <name>PlanetScale (MySQL)</name>
      <pros>Serverless scaling, branching workflow, excellent DX</pros>
      <cons>MySQL not Postgres, no foreign keys in free tier</cons>
    </option>
    <option id="convex">
      <name>Convex</name>
      <pros>Real-time by default, TypeScript-native, automatic caching</pros>
      <cons>Newer platform, different mental model, less SQL flexibility</cons>
    </option>
  </options>
  <resume-signal>Select: supabase, planetscale, or convex</resume-signal>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action (1% - Raro)

**Quando:** A ação NÃO TEM CLI/API e requer interação somente humana, OU Claude encontrou um portão de autenticação durante a automação.

**Use APENAS para:**
- **Portões de autenticação** - Claude tentou CLI/API mas precisa de credenciais (isso NÃO é uma falha)
- Links de verificação por e-mail (clicar no e-mail)
- Códigos de SMS 2FA (verificação por telefone)
- Aprovações manuais de conta (a plataforma requer revisão humana)
- Fluxos 3D Secure de cartão de crédito (autorização de pagamento via web)
- Aprovações de apps OAuth (aprovação via web)

**NÃO use para trabalho manual pré-planejado:**
- Implantação (use CLI - portão de auth se necessário)
- Criação de webhooks/bancos de dados (use API/CLI - portão de auth se necessário)
- Executar builds/testes (use a ferramenta Bash)
- Criar arquivos (use a ferramenta Write)

**Estrutura:**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[O que o humano deve fazer - Claude já fez tudo que é automatizável]</action>
  <instructions>
    [O que Claude já automatizou]
    [A UMA coisa que requer ação humana]
  </instructions>
  <verification>[O que Claude pode verificar depois]</verification>
  <resume-signal>[Como continuar]</resume-signal>
</task>
```

**Exemplo: Verificação de E-mail**
```xml
<task type="auto">
  <name>Create SendGrid account via API</name>
  <action>Use SendGrid API to create subuser account with provided email. Request verification email.</action>
  <verify>API returns 201, account created</verify>
  <done>Account created, verification email sent</done>
</task>

<task type="checkpoint:human-action" gate="blocking">
  <action>Complete email verification for SendGrid account</action>
  <instructions>
    I created the account and requested verification email.
    Check your inbox for SendGrid verification link and click it.
  </instructions>
  <verification>SendGrid API key works: curl test succeeds</verification>
  <resume-signal>Type "done" when email verified</resume-signal>
</task>
```

**Exemplo: Portão de Autenticação (Checkpoint Dinâmico)**
```xml
<task type="auto">
  <name>Deploy to Vercel</name>
  <files>.vercel/, vercel.json</files>
  <action>Run `vercel --yes` to deploy</action>
  <verify>vercel ls shows deployment, fetch returns 200</verify>
</task>

<!-- Se vercel retornar "Error: Not authenticated", Claude cria o checkpoint dinamicamente -->

<task type="checkpoint:human-action" gate="blocking">
  <action>Authenticate Vercel CLI so I can continue deployment</action>
  <instructions>
    I tried to deploy but got authentication error.
    Run: vercel login
    This will open your browser - complete the authentication flow.
  </instructions>
  <verification>vercel whoami returns your account email</verification>
  <resume-signal>Type "done" when authenticated</resume-signal>
</task>

<!-- Após a autenticação, Claude tenta a implantação novamente -->

<task type="auto">
  <name>Retry Vercel deployment</name>
  <action>Run `vercel --yes` (now authenticated)</action>
  <verify>vercel ls shows deployment, fetch returns 200</verify>
</task>
```

**Distinção-chave:** Portões de auth são criados dinamicamente quando Claude encontra erros de auth. NÃO são pré-planejados — Claude automatiza primeiro, pede credenciais apenas quando bloqueado.
</type>
</checkpoint_types>

<execution_protocol>

Quando Claude encontra `type="checkpoint:*"`:

1. **Pare imediatamente** - não avance para a próxima tarefa
2. **Exiba o checkpoint claramente** usando o formato abaixo
3. **Aguarde a resposta do usuário** - não simule uma conclusão
4. **Verifique se possível** - verifique arquivos, execute testes, o que for especificado
5. **Retome a execução** - continue para a próxima tarefa apenas após confirmação

**Para checkpoint:human-verify:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Verificação Necessária                   ║
╚═══════════════════════════════════════════════════════╝

Progresso: 5/8 tarefas concluídas
Tarefa: Layout responsivo do dashboard

Construído: Dashboard responsivo em /dashboard

Como verificar:
  1. Acesse: http://localhost:3000/dashboard
  2. Desktop (>1024px): Barra lateral visível, conteúdo preenche o espaço restante
  3. Tablet (768px): Barra lateral colapsa para ícones
  4. Mobile (375px): Barra lateral oculta, menu hambúrguer aparece

────────────────────────────────────────────────────────
→ SUA AÇÃO: Digite "approved" ou descreva os problemas
────────────────────────────────────────────────────────
```

**Para checkpoint:decision:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Decisão Necessária                       ║
╚═══════════════════════════════════════════════════════╝

Progresso: 2/6 tarefas concluídas
Tarefa: Selecionar provedor de autenticação

Decisão: Qual provedor de auth devemos usar?

Contexto: Precisamos de autenticação de usuários. Três opções com diferentes trocas.

Opções:
  1. supabase - Integrado com nosso BD, tier gratuito
     Prós: Integração com segurança de linha, tier gratuito generoso
     Contras: UI menos customizável, lock-in de ecossistema

  2. clerk - Melhor DX, pago após 10k usuários
     Prós: UI pré-construída bonita, documentação excelente
     Contras: Lock-in de fornecedor, preços em escala

  3. nextauth - Auto-hospedado, controle máximo
     Prós: Gratuito, sem lock-in de fornecedor, amplamente adotado
     Contras: Mais trabalho de configuração, atualizações de segurança DIY

────────────────────────────────────────────────────────
→ SUA AÇÃO: Selecione supabase, clerk ou nextauth
────────────────────────────────────────────────────────
```

**Para checkpoint:human-action:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Ação Necessária                          ║
╚═══════════════════════════════════════════════════════╝

Progresso: 3/8 tarefas concluídas
Tarefa: Implantar no Vercel

Tentativa: vercel --yes
Erro: Não autenticado. Por favor, execute 'vercel login'

O que você precisa fazer:
  1. Execute: vercel login
  2. Complete a autenticação no navegador quando abrir
  3. Retorne aqui quando concluir

Vou verificar: vercel whoami retorna sua conta

────────────────────────────────────────────────────────
→ SUA AÇÃO: Digite "done" quando autenticado
────────────────────────────────────────────────────────
```
</execution_protocol>

<authentication_gates>

**Portão de auth = Claude tentou CLI/API, obteve erro de auth.** Não é uma falha — é um portão que requer entrada humana para desbloquear.

**Padrão:** Claude tenta automação → erro de auth → cria checkpoint:human-action → usuário autentica → Claude tenta novamente → continua

**Protocolo do portão:**
1. Reconheça que não é uma falha - auth ausente é esperado
2. Pare a tarefa atual - não tente repetidamente
3. Crie checkpoint:human-action dinamicamente
4. Forneça passos exatos de autenticação
5. Verifique se a autenticação funciona
6. Tente novamente a tarefa original
7. Continue normalmente

**Distinção-chave:**
- Checkpoint pré-planejado: "Eu preciso que você faça X" (errado - Claude deveria automatizar)
- Portão de auth: "Tentei automatizar X mas preciso de credenciais" (correto - desbloqueia a automação)

</authentication_gates>

<automation_reference>

**A regra:** Se tem CLI/API, Claude faz. Nunca peça ao humano para realizar trabalho automatizável.

## Referência de CLI de Serviços

| Serviço | CLI/API | Comandos Principais | Portão de Auth |
|---------|---------|--------------|-----------|
| Vercel | `vercel` | `--yes`, `env add`, `--prod`, `ls` | `vercel login` |
| Railway | `railway` | `init`, `up`, `variables set` | `railway login` |
| Fly | `fly` | `launch`, `deploy`, `secrets set` | `fly auth login` |
| Stripe | `stripe` + API | `listen`, `trigger`, chamadas de API | Chave de API em .env |
| Supabase | `supabase` | `init`, `link`, `db push`, `gen types` | `supabase login` |
| Upstash | `upstash` | `redis create`, `redis get` | `upstash auth login` |
| PlanetScale | `pscale` | `database create`, `branch create` | `pscale auth login` |
| GitHub | `gh` | `repo create`, `pr create`, `secret set` | `gh auth login` |
| Node | `npm`/`pnpm` | `install`, `run build`, `test`, `run dev` | N/A |
| Xcode | `xcodebuild` | `-project`, `-scheme`, `build`, `test` | N/A |
| Convex | `npx convex` | `dev`, `deploy`, `env set`, `env get` | `npx convex login` |

## Automação de Variáveis de Ambiente

**Arquivos env:** Use as ferramentas Write/Edit. Nunca peça ao humano para criar .env manualmente.

**Variáveis de ambiente do dashboard via CLI:**

| Plataforma | Comando CLI | Exemplo |
|----------|-------------|---------|
| Convex | `npx convex env set` | `npx convex env set OPENAI_API_KEY sk-...` |
| Vercel | `vercel env add` | `vercel env add STRIPE_KEY production` |
| Railway | `railway variables set` | `railway variables set API_KEY=value` |
| Fly | `fly secrets set` | `fly secrets set DATABASE_URL=...` |
| Supabase | `supabase secrets set` | `supabase secrets set MY_SECRET=value` |

**Padrão de coleta de segredos:**
```xml
<!-- ERRADO: Pedir ao usuário para adicionar variáveis de ambiente no dashboard -->
<task type="checkpoint:human-action">
  <action>Add OPENAI_API_KEY to Convex dashboard</action>
  <instructions>Go to dashboard.convex.dev → Settings → Environment Variables → Add</instructions>
</task>

<!-- CERTO: Claude pede o valor, depois adiciona via CLI -->
<task type="checkpoint:human-action">
  <action>Provide your OpenAI API key</action>
  <instructions>
    I need your OpenAI API key for Convex backend.
    Get it from: https://platform.openai.com/api-keys
    Paste the key (starts with sk-)
  </instructions>
  <verification>I'll add it via `npx convex env set` and verify</verification>
  <resume-signal>Paste your API key</resume-signal>
</task>

<task type="auto">
  <name>Configure OpenAI key in Convex</name>
  <action>Run `npx convex env set OPENAI_API_KEY {user-provided-key}`</action>
  <verify>`npx convex env get OPENAI_API_KEY` returns the key (masked)</verify>
</task>
```

## Automação de Servidor de Desenvolvimento

| Framework | Comando de Início | Sinal de Pronto | URL Padrão |
|-----------|---------------|--------------|-------------|
| Next.js | `npm run dev` | "Ready in" ou "started server" | http://localhost:3000 |
| Vite | `npm run dev` | "ready in" | http://localhost:5173 |
| Convex | `npx convex dev` | "Convex functions ready" | N/A (somente backend) |
| Express | `npm start` | "listening on port" | http://localhost:3000 |
| Django | `python manage.py runserver` | "Starting development server" | http://localhost:8000 |

**Ciclo de vida do servidor:**
```bash
# Run in background, capture PID
npm run dev &
DEV_SERVER_PID=$!

# Wait for ready (max 30s) — uses fetch() for cross-platform compatibility
timeout 30 bash -c 'until node -e "fetch(\"http://localhost:3000\").then(r=>{process.exit(r.ok?0:1)}).catch(()=>process.exit(1))" 2>/dev/null; do sleep 1; done'
```

**Conflitos de porta:** Mate o processo parado (`lsof -ti:3000 | xargs kill`) ou use porta alternativa (`--port 3001`).

**O servidor continua rodando** pelos checkpoints. Só encerre quando o plano estiver completo, ao mudar para produção, ou quando a porta for necessária para um serviço diferente.

## Tratamento de Instalação de CLI

| CLI | Instalação automática? | Comando |
|-----|---------------|---------|
| npm/pnpm/yarn | Não - pergunte ao usuário | Usuário escolhe o gerenciador de pacotes |
| vercel | Sim | `npm i -g vercel` |
| gh (GitHub) | Sim | `brew install gh` (macOS) ou `apt install gh` (Linux) |
| stripe | Sim | `npm i -g stripe` |
| supabase | Sim | `npm i -g supabase` |
| convex | Não - use npx | `npx convex` (sem instalação necessária) |
| fly | Sim | `brew install flyctl` ou instalador curl |
| railway | Sim | `npm i -g @railway/cli` |

**Protocolo:** Tente o comando → "command not found" → pode instalar automaticamente? → sim: instale silenciosamente, tente novamente → não: checkpoint pedindo ao usuário que instale.

## Falhas de Automação Antes do Checkpoint

| Falha | Resposta |
|---------|----------|
| Servidor não inicia | Verifique o erro, corrija o problema, tente novamente (não prossiga para o checkpoint) |
| Porta em uso | Mate o processo parado ou use porta alternativa |
| Dependência faltando | Execute `npm install`, tente novamente |
| Erro de build | Corrija o erro primeiro (bug, não problema de checkpoint) |
| Erro de auth | Crie checkpoint de portão de auth |
| Timeout de rede | Tente novamente com backoff, depois checkpoint se persistente |

**Nunca apresente um checkpoint com ambiente de verificação quebrado.** Se o servidor local não está respondendo, não peça ao usuário para "visitar localhost:3000".

> **Nota de compatibilidade:** Use `node -e "fetch('http://localhost:3000').then(r=>console.log(r.status))"` em vez de `curl` para verificações de saúde. `curl` tem problemas no Windows MSYS/Git Bash devido a problemas de SSL/manipulação de caminho.

```xml
<!-- ERRADO: Checkpoint com ambiente quebrado -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard (server failed to start)</what-built>
  <how-to-verify>Visit http://localhost:3000...</how-to-verify>
</task>

<!-- CERTO: Corrija primeiro, depois o checkpoint -->
<task type="auto">
  <name>Fix server startup issue</name>
  <action>Investigate error, fix root cause, restart server</action>
  <verify>fetch http://localhost:3000 returns 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Dashboard - server running at http://localhost:3000</what-built>
  <how-to-verify>Visit http://localhost:3000/dashboard...</how-to-verify>
</task>
```

## Referência Rápida de Automatizável

| Ação | Automatizável? | Claude faz? |
|--------|--------------|-----------------|
| Implantar no Vercel | Sim (`vercel`) | SIM |
| Criar webhook do Stripe | Sim (API) | SIM |
| Escrever arquivo .env | Sim (ferramenta Write) | SIM |
| Criar BD Upstash | Sim (`upstash`) | SIM |
| Executar testes | Sim (`npm test`) | SIM |
| Iniciar servidor de desenvolvimento | Sim (`npm run dev`) | SIM |
| Adicionar variáveis de ambiente ao Convex | Sim (`npx convex env set`) | SIM |
| Adicionar variáveis de ambiente ao Vercel | Sim (`vercel env add`) | SIM |
| Popular banco de dados | Sim (CLI/API) | SIM |
| Clicar em link de verificação de e-mail | Não | NÃO |
| Inserir cartão de crédito com 3DS | Não | NÃO |
| Completar OAuth no navegador | Não | NÃO |
| Verificar visualmente se a UI parece correta | Não | NÃO |
| Testar fluxos interativos do usuário | Não | NÃO |

</automation_reference>

<writing_guidelines>

**FAÇA:**
- Automatize tudo com CLI/API antes do checkpoint
- Seja específico: "Acesse https://myapp.vercel.app" não "verifique a implantação"
- Numere os passos de verificação
- Declare os resultados esperados: "Você deve ver X"
- Forneça contexto: por que este checkpoint existe

**NÃO FAÇA:**
- Pedir ao humano para fazer trabalho que Claude pode automatizar ❌
- Assumir conhecimento: "Configure as configurações habituais" ❌
- Pular passos: "Configure o banco de dados" (muito vago) ❌
- Misturar múltiplas verificações em um único checkpoint ❌

**Posicionamento:**
- **Após a automação ser concluída** - não antes de Claude fazer o trabalho
- **Após a construção da UI** - antes de declarar a fase completa
- **Antes do trabalho dependente** - decisões antes da implementação
- **Em pontos de integração** - após configurar serviços externos

**Posicionamento ruim:** Antes da automação ❌ | Muito frequente ❌ | Muito tarde (tarefas dependentes já precisavam do resultado) ❌
</writing_guidelines>

<examples>

### Exemplo 1: Configuração de Banco de Dados (Sem Checkpoint Necessário)

```xml
<task type="auto">
  <name>Create Upstash Redis database</name>
  <files>.env</files>
  <action>
    1. Run `upstash redis create myapp-cache --region us-east-1`
    2. Capture connection URL from output
    3. Write to .env: UPSTASH_REDIS_URL={url}
    4. Verify connection with test command
  </action>
  <verify>
    - upstash redis list shows database
    - .env contains UPSTASH_REDIS_URL
    - Test connection succeeds
  </verify>
  <done>Redis database created and configured</done>
</task>

<!-- SEM CHECKPOINT NECESSÁRIO - Claude automatizou tudo e verificou programaticamente -->
```

### Exemplo 2: Fluxo Completo de Auth (Checkpoint único ao final)

```xml
<task type="auto">
  <name>Create user schema</name>
  <files>src/db/schema.ts</files>
  <action>Define User, Session, Account tables with Drizzle ORM</action>
  <verify>npm run db:generate succeeds</verify>
</task>

<task type="auto">
  <name>Create auth API routes</name>
  <files>src/app/api/auth/[...nextauth]/route.ts</files>
  <action>Set up NextAuth with GitHub provider, JWT strategy</action>
  <verify>TypeScript compiles, no errors</verify>
</task>

<task type="auto">
  <name>Create login UI</name>
  <files>src/app/login/page.tsx, src/components/LoginButton.tsx</files>
  <action>Create login page with GitHub OAuth button</action>
  <verify>npm run build succeeds</verify>
</task>

<task type="auto">
  <name>Start dev server for auth testing</name>
  <action>Run `npm run dev` in background, wait for ready signal</action>
  <verify>fetch http://localhost:3000 returns 200</verify>
  <done>Dev server running at http://localhost:3000</done>
</task>

<!-- UM checkpoint ao final verifica o fluxo completo -->
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Complete authentication flow - dev server running at http://localhost:3000</what-built>
  <how-to-verify>
    1. Visit: http://localhost:3000/login
    2. Click "Sign in with GitHub"
    3. Complete GitHub OAuth flow
    4. Verify: Redirected to /dashboard, user name displayed
    5. Refresh page: Session persists
    6. Click logout: Session cleared
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```
</examples>

<anti_patterns>

### ❌ RUIM: Pedir ao usuário para iniciar o servidor de desenvolvimento

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard component</what-built>
  <how-to-verify>
    1. Run: npm run dev
    2. Visit: http://localhost:3000/dashboard
    3. Check layout is correct
  </how-to-verify>
</task>
```

**Por que é ruim:** Claude pode executar `npm run dev`. O usuário deve apenas visitar URLs, não executar comandos.

### ✅ BOM: Claude inicia o servidor, usuário visita

```xml
<task type="auto">
  <name>Start dev server</name>
  <action>Run `npm run dev` in background</action>
  <verify>fetch http://localhost:3000 returns 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard at http://localhost:3000/dashboard (server running)</what-built>
  <how-to-verify>
    Visit http://localhost:3000/dashboard and verify:
    1. Layout matches design
    2. No console errors
  </how-to-verify>
</task>
```

### ❌ RUIM: Pedir ao humano para implantar / ✅ BOM: Claude automatiza

```xml
<!-- RUIM: Pedir ao usuário para implantar via dashboard -->
<task type="checkpoint:human-action" gate="blocking">
  <action>Deploy to Vercel</action>
  <instructions>Visit vercel.com/new → Import repo → Click Deploy → Copy URL</instructions>
</task>

<!-- BOM: Claude implanta, usuário verifica -->
<task type="auto">
  <name>Deploy to Vercel</name>
  <action>Run `vercel --yes`. Capture URL.</action>
  <verify>vercel ls shows deployment, fetch returns 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Deployed to {url}</what-built>
  <how-to-verify>Visit {url}, check homepage loads</how-to-verify>
  <resume-signal>Type "approved"</resume-signal>
</task>
```

### ❌ RUIM: Muitos checkpoints / ✅ BOM: Checkpoint único

```xml
<!-- RUIM: Checkpoint após cada tarefa -->
<task type="auto">Create schema</task>
<task type="checkpoint:human-verify">Check schema</task>
<task type="auto">Create API route</task>
<task type="checkpoint:human-verify">Check API</task>
<task type="auto">Create UI form</task>
<task type="checkpoint:human-verify">Check form</task>

<!-- BOM: Um checkpoint ao final -->
<task type="auto">Create schema</task>
<task type="auto">Create API route</task>
<task type="auto">Create UI form</task>

<task type="checkpoint:human-verify">
  <what-built>Complete auth flow (schema + API + UI)</what-built>
  <how-to-verify>Test full flow: register, login, access protected page</how-to-verify>
  <resume-signal>Type "approved"</resume-signal>
</task>
```

### ❌ RUIM: Verificação vaga / ✅ BOM: Passos específicos

```xml
<!-- RUIM -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard</what-built>
  <how-to-verify>Check it works</how-to-verify>
</task>

<!-- BOM -->
<task type="checkpoint:human-verify">
  <what-built>Responsive dashboard - server running at http://localhost:3000</what-built>
  <how-to-verify>
    Visit http://localhost:3000/dashboard and verify:
    1. Desktop (>1024px): Sidebar visible, content area fills remaining space
    2. Tablet (768px): Sidebar collapses to icons
    3. Mobile (375px): Sidebar hidden, hamburger menu in header
    4. No horizontal scroll at any size
  </how-to-verify>
  <resume-signal>Type "approved" or describe layout issues</resume-signal>
</task>
```

### ❌ RUIM: Pedir ao usuário para executar comandos CLI

```xml
<task type="checkpoint:human-action">
  <action>Run database migrations</action>
  <instructions>Run: npx prisma migrate deploy && npx prisma db seed</instructions>
</task>
```

**Por que é ruim:** Claude pode executar esses comandos. O usuário nunca deve executar comandos CLI.

### ❌ RUIM: Pedir ao usuário para copiar valores entre serviços

```xml
<task type="checkpoint:human-action">
  <action>Configure webhook URL in Stripe</action>
  <instructions>Copy deployment URL → Stripe Dashboard → Webhooks → Add endpoint → Copy secret → Add to .env</instructions>
</task>
```

**Por que é ruim:** O Stripe tem uma API. Claude deve criar o webhook via API e escrever no .env diretamente.

</anti_patterns>

<type name="tdd-review">
## checkpoint:tdd-review (Somente Modo TDD)

**Quando:** Todas as ondas em uma fase são concluídas e `workflow.tdd_mode` está habilitado. Inserido pelo orquestrador execute-phase após `aggregate_results`.

**Propósito:** Revisão colaborativa da conformidade com o portão TDD em todos os planos `type: tdd` na fase. Consultivo — não bloqueia a execução.

**Use para:**
- Verificar sequência de commits RED/GREEN/REFACTOR para cada plano TDD
- Identificar violações de portão (commits RED ou GREEN ausentes)
- Revisar qualidade dos testes (testes falham pelo motivo certo)
- Confirmar implementações GREEN mínimas

**Estrutura:**
```xml
<task type="checkpoint:tdd-review" gate="advisory">
  <what-checked>TDD gate compliance for {count} plans in Phase {X}</what-checked>
  <gate-results>
    | Plan | RED | GREEN | REFACTOR | Status |
    |------|-----|-------|----------|--------|
    | {id} |  ✓  |   ✓   |    ✓     | Pass   |
  </gate-results>
  <violations>[List of gate violations, or "None"]</violations>
  <resume-signal>Review complete — proceed to phase verification</resume-signal>
</task>
```

**Comportamento em modo automático:** Quando `workflow._auto_chain_active` ou `workflow.auto_advance` é verdadeiro, o checkpoint de revisão TDD aprova automaticamente (portão consultivo — nunca bloqueia).
</type>

<summary>

Os checkpoints formalizam pontos de interação humana para verificação e decisões, não para trabalho manual.

**A regra de ouro:** Se Claude PODE automatizar, Claude DEVE automatizar.

**Prioridade dos checkpoints:**
1. **checkpoint:human-verify** (90%) - Claude automatizou tudo, humano confirma a correção visual/funcional
2. **checkpoint:decision** (9%) - Humano faz escolhas arquitetônicas/tecnológicas
3. **checkpoint:human-action** (1%) - Passos manuais verdadeiramente inevitáveis sem API/CLI

**Quando NÃO usar checkpoints:**
- Coisas que Claude pode verificar programaticamente (testes, builds)
- Operações de arquivo (Claude pode ler arquivos)
- Correção de código (testes e análise estática)
- Qualquer coisa automatizável via CLI/API
</summary>
