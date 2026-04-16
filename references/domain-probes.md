# Padrões de Sondagem por Domínio

Referência compartilhada para `/gsd-begin`, `/gsd-discuss-phase` e workflows de exploração de domínio.

Quando o usuário menciona uma área de tecnologia, use essas sondas para fazer perguntas de acompanhamento perspicazes. Não as percorra como uma lista de verificação — escolha as 2-3 mais relevantes com base no contexto. O objetivo é revelar suposições ocultas e trocas que o usuário ainda pode não ter considerado.

---

## Autenticação

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "login" ou "auth" | OAuth (quais provedores?), JWT ou baseado em sessão? Precisa de login social ou apenas email/senha? |
| "usuários" ou "contas" | MFA obrigatório? Fluxo de redefinição de senha? Verificação de email? |
| "sessões" | Duração da sessão e estratégia de atualização? Sessões do lado do servidor ou tokens sem estado? |
| "papéis" ou "permissões" | RBAC, ABAC ou verificações simples de papel? Quantos papéis distintos? |
| "chaves de API" | Estratégia de rotação de chaves? Permissões com escopo por chave? Limitação de taxa por chave? |

---

## Atualizações em Tempo Real

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "tempo real" ou "atualizações ao vivo" | WebSockets, SSE ou polling? O que especificamente precisa ser em tempo real vs. eventual? |
| "notificações" | Notificações push (navegador/mobile), somente no aplicativo ou ambas? Persistência e confirmações de leitura? |
| "colaboração" ou "multiplayer" | Estratégia de resolução de conflitos? Transforms operacionais ou CRDTs? Usuários simultâneos esperados? |
| "chat" ou "mensagens" | Histórico e busca de mensagens? Indicadores de digitação? Confirmações de leitura? |
| "streaming" | Estratégia de reconexão? O que acontece quando a conexão cai — enfileirar ou descartar? |

---

## Dashboard

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "dashboard" | Quais fontes de dados o alimentam? Quantas visões distintas? |
| "gráficos" ou "diagramas" | Interativos ou estáticos? Capacidade de drill-down? Exportação para CSV/PDF? |
| "métricas" ou "KPIs" | Estratégia de atualização — tempo real, polling periódico ou sob demanda? Desatualização aceitável? |
| "painel administrativo" | Visibilidade baseada em papel? Quais ações além de visualizar (editar, excluir, aprovar)? |
| "mobile" ou "responsivo" | Visão mobile simplificada ou paridade completa? Interações de toque para gráficos? |

---

## Design de API

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "API" | REST, GraphQL ou estilo RPC? Somente interno ou voltado ao público? |
| "endpoints" ou "rotas" | Estratégia de versionamento (caminho da URL, cabeçalho, parâmetro de consulta)? Política de mudanças incompatíveis? |
| "paginação" | Baseada em cursor ou offset? Tamanhos esperados de conjunto de resultados? Garantia de ordenação estável? |
| "limitação de taxa" | Por usuário, por IP ou por chave de API? Tolerância a picos? Como comunicar limites aos clientes? |
| "erros" | Formato de erro estruturado? Códigos de erro vs. mensagens? Quanto detalhe em erros de produção? |

---

## Banco de Dados

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "banco de dados" ou "armazenamento" | SQL ou NoSQL? O que orienta a escolha — integridade relacional, flexibilidade, escala? |
| "ORM" ou "consultas" | ORM (qual?) ou consultas brutas? Query builder como meio-termo? |
| "migrações" | Ferramenta de migração? Estratégia de rollback? Como lidar com migrações de dados vs. migrações de esquema? |
| "seed" ou "dados de teste" | Dados de seed para desenvolvimento? Dados falsos realistas ou fixtures mínimas? |
| "escala" ou "desempenho" | Proporção leitura/escrita? Réplicas de leitura? Estratégia de pool de conexões? |

---

## Busca

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "busca" | Texto completo ou correspondência exata? Motor de busca dedicado (Elasticsearch, Meilisearch) ou nível de banco de dados? |
| "filtros" ou "facetas" | Filtragem por facetas? Quantas dimensões de filtro? Filtros combinados (E/OU)? |
| "autocompletar" ou "typeahead" | Estratégia de debounce? Limiar mínimo de caracteres? Classificação de resultados? |
| "indexação" | Tamanho do índice e frequência de atualização? Indexação em tempo real ou em lote? Lag de índice aceitável? |
| "fuzzy" ou "tolerância a erros de digitação" | Correspondência fuzzy? Suporte a sinônimos? Stemming específico por idioma? |

---

## Upload / Armazenamento de Arquivos

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "upload" ou "upload de arquivo" | Sistema de arquivos local ou nuvem (S3, GCS, Azure Blob)? Upload direto ou pelo servidor? |
| "imagens" ou "mídia" | Pipeline de processamento — redimensionamento, compressão, geração de miniaturas? Conversão de formato? |
| "limites de tamanho" | Tamanho máximo de arquivo? Armazenamento total máximo por usuário? O que acontece quando os limites são atingidos? |
| "CDN" | CDN para entrega? Invalidação de cache para arquivos atualizados? URLs assinadas para controle de acesso? |
| "documentos" ou "anexos" | Verificação de vírus? Geração de prévia? Versionamento de arquivos enviados? |

---

## Cache

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "cache" ou "desempenho" | Onde fazer cache — navegador, CDN, camada de aplicação, cache de consulta de banco de dados? |
| "invalidação" | Estratégia de invalidação — TTL, orientada a eventos ou manual? Cache-aside vs. write-through? |
| "dados desatualizados" | Janela de desatualização aceitável? Padrão stale-while-revalidate? |
| "Redis" ou "Memcached" | Topologia de cache — nó único ou clusterizado? Persistência necessária ou cache puro? |
| "CDN" ou "edge" | Cache de edge para ativos estáticos? Conteúdo dinâmico no edge? Estratégia de chave de cache? |

---

## Testes

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "testes" | Equilíbrio entre testes unitários, de integração e E2E? Onde você investe mais esforço de teste? |
| "mocks" ou "stubs" | Simular serviços externos ou usar containers de teste? Estratégia de mock de banco de dados? |
| "CI" ou "pipeline" | Testes em CI? Execução paralela de testes? Testar em PR ou em push? |
| "cobertura" | Metas de cobertura? Cobertura como portão ou consultiva? Quais métricas (linha, branch, função)? |
| "E2E" ou "testes de navegador" | Playwright, Cypress ou outro? Com cabeçalho vs. sem cabeçalho? Testes de regressão visual? |

---

## Implantação

| Usuário menciona | Agente sonda com conhecimento do domínio |
|---|---|
| "implantar" ou "hospedagem" | Container, serverless ou VM/VPS tradicional? Plataforma gerenciada (Vercel, Railway) ou auto-hospedado? |
| "CI/CD" ou "pipeline" | GitHub Actions, GitLab CI ou outro? Implantar ao fazer merge na main ou acionamento manual? |
| "ambientes" | Quantos ambientes (dev, staging, prod)? Estratégia de paridade de ambiente? |
| "rollback" | Estratégia de rollback? Blue-green, canary ou rollback instantâneo? Considerações de rollback de banco de dados? |
| "segredos" ou "configuração" | Gerenciamento de segredos — variáveis de ambiente, vault ou nativo da plataforma? Estratégia de configuração por ambiente? |
