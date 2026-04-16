# Matriz de Decisão de Frameworks de IA

> Referência usada por `gsd-framework-selector` e `gsd-ai-researcher`.
> Destilado de documentações oficiais, benchmarks e relatos de desenvolvedores (2026).

---

## Escolhas Rápidas

| Situação | Escolha |
|-----------|------|
| Caminho mais simples para um agente funcional (OpenAI) | OpenAI Agents SDK |
| Caminho mais simples para um agente funcional (agnóstico de modelo) | CrewAI |
| RAG em produção / perguntas e respostas em documentos | LlamaIndex |
| Workflows complexos com estado e ramificações | LangGraph |
| Equipes de múltiplos agentes com papéis definidos | CrewAI |
| Agentes autônomos cientes de código (Anthropic) | Claude Agent SDK |
| "Ainda não sei meus requisitos" | LangChain |
| Setor regulamentado / trilha de auditoria obrigatória | LangGraph |
| Empresas Microsoft/.NET | AutoGen/AG2 |
| Equipes comprometidas com Google Cloud / Gemini | Google ADK |
| Pipelines de NLP puro com controle explícito | Haystack |

---

## Perfis dos Frameworks

### CrewAI
- **Tipo:** Orquestração multi-agente
- **Linguagem:** Somente Python
- **Suporte a modelos:** Agnóstico de modelo
- **Curva de aprendizado:** Iniciante (papel/tarefa/equipe mapeia para times reais)
- **Melhor para:** Pipelines de conteúdo, automação de pesquisa, workflows de processos de negócio, prototipagem rápida
- **Evite se:** Gerenciamento granular de estado, TypeScript, checkpointing tolerante a falhas, ramificação condicional complexa
- **Pontos fortes:** Prototipagem multi-agente mais rápida, 5,76x mais rápido que LangGraph em tarefas de QA, memória integrada (curto/longo prazo/entidade/contextual), arquitetura Flows, standalone (sem dependência de LangChain)
- **Pontos fracos:** Checkpointing limitado, tratamento de erros grosseiro, somente Python
- **Preocupações de avaliação:** Precisão de decomposição de tarefas, handoff entre agentes, taxa de conclusão de objetivos, detecção de loops

### LlamaIndex
- **Tipo:** RAG e ingestão de dados
- **Linguagem:** Python + TypeScript
- **Suporte a modelos:** Agnóstico de modelo
- **Curva de aprendizado:** Intermediário
- **Melhor para:** Pesquisa jurídica, assistentes de conhecimento interno, busca em documentos corporativos, qualquer sistema onde a qualidade da recuperação é a prioridade #1
- **Evite se:** A necessidade principal é orquestração de agentes, colaboração multi-agente ou fluxo de conversa em chatbot
- **Pontos fortes:** Análise de documentos de classe mundial (LlamaParse), melhoria de 35% na precisão de recuperação, consultas 20-30% mais rápidas, estratégias de recuperação mistas (vetor + grafo + reranker)
- **Pontos fracos:** Framework de dados em primeiro lugar — orquestração de agentes é secundária
- **Preocupações de avaliação:** Fidelidade ao contexto, alucinação, relevância de resposta, precisão/recall de recuperação

### LangChain
- **Tipo:** Framework de LLM de propósito geral
- **Linguagem:** Python + TypeScript
- **Suporte a modelos:** Agnóstico de modelo (ecossistema mais amplo)
- **Curva de aprendizado:** Intermediário–Avançado
- **Melhor para:** Requisitos em evolução, muitas integrações de terceiros, equipes que querem um único framework para tudo, RAG + agentes + cadeias
- **Evite se:** Caso de uso simples e bem definido, RAG-primário (use LlamaIndex), workflows complexos com estado (use LangGraph), desempenho em escala é crítico
- **Pontos fortes:** Maior comunidade e ecossistema de integração, desenvolvimento 25% mais rápido vs. partir do zero, cobre RAG/agentes/cadeias/memória
- **Pontos fracos:** Overhead de abstração, latência p99 degrada sob carga, risco de crescimento de complexidade
- **Preocupações de avaliação:** Conclusão de tarefas de ponta a ponta, correção da cadeia, qualidade de recuperação

### LangGraph
- **Tipo:** Workflows de agentes com estado (baseado em grafo)
- **Linguagem:** Python + TypeScript (paridade completa)
- **Suporte a modelos:** Agnóstico de modelo (herda integrações do LangChain)
- **Curva de aprendizado:** Intermediário–Avançado (modelo mental de grafo)
- **Melhor para:** Workflows com estado de grau de produção, setores regulamentados, trilhas de auditoria, fluxos com humano no loop, agentes multi-etapas tolerantes a falhas
- **Evite se:** Chatbot simples, workflow puramente linear, prototipagem rápida
- **Pontos fortes:** Melhor checkpointing (cada nó), depuração com viagem no tempo, persistência nativa em Postgres/Redis, suporte a streaming, escolhido por 62% dos desenvolvedores para trabalho com agentes com estado (2026)
- **Pontos fracos:** Mais scaffolding inicial, curva mais íngreme, excessivo para casos simples
- **Preocupações de avaliação:** Correção de transição de estado, taxa de conclusão de objetivos, precisão no uso de ferramentas, guardrails de segurança

