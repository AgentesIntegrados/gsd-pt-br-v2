# Template de Integrações Externas

Template para `.planning/codebase/INTEGRATIONS.md` - captura as dependências de serviços externos.

**Propósito:** Documentar com quais sistemas externos este codebase se comunica. Focado em "o que vive fora do nosso código do qual dependemos."

---

## Template do Arquivo

```markdown
# Integrações Externas

**Data da Análise:** [YYYY-MM-DD]

## APIs e Serviços Externos

**Processamento de Pagamentos:**
- [Serviço] - [Para que é usado: ex.: "cobrança por assinatura, pagamentos únicos"]
  - SDK/Cliente: [ex.: "pacote npm stripe v14.x"]
  - Auth: [ex.: "chave de API em variável de ambiente STRIPE_SECRET_KEY"]
  - Endpoints usados: [ex.: "sessões de checkout, webhooks"]

**E-mail/SMS:**
- [Serviço] - [Para que é usado: ex.: "e-mails transacionais"]
  - SDK/Cliente: [ex.: "@sendgrid/mail v8.x"]
  - Auth: [ex.: "chave de API em variável de ambiente SENDGRID_API_KEY"]
  - Templates: [ex.: "gerenciados no painel SendGrid"]

**APIs Externas:**
- [Serviço] - [Para que é usado]
  - Método de integração: [ex.: "API REST via fetch", "cliente GraphQL"]
  - Auth: [ex.: "token OAuth2 em variável de ambiente AUTH_TOKEN"]
  - Limites de taxa: [se aplicável]

## Armazenamento de Dados

**Bancos de Dados:**
- [Tipo/Provedor] - [ex.: "PostgreSQL no Supabase"]
  - Conexão: [ex.: "via variável de ambiente DATABASE_URL"]
  - Cliente: [ex.: "Prisma ORM v5.x"]
  - Migrações: [ex.: "prisma migrate em migrations/"]

**Armazenamento de Arquivos:**
- [Serviço] - [ex.: "AWS S3 para uploads de usuários"]
  - SDK/Cliente: [ex.: "@aws-sdk/client-s3"]
  - Auth: [ex.: "credenciais IAM em variáveis de ambiente AWS_*"]
  - Buckets: [ex.: "prod-uploads, dev-uploads"]

**Cache:**
- [Serviço] - [ex.: "Redis para armazenamento de sessão"]
  - Conexão: [ex.: "variável de ambiente REDIS_URL"]
  - Cliente: [ex.: "ioredis v5.x"]

## Autenticação e Identidade

**Provedor de Auth:**
- [Serviço] - [ex.: "Supabase Auth", "Auth0", "JWT customizado"]
  - Implementação: [ex.: "SDK cliente Supabase"]
  - Armazenamento de token: [ex.: "cookies httpOnly", "localStorage"]
  - Gerenciamento de sessão: [ex.: "tokens de refresh JWT"]

**Integrações OAuth:**
- [Provedor] - [ex.: "Google OAuth para login"]
  - Credenciais: [ex.: "GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET"]
  - Escopos: [ex.: "email, profile"]

## Monitoramento e Observabilidade

**Rastreamento de Erros:**
- [Serviço] - [ex.: "Sentry"]
  - DSN: [ex.: "variável de ambiente SENTRY_DSN"]
  - Rastreamento de release: [ex.: "via SENTRY_RELEASE"]

**Analytics:**
- [Serviço] - [ex.: "Mixpanel para analytics de produto"]
  - Token: [ex.: "variável de ambiente MIXPANEL_TOKEN"]
  - Eventos rastreados: [ex.: "ações do usuário, visualizações de página"]

**Logs:**
- [Serviço] - [ex.: "CloudWatch", "Datadog", "nenhum (apenas stdout)"]
  - Integração: [ex.: "AWS Lambda built-in"]

## CI/CD e Deploy

**Hospedagem:**
- [Plataforma] - [ex.: "Vercel", "AWS Lambda", "Docker no ECS"]
  - Deploy: [ex.: "automático no push para o branch main"]
  - Variáveis de ambiente: [ex.: "configuradas no painel Vercel"]

**Pipeline de CI:**
- [Serviço] - [ex.: "GitHub Actions"]
  - Workflows: [ex.: "test.yml, deploy.yml"]
  - Segredos: [ex.: "armazenados nos segredos do repositório GitHub"]

## Configuração de Ambiente

**Desenvolvimento:**
- Variáveis de ambiente necessárias: [Liste as variáveis críticas]
- Local dos segredos: [ex.: ".env.local (no .gitignore)", "cofre no 1Password"]
- Serviços mock/stub: [ex.: "modo de teste Stripe", "PostgreSQL local"]

**Staging:**
- Diferenças específicas do ambiente: [ex.: "usa conta Stripe de staging"]
- Dados: [ex.: "banco de dados de staging separado"]

**Produção:**
- Gerenciamento de segredos: [ex.: "variáveis de ambiente do Vercel"]
- Failover/redundância: [ex.: "replicação de BD multi-região"]

## Webhooks e Callbacks

**Entrada:**
- [Serviço] - [Endpoint: ex.: "/api/webhooks/stripe"]
  - Verificação: [ex.: "validação de assinatura via stripe.webhooks.constructEvent"]
  - Eventos: [ex.: "payment_intent.succeeded, customer.subscription.updated"]

**Saída:**
- [Serviço] - [O que aciona]
  - Endpoint: [ex.: "webhook de CRM externo no cadastro de usuário"]
  - Lógica de retry: [se aplicável]

---

*Auditoria de integrações: [data]*
*Atualize ao adicionar/remover serviços externos*
```

