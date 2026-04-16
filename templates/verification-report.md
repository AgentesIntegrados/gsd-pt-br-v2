# Template de Relatório de Verificação

Template para `.planning/phases/XX-name/{phase_num}-VERIFICATION.md` — resultados de verificação do objetivo da fase.

---

## Template do Arquivo

```markdown
---
phase: XX-name
verified: YYYY-MM-DDTHH:MM:SSZ
status: passed | gaps_found | human_needed
score: N/M must-haves verified
---

# Fase {X}: {Nome} - Relatório de Verificação

**Objetivo da Fase:** {objetivo do ROADMAP.md}
**Verificado em:** {timestamp}
**Status:** {passed | gaps_found | human_needed}

## Atingimento do Objetivo

### Verdades Observáveis

| # | Verdade | Status | Evidência |
|---|---------|--------|-----------|
| 1 | {verdade dos must_haves} | ✓ VERIFICADO | {o que confirmou} |
| 2 | {verdade dos must_haves} | ✗ FALHOU | {o que está errado} |
| 3 | {verdade dos must_haves} | ? INCERTO | {por que não é possível verificar} |

**Pontuação:** {N}/{M} verdades verificadas

### Artefatos Necessários

| Artefato | Esperado | Status | Detalhes |
|----------|----------|--------|---------|
| `src/components/Chat.tsx` | Componente de lista de mensagens | ✓ EXISTE + SUBSTANTIVO | Exporta ChatList, renderiza Message[], sem stubs |
| `src/app/api/chat/route.ts` | CRUD de mensagens | ✗ STUB | Arquivo existe, mas POST retorna placeholder |
| `prisma/schema.prisma` | Model de mensagem | ✓ EXISTE + SUBSTANTIVO | Model definido com todos os campos |

**Artefatos:** {N}/{M} verificados

### Verificação de Links-Chave

| De | Para | Via | Status | Detalhes |
|----|------|-----|--------|---------|
| Chat.tsx | /api/chat | fetch no useEffect | ✓ CONECTADO | Linha 23: `fetch('/api/chat')` com tratamento de resposta |
| ChatInput | /api/chat POST | handler onSubmit | ✗ NÃO CONECTADO | onSubmit apenas chama console.log |
| /api/chat POST | banco de dados | prisma.message.create | ✗ NÃO CONECTADO | Retorna resposta fixada, sem chamada ao BD |

**Ligações:** {N}/{M} conexões verificadas

## Cobertura de Requisitos

| Requisito | Status | Problema Bloqueante |
|-----------|--------|---------------------|
| {REQ-01}: {descrição} | ✓ SATISFEITO | - |
| {REQ-02}: {descrição} | ✗ BLOQUEADO | Rota de API é um stub |
| {REQ-03}: {descrição} | ? REQUER HUMANO | Não é possível verificar WebSocket programaticamente |

**Cobertura:** {N}/{M} requisitos satisfeitos

## Anti-Padrões Encontrados

| Arquivo | Linha | Padrão | Severidade | Impacto |
|---------|-------|--------|------------|---------|
| src/app/api/chat/route.ts | 12 | `// TODO: implement` | ⚠️ Aviso | Indica incompleto |
| src/components/Chat.tsx | 45 | `return <div>Placeholder</div>` | 🛑 Bloqueante | Não renderiza conteúdo |
| src/hooks/useChat.ts | - | Arquivo ausente | 🛑 Bloqueante | Hook esperado não existe |

**Anti-padrões:** {N} encontrados ({blockers} bloqueantes, {warnings} avisos)

## Verificação Humana Necessária

{Se nenhuma verificação humana for necessária:}
Nenhuma — todos os itens verificáveis foram checados programaticamente.

{Se a verificação humana for necessária:}

### 1. {Nome do Teste}
**Teste:** {O que fazer}
**Esperado:** {O que deve acontecer}
**Por que humano:** {Por que não é possível verificar programaticamente}

### 2. {Nome do Teste}
**Teste:** {O que fazer}
**Esperado:** {O que deve acontecer}
**Por que humano:** {Por que não é possível verificar programaticamente}

## Resumo das Lacunas

{Se sem lacunas:}
**Nenhuma lacuna encontrada.** Objetivo da fase atingido. Pronto para prosseguir.

{Se houver lacunas:}

### Lacunas Críticas (Bloqueiam o Progresso)

