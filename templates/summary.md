# Template de Resumo

Template para `.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md` - documentação de conclusão de fase.

---

## Template do Arquivo

```markdown
---
phase: XX-name
plan: YY
subsystem: [categoria principal: auth, payments, ui, api, database, infra, testing, etc.]
tags: [tecnologia pesquisável: jwt, stripe, react, postgres, prisma]

# Grafo de dependência
requires:
  - phase: [fase anterior da qual este depende]
    provides: [o que aquela fase construiu que este usa]
provides:
  - [lista de itens que esta fase construiu/entregou]
affects: [lista de nomes de fases ou palavras-chave que precisarão deste contexto]

# Rastreamento de tecnologia
tech-stack:
  added: [bibliotecas/ferramentas adicionadas nesta fase]
  patterns: [padrões arquiteturais/de código estabelecidos]

key-files:
  created: [arquivos importantes criados]
  modified: [arquivos importantes modificados]

key-decisions:
  - "Decisão 1"
  - "Decisão 2"

patterns-established:
  - "Padrão 1: descrição"
  - "Padrão 2: descrição"

requirements-completed: []  # OBRIGATÓRIO — Copie TODOS os IDs de requisitos do campo `requirements` do frontmatter deste plano.

# Métricas
duration: Xmin
completed: YYYY-MM-DD
---

# Fase [X]: [Nome] - Resumo

**[Frase substantiva de uma linha descrevendo o resultado - NÃO "fase concluída" ou "implementação finalizada"]**

## Performance

- **Duração:** [tempo] (ex.: 23 min, 1h 15m)
- **Iniciado:** [timestamp ISO]
- **Concluído:** [timestamp ISO]
- **Tarefas:** [quantidade concluída]
- **Arquivos modificados:** [quantidade]

## Conquistas
- [Resultado mais importante]
- [Segunda conquista principal]
- [Terceira, se aplicável]

## Commits de Tarefas

Cada tarefa foi commitada atomicamente:

1. **Tarefa 1: [nome da tarefa]** - `abc123f` (feat/fix/test/refactor)
2. **Tarefa 2: [nome da tarefa]** - `def456g` (feat/fix/test/refactor)
3. **Tarefa 3: [nome da tarefa]** - `hij789k` (feat/fix/test/refactor)

**Metadados do plano:** `lmn012o` (docs: plano completo)

_Nota: Tarefas TDD podem ter múltiplos commits (test → feat → refactor)_

## Arquivos Criados/Modificados
- `path/to/file.ts` - O que faz
- `path/to/another.ts` - O que faz

## Decisões Tomadas
[Decisões-chave com breve justificativa, ou "Nenhum - seguido o plano conforme especificado"]

## Desvios do Plano

[Se sem desvios: "Nenhum - plano executado exatamente como escrito"]

[Se houve desvios:]

### Problemas Auto-corrigidos

**1. [Regra X - Categoria] Breve descrição**
- **Encontrado durante:** Tarefa [N] ([nome da tarefa])
- **Problema:** [O que estava errado]
- **Correção:** [O que foi feito]
- **Arquivos modificados:** [caminhos de arquivo]
- **Verificação:** [Como foi verificado]
- **Commitado em:** [hash] (parte do commit da tarefa)

[... repita para cada autocorreção ...]

---

**Total de desvios:** [N] auto-corrigidos ([detalhamento por regra])
**Impacto no plano:** [Breve avaliação - ex.: "Todas as autocorreções necessárias para correção/segurança. Sem expansão de escopo."]

## Problemas Encontrados
[Problemas e como foram resolvidos, ou "Nenhum"]

[Nota: "Desvios do Plano" documenta o trabalho não planejado que foi tratado automaticamente pelas regras de desvio. "Problemas Encontrados" documenta problemas durante o trabalho planejado que exigiram resolução de problemas.]

## Configuração do Usuário Necessária

[Se USER-SETUP.md foi gerado:]
**Serviços externos requerem configuração manual.** Veja [{phase}-USER-SETUP.md](./{phase}-USER-SETUP.md) para:
- Variáveis de ambiente a adicionar
- Etapas de configuração no painel
- Comandos de verificação

[Se não há USER-SETUP.md:]
Nenhum - nenhuma configuração de serviço externo necessária.

## Prontidão para Próxima Fase
[O que está pronto para a próxima fase]
[Quaisquer bloqueios ou preocupações]

---
*Fase: XX-name*
*Concluído em: [data]*
```

<frontmatter_guidance>
**Propósito:** Habilitar a montagem automática de contexto via grafo de dependência. O frontmatter torna os metadados do resumo legíveis por máquina para que o plan-phase possa escanear todos os resumos rapidamente e selecionar os relevantes com base nas dependências.

**Escaneamento rápido:** O frontmatter ocupa as primeiras ~25 linhas, barato de escanear em todos os resumos sem ler o conteúdo completo.

