# Template de Pesquisa de Stack

Template para `.planning/research/STACK.md` — tecnologias recomendadas para o domínio do projeto.

<template>

```markdown
# Pesquisa de Stack

**Domínio:** [tipo de domínio]
**Pesquisada em:** [data]
**Confiança:** [ALTA/MÉDIA/BAIXA]

## Stack Recomendada

### Tecnologias Principais

| Tecnologia | Versão | Propósito | Por Que É Recomendada |
|------------|--------|-----------|----------------------|
| [nome] | [versão] | [o que faz] | [por que especialistas usam para este domínio] |
| [nome] | [versão] | [o que faz] | [por que especialistas usam para este domínio] |
| [nome] | [versão] | [o que faz] | [por que especialistas usam para este domínio] |

### Bibliotecas de Suporte

| Biblioteca | Versão | Propósito | Quando Usar |
|------------|--------|-----------|-------------|
| [nome] | [versão] | [o que faz] | [caso de uso específico] |
| [nome] | [versão] | [o que faz] | [caso de uso específico] |
| [nome] | [versão] | [o que faz] | [caso de uso específico] |

### Ferramentas de Desenvolvimento

| Ferramenta | Propósito | Observações |
|------------|-----------|-------------|
| [nome] | [o que faz] | [dicas de configuração] |
| [nome] | [o que faz] | [dicas de configuração] |

## Instalação

```bash
# Principais
npm install [pacotes]

# Suporte
npm install [pacotes]

# Dependências de desenvolvimento
npm install -D [pacotes]
```

## Alternativas Consideradas

| Recomendado | Alternativa | Quando Usar a Alternativa |
|-------------|-------------|--------------------------|
| [nossa escolha] | [outra opção] | [condições em que a alternativa é melhor] |
| [nossa escolha] | [outra opção] | [condições em que a alternativa é melhor] |

## O Que NÃO Usar

| Evitar | Por Quê | Use Em Vez Disso |
|--------|---------|-----------------|
| [tecnologia] | [problema específico] | [alternativa recomendada] |
| [tecnologia] | [problema específico] | [alternativa recomendada] |

## Padrões de Stack por Variante

**Se [condição]:**
- Use [variação]
- Porque [motivo]

**Se [condição]:**
- Use [variação]
- Porque [motivo]

## Compatibilidade de Versões

| Pacote A | Compatível Com | Observações |
|----------|----------------|-------------|
| [pacote@versão] | [pacote@versão] | [notas de compatibilidade] |

## Fontes

- [ID da biblioteca Context7] — [tópicos consultados]
- [URL da documentação oficial] — [o que foi verificado]
- [Outra fonte] — [nível de confiança]

---
*Pesquisa de stack para: [domínio]*
*Pesquisada em: [data]*
```

</template>

<guidelines>

**Tecnologias Principais:**
- Inclua números de versão específicos
- Explique por que esta é a escolha padrão, não apenas o que faz
- Foque em tecnologias que afetam as decisões de arquitetura

**Bibliotecas de Suporte:**
- Inclua bibliotecas comumente necessárias para este domínio
- Anote quando cada uma é necessária (nem todos os projetos precisam de todas as bibliotecas)

**Alternativas:**
- Não apenas descarte as alternativas
- Explique quando as alternativas fazem sentido
- Ajuda o usuário a tomar decisões informadas se discordar

**O Que NÃO Usar:**
- Avise ativamente contra escolhas desatualizadas ou problemáticas
- Explique o problema específico, não apenas "é velho"
- Forneça a alternativa recomendada

**Compatibilidade de Versões:**
- Anote quaisquer problemas de compatibilidade conhecidos
- Crítico para evitar tempo de depuração mais tarde

</guidelines>