### OpenAI Agents SDK
- **Tipo:** Framework nativo de agentes da OpenAI
- **Linguagem:** Python + TypeScript
- **Suporte a modelos:** Otimizado para OpenAI (suporta 100+ via compatibilidade com Chat Completions)
- **Curva de aprendizado:** Iniciante (4 primitivas: Agents, Handoffs, Guardrails, Tracing)
- **Melhor para:** Equipes comprometidas com OpenAI, prototipagem rápida de agentes, agentes de voz (gpt-realtime), equipes que querem construtor visual (AgentKit)
- **Evite se:** Flexibilidade de modelo necessária, colaboração multi-agente complexa, gerenciamento de estado persistente necessário, preocupação com lock-in de fornecedor
- **Pontos fortes:** Modelo mental mais simples, rastreamento e guardrails integrados, Handoffs para delegação de agentes, Realtime Agents para voz
- **Pontos fracos:** Lock-in com OpenAI, sem estado persistente integrado, ecossistema mais jovem
- **Preocupações de avaliação:** Seguimento de instruções, guardrails de segurança, precisão de escalada, consistência de tom

### Claude Agent SDK (Anthropic)
- **Tipo:** Framework de agentes autônomos cientes de código
- **Linguagem:** Python + TypeScript
- **Suporte a modelos:** Somente modelos Claude
- **Curva de aprendizado:** Intermediário (18 eventos de hook, MCP, decoradores de ferramentas)
- **Melhor para:** Ferramentas de desenvolvedor, agentes de geração/revisão de código, assistentes de codificação autônomos, arquiteturas com uso intenso de MCP, aplicações críticas de segurança
- **Evite se:** Flexibilidade de modelo necessária, API estável/madura necessária, caso de uso não relacionado a código/uso de ferramentas
- **Pontos fortes:** Integração MCP mais profunda, acesso integrado ao sistema de arquivos/shell, 18 hooks de ciclo de vida, compactação automática de contexto, raciocínio estendido, design orientado à segurança
- **Pontos fracos:** Lock-in com Claude apenas, API mais nova/em evolução, comunidade menor
- **Preocupações de avaliação:** Correção no uso de ferramentas, segurança, qualidade de código, seguimento de instruções

### AutoGen / AG2 / Microsoft Agent Framework
- **Tipo:** Framework de agentes conversacionais multi-agente
- **Linguagem:** Python (AG2), Python + .NET (Microsoft Agent Framework)
- **Suporte a modelos:** Agnóstico de modelo
- **Curva de aprendizado:** Intermediário–Avançado
- **Melhor para:** Aplicações de pesquisa, resolução conversacional de problemas, loops de geração + execução de código, empresas Microsoft/.NET
- **Evite se:** Você quer estabilidade de ecossistema, workflows determinísticos, ou "aposta mais segura a longo prazo" (risco de fragmentação)
- **Pontos fortes:** Padrões de agentes conversacionais mais sofisticados, loop de geração + execução de código, assíncrono orientado a eventos (v0.4+), interoperabilidade entre linguagens (Microsoft Agent Framework)
- **Pontos fracos:** Ecossistema fragmentado (AutoGen em modo de manutenção, fork AG2, Microsoft Agent Framework em preview) — risco genuíno a longo prazo
- **Preocupações de avaliação:** Conclusão de objetivo de conversa, qualidade de consenso, correção na execução de código

### Google ADK (Agent Development Kit)
- **Tipo:** Framework de orquestração multi-agente
- **Linguagem:** Python + Java
- **Suporte a modelos:** Otimizado para Gemini; suporta outros modelos via LiteLLM
- **Curva de aprendizado:** Intermediário (modelo de agente/ferramenta/sessão, familiar se conhece LangGraph)
- **Melhor para:** Equipes com Google Cloud / Vertex AI, workflows multi-agente precisando de gerenciamento integrado de sessão e memória, equipes já comprometidas com Gemini, pipelines de agentes que precisam de integração com Google Search / BigQuery
- **Evite se:** Flexibilidade de modelo além do Gemini é necessária, dependência do Google Cloud não é aceitável, stack somente TypeScript
- **Pontos fortes:** Suporte de primeira parte do Google, gerenciamento integrado de sessão/memória/artefatos, integração estreita com Vertex AI e Google Search, framework de avaliação próprio (compatível com RAGAS), multi-agente por design (padrões sequencial, paralelo, loop), SDK Java para equipes corporativas
- **Pontos fracos:** Lock-in com Gemini na prática, comunidade mais jovem que LangChain/LlamaIndex, menos profundidade de integração de terceiros
- **Preocupações de avaliação:** Decomposição de tarefas multi-agente, correção no uso de ferramentas, consistência de estado de sessão, taxa de conclusão de objetivos