<good_examples>
```markdown
# Integrações Externas

**Data da Análise:** 2025-01-20

## APIs e Serviços Externos

**Processamento de Pagamentos:**
- Stripe - Cobrança por assinatura e pagamentos únicos de cursos
  - SDK/Cliente: pacote npm stripe v14.8
  - Auth: chave de API em variável de ambiente STRIPE_SECRET_KEY
  - Endpoints usados: sessões de checkout, portal do cliente, webhooks

**E-mail/SMS:**
- SendGrid - E-mails transacionais (recibos, redefinição de senha)
  - SDK/Cliente: @sendgrid/mail v8.1
  - Auth: chave de API em variável de ambiente SENDGRID_API_KEY
  - Templates: Gerenciados no painel SendGrid (IDs de template no código)

**APIs Externas:**
- API OpenAI - Geração de conteúdo de cursos
  - Método de integração: API REST via pacote npm openai v4.x
  - Auth: token Bearer em variável de ambiente OPENAI_API_KEY
  - Limites de taxa: 3.500 requisições/min (nível 3)

## Armazenamento de Dados

**Bancos de Dados:**
- PostgreSQL no Supabase - Armazenamento de dados principal
  - Conexão: via variável de ambiente DATABASE_URL
  - Cliente: Prisma ORM v5.8
  - Migrações: prisma migrate em prisma/migrations/

**Armazenamento de Arquivos:**
- Supabase Storage - Uploads de usuários (imagens de perfil, materiais do curso)
  - SDK/Cliente: @supabase/supabase-js v2.x
  - Auth: chave de role de serviço em SUPABASE_SERVICE_ROLE_KEY
  - Buckets: avatars (público), course-materials (privado)

**Cache:**
- Nenhum no momento (todas as consultas ao banco de dados, sem Redis)

## Autenticação e Identidade

**Provedor de Auth:**
- Supabase Auth - E-mail/senha + OAuth
  - Implementação: SDK cliente Supabase com gerenciamento de sessão no servidor
  - Armazenamento de token: cookies httpOnly via @supabase/ssr
  - Gerenciamento de sessão: tokens de refresh JWT gerenciados pelo Supabase

**Integrações OAuth:**
- Google OAuth - Login social
  - Credenciais: GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET (painel Supabase)
  - Escopos: email, profile

## Monitoramento e Observabilidade

**Rastreamento de Erros:**
- Sentry - Erros no servidor e no cliente
  - DSN: variável de ambiente SENTRY_DSN
  - Rastreamento de release: SHA do commit Git via SENTRY_RELEASE

**Analytics:**
- Nenhum (planejado: Mixpanel)

**Logs:**
- Logs do Vercel - apenas stdout/stderr
  - Retenção: 7 dias no plano Pro

## CI/CD e Deploy

**Hospedagem:**
- Vercel - Hospedagem do aplicativo Next.js
  - Deploy: Automático no push para o branch main
  - Variáveis de ambiente: Configuradas no painel Vercel (sincronizadas com .env.example)

**Pipeline de CI:**
- GitHub Actions - Testes e verificação de tipos
  - Workflows: .github/workflows/ci.yml
  - Segredos: Nenhum necessário (testes de repositório público apenas)

## Configuração de Ambiente

**Desenvolvimento:**
- Variáveis de ambiente necessárias: DATABASE_URL, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
- Local dos segredos: .env.local (no .gitignore), compartilhado pela equipe via cofre no 1Password
- Serviços mock/stub: modo de teste Stripe, projeto local de desenvolvimento Supabase

**Staging:**
- Usa projeto Supabase de staging separado
- Modo de teste Stripe
- Mesma conta Vercel, ambiente diferente

**Produção:**
- Gerenciamento de segredos: variáveis de ambiente do Vercel
- Banco de dados: projeto Supabase de produção com backups diários

## Webhooks e Callbacks

**Entrada:**
- Stripe - /api/webhooks/stripe
  - Verificação: Validação de assinatura via stripe.webhooks.constructEvent
  - Eventos: payment_intent.succeeded, customer.subscription.updated, customer.subscription.deleted

**Saída:**
- Nenhuma

---

*Auditoria de integrações: 2025-01-20*
*Atualize ao adicionar/remover serviços externos*
```
</good_examples>

