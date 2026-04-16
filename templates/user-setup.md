# Template de Configuração do Usuário

Template para `.planning/phases/XX-name/{phase}-USER-SETUP.md` - configuração exigida por humanos que Claude não pode automatizar.

**Propósito:** Documentar tarefas de configuração que literalmente requerem ação humana - criação de conta, configuração no painel, recuperação de segredos. Claude automatiza tudo o que for possível; este arquivo captura apenas o que resta.

---

## Template do Arquivo

```markdown
# Fase {X}: Configuração do Usuário Necessária

**Gerado em:** [YYYY-MM-DD]
**Fase:** {phase-name}
**Status:** Incompleto

Conclua estes itens para que a integração funcione. Claude automatizou tudo o que foi possível; estes itens requerem acesso humano a painéis/contas externos.

## Variáveis de Ambiente

| Status | Variável | Fonte | Adicionar em |
|--------|----------|-------|--------------|
| [ ] | `ENV_VAR_NAME` | [Painel do Serviço → Caminho → Para → Valor] | `.env.local` |
| [ ] | `ANOTHER_VAR` | [Painel do Serviço → Caminho → Para → Valor] | `.env.local` |

## Configuração de Conta

[Apenas se a criação de nova conta for necessária]

- [ ] **Criar conta no [Serviço]**
  - URL: [URL de cadastro]
  - Pule se: Já possui conta

## Configuração no Painel

[Apenas se a configuração no painel for necessária]

- [ ] **[Tarefa de configuração]**
  - Local: [Painel do Serviço → Caminho → Para → Configuração]
  - Defina como: [Valor ou configuração necessária]
  - Observações: [Quaisquer detalhes importantes]

## Verificação

Após concluir a configuração, verifique com:

```bash
# [Comandos de verificação]
```

Resultados esperados:
- [Como o sucesso se parece]

---

**Quando todos os itens estiverem concluídos:** Marque o status como "Completo" no topo do arquivo.
```

---

## Quando Gerar

Gere `{phase}-USER-SETUP.md` quando o frontmatter do plano contiver o campo `user_setup`.

**Gatilho:** `user_setup` existe no frontmatter do PLAN.md e tem itens.

**Local:** Mesmo diretório que o PLAN.md e o SUMMARY.md.

**Momento:** Gerado durante o execute-plan.md após as tarefas serem concluídas, antes da criação do SUMMARY.md.

---

## Schema do Frontmatter

No PLAN.md, `user_setup` declara a configuração exigida por humanos:

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
        details: "URL: https://[seu-domínio]/api/webhooks/stripe, Eventos: checkout.session.completed, customer.subscription.*"
    local_dev:
      - "Execute: stripe listen --forward-to localhost:3000/api/webhooks/stripe"
      - "Use o segredo de webhook da saída da CLI para testes locais"
```

---

## A Regra de Automação em Primeiro Lugar

**USER-SETUP.md contém APENAS o que Claude literalmente não pode fazer.**

| Claude PODE Fazer (não em USER-SETUP) | Claude NÃO PODE Fazer (→ USER-SETUP) |
|---------------------------------------|--------------------------------------|
| `npm install stripe` | Criar conta Stripe |
| Escrever código do handler de webhook | Obter chaves de API do painel |
| Criar estrutura do arquivo `.env.local` | Copiar valores de segredos reais |
| Executar `stripe listen` | Autenticar a CLI do Stripe (OAuth no navegador) |
| Configurar package.json | Acessar painéis de serviços externos |
| Escrever qualquer código | Recuperar segredos de sistemas de terceiros |

**O teste:** "Isso requer um humano em um navegador, acessando uma conta que Claude não tem credenciais?"
- Sim → USER-SETUP.md
- Não → Claude faz automaticamente

---

## Exemplos por Serviço

<stripe_example>
```markdown
# Fase 10: Configuração do Usuário Necessária

**Gerado em:** 2025-01-14
**Fase:** 10-monetization
**Status:** Incompleto

Conclua estes itens para que a integração com Stripe funcione.

## Variáveis de Ambiente

| Status | Variável | Fonte | Adicionar em |
|--------|----------|-------|--------------|
| [ ] | `STRIPE_SECRET_KEY` | Painel Stripe → Desenvolvedores → Chaves de API → Chave secreta | `.env.local` |
| [ ] | `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Painel Stripe → Desenvolvedores → Chaves de API → Chave publicável | `.env.local` |
| [ ] | `STRIPE_WEBHOOK_SECRET` | Painel Stripe → Desenvolvedores → Webhooks → [endpoint] → Segredo de assinatura | `.env.local` |

## Configuração de Conta

- [ ] **Criar conta Stripe** (se necessário)
  - URL: https://dashboard.stripe.com/register
  - Pule se: Já possui conta Stripe

## Configuração no Painel

- [ ] **Criar endpoint de webhook**
  - Local: Painel Stripe → Desenvolvedores → Webhooks → Adicionar endpoint
  - URL do endpoint: `https://[seu-domínio]/api/webhooks/stripe`
  - Eventos a enviar:
    - `checkout.session.completed`
    - `customer.subscription.created`
    - `customer.subscription.updated`
    - `customer.subscription.deleted`

