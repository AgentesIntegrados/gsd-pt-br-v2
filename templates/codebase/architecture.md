# Template de Arquitetura

Template para `.planning/codebase/ARCHITECTURE.md` - captura a organização conceitual do código.

**Propósito:** Documentar como o código está organizado em um nível conceitual. Complementa o STRUCTURE.md (que mostra os locais físicos dos arquivos).

---

## Template do Arquivo

```markdown
# Arquitetura

**Data da Análise:** [YYYY-MM-DD]

## Visão Geral do Padrão

**Geral:** [Nome do padrão: ex.: "CLI Monolítica", "API Serverless", "MVC Full-Stack"]

**Características Principais:**
- [Característica 1: ex.: "Executável único"]
- [Característica 2: ex.: "Tratamento de requisições sem estado"]
- [Característica 3: ex.: "Orientado a eventos"]

## Camadas

[Descreva as camadas conceituais e suas responsabilidades]

**[Nome da Camada]:**
- Propósito: [O que esta camada faz]
- Contém: [Tipos de código: ex.: "handlers de rota", "lógica de negócios"]
- Depende de: [O que usa: ex.: "apenas a camada de dados"]
- Usado por: [O que a usa: ex.: "rotas de API"]

**[Nome da Camada]:**
- Propósito: [O que esta camada faz]
- Contém: [Tipos de código]
- Depende de: [O que usa]
- Usado por: [O que a usa]

## Fluxo de Dados

[Descreva o ciclo de vida típico de uma requisição/execução]

**[Nome do Fluxo] (ex.: "Requisição HTTP", "Comando CLI", "Processamento de Eventos"):**

1. [Ponto de entrada: ex.: "Usuário executa o comando"]
2. [Etapa de processamento: ex.: "Router corresponde ao caminho"]
3. [Etapa de processamento: ex.: "Controller valida a entrada"]
4. [Etapa de processamento: ex.: "Service executa a lógica"]
5. [Saída: ex.: "Resposta retornada"]

**Gerenciamento de Estado:**
- [Como o estado é tratado: ex.: "Sem estado - sem estado persistente", "Banco de dados por requisição", "Cache em memória"]

## Abstrações-Chave

[Conceitos/padrões principais usados em todo o codebase]

**[Nome da Abstração]:**
- Propósito: [O que representa]
- Exemplos: [ex.: "UserService, ProjectService"]
- Padrão: [ex.: "Singleton", "Factory", "Repository"]

**[Nome da Abstração]:**
- Propósito: [O que representa]
- Exemplos: [Exemplos concretos]
- Padrão: [Padrão usado]

## Pontos de Entrada

[Onde a execução começa]

**[Ponto de Entrada]:**
- Local: [Breve: ex.: "src/index.ts", "API Gateway dispara"]
- Acionado por: [O que o invoca: ex.: "Invocação via CLI", "Requisição HTTP"]
- Responsabilidades: [O que faz: ex.: "Analisar argumentos, rotear para o comando"]

## Tratamento de Erros

**Estratégia:** [Como os erros são tratados: ex.: "Propagação de exceções para o handler de nível superior", "Middleware de erro por rota"]

**Padrões:**
- [Padrão: ex.: "try/catch no nível do controller"]
- [Padrão: ex.: "Códigos de erro retornados ao usuário"]

## Preocupações Transversais

[Aspectos que afetam múltiplas camadas]

**Logging:**
- [Abordagem: ex.: "Logger Winston, injetado por requisição"]

**Validação:**
- [Abordagem: ex.: "Schemas Zod na fronteira da API"]

**Autenticação:**
- [Abordagem: ex.: "Middleware JWT em rotas protegidas"]

---

*Análise de arquitetura: [data]*
*Atualize quando os padrões principais mudarem*
```

