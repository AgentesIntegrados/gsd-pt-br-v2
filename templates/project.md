# Template do PROJECT.md

Template para `.planning/PROJECT.md` — o documento de contexto vivo do projeto.

<template>

```markdown
# [Nome do Projeto]

## O Que É Isso

[Descrição atual e precisa — 2-3 frases. O que este produto faz e para quem é?
Use a linguagem e o enquadramento do usuário. Atualize sempre que a realidade divergir desta descrição.]

## Valor Principal

[A UMA coisa que mais importa. Se tudo o mais falhar, isso deve funcionar.
Uma frase que orienta a priorização quando surgem compensações.]

## Requisitos

### Validados

<!-- Entregues e confirmados como valiosos. -->

(Nenhum ainda — entregue para validar)

### Ativos

<!-- Escopo atual. Construindo em direção a esses. -->

- [ ] [Requisito 1]
- [ ] [Requisito 2]
- [ ] [Requisito 3]

### Fora do Escopo

<!-- Limites explícitos. Inclui raciocínio para evitar re-adição futura. -->

- [Exclusão 1] — [por quê]
- [Exclusão 2] — [por quê]

## Contexto

[Informações de fundo que orientam a implementação:
- Ambiente ou ecossistema técnico
- Trabalho anterior relevante ou experiência
- Temas de pesquisa ou feedback de usuários
- Problemas conhecidos a resolver]

## Restrições

- **[Tipo]**: [O quê] — [Por quê]
- **[Tipo]**: [O quê] — [Por quê]

Tipos comuns: Stack tecnológica, Prazo, Orçamento, Dependências, Compatibilidade, Performance, Segurança

## Decisões-Chave

<!-- Decisões que restringem trabalhos futuros. Adicione ao longo do ciclo de vida do projeto. -->

| Decisão | Justificativa | Resultado |
|---------|---------------|-----------|
| [Escolha] | [Por quê] | [✓ Boa / ⚠️ Revisar / — Pendente] |

---
*Última atualização: [data] após [gatilho]*
```

</template>

<guidelines>

**O Que É Isso:**
- Descrição atual e precisa do produto
- 2-3 frases capturando o que ele faz e para quem é
- Use as palavras e o enquadramento do usuário
- Atualize quando o produto evoluir além desta descrição

**Valor Principal:**
- A coisa mais importante
- Tudo o mais pode falhar; isso não pode
- Orienta a priorização quando surgem compensações
- Raramente muda; se mudar, é uma mudança significativa de direção

**Requisitos — Validados:**
- Requisitos que foram entregues e provaram valor
- Formato: `- ✓ [Requisito] — [versão/fase]`
- Estes estão bloqueados — alterá-los requer discussão explícita

**Requisitos — Ativos:**
- Escopo atual sendo construído
- São hipóteses até serem entregues e validados
- Mova para Validados quando entregues, para Fora do Escopo se invalidados

**Requisitos — Fora do Escopo:**
- Limites explícitos sobre o que não estamos construindo
- Sempre inclua o raciocínio (evita re-adição posterior)
- Inclui: considerados e rejeitados, adiados para o futuro, explicitamente excluídos

**Contexto:**
- Informações de fundo que orientam decisões de implementação
- Ambiente técnico, trabalho anterior, feedback de usuários
- Problemas conhecidos ou dívida técnica a resolver
- Atualize conforme novos contextos surjam

**Restrições:**
- Limites rígidos nas escolhas de implementação
- Stack tecnológica, prazo, orçamento, compatibilidade, dependências
- Inclua o "por quê" — restrições sem justificativa são questionadas

**Decisões-Chave:**
- Escolhas significativas que afetam trabalhos futuros
- Adicione decisões conforme forem tomadas ao longo do projeto
- Acompanhe o resultado quando conhecido:
  - ✓ Boa — decisão provou estar correta
  - ⚠️ Revisar — decisão pode precisar de reconsideração
  - — Pendente — cedo demais para avaliar

**Última Atualização:**
- Sempre anote quando e por que o documento foi atualizado
- Formato: `após a Fase 2` ou `após o milestone v1.0`
- Aciona revisão de se o conteúdo ainda está preciso

</guidelines>

<evolution>

O PROJECT.md evolui ao longo do ciclo de vida do projeto.
Estas regras estão incorporadas no PROJECT.md gerado (seção ## Evolução)
e implementadas por workflows/transition.md e workflows/complete-milestone.md.

**Após cada transição de fase:**
1. Requisitos invalidados? → Mover para Fora do Escopo com justificativa
2. Requisitos validados? → Mover para Validados com referência de fase
3. Novos requisitos surgiram? → Adicionar aos Ativos
4. Decisões a registrar? → Adicionar às Decisões-Chave
5. "O Que É Isso" ainda está preciso? → Atualizar se houve desvio

**Após cada milestone:**
1. Revisão completa de todas as seções
2. Verificação do Valor Principal — ainda é a prioridade certa?
3. Auditoria de Fora do Escopo — os motivos ainda são válidos?
4. Atualizar Contexto com o estado atual (usuários, feedback, métricas)

</evolution>

<brownfield>

Para codebases existentes:

1. **Mapeie o codebase primeiro** via `/gsd-map-codebase`

2. **Inferir Requisitos Validados** a partir do código existente:
   - O que o codebase realmente faz?
   - Quais padrões estão estabelecidos?
   - O que está claramente funcionando e sendo utilizado?

3. **Coletar Requisitos Ativos** do usuário:
   - Apresentar o estado atual inferido
   - Perguntar o que querem construir a seguir

4. **Inicializar:**
   - Validados = inferidos a partir do código existente
   - Ativos = objetivos do usuário para este trabalho
   - Fora do Escopo = limites especificados pelo usuário
   - Contexto = inclui o estado atual do codebase

</brownfield>

<state_reference>

O STATE.md referencia o PROJECT.md:

```markdown
## Referência do Projeto

Veja: .planning/PROJECT.md (atualizado em [data])

**Valor principal:** [Frase única da seção Valor Principal]
**Foco atual:** [Nome da fase atual]
```

Isso garante que Claude leia o contexto atual do PROJECT.md.

</state_reference>
