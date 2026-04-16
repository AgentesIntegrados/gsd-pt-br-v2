# Exemplos Estendidos do Executor

> Arquivo de referência para o agente gsd-executor. Carregado sob demanda via referência `@`.
> Para janelas de contexto menores que 200K, este conteúdo é removido do prompt do agente e fica disponível aqui para carregamento sob demanda.

## Exemplos de Regras de Desvio

### Regra 1 — Corrigir bugs automaticamente

**Exemplos de acionadores da Regra 1:**
- Consultas erradas retornando dados incorretos
- Erros de lógica em condicionais
- Erros de tipo e incompatibilidades de tipo
- Exceções de ponteiro nulo / acesso indefinido
- Validação quebrada (aceita entrada inválida)
- Vulnerabilidades de segurança (XSS, injeção de SQL)
- Condições de corrida em código assíncrono
- Vazamentos de memória de recursos não limpos

### Regra 2 — Adicionar automaticamente funcionalidades críticas ausentes

**Exemplos de acionadores da Regra 2:**
- Tratamento de erros ausente (rejeições de promise não tratadas, sem try/catch em E/S)
- Sem validação de entrada em endpoints voltados ao usuário
- Verificações de nulo ausentes antes do acesso a propriedades
- Sem auth em rotas protegidas
- Verificações de autorização ausentes (usuário pode acessar dados de outros usuários)
- Sem configuração de CSRF/CORS
- Sem limitação de taxa em endpoints públicos
- Índices de BD ausentes em colunas consultadas frequentemente
- Sem registro de erros (falhas silenciosamente engolidas)

### Regra 3 — Corrigir automaticamente problemas bloqueantes

**Exemplos de acionadores da Regra 3:**
- Dependência ausente não está em package.json
- Tipos errados impedindo a compilação
- Importações quebradas (caminho errado, nome de exportação errado)
- Variável de ambiente ausente necessária em tempo de execução
- Erro de conexão com BD (URL errada, credenciais ausentes)
- Erro de configuração de build (ponto de entrada errado, loader ausente)
- Arquivo referenciado ausente (importação aponta para módulo inexistente)
- Dependência circular impedindo o carregamento do módulo

### Regra 4 — Perguntar sobre mudanças arquitetônicas

**Exemplos de acionadores da Regra 4:**
- Nova tabela no BD (não apenas adicionar uma coluna)
- Mudanças maiores de esquema (renomear tabelas, alterar relacionamentos)
- Nova camada de serviço (adicionar uma fila, cache ou barramento de mensagens)
- Trocar bibliotecas/frameworks (ex.: substituir Express por Fastify)
- Mudar a abordagem de auth (trocar de sessão para JWT)
- Nova infraestrutura (adicionar Redis, S3, etc.)
- Mudanças incompatíveis de API (remover ou renomear endpoints)

## Guia de Decisão para Casos Extremos

| Cenário | Regra | Justificativa |
|----------|------|-----------|
| Validação ausente na entrada | Regra 2 | Requisito de segurança |
| Falha em entrada nula | Regra 1 | Bug — comportamento incorreto |
| Necessidade de nova tabela no banco de dados | Regra 4 | Decisão arquitetônica |
| Necessidade de nova coluna em tabela existente | Regra 1 ou 2 | Depende do contexto |
| Avisos de linting pré-existentes | Fora do escopo | Não causado pela tarefa atual |
| Falhas de teste não relacionadas | Fora do escopo | Não causado pela tarefa atual |

**Heurística de decisão:** "Isso afeta a correção, segurança ou capacidade de concluir a tarefa atual?"
- SIM → Regras 1-3 (corrigir automaticamente)
- TALVEZ → Regra 4 (perguntar ao usuário)
- NÃO → Fora do escopo (registrar em deferred-items.md)

## Exemplos de Checkpoints

### Bom posicionamento de checkpoint

```xml
<!-- Automatize tudo, depois verifique ao final -->
<task type="auto">Create database schema</task>
<task type="auto">Create API endpoints</task>
<task type="auto">Create UI components</task>
<task type="checkpoint:human-verify">
  <what-built>Complete auth flow (schema + API + UI)</what-built>
  <how-to-verify>
    1. Visit http://localhost:3000/register
    2. Create account with test@example.com
    3. Log in with those credentials
    4. Verify dashboard loads with user name
  </how-to-verify>
</task>
```

### Mau posicionamento de checkpoint

```xml
<!-- Checkpoints demais — causa fadiga de verificação -->
<task type="auto">Create schema</task>
<task type="checkpoint:human-verify">Check schema</task>
<task type="auto">Create API</task>
<task type="checkpoint:human-verify">Check API</task>
<task type="auto">Create UI</task>
<task type="checkpoint:human-verify">Check UI</task>
```

### Tratamento de portão de autenticação

Quando um erro de auth ocorre durante a execução de `type="auto"`:
1. Reconheça como um portão de auth (não um bug) — indicadores: "Not authenticated", "401", "403", "Please run X login"
2. PARE a tarefa atual
3. Retorne um `checkpoint:human-action` com passos exatos de auth
4. No SUMMARY.md, documente portões de auth como fluxo normal, não como desvios
