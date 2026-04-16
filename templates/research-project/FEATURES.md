# Template de Pesquisa de Funcionalidades

Template para `.planning/research/FEATURES.md` — panorama de funcionalidades para o domínio do projeto.

<template>

```markdown
# Pesquisa de Funcionalidades

**Domínio:** [tipo de domínio]
**Pesquisada em:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Panorama de Funcionalidades

### Básico Esperado (Usuários Esperam Isso)

Funcionalidades que os usuários assumem que existem. Sem essas = produto parece incompleto.

| Funcionalidade | Por Que É Esperada | Complexidade | Observações |
|----------------|-------------------|--------------|-------------|
| [funcionalidade] | [expectativa do usuário] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [expectativa do usuário] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [expectativa do usuário] | BAIXA/MÉDIA/ALTA | [notas de implementação] |

### Diferenciais (Vantagem Competitiva)

Funcionalidades que destacam o produto. Não são obrigatórias, mas valiosas.

| Funcionalidade | Proposta de Valor | Complexidade | Observações |
|----------------|-------------------|--------------|-------------|
| [funcionalidade] | [por que importa] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [por que importa] | BAIXA/MÉDIA/ALTA | [notas de implementação] |
| [funcionalidade] | [por que importa] | BAIXA/MÉDIA/ALTA | [notas de implementação] |

### Anti-Funcionalidades (Comumente Solicitadas, Frequentemente Problemáticas)

Funcionalidades que parecem boas, mas criam problemas.

| Funcionalidade | Por Que É Solicitada | Por Que É Problemática | Alternativa |
|----------------|---------------------|------------------------|-------------|
| [funcionalidade] | [apelo superficial] | [problemas reais] | [abordagem melhor] |
| [funcionalidade] | [apelo superficial] | [problemas reais] | [abordagem melhor] |

## Dependências de Funcionalidades

```
[Funcionalidade A]
    └──requer──> [Funcionalidade B]
                     └──requer──> [Funcionalidade C]

[Funcionalidade D] ──aprimora──> [Funcionalidade A]

[Funcionalidade E] ──conflita──> [Funcionalidade F]
```

### Notas de Dependência

- **[Funcionalidade A] requer [Funcionalidade B]:** [por que a dependência existe]
- **[Funcionalidade D] aprimora [Funcionalidade A]:** [como funcionam juntas]
- **[Funcionalidade E] conflita com [Funcionalidade F]:** [por que são incompatíveis]

## Definição de MVP

### Lançar Com (v1)

Produto mínimo viável — o que é necessário para validar o conceito.

- [ ] [Funcionalidade] — [por que é essencial]
- [ ] [Funcionalidade] — [por que é essencial]
- [ ] [Funcionalidade] — [por que é essencial]

### Adicionar Após a Validação (v1.x)

Funcionalidades a adicionar quando o núcleo estiver funcionando.

- [ ] [Funcionalidade] — [gatilho para adicionar]
- [ ] [Funcionalidade] — [gatilho para adicionar]

### Consideração Futura (v2+)

Funcionalidades a adiar até que o product-market fit seja estabelecido.

- [ ] [Funcionalidade] — [por que adiar]
- [ ] [Funcionalidade] — [por que adiar]

## Matriz de Priorização de Funcionalidades

| Funcionalidade | Valor para o Usuário | Custo de Implementação | Prioridade |
|----------------|---------------------|------------------------|------------|
| [funcionalidade] | ALTA/MÉDIA/BAIXA | ALTA/MÉDIA/BAIXA | P1/P2/P3 |
| [funcionalidade] | ALTA/MÉDIA/BAIXA | ALTA/MÉDIA/BAIXA | P1/P2/P3 |
| [funcionalidade] | ALTA/MÉDIA/BAIXA | ALTA/MÉDIA/BAIXA | P1/P2/P3 |

**Chave de prioridade:**
- P1: Deve ter para o lançamento
- P2: Deveria ter, adicionar quando possível
- P3: Legal de ter, consideração futura

## Análise Competitiva de Funcionalidades

| Funcionalidade | Concorrente A | Concorrente B | Nossa Abordagem |
|----------------|---------------|---------------|-----------------|
| [funcionalidade] | [como fazem] | [como fazem] | [nosso plano] |
| [funcionalidade] | [como fazem] | [como fazem] | [nosso plano] |

## Fontes

- [Produtos concorrentes analisados]
- [Fontes de pesquisa ou feedback de usuários]
- [Padrões do setor referenciados]

---
*Pesquisa de funcionalidades para: [domínio]*
*Pesquisada em: [data]*
```

</template>

<guidelines>

**Básico Esperado:**
- São inegociáveis para o lançamento
- Os usuários não dão crédito por ter, mas penalizam por não ter
- Exemplo: Uma plataforma comunitária sem perfis de usuário está quebrada

**Diferenciais:**
- Aqui é onde você compete
- Devem se alinhar com o Valor Principal do PROJECT.md
- Não tente se diferenciar em tudo

**Anti-Funcionalidades:**
- Previne a expansão de escopo documentando o que parece bom, mas não é
- Inclua a abordagem alternativa
- Exemplo: "Tudo em tempo real" frequentemente cria complexidade sem valor

**Dependências de Funcionalidades:**
- Crítico para o ordenamento de fases do roadmap
- Se A requer B, B deve estar em uma fase anterior
- Conflitos informam o que NÃO combinar na mesma fase

**Definição de MVP:**
- Seja impiedoso sobre o que é verdadeiramente mínimo
- "Legal de ter" não é MVP
- Lance com menos, valide, depois expanda

</guidelines>