### Haystack
- **Tipo:** Framework de pipeline de NLP
- **Linguagem:** Python
- **Suporte a modelos:** Agnóstico de modelo
- **Curva de aprendizado:** Intermediário
- **Melhor para:** Pipelines de NLP explícitos e auditáveis, processamento de documentos com controle refinado, busca corporativa, setores regulamentados que precisam de transparência
- **Evite se:** Prototipagem rápida, workflows multi-agente, ou se você quer uma comunidade grande
- **Pontos fortes:** Controle explícito do pipeline, forte para pipelines de dados estruturados, boa documentação
- **Pontos fracos:** Comunidade menor, menos orientado a agentes que as alternativas
- **Preocupações de avaliação:** Precisão de extração, validade da saída do pipeline, qualidade de recuperação

---

## Dimensões de Decisão

### Por Tipo de Sistema

| Tipo de Sistema | Frameworks Primários | Principais Preocupações de Avaliação |
|-------------|---------------------|-------------------|
| RAG / Perguntas e Respostas sobre Conhecimento | LlamaIndex, LangChain | Fidelidade ao contexto, alucinação, precisão/recall de recuperação |
| Orquestração multi-agente | CrewAI, LangGraph, Google ADK | Decomposição de tarefas, qualidade de handoff, conclusão de objetivos |
| Assistentes conversacionais | OpenAI Agents SDK, Claude Agent SDK | Tom, segurança, seguimento de instruções, escalada |
| Extração de dados estruturados | LangChain, LlamaIndex | Conformidade com esquema, precisão de extração |
| Agentes de tarefas autônomas | LangGraph, OpenAI Agents SDK | Guardrails de segurança, correção de ferramentas, adesão a custos |
| Geração de conteúdo | Claude Agent SDK, OpenAI Agents SDK | Voz da marca, precisão factual, tom |
| Automação de código | Claude Agent SDK | Correção do código, segurança, taxa de aprovação em testes |

### Por Tamanho da Equipe e Estágio

| Contexto | Recomendação |
|---------|----------------|
| Dev solo, prototipagem | OpenAI Agents SDK ou CrewAI (mais rápido para funcionar) |
| Dev solo, RAG | LlamaIndex (baterias incluídas) |
| Equipe, produção, com estado | LangGraph (melhor tolerância a falhas) |
| Equipe, requisitos em evolução | LangChain (maior capacidade de escape) |
| Equipe, multi-agente | CrewAI (abstração de papel mais simples) |
| Corporativo, .NET | AutoGen AG2 / Microsoft Agent Framework |

### Por Comprometimento com Modelo

| Preferência | Framework |
|-----------|-----------|
| Somente OpenAI | OpenAI Agents SDK |
| Somente Anthropic/Claude | Claude Agent SDK |
| Comprometido com Google/Gemini | Google ADK |
| Agnóstico de modelo (flexibilidade total) | LangChain, LlamaIndex, CrewAI, LangGraph, Haystack |

---

## Anti-Padrões

1. **Usar LangChain para chatbots simples** — chamada direta ao SDK tem menos código, é mais rápida e mais fácil de depurar
2. **Usar CrewAI para workflows complexos com estado** — lacunas no checkpointing vão te prejudicar em produção
3. **Usar OpenAI Agents SDK com modelos não-OpenAI** — perde os benefícios de integração pelos quais você o escolheu
4. **Usar LlamaIndex como framework multi-agente** — pode fazer agentes, mas não é seu ponto forte
5. **Usar LangChain por padrão sem avaliar alternativas** — "todo mundo usa" ≠ certo para seu caso de uso
6. **Iniciar um novo projeto com AutoGen (não AG2)** — AutoGen está em modo de manutenção; use AG2 ou aguarde o GA do Microsoft Agent Framework
7. **Escolher LangGraph para fluxos lineares simples** — o overhead do grafo não vale a pena; use cadeias do LangChain
8. **Ignorar lock-in de fornecedor** — SDKs nativos de provedores (OpenAI, Claude) trocam flexibilidade por profundidade de integração; decida conscientemente

---

## Combinações (Stacks Multi-Framework)

| Padrão em Produção | Stack |
|-------------------|-------|
| RAG com observabilidade | LlamaIndex + LangSmith ou Langfuse |
| Agente com estado + RAG | LangGraph + LlamaIndex |
| Multi-agente com rastreamento | CrewAI + Langfuse |
| Agentes OpenAI com avaliações | OpenAI Agents SDK + Promptfoo ou Braintrust |
| Agentes Claude com MCP | Claude Agent SDK + LangSmith ou Arize Phoenix |