- [ ] **Criar produtos e preços** (se usando níveis de assinatura)
  - Local: Painel Stripe → Produtos → Adicionar produto
  - Crie cada nível de assinatura
  - Copie os IDs de Preço para:
    - `STRIPE_STARTER_PRICE_ID`
    - `STRIPE_PRO_PRICE_ID`

## Desenvolvimento Local

Para testes de webhook locais:
```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```
Use o segredo de assinatura do webhook da saída da CLI (começa com `whsec_`).

## Verificação

Após concluir a configuração:

```bash
# Verificar se as variáveis de ambiente estão definidas
grep STRIPE .env.local

# Verificar se o build passa
npm run build

# Testar endpoint de webhook (deve retornar 400 assinatura inválida, não 500 crash)
curl -X POST http://localhost:3000/api/webhooks/stripe \
  -H "Content-Type: application/json" \
  -d '{}'
```

Esperado: Build passa, webhook retorna 400 (validação de assinatura funcionando).

---

**Quando todos os itens estiverem concluídos:** Marque o status como "Completo" no topo do arquivo.
```
</stripe_example>

<supabase_example>
```markdown
# Fase 2: Configuração do Usuário Necessária

**Gerado em:** 2025-01-14
**Fase:** 02-authentication
**Status:** Incompleto

Conclua estes itens para que o Supabase Auth funcione.

## Variáveis de Ambiente

| Status | Variável | Fonte | Adicionar em |
|--------|----------|-------|--------------|
| [ ] | `NEXT_PUBLIC_SUPABASE_URL` | Painel Supabase → Configurações → API → URL do Projeto | `.env.local` |
| [ ] | `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Painel Supabase → Configurações → API → anon public | `.env.local` |
| [ ] | `SUPABASE_SERVICE_ROLE_KEY` | Painel Supabase → Configurações → API → service_role | `.env.local` |

## Configuração de Conta

- [ ] **Criar projeto Supabase**
  - URL: https://supabase.com/dashboard/new
  - Pule se: Já possui projeto para este aplicativo

## Configuração no Painel

- [ ] **Habilitar Auth por E-mail**
  - Local: Painel Supabase → Autenticação → Provedores
  - Habilitar: Provedor de e-mail
  - Configurar: Confirmar e-mail (ligado/desligado conforme preferência)

- [ ] **Configurar provedores OAuth** (se usando login social)
  - Local: Painel Supabase → Autenticação → Provedores
  - Para Google: Adicionar ID do Cliente e Segredo do Google Cloud Console
  - Para GitHub: Adicionar ID do Cliente e Segredo dos OAuth Apps do GitHub

## Verificação

Após concluir a configuração:

```bash
# Verificar variáveis de ambiente
grep SUPABASE .env.local

# Verificar conexão (execute no diretório do projeto)
npx supabase status
```

---

**Quando todos os itens estiverem concluídos:** Marque o status como "Completo" no topo do arquivo.
```
</supabase_example>

<sendgrid_example>
```markdown
# Fase 5: Configuração do Usuário Necessária

**Gerado em:** 2025-01-14
**Fase:** 05-notifications
**Status:** Incompleto

Conclua estes itens para que o e-mail SendGrid funcione.

## Variáveis de Ambiente

| Status | Variável | Fonte | Adicionar em |
|--------|----------|-------|--------------|
| [ ] | `SENDGRID_API_KEY` | Painel SendGrid → Configurações → Chaves de API → Criar Chave de API | `.env.local` |
| [ ] | `SENDGRID_FROM_EMAIL` | Seu endereço de e-mail de remetente verificado | `.env.local` |

## Configuração de Conta

- [ ] **Criar conta SendGrid**
  - URL: https://signup.sendgrid.com/
  - Pule se: Já possui conta

## Configuração no Painel

- [ ] **Verificar identidade do remetente**
  - Local: Painel SendGrid → Configurações → Autenticação do Remetente
  - Opção 1: Verificação de Remetente Único (rápido, para desenvolvimento)
  - Opção 2: Autenticação de Domínio (produção)

- [ ] **Criar Chave de API**
  - Local: Painel SendGrid → Configurações → Chaves de API → Criar Chave de API
  - Permissão: Acesso Restrito → Envio de E-mail (Acesso Total)
  - Copie a chave imediatamente (exibida apenas uma vez)

## Verificação

Após concluir a configuração:

```bash
# Verificar variável de ambiente
grep SENDGRID .env.local

# Testar envio de e-mail (substitua pelo seu e-mail de teste)
curl -X POST http://localhost:3000/api/test-email \
  -H "Content-Type: application/json" \
  -d '{"to": "seu@email.com"}'
```

---

**Quando todos os itens estiverem concluídos:** Marque o status como "Completo" no topo do arquivo.
```
</sendgrid_example>

---

## Diretrizes

**Nunca inclua:** Valores reais de segredos. Etapas que Claude pode automatizar (instalações de pacotes, alterações de código).

**Nomenclatura:** `{phase}-USER-SETUP.md` corresponde ao padrão de número de fase.
**Rastreamento de status:** O usuário marca os checkboxes e atualiza a linha de status quando concluído.
**Pesquisabilidade:** `grep -r "USER-SETUP" .planning/` encontra todas as fases com requisitos de usuário.
