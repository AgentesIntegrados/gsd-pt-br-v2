# AI-SPEC — Fase {N}: {phase_name}

> Contrato de design de IA gerado por `/gsd-ai-integration-phase`. Consumido por `gsd-planner` e `gsd-eval-auditor`.
> Bloqueia a seleção de framework, orientações de implementação e estratégia de avaliação antes do início do planejamento.

---

## 1. Classificação do Sistema

**Tipo de Sistema:** <!-- RAG | Multi-Agente | Conversacional | Extração | Agente Autônomo | Geração de Conteúdo | Automação de Código | Híbrido -->

**Descrição:**
<!-- Descrição em um parágrafo do que este sistema de IA faz, quem o utiliza e como é o "bom funcionamento" -->

**Modos de Falha Críticos:**
<!-- Os 3-5 comportamentos que absolutamente não podem dar errado neste sistema -->
1.
2.
3.

---

## 1b. Contexto de Domínio

> Pesquisado por `gsd-domain-researcher`. Fundamenta a estratégia de avaliação no conhecimento de especialistas do domínio.

**Vertical da Indústria:** <!-- saúde | jurídico | finanças | atendimento ao cliente | educação | ferramentas de desenvolvimento | e-commerce | etc. -->

**População de Usuários:** <!-- quem usa este sistema e em que contexto -->

**Nível de Criticidade:** <!-- Baixo | Médio | Alto | Crítico -->

**Consequência da Saída:** <!-- o que acontece quando a saída da IA é colocada em prática -->

### O que Especialistas de Domínio Avaliam

<!-- Ingredientes de rubrica específicos do domínio — em linguagem de profissional, não jargão de IA -->
<!-- Formato: Dimensão / Bom (especialista aceita) / Ruim (especialista sinaliza) / Criticidade / Fonte -->

### Modos de Falha Conhecidos Neste Domínio

<!-- Modos de falha específicos do domínio a partir da pesquisa — não alucinação genérica, mas como ela se manifesta aqui -->

### Contexto Regulatório / de Conformidade

<!-- Regulamentações ou restrições relevantes — ou "Nenhuma identificada" se genuinamente não se aplicar -->

### Funções de Especialistas de Domínio para Avaliação

| Função | Responsabilidade |
|--------|-----------------|
| <!-- ex.: Profissional sênior --> | <!-- Rotulagem de dataset / calibração de rubrica / amostragem de produção --> |

---

## 2. Decisão de Framework

**Framework Selecionado:** <!-- ex.: LlamaIndex v0.10.x -->

**Versão:** <!-- Fixe a versão -->

**Justificativa:**
<!-- Por que este framework se adequa a este tipo de sistema, contexto da equipe e requisitos de produção -->

**Alternativas Consideradas:**

| Framework | Descartado Porque |
|-----------|------------------|
| | |

**Acoplamento ao Fornecedor Aceito:** <!-- Sim / Não / Parcial — documente o trade-off conscientemente -->

---

## 3. Referência Rápida do Framework

> Obtido da documentação oficial pelo `gsd-ai-researcher`. Destilado para este caso de uso específico.

### Instalação
```bash
# Comando(s) de instalação
```

### Imports Principais
```python
# Imports essenciais para este caso de uso
```

### Padrão de Ponto de Entrada
```python
# Exemplo mínimo funcional para este tipo de sistema
```

### Abstrações Principais
<!-- Conceitos específicos do framework que o desenvolvedor deve entender antes de codificar -->
| Conceito | O Que É | Quando Usar |
|----------|---------|-------------|
| | | |

### Armadilhas Comuns
<!-- Pegadinhas específicas deste framework e tipo de sistema — de docs, issues e relatos da comunidade -->
1.
2.
3.

### Estrutura de Projeto Recomendada
```
projeto/
├── # Layout de pastas específico do framework
```

---

## 4. Orientação de Implementação

**Configuração do Modelo:**
<!-- Qual(is) modelo(s), temperatura, max tokens e outros parâmetros principais -->

**Padrão Principal:**
<!-- O padrão de implementação primário para este tipo de sistema neste framework -->

**Uso de Ferramentas:**
<!-- Ferramentas/integrações necessárias e como configurá-las -->

**Gerenciamento de Estado:**
<!-- Como o estado é persistido, recuperado e atualizado -->

