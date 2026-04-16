# Template de Pesquisa de Armadilhas

Template para `.planning/research/PITFALLS.md` — erros comuns a evitar no domínio do projeto.

<template>

```markdown
# Pesquisa de Armadilhas

**Domínio:** [tipo de domínio]
**Pesquisada em:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Armadilhas Críticas

### Armadilha 1: [Nome]

**O que dá errado:**
[Descrição do modo de falha]

**Por que acontece:**
[Causa raiz — por que os desenvolvedores cometem este erro]

**Como evitar:**
[Estratégia específica de prevenção]

**Sinais de alerta:**
[Como detectar isso cedo antes que se torne um problema]

**Fase para tratar:**
[Qual fase do roadmap deve prevenir isso]

---

### Armadilha 2: [Nome]

**O que dá errado:**
[Descrição do modo de falha]

**Por que acontece:**
[Causa raiz — por que os desenvolvedores cometem este erro]

**Como evitar:**
[Estratégia específica de prevenção]

**Sinais de alerta:**
[Como detectar isso cedo antes que se torne um problema]

**Fase para tratar:**
[Qual fase do roadmap deve prevenir isso]

---

### Armadilha 3: [Nome]

**O que dá errado:**
[Descrição do modo de falha]

**Por que acontece:**
[Causa raiz — por que os desenvolvedores cometem este erro]

**Como evitar:**
[Estratégia específica de prevenção]

**Sinais de alerta:**
[Como detectar isso cedo antes que se torne um problema]

**Fase para tratar:**
[Qual fase do roadmap deve prevenir isso]

---

[Continue para todas as armadilhas críticas...]

## Padrões de Dívida Técnica

Atalhos que parecem razoáveis, mas criam problemas a longo prazo.

| Atalho | Benefício Imediato | Custo a Longo Prazo | Quando É Aceitável |
|--------|-------------------|---------------------|-------------------|
| [atalho] | [benefício] | [custo] | [condições, ou "nunca"] |
| [atalho] | [benefício] | [custo] | [condições, ou "nunca"] |
| [atalho] | [benefício] | [custo] | [condições, ou "nunca"] |

## Armadilhas de Integração

Erros comuns ao conectar a serviços externos.

| Integração | Erro Comum | Abordagem Correta |
|------------|------------|-------------------|
| [serviço] | [o que as pessoas fazem errado] | [o que fazer em vez disso] |
| [serviço] | [o que as pessoas fazem errado] | [o que fazer em vez disso] |
| [serviço] | [o que as pessoas fazem errado] | [o que fazer em vez disso] |

## Armadilhas de Performance

Padrões que funcionam em pequena escala, mas falham conforme o uso cresce.

| Armadilha | Sintomas | Prevenção | Quando Quebra |
|-----------|---------|-----------|---------------|
| [armadilha] | [como você percebe] | [como evitar] | [limiar de escala] |
| [armadilha] | [como você percebe] | [como evitar] | [limiar de escala] |
| [armadilha] | [como você percebe] | [como evitar] | [limiar de escala] |

## Erros de Segurança

Problemas de segurança específicos do domínio além da segurança web geral.

| Erro | Risco | Prevenção |
|------|-------|-----------|
| [erro] | [o que pode acontecer] | [como evitar] |
| [erro] | [o que pode acontecer] | [como evitar] |
| [erro] | [o que pode acontecer] | [como evitar] |

## Armadilhas de UX

Erros comuns de experiência do usuário neste domínio.

| Armadilha | Impacto no Usuário | Abordagem Melhor |
|-----------|-------------------|-----------------|
| [armadilha] | [como os usuários sofrem] | [o que fazer em vez disso] |
| [armadilha] | [como os usuários sofrem] | [o que fazer em vez disso] |
| [armadilha] | [como os usuários sofrem] | [o que fazer em vez disso] |

## Checklist "Parece Pronto, Mas Não Está"

Coisas que parecem completas, mas estão faltando peças críticas.

- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [verificação]
- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [verificação]
- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [verificação]
- [ ] **[Funcionalidade]:** Frequentemente faltando [coisa] — verifique [verificação]

## Estratégias de Recuperação

Quando as armadilhas ocorrem apesar da prevenção, como se recuperar.

| Armadilha | Custo de Recuperação | Etapas de Recuperação |
|-----------|----------------------|-----------------------|
| [armadilha] | BAIXO/MÉDIO/ALTO | [o que fazer] |
| [armadilha] | BAIXO/MÉDIO/ALTO | [o que fazer] |
| [armadilha] | BAIXO/MÉDIO/ALTO | [o que fazer] |

## Mapeamento Armadilha-para-Fase

Como as fases do roadmap devem tratar essas armadilhas.

| Armadilha | Fase de Prevenção | Verificação |
|-----------|------------------|-------------|
| [armadilha] | Fase [X] | [como verificar se a prevenção funcionou] |
| [armadilha] | Fase [X] | [como verificar se a prevenção funcionou] |
| [armadilha] | Fase [X] | [como verificar se a prevenção funcionou] |

## Fontes

- [Post-mortems referenciados]
- [Discussões da comunidade]
- [Documentação oficial de "armadilhas"]
- [Experiência pessoal / problemas conhecidos]

---
*Pesquisa de armadilhas para: [domínio]*
*Pesquisada em: [data]*
```

</template>

<guidelines>

**Armadilhas Críticas:**
- Foque em problemas específicos do domínio, não em erros genéricos
- Inclua sinais de alerta — a detecção precoce previne desastres
- Vincule a fases específicas — torna as armadilhas acionáveis

**Dívida Técnica:**
- Seja realista — alguns atalhos são aceitáveis
- Anote quando os atalhos são "nunca aceitáveis" vs. "apenas no MVP"
- Inclua o custo a longo prazo para informar as decisões de compensação

**Armadilhas de Performance:**
- Inclua limiares de escala ("quebra com 10k usuários")
- Foque no que é relevante para a escala esperada deste projeto
- Não super-engenheiro para escala hipotética

**Erros de Segurança:**
- Além do básico do OWASP — problemas específicos do domínio
- Exemplo: Plataformas comunitárias têm preocupações de segurança diferentes de e-commerce
- Inclua o nível de risco para priorizar

**"Parece Pronto, Mas Não Está":**
- Formato de checklist para verificação durante a execução
- Comum em demos vs. produção
- Previne problemas de "funciona na minha máquina"

**Mapeamento Armadilha-para-Fase:**
- Crítico para a criação do roadmap
- Cada armadilha deve mapear para uma fase que a previne
- Informa o ordenamento de fases e os critérios de sucesso

</guidelines>
