# Template de Requisitos

Template para `.planning/REQUIREMENTS.md` — requisitos verificáveis que definem "concluído".

<template>

```markdown
# Requisitos: [Nome do Projeto]

**Definidos em:** [data]
**Valor Principal:** [do PROJECT.md]

## Requisitos v1

Requisitos para o lançamento inicial. Cada um mapeia para fases do roadmap.

### Autenticação

- [ ] **AUTH-01**: Usuário pode se cadastrar com e-mail e senha
- [ ] **AUTH-02**: Usuário recebe verificação por e-mail após o cadastro
- [ ] **AUTH-03**: Usuário pode redefinir senha via link por e-mail
- [ ] **AUTH-04**: Sessão do usuário persiste após atualização do navegador

### [Categoria 2]

- [ ] **[CAT]-01**: [Descrição do requisito]
- [ ] **[CAT]-02**: [Descrição do requisito]
- [ ] **[CAT]-03**: [Descrição do requisito]

### [Categoria 3]

- [ ] **[CAT]-01**: [Descrição do requisito]
- [ ] **[CAT]-02**: [Descrição do requisito]

## Requisitos v2

Adiados para versão futura. Rastreados, mas não no roadmap atual.

### [Categoria]

- **[CAT]-01**: [Descrição do requisito]
- **[CAT]-02**: [Descrição do requisito]

## Fora do Escopo

Explicitamente excluído. Documentado para evitar expansão de escopo.

| Funcionalidade | Motivo |
|----------------|--------|
| [Funcionalidade] | [Por que excluída] |
| [Funcionalidade] | [Por que excluída] |

## Rastreabilidade

Quais fases cobrem quais requisitos. Atualizado durante a criação do roadmap.

| Requisito | Fase | Status |
|-----------|------|--------|
| AUTH-01 | Fase 1 | Pendente |
| AUTH-02 | Fase 1 | Pendente |
| AUTH-03 | Fase 1 | Pendente |
| AUTH-04 | Fase 1 | Pendente |
| [REQ-ID] | Fase [N] | Pendente |

**Cobertura:**
- Requisitos v1: [X] total
- Mapeados para fases: [Y]
- Não mapeados: [Z] ⚠️

---
*Requisitos definidos em: [data]*
*Última atualização: [data] após [gatilho]*
```

</template>

<guidelines>

**Formato dos Requisitos:**
- ID: `[CATEGORIA]-[NÚMERO]` (AUTH-01, CONTENT-02, SOCIAL-03)
- Descrição: Centrada no usuário, testável, atômica
- Checkbox: Apenas para requisitos v1 (v2 ainda não é acionável)

**Categorias:**
- Derivar das categorias do FEATURES.md de pesquisa
- Manter consistência com as convenções do domínio
- Típicas: Autenticação, Conteúdo, Social, Notificações, Moderação, Pagamentos, Admin

**v1 vs v2:**
- v1: Escopo comprometido, estará nas fases do roadmap
- v2: Reconhecido, mas adiado, não no roadmap atual
- Mover v2 → v1 requer atualização do roadmap

**Fora do Escopo:**
- Exclusões explícitas com justificativa
- Evita "por que você não incluiu X?" depois
- Anti-funcionalidades da pesquisa pertencem aqui com avisos

**Rastreabilidade:**
- Vazia inicialmente, preenchida durante a criação do roadmap
- Cada requisito mapeia para exatamente uma fase
- Requisitos não mapeados = lacuna no roadmap

**Valores de Status:**
- Pendente: Não iniciado
- Em Progresso: Fase ativa
- Completo: Requisito verificado
- Bloqueado: Aguardando fator externo

</guidelines>

<evolution>

**Após a conclusão de cada fase:**
1. Marcar requisitos cobertos como Completo
2. Atualizar o status de rastreabilidade
3. Anotar quaisquer requisitos que mudaram de escopo

**Após atualizações do roadmap:**
1. Verificar se todos os requisitos v1 ainda estão mapeados
2. Adicionar novos requisitos se o escopo expandiu
3. Mover requisitos para v2/fora do escopo se reduzidos

**Critérios de conclusão de requisito:**
- Requisito é "Completo" quando:
  - A funcionalidade está implementada
  - A funcionalidade está verificada (testes passam, verificação manual feita)
  - A funcionalidade está commitada

