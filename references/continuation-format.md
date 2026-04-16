# Formato de Continuação

Formato padrão para apresentar os próximos passos após concluir um comando ou workflow.

## Estrutura Principal

```
---

## ▶ A Seguir

**{identificador}: {nome}** — {descrição em uma linha}

`/clear` e depois:

`{comando para copiar e colar}`

---

**Também disponível:**
- `{opção alternativa 1}` — descrição
- `{opção alternativa 2}` — descrição

---
```

## Regras de Formato

1. **Sempre mostre o que é** — nome + descrição, nunca apenas um caminho de comando
2. **Extraia contexto da fonte** — ROADMAP.md para fases, `<objective>` do PLAN.md para planos
3. **Comando em código inline** — backticks, fácil de copiar e colar, renderiza como link clicável
4. **`/clear` primeiro** — sempre mostre `/clear` antes do comando para que os usuários o executem na ordem correta
5. **"Também disponível" não "Outras opções"** — soa mais como um aplicativo
6. **Separadores visuais** — `---` acima e abaixo para destacar

## Variantes

### Executar Próximo Plano

```
---

## ▶ A Seguir

**02-03: Rotação de Token de Atualização** — Adicionar /api/auth/refresh com expiração deslizante

`/clear` e depois:

`/gsd-execute-phase 2`

---

**Também disponível:**
- Revisar o plano antes de executar
- `/gsd-list-phase-assumptions 2` — verificar suposições

---
```

### Executar Último Plano na Fase

Adicione uma nota de que este é o último plano e o que vem depois:

```
---

## ▶ A Seguir

**02-03: Rotação de Token de Atualização** — Adicionar /api/auth/refresh com expiração deslizante
<sub>Último plano na Fase 2</sub>

`/clear` e depois:

`/gsd-execute-phase 2`

---

**Após isso ser concluído:**
- Transição da Fase 2 → Fase 3
- Próximo: **Fase 3: Funcionalidades Principais** — Dashboard e configurações do usuário

---
```

### Planejar uma Fase

```
---

## ▶ A Seguir

**Fase 2: Autenticação** — Fluxo de login JWT com tokens de atualização

`/clear` e depois:

`/gsd-plan-phase 2`

---

**Também disponível:**
- `/gsd-discuss-phase 2` — coletar contexto primeiro
- `/gsd-research-phase 2` — investigar desconhecidos
- Revisar o roadmap

---
```

### Fase Completa, Pronta para a Próxima

Mostre o status de conclusão antes da próxima ação:

```
---

## ✓ Fase 2 Completa

3/3 planos executados

## ▶ A Seguir

**Fase 3: Funcionalidades Principais** — Dashboard do usuário, configurações e exportação de dados

`/clear` e depois:

`/gsd-plan-phase 3`

---

**Também disponível:**
- `/gsd-discuss-phase 3` — coletar contexto primeiro
- `/gsd-research-phase 3` — investigar desconhecidos
- Revisar o que a Fase 2 construiu

---
```

### Múltiplas Opções Iguais

Quando não há uma ação principal clara:

```
---

## ▶ A Seguir

**Fase 3: Funcionalidades Principais** — Dashboard do usuário, configurações e exportação de dados

`/clear` e depois uma das opções:

**Para planejar diretamente:** `/gsd-plan-phase 3`

**Para discutir o contexto primeiro:** `/gsd-discuss-phase 3`

**Para pesquisar desconhecidos:** `/gsd-research-phase 3`

---
```

### Marco Completo

```
---

## 🎉 Marco v1.0 Completo

Todas as 4 fases entregues

## ▶ A Seguir

**Iniciar v1.1** — questionamento → pesquisa → requisitos → roadmap

`/clear` e depois:

`/gsd-new-milestone`

---
```

## Extraindo Contexto

### Para fases (do ROADMAP.md):

```markdown
### Fase 2: Autenticação
**Objetivo**: Fluxo de login JWT com tokens de atualização
```

Extraia: `**Fase 2: Autenticação** — Fluxo de login JWT com tokens de atualização`

### Para planos (do ROADMAP.md):

```markdown
Planos:
- [ ] 02-03: Adicionar rotação de token de atualização
```

Ou do `<objective>` do PLAN.md:

```xml
<objective>
Adicionar rotação de token de atualização com janela de expiração deslizante.

Propósito: Estender o tempo de vida da sessão sem comprometer a segurança.
</objective>
```

Extraia: `**02-03: Rotação de Token de Atualização** — Adicionar /api/auth/refresh com expiração deslizante`

## Anti-Padrões

### Não faça: Somente comando (sem contexto)

```
## Para Continuar

Execute `/clear`, depois cole:
/gsd-execute-phase 2
```

O usuário não tem ideia do que é o 02-03.

### Não faça: Explicação de /clear ausente

```
`/gsd-plan-phase 3`

Execute /clear primeiro.
```

Não explica o porquê. O usuário pode pular.

### Não faça: Linguagem "Outras opções"

```
Outras opções:
- Revisar o roadmap
```

Parece uma reflexão tardia. Use "Também disponível:" em vez disso.

### Não faça: Blocos de código cercados para comandos

```
```
/gsd-plan-phase 3
```
```

Blocos cercados dentro de templates criam ambiguidade de aninhamento. Use backticks inline em vez disso.