**Grafo de dependência:** `requires`/`provides`/`affects` criam links explícitos entre fases, habilitando o fechamento transitivo para seleção de contexto.

**Subsystem:** Categorização primária (auth, payments, ui, api, database, infra, testing) para detectar fases relacionadas.

**Tags:** Palavras-chave técnicas pesquisáveis (bibliotecas, frameworks, ferramentas) para consciência da stack tecnológica.

**Key-files:** Arquivos importantes para referências @context no PLAN.md.

**Patterns:** Convenções estabelecidas que fases futuras devem manter.

**Preenchimento:** O frontmatter é preenchido durante a criação do resumo no execute-plan.md. Veja `<step name="create_summary">` para orientação campo a campo.
</frontmatter_guidance>

<one_liner_rules>
A frase de uma linha DEVE ser substantiva:

**Boa:**
- "Auth JWT com rotação de refresh usando a biblioteca jose"
- "Schema Prisma com models User, Session e Product"
- "Dashboard com métricas em tempo real via Server-Sent Events"

**Ruim:**
- "Fase concluída"
- "Autenticação implementada"
- "Fundação finalizada"
- "Todas as tarefas concluídas"

A frase de uma linha deve dizer a alguém o que foi realmente entregue.
</one_liner_rules>

<example>
```markdown
# Fase 1: Resumo da Fundação

**Auth JWT com rotação de refresh usando a biblioteca jose, model Prisma de Usuário e middleware de API protegida**

## Performance

- **Duração:** 28 min
- **Iniciado:** 2025-01-15T14:22:10Z
- **Concluído:** 2025-01-15T14:50:33Z
- **Tarefas:** 5
- **Arquivos modificados:** 8

## Conquistas
- Model de usuário com auth de e-mail/senha
- Endpoints de login/logout com cookies JWT httpOnly
- Middleware de rota protegida verificando validade do token
- Rotação de token de refresh a cada requisição

## Arquivos Criados/Modificados
- `prisma/schema.prisma` - Models User e Session
- `src/app/api/auth/login/route.ts` - Endpoint de login
- `src/app/api/auth/logout/route.ts` - Endpoint de logout
- `src/middleware.ts` - Verificações de rota protegida
- `src/lib/auth.ts` - Helpers JWT usando jose

## Decisões Tomadas
- Usado jose em vez de jsonwebtoken (nativo ESM, compatível com Edge)
- Tokens de acesso de 15 min com tokens de refresh de 7 dias
- Armazenando tokens de refresh no banco de dados para capacidade de revogação

## Desvios do Plano

### Problemas Auto-corrigidos

**1. [Regra 2 - Crítico Ausente] Adicionado hash de senha com bcrypt**
- **Encontrado durante:** Tarefa 2 (Implementação do endpoint de login)
- **Problema:** O plano não especificou hash de senha - armazenar em texto simples seria falha de segurança crítica
- **Correção:** Adicionado hash bcrypt no registro, comparação no login com salt rounds 10
- **Arquivos modificados:** src/app/api/auth/login/route.ts, src/lib/auth.ts
- **Verificação:** Teste de hash de senha passa, texto simples nunca armazenado
- **Commitado em:** abc123f (commit da Tarefa 2)

**2. [Regra 3 - Bloqueante] Instalada dependência jose ausente**
- **Encontrado durante:** Tarefa 4 (Geração de token JWT)
- **Problema:** Pacote jose não estava no package.json, import falhando
- **Correção:** Executado `npm install jose`
- **Arquivos modificados:** package.json, package-lock.json
- **Verificação:** Import bem-sucedido, build passa
- **Commitado em:** def456g (commit da Tarefa 4)

---

**Total de desvios:** 2 auto-corrigidos (1 crítico ausente, 1 bloqueante)
**Impacto no plano:** Ambas as autocorreções essenciais para segurança e funcionalidade. Sem expansão de escopo.

## Problemas Encontrados
- Import CommonJS do jsonwebtoken falhou no Edge runtime - trocado para jose (mudança de biblioteca planejada, funcionou como esperado)

## Prontidão para Próxima Fase
- Fundação de auth completa, pronta para desenvolvimento de funcionalidades
- Endpoint de registro de usuário necessário antes do lançamento público

---
*Fase: 01-foundation*
*Concluído em: 2025-01-15*
```
</example>

<guidelines>
**Frontmatter:** OBRIGATÓRIO - complete todos os campos. Habilita a montagem automática de contexto para planejamento futuro.

**Frase de uma linha:** Deve ser substantiva. "Auth JWT com rotação de refresh usando a biblioteca jose" e não "Autenticação implementada".

**Seção de Decisões:**
- Decisões-chave tomadas durante a execução com justificativa
- Extraídas para o contexto acumulado do STATE.md
- Use "Nenhum - seguido o plano conforme especificado" se não houver desvios

**Após a criação:** STATE.md atualizado com posição, decisões, problemas.
</guidelines>