</evolution>

<example>

```markdown
# Requisitos: CommunityApp

**Definidos em:** 2025-01-14
**Valor Principal:** Usuários podem compartilhar e discutir conteúdo com pessoas que têm os mesmos interesses

## Requisitos v1

### Autenticação

- [ ] **AUTH-01**: Usuário pode se cadastrar com e-mail e senha
- [ ] **AUTH-02**: Usuário recebe verificação por e-mail após o cadastro
- [ ] **AUTH-03**: Usuário pode redefinir senha via link por e-mail
- [ ] **AUTH-04**: Sessão do usuário persiste após atualização do navegador

### Perfis

- [ ] **PROF-01**: Usuário pode criar perfil com nome de exibição
- [ ] **PROF-02**: Usuário pode enviar foto de avatar
- [ ] **PROF-03**: Usuário pode escrever bio (máx. 500 caracteres)
- [ ] **PROF-04**: Usuário pode visualizar perfis de outros usuários

### Conteúdo

- [ ] **CONT-01**: Usuário pode criar postagem de texto
- [ ] **CONT-02**: Usuário pode enviar imagem junto com a postagem
- [ ] **CONT-03**: Usuário pode editar suas próprias postagens
- [ ] **CONT-04**: Usuário pode excluir suas próprias postagens
- [ ] **CONT-05**: Usuário pode visualizar feed de postagens

### Social

- [ ] **SOCL-01**: Usuário pode seguir outros usuários
- [ ] **SOCL-02**: Usuário pode deixar de seguir usuários
- [ ] **SOCL-03**: Usuário pode curtir postagens
- [ ] **SOCL-04**: Usuário pode comentar em postagens
- [ ] **SOCL-05**: Usuário pode visualizar feed de atividades (postagens de usuários seguidos)

## Requisitos v2

### Notificações

- **NOTF-01**: Usuário recebe notificações no aplicativo
- **NOTF-02**: Usuário recebe e-mail para novos seguidores
- **NOTF-03**: Usuário recebe e-mail para comentários em suas postagens
- **NOTF-04**: Usuário pode configurar preferências de notificação

### Moderação

- **MODR-01**: Usuário pode denunciar conteúdo
- **MODR-02**: Usuário pode bloquear outros usuários
- **MODR-03**: Admin pode visualizar conteúdo denunciado
- **MODR-04**: Admin pode remover conteúdo
- **MODR-05**: Admin pode banir usuários

## Fora do Escopo

| Funcionalidade | Motivo |
|----------------|--------|
| Chat em tempo real | Alta complexidade, não é central para o valor da comunidade |
| Postagens em vídeo | Custos de armazenamento/largura de banda, adiar para v2+ |
| Login com OAuth | E-mail/senha suficiente para v1 |
| Aplicativo mobile | Web primeiro, mobile depois |

## Rastreabilidade

| Requisito | Fase | Status |
|-----------|------|--------|
| AUTH-01 | Fase 1 | Pendente |
| AUTH-02 | Fase 1 | Pendente |
| AUTH-03 | Fase 1 | Pendente |
| AUTH-04 | Fase 1 | Pendente |
| PROF-01 | Fase 2 | Pendente |
| PROF-02 | Fase 2 | Pendente |
| PROF-03 | Fase 2 | Pendente |
| PROF-04 | Fase 2 | Pendente |
| CONT-01 | Fase 3 | Pendente |
| CONT-02 | Fase 3 | Pendente |
| CONT-03 | Fase 3 | Pendente |
| CONT-04 | Fase 3 | Pendente |
| CONT-05 | Fase 3 | Pendente |
| SOCL-01 | Fase 4 | Pendente |
| SOCL-02 | Fase 4 | Pendente |
| SOCL-03 | Fase 4 | Pendente |
| SOCL-04 | Fase 4 | Pendente |
| SOCL-05 | Fase 4 | Pendente |

**Cobertura:**
- Requisitos v1: 18 total
- Mapeados para fases: 18
- Não mapeados: 0 ✓

---
*Requisitos definidos em: 2025-01-14*
*Última atualização: 2025-01-14 após a definição inicial*
```

</example>
