# Referência de Avaliação de IA

> Referência usada por `gsd-eval-planner` e `gsd-eval-auditor`.
> Baseado no curso "AI Evals for Everyone" (Reganti & Badam) + práticas da indústria.

---

## Conceitos Fundamentais

### Por Que Avaliações Existem
Sistemas de IA são não-determinísticos. A entrada X não produz de forma confiável a saída Y em diferentes execuções, usuários ou casos extremos. Avaliações são o processo contínuo de verificar se o comportamento do seu sistema atende às expectativas em condições do mundo real — testes unitários e de integração isolados são insuficientes.

### Avaliação de Modelo vs. Produto
- **Avaliações de modelo** (MMLU, HumanEval, GSM8K) — medem capacidade geral em condições padronizadas. Use apenas como filtro inicial.
- **Avaliações de produto** — medem o comportamento dentro do seu sistema específico, com seus dados, seus usuários, suas regras de domínio. É aqui que 80% do esforço de avaliação deve ser investido.

### Os Três Componentes de Toda Avaliação
- **Entrada** — tudo que afeta o sistema: consulta, histórico, documentos recuperados, prompt de sistema, configuração
- **Esperado** — como deve ser o bom comportamento, definido por rubricas
- **Real** — o que o sistema produziu, incluindo passos intermediários, chamadas de ferramentas e rastros de raciocínio

### Três Abordagens de Medição
1. **Métricas baseadas em código** — verificações determinísticas: validação de JSON, disclaimers obrigatórios, limiares de desempenho, flags de classificação. Rápidas, baratas, confiáveis. Use primeiro.
2. **Juízes de LLM** — um modelo avalia outro contra uma rubrica. Poderoso para qualidades subjetivas (tom, raciocínio, escalada). Requer calibração com julgamento humano antes de confiar.
3. **Avaliação humana** — padrão ouro para julgamentos nuançados. Não escala. Use para calibração, casos extremos, amostragem periódica e decisões de alto risco.

Os sistemas mais eficazes combinam as três abordagens.

---

## Dimensões de Avaliação

### Pré-Implantação (Fase de Desenvolvimento)

| Dimensão | O Que Mede | Quando É Importante |
|-----------|-----------------|-----------------|
| **Precisão factual** | Correção das afirmações em relação à verdade | RAG, bases de conhecimento, quaisquer afirmações factuais |
| **Fidelidade ao contexto** | Resposta baseada no contexto fornecido vs. fabricada | Pipelines RAG, perguntas e respostas em documentos, sistemas com recuperação aumentada |
| **Detecção de alucinações** | Afirmações plausíveis mas sem suporte | Todos os sistemas generativos, domínios de alto risco |
| **Precisão de escalada** | Identificação correta de quando a intervenção humana é necessária | Atendimento ao cliente, saúde, consultoria financeira |
| **Conformidade com políticas** | Adesão a regras de negócio, requisitos legais, disclaimers | Setores regulamentados, implantações corporativas |
| **Adequação de tom/estilo** | Correspondência com a voz da marca, expectativas do público, contexto emocional | Sistemas voltados ao cliente, geração de conteúdo |
| **Validade da estrutura de saída** | Conformidade com esquema, campos obrigatórios, correção de formato | Extração estruturada, integrações de API, pipelines de dados |
| **Conclusão de tarefas** | Se o sistema alcançou o objetivo declarado | Workflows agênticos, tarefas de múltiplas etapas |
| **Correção no uso de ferramentas** | Seleção e invocação corretas de ferramentas | Sistemas de agentes com chamadas de ferramentas |
| **Segurança** | Ausência de saídas prejudiciais, tendenciosas ou inadequadas | Todos os sistemas voltados ao usuário |

### Monitoramento em Produção

| Dimensão | Abordagem de Monitoramento |
|-----------|---------------------|
| **Violações de segurança** | Guardrail online — tempo real, intervenção imediata |
| **Falhas de conformidade** | Guardrail online — bloquear ou escalar antes que o usuário veja a saída |
| **Tendências de degradação de qualidade** | Flywheel offline — análise em lote de interações amostradas |
| **Modos de falha emergentes** | Divergência sinal-métrica — quando sinais de comportamento do usuário divergem das pontuações métricas, investigar manualmente |
| **Deriva de custo/latência** | Métricas baseadas em código — alertas automáticos de limiar |

---

## A Decisão: Guardrail vs. Flywheel

Pergunte: "Se esse comportamento der errado, seria catastrófico para o meu negócio?"

- **Sim → Guardrail** — execute online, em tempo real, com intervenção imediata (bloquear, escalar, transferir). Seja seletivo: guardrails adicionam latência.
- **Não → Flywheel** — execute offline como análise em lote que alimenta refinamentos do sistema ao longo do tempo.

---

## Design de Rubricas

Métricas genéricas não têm sentido sem contexto. "Utilidade" no setor imobiliário significa resumir listagens claramente. Na saúde, significa saber quando *não* responder.

Uma rubrica deve definir:
1. A dimensão sendo medida
2. O que pontua 1, 3 e 5 em uma escala de 5 pontos (ou critérios de aprovação/reprovação)
3. Exemplos específicos do domínio de comportamento aceitável vs. inaceitável