<guidelines>
**O que pertence ao INTEGRATIONS.md:**
- Serviços externos com os quais o código se comunica
- Padrões de autenticação (onde os segredos ficam, não os segredos em si)
- SDKs e bibliotecas cliente usadas
- Nomes de variáveis de ambiente (não os valores)
- Endpoints de webhook e métodos de verificação
- Padrões de conexão ao banco de dados
- Locais de armazenamento de arquivos
- Serviços de monitoramento e logging

**O que NÃO pertence aqui:**
- Chaves de API ou segredos reais (NUNCA escreva estes)
- Arquitetura interna (isso é o ARCHITECTURE.md)
- Padrões de código (isso é o PATTERNS.md)
- Escolhas tecnológicas (isso é o STACK.md)
- Problemas de performance (isso é o CONCERNS.md)

**Ao preencher este template:**
- Verifique .env.example ou .env.template para variáveis de ambiente necessárias
- Procure imports de SDK (stripe, @sendgrid/mail, etc.)
- Verifique handlers de webhook em rotas/endpoints
- Anote onde os segredos são gerenciados (não os segredos em si)
- Documente diferenças específicas de ambiente (dev/staging/prod)
- Inclua padrões de auth para cada serviço

**Útil para planejamento de fase quando:**
- Adicionando novas integrações de serviços externos
- Depurando problemas de autenticação
- Entendendo o fluxo de dados fora do aplicativo
- Configurando novos ambientes
- Auditando dependências de terceiros
- Planejando falhas de serviço ou migrações

**Nota de segurança:**
Documente ONDE os segredos ficam (variáveis de ambiente, painel Vercel, 1Password), nunca O QUE são os segredos.
</guidelines>