<good_examples>
```markdown
# Arquitetura

**Data da Análise:** 2025-01-20

## Visão Geral do Padrão

**Geral:** Aplicativo CLI com Sistema de Plugins

**Características Principais:**
- Executável único com subcomandos
- Extensibilidade baseada em plugins
- Estado baseado em arquivos (sem banco de dados)
- Modelo de execução síncrono

## Camadas

**Camada de Comando:**
- Propósito: Analisar a entrada do usuário e rotear para o handler adequado
- Contém: Definições de comandos, análise de argumentos, texto de ajuda
- Local: `src/commands/*.ts`
- Depende de: Camada de serviço para lógica de negócios
- Usado por: Ponto de entrada da CLI (`src/index.ts`)

**Camada de Serviço:**
- Propósito: Lógica de negócios principal
- Contém: FileService, TemplateService, InstallService
- Local: `src/services/*.ts`
- Depende de: Utilitários do sistema de arquivos, ferramentas externas
- Usado por: Handlers de comando

**Camada de Utilitários:**
- Propósito: Helpers compartilhados e abstrações
- Contém: Wrappers de E/S de arquivo, resolução de caminhos, formatação de strings
- Local: `src/utils/*.ts`
- Depende de: Apenas os built-ins do Node.js
- Usado por: Camada de serviço

## Fluxo de Dados

**Execução de Comando CLI:**

1. Usuário executa: `gsd new-project`
2. Commander analisa argumentos e flags
3. Handler do comando invocado (`src/commands/new-project.ts`)
4. Handler chama métodos de serviço (`src/services/project.ts` → `create()`)
5. Serviço lê templates, processa arquivos, escreve saída
6. Resultados registrados no console
7. Processo encerra com código de status

**Gerenciamento de Estado:**
- Baseado em arquivo: Todo o estado reside no diretório `.planning/`
- Sem estado persistente em memória
- Cada execução de comando é independente

## Abstrações-Chave

**Service:**
- Propósito: Encapsular lógica de negócios para um domínio
- Exemplos: `src/services/file.ts`, `src/services/template.ts`, `src/services/project.ts`
- Padrão: Semelhante a Singleton (importados como módulos, não instanciados)

**Command:**
- Propósito: Definição de comando CLI
- Exemplos: `src/commands/new-project.ts`, `src/commands/plan-phase.ts`
- Padrão: Registro de comando Commander.js

**Template:**
- Propósito: Estruturas de documento reutilizáveis
- Exemplos: Templates PROJECT.md, PLAN.md
- Padrão: Arquivos Markdown com variáveis de substituição

## Pontos de Entrada

**Entrada CLI:**
- Local: `src/index.ts`
- Acionado por: Usuário executa `gsd <comando>`
- Responsabilidades: Registrar comandos, analisar argumentos, exibir ajuda

**Comandos:**
- Local: `src/commands/*.ts`
- Acionado por: Comando correspondido na CLI
- Responsabilidades: Validar entrada, chamar serviços, formatar saída

## Tratamento de Erros

**Estratégia:** Lançar exceções, capturar no nível do comando, registrar e encerrar

**Padrões:**
- Serviços lançam Error com mensagens descritivas
- Handlers de comando capturam, registram o erro no stderr, encerram com exit(1)
- Erros de validação exibidos antes da execução (falha rápida)

## Preocupações Transversais

**Logging:**
- Console.log para saída normal
- Console.error para erros
- Chalk para saída colorida

**Validação:**
- Schemas Zod para análise de arquivo de configuração
- Validação manual nos handlers de comando
- Falha rápida em entrada inválida

**Operações de Arquivo:**
- Abstração FileService sobre fs-extra
- Todos os caminhos validados antes das operações
- Escritas atômicas (arquivo temporário + renomeação)

---

*Análise de arquitetura: 2025-01-20*
*Atualize quando os padrões principais mudarem*
```
</good_examples>

<guidelines>
**O que pertence ao ARCHITECTURE.md:**
- Padrão arquitetural geral (monolito, microsserviços, em camadas, etc.)
- Camadas conceituais e seus relacionamentos
- Fluxo de dados / ciclo de vida da requisição
- Abstrações e padrões-chave
- Pontos de entrada
- Estratégia de tratamento de erros
- Preocupações transversais (logging, auth, validação)

**O que NÃO pertence aqui:**
- Listagens exaustivas de arquivos (isso é o STRUCTURE.md)
- Escolhas tecnológicas (isso é o STACK.md)
- Walkthrough de código linha por linha (deixar para leitura de código)
- Detalhes de implementação de funcionalidades específicas

**Caminhos de arquivo SÃO bem-vindos:**
Inclua caminhos de arquivo como exemplos concretos de abstrações. Use formatação com backtick: `src/services/user.ts`. Isso torna o documento de arquitetura acionável para o Claude durante o planejamento.

**Ao preencher este template:**
- Leia os principais pontos de entrada (index, server, main)
- Identifique camadas lendo imports/dependências
- Trace uma execução típica de requisição/comando
- Note padrões recorrentes (services, controllers, repositories)
- Mantenha as descrições conceituais, não mecânicas

**Útil para planejamento de fase quando:**
- Adicionando novas funcionalidades (onde se encaixa nas camadas?)
- Refatorando (entendendo os padrões atuais)
- Identificando onde adicionar código (qual camada lida com X?)
- Entendendo dependências entre componentes
</guidelines>