1. **{Nome da lacuna}**
   - Ausente: {o que está faltando}
   - Impacto: {por que isso bloqueia o objetivo}
   - Correção: {o que precisa acontecer}

2. **{Nome da lacuna}**
   - Ausente: {o que está faltando}
   - Impacto: {por que isso bloqueia o objetivo}
   - Correção: {o que precisa acontecer}

### Lacunas Não Críticas (Podem ser Adiadas)

1. **{Nome da lacuna}**
   - Problema: {o que está errado}
   - Impacto: {impacto limitado porque...}
   - Recomendação: {corrigir agora ou adiar}

## Planos de Correção Recomendados

{Se houver lacunas, gere recomendações de planos de correção:}

### {phase}-{next}-PLAN.md: {Nome da Correção}

**Objetivo:** {O que isso corrige}

**Tarefas:**
1. {Tarefa para corrigir lacuna 1}
2. {Tarefa para corrigir lacuna 2}
3. {Tarefa de verificação}

**Escopo estimado:** {Pequeno / Médio}

---

### {phase}-{next+1}-PLAN.md: {Nome da Correção}

**Objetivo:** {O que isso corrige}

**Tarefas:**
1. {Tarefa}
2. {Tarefa}

**Escopo estimado:** {Pequeno / Médio}

---

## Metadados de Verificação

**Abordagem de verificação:** Retroativa a partir do objetivo (derivada do objetivo da fase)
**Fonte dos must-haves:** {frontmatter do PLAN.md | derivado do objetivo do ROADMAP.md}
**Verificações automatizadas:** {N} aprovadas, {M} falharam
**Verificações humanas necessárias:** {N}
**Tempo total de verificação:** {duração}

---
*Verificado em: {timestamp}*
*Verificador: Claude (subagente)*
```

---

## Diretrizes

**Valores de status:**
- `passed` — Todos os must-haves verificados, sem bloqueios
- `gaps_found` — Uma ou mais lacunas críticas encontradas
- `human_needed` — Verificações automatizadas passam, mas verificação humana é necessária

**Tipos de evidência:**
- Para EXISTS: "Arquivo no caminho, exporta X"
- Para SUBSTANTIVE: "N linhas, tem padrões X, Y, Z"
- Para WIRED: "Linha N: código que conecta A a B"
- Para FAILED: "Ausente porque X" ou "Stub porque Y"

**Níveis de severidade:**
- 🛑 Bloqueante: Impede o atingimento do objetivo, deve corrigir
- ⚠️ Aviso: Indica incompleto, mas não bloqueia
- ℹ️ Informativo: Notável, mas não problemático

**Geração de planos de correção:**
- Gere apenas se gaps_found
- Agrupe correções relacionadas em planos únicos
- Limite a 2-3 tarefas por plano
- Inclua tarefa de verificação em cada plano

---

## Exemplo

```markdown
---
phase: 03-chat
verified: 2025-01-15T14:30:00Z
status: gaps_found
score: 2/5 must-haves verified
---

# Fase 3: Relatório de Verificação da Interface de Chat

**Objetivo da Fase:** Interface de chat funcional onde usuários podem enviar e receber mensagens
**Verificado em:** 2025-01-15T14:30:00Z
**Status:** gaps_found

## Atingimento do Objetivo

### Verdades Observáveis

| # | Verdade | Status | Evidência |
|---|---------|--------|-----------|
| 1 | Usuário consegue ver mensagens existentes | ✗ FALHOU | Componente renderiza placeholder, não dados de mensagens |
| 2 | Usuário consegue digitar uma mensagem | ✓ VERIFICADO | Campo de entrada existe com handler onChange |
| 3 | Usuário consegue enviar uma mensagem | ✗ FALHOU | Handler onSubmit é apenas console.log |
| 4 | Mensagem enviada aparece na lista | ✗ FALHOU | Nenhuma atualização de estado após o envio |
| 5 | Mensagens persistem após atualização | ? INCERTO | Não é possível verificar - envio não funciona |

**Pontuação:** 1/5 verdades verificadas

### Artefatos Necessários

| Artefato | Esperado | Status | Detalhes |
|----------|----------|--------|---------|
| `src/components/Chat.tsx` | Componente de lista de mensagens | ✗ STUB | Retorna `<div>Chat will be here</div>` |
| `src/components/ChatInput.tsx` | Entrada de mensagem | ✓ EXISTE + SUBSTANTIVO | Formulário com input, botão de envio, handlers |
| `src/app/api/chat/route.ts` | CRUD de mensagens | ✗ STUB | GET retorna [], POST retorna { ok: true } |
| `prisma/schema.prisma` | Model de mensagem | ✓ EXISTE + SUBSTANTIVO | Model Message com id, content, userId, createdAt |