**Estratégia de Janela de Contexto:**
<!-- Como gerenciar limites de contexto para este tipo de sistema -->

---

## 4b. Boas Práticas para Sistemas de IA

> Escrito por `gsd-ai-researcher`. Padrões transversais que todo desenvolvedor de sistemas de IA precisa — independente da escolha de framework.

### Saídas Estruturadas com Pydantic

<!-- Padrão de integração do Pydantic específico para este framework e caso de uso -->
<!-- Inclua: definição do modelo de saída, como o framework o utiliza, lógica de retry em falha de validação -->

```python
# Modelo Pydantic de saída para este tipo de sistema
```

### Design Async-First

<!-- Como async é tratado neste framework, o erro mais comum e quando usar stream vs. await -->

### Disciplina em Engenharia de Prompts

<!-- Separação de prompt de sistema vs. usuário, orientação few-shot, estratégia de orçamento de tokens -->

### Gerenciamento de Janela de Contexto

<!-- Estratégia específica para este tipo de sistema: chunking de RAG / sumarização de conversa / compactação de agente -->

### Orçamento de Custo e Latência

<!-- Estimativa de custo por chamada, estratégia de cache, roteamento de modelo por subtarefa -->

---

## 5. Estratégia de Avaliação

### Dimensões

| Dimensão | Rubrica (Aprovado/Reprovado ou 1-5) | Abordagem de Medição | Prioridade |
|----------|-------------------------------------|---------------------|------------|
| | | Código / Juiz LLM / Humano | Crítico / Alto / Médio |

### Ferramentas de Avaliação

**Ferramenta Principal:** <!-- ex.: RAGAS + Langfuse -->

**Configuração:**
```bash
# Instalar e configurar
```

**Integração CI/CD:**
```bash
# Comando para executar avaliações no pipeline CI/CD
```

### Dataset de Referência

**Tamanho:** <!-- ex.: 20 exemplos para começar -->

**Composição:**
<!-- Que tipos de cenários o dataset cobre: caminhos críticos, casos extremos, modos de falha -->

**Rotulagem:**
<!-- Quem rotula os exemplos e como (especialista de domínio, juiz LLM com calibração, etc.) -->

---

## 6. Guardrails

### Online (Tempo Real)

| Guardrail | Gatilho | Intervenção |
|-----------|---------|-------------|
| | | Bloquear / Escalar / Sinalizar |

### Offline (Ciclo de Melhoria)

| Métrica | Estratégia de Amostragem | Ação em Caso de Degradação |
|---------|--------------------------|---------------------------|
| | | |

---

## 7. Monitoramento em Produção

**Ferramenta de Rastreamento:** <!-- ex.: Langfuse self-hosted -->

**Métricas Principais a Monitorar:**
<!-- 3-5 métricas que serão monitoradas em produção -->

**Limiares de Alerta:**
<!-- Quando alertar/notificar -->

**Estratégia de Amostragem Inteligente:**
<!-- Como selecionar interações para revisão humana — filtros baseados em sinal -->

---

## Checklist

- [ ] Tipo de sistema classificado
- [ ] Modos de falha críticos identificados (≥ 3)
- [ ] Contexto de domínio pesquisado (Seção 1b: vertical, criticidade, critérios de especialistas, modos de falha)
- [ ] Contexto regulatório/de conformidade identificado ou explicitamente anotado como nenhum
- [ ] Funções de especialistas de domínio definidas para envolvimento na avaliação
- [ ] Framework selecionado com justificativa documentada
- [ ] Alternativas consideradas e descartadas
- [ ] Referência rápida do framework escrita (instalação, imports, padrão, armadilhas)
- [ ] Boas práticas de sistemas de IA escritas (Seção 4b: Pydantic, async, disciplina de prompt, contexto)
- [ ] Dimensões de avaliação fundamentadas nos ingredientes de rubrica do domínio
- [ ] Cada dimensão de avaliação tem uma rubrica concreta (Bom/Ruim em linguagem de domínio)
- [ ] Ferramentas de avaliação selecionadas — padrão Arize Phoenix confirmado ou substituição anotada
- [ ] Especificação do dataset de referência escrita (tamanho ≥ 10, composição + rotulagem definidas)
- [ ] Integração CI/CD de avaliação especificada
- [ ] Guardrails online definidos
- [ ] Monitoramento de produção configurado (ferramenta de rastreamento + estratégia de amostragem)