Sem rubricas, os juízes de LLM produzem ruído em vez de sinal.

---

## Diretrizes para Conjuntos de Dados de Referência

- Comece com **10-20 exemplos de alta qualidade** — não 200 mediocres
- Cubra: cenários críticos de sucesso, fluxos comuns do usuário, casos extremos conhecidos, modos de falha históricos
- Peça que especialistas do domínio rotule os exemplos (não apenas engenheiros)
- Expanda com base no que aprender em produção — não construa para cobertura hipotética

---

## Guia de Ferramentas de Avaliação

| Ferramenta | Tipo | Melhor Para | Ponto Forte |
|------|------|----------|-------------|
| **RAGAS** | Biblioteca Python | Avaliação de RAG | Métricas específicas: fidelidade, relevância de resposta, precisão/recall de contexto |
| **Langfuse** | Plataforma (código aberto, auto-hospedável) | Todos os tipos de sistema | Rastreamento robusto, gerenciamento de prompts, bom para equipes que querem controle de infraestrutura |
| **LangSmith** | Plataforma (comercial) | Ecossistemas LangChain/LangGraph | Integração mais estreita com LangChain; melhor se já estiver nesse ecossistema |
| **Arize Phoenix** | Plataforma (código aberto + hospedado) | Rastreamento RAG + multi-agente | Avaliação RAG robusta + visualização de traces; código aberto com opção hospedada |
| **Braintrust** | Plataforma (comercial) | Avaliação agnóstica de modelo | Gerenciamento de datasets e experimentos; bom para comparar entre frameworks |
| **Promptfoo** | Ferramenta CLI (código aberto) | Teste de prompts, CI/CD | CLI-first, excelente para testes de regressão de prompts em CI/CD |

### Seleção de Ferramenta por Tipo de Sistema

| Tipo de Sistema | Ferramentas Recomendadas |
|-------------|---------------------|
| RAG / Perguntas e Respostas sobre Conhecimento | RAGAS + Arize Phoenix ou Braintrust |
| Sistemas multi-agente | Langfuse + Arize Phoenix |
| Conversacional / modelo único | Promptfoo + Braintrust |
| Extração estruturada | Promptfoo + validadores baseados em código |
| Projetos LangChain/LangGraph | LangSmith (integração nativa) |
| Monitoramento em produção (todos os tipos) | Langfuse, Arize Phoenix ou LangSmith |

---

## Avaliações no Ciclo de Desenvolvimento

### Fase de Plano (Design Orientado a Avaliações)
Antes de escrever código, defina:
1. Que tipo de sistema de IA está sendo construído → determina o framework e as principais preocupações de avaliação
2. Modos críticos de falha (3-5 comportamentos que não podem dar errado)
3. Rubricas — definições explícitas de comportamento aceitável/inaceitável por dimensão
4. Estratégia de avaliação — quais dimensões usam métricas de código, juízes de LLM ou revisão humana
5. Requisitos do conjunto de dados de referência — tamanho, composição, abordagem de rotulagem
6. Seleção de ferramentas de avaliação

Saída: seção EVALS-SPEC do AI-SPEC.md

### Fase de Execução (Instrumentar Durante a Construção)
- Adicione rastreamento desde o primeiro dia (Langfuse, Arize Phoenix ou LangSmith)
- Construa o conjunto de dados de referência simultaneamente com a implementação
- Implemente verificações baseadas em código primeiro; adicione juízes de LLM apenas para dimensões subjetivas
- Execute avaliações em CI/CD via Promptfoo ou Braintrust

### Fase de Verificação (Validação Pré-Implantação)
- Execute o conjunto de dados de referência completo contra todas as métricas
- Conduza revisão humana de casos extremos e desacordos entre juízes de LLM
- Calibre juízes de LLM contra pontuações humanas (meta ≥ 0,7 de correlação antes de confiar)
- Defina e configure guardrails de produção
- Estabeleça linha de base de monitoramento

### Fase de Monitoramento (Loop de Avaliação em Produção)
- Amostragem inteligente — dê peso a interações com sinais preocupantes (novas tentativas, comprimento incomum, escaladas explícitas)
- Guardrails online em cada interação
- Flywheel offline em lote amostrado
- Observe divergência sinal-métrica — o sistema de alerta precoce para lacunas de avaliação

---

## Armadilhas Comuns

1. **Assumir que benchmarks preveem sucesso do produto** — não preveem; avaliações de modelo são um filtro, não um veredicto
2. **Desenvolver avaliações em isolamento** — especialistas do domínio devem co-definir rubricas; engenheiros sozinhos perdem nuances críticas
3. **Construir cobertura abrangente no primeiro dia** — comece pequeno (10-20 exemplos), expanda a partir de modos de falha reais
4. **Confiar em juízes de LLM não calibrados** — valide contra julgamento humano antes de depender deles
5. **Medir tudo** — rastreie apenas métricas que orientam decisões; "coletar tudo" produz ruído
6. **Tratar avaliação como configuração única** — o comportamento do usuário evolui, os requisitos mudam, modos de falha emergem; avaliação é contínua