**Artefatos:** 2/4 verificados

### Verificação de Links-Chave

| De | Para | Via | Status | Detalhes |
|----|------|-----|--------|---------|
| Chat.tsx | /api/chat GET | fetch | ✗ NÃO CONECTADO | Nenhuma chamada fetch no componente |
| ChatInput | /api/chat POST | onSubmit | ✗ NÃO CONECTADO | Handler apenas registra, não faz fetch |
| /api/chat GET | banco de dados | prisma.message.findMany | ✗ NÃO CONECTADO | Retorna [] fixado |
| /api/chat POST | banco de dados | prisma.message.create | ✗ NÃO CONECTADO | Retorna { ok: true }, sem chamada ao BD |

**Ligações:** 0/4 conexões verificadas

## Cobertura de Requisitos

| Requisito | Status | Problema Bloqueante |
|-----------|--------|---------------------|
| CHAT-01: Usuário consegue enviar mensagem | ✗ BLOQUEADO | API POST é um stub |
| CHAT-02: Usuário consegue visualizar mensagens | ✗ BLOQUEADO | Componente é um placeholder |
| CHAT-03: Mensagens persistem | ✗ BLOQUEADO | Sem integração com banco de dados |

**Cobertura:** 0/3 requisitos satisfeitos

## Anti-Padrões Encontrados

| Arquivo | Linha | Padrão | Severidade | Impacto |
|---------|-------|--------|------------|---------|
| src/components/Chat.tsx | 8 | `<div>Chat will be here</div>` | 🛑 Bloqueante | Nenhum conteúdo real |
| src/app/api/chat/route.ts | 5 | `return Response.json([])` | 🛑 Bloqueante | Vazio fixado |
| src/app/api/chat/route.ts | 12 | `// TODO: save to database` | ⚠️ Aviso | Incompleto |

**Anti-padrões:** 3 encontrados (2 bloqueantes, 1 aviso)

## Verificação Humana Necessária

Nenhuma necessária até que as lacunas automatizadas sejam corrigidas.

## Resumo das Lacunas

### Lacunas Críticas (Bloqueiam o Progresso)

1. **Componente Chat é um placeholder**
   - Ausente: Renderização real da lista de mensagens
   - Impacto: Usuários veem "Chat will be here" em vez das mensagens
   - Correção: Implementar Chat.tsx para buscar e renderizar mensagens

2. **Rotas de API são stubs**
   - Ausente: Integração com banco de dados em GET e POST
   - Impacto: Sem persistência de dados, sem funcionalidade real
   - Correção: Conectar chamadas prisma nos handlers de rota

3. **Sem ligação entre frontend e backend**
   - Ausente: Chamadas fetch nos componentes
   - Impacto: Mesmo que a API funcionasse, a UI não a chamaria
   - Correção: Adicionar fetch no useEffect do Chat, fetch no onSubmit do ChatInput

## Planos de Correção Recomendados

### 03-04-PLAN.md: Implementar API de Chat

**Objetivo:** Conectar rotas de API ao banco de dados

**Tarefas:**
1. Implementar GET /api/chat com prisma.message.findMany
2. Implementar POST /api/chat com prisma.message.create
3. Verificar: API retorna dados reais, POST cria registros

**Escopo estimado:** Pequeno

---

### 03-05-PLAN.md: Implementar UI de Chat

**Objetivo:** Conectar componente Chat à API

**Tarefas:**
1. Implementar Chat.tsx com fetch via useEffect e renderização de mensagens
2. Conectar onSubmit do ChatInput ao POST /api/chat
3. Verificar: Mensagens são exibidas, novas mensagens aparecem após o envio

**Escopo estimado:** Pequeno

---

## Metadados de Verificação

**Abordagem de verificação:** Retroativa a partir do objetivo (derivada do objetivo da fase)
**Fonte dos must-haves:** frontmatter de 03-01-PLAN.md
**Verificações automatizadas:** 2 aprovadas, 8 falharam
**Verificações humanas necessárias:** 0 (bloqueadas por falhas automatizadas)
**Tempo total de verificação:** 2 min

---
*Verificado em: 2025-01-15T14:30:00Z*
*Verificador: Claude (subagente)*
```
