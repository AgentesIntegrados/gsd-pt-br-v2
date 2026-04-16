# Template de Convenções de Código

Template para `.planning/codebase/CONVENTIONS.md` - captura o estilo e padrões de código.

**Propósito:** Documentar como o código é escrito neste codebase. Guia prescritivo para o Claude corresponder ao estilo existente.

---

## Template do Arquivo

```markdown
# Convenções de Código

**Data da Análise:** [YYYY-MM-DD]

## Padrões de Nomenclatura

**Arquivos:**
- [Padrão: ex.: "kebab-case para todos os arquivos"]
- [Arquivos de teste: ex.: "*.test.ts ao lado do código-fonte"]
- [Componentes: ex.: "PascalCase.tsx para componentes React"]

**Funções:**
- [Padrão: ex.: "camelCase para todas as funções"]
- [Assíncronas: ex.: "sem prefixo especial para funções assíncronas"]
- [Handlers: ex.: "handleNomeDoEvento para handlers de eventos"]

**Variáveis:**
- [Padrão: ex.: "camelCase para variáveis"]
- [Constantes: ex.: "UPPER_SNAKE_CASE para constantes"]
- [Privadas: ex.: "_prefixo para membros privados" ou "sem prefixo"]

**Tipos:**
- [Interfaces: ex.: "PascalCase, sem prefixo I"]
- [Types: ex.: "PascalCase para aliases de tipo"]
- [Enums: ex.: "PascalCase para o nome do enum, UPPER_CASE para os valores"]

## Estilo de Código

**Formatação:**
- [Ferramenta: ex.: "Prettier com configuração em .prettierrc"]
- [Comprimento de linha: ex.: "máximo de 100 caracteres"]
- [Aspas: ex.: "aspas simples para strings"]
- [Ponto e vírgula: ex.: "obrigatório" ou "omitido"]

**Linting:**
- [Ferramenta: ex.: "ESLint com eslint.config.js"]
- [Regras: ex.: "extends airbnb-base, sem console em produção"]
- [Executar: ex.: "npm run lint"]

## Organização de Imports

**Ordem:**
1. [ex.: "Pacotes externos (react, express, etc.)"]
2. [ex.: "Módulos internos (@/lib, @/components)"]
3. [ex.: "Imports relativos (., ..)"]
4. [ex.: "Imports de tipo (import type {})"]

**Agrupamento:**
- [Linhas em branco: ex.: "linha em branco entre grupos"]
- [Ordenação: ex.: "alfabética dentro de cada grupo"]

**Aliases de Caminho:**
- [Aliases usados: ex.: "@/ para src/, @components/ para src/components/"]

## Tratamento de Erros

**Padrões:**
- [Estratégia: ex.: "lançar erros, capturar nas fronteiras"]
- [Erros customizados: ex.: "estender a classe Error, nomeados *Error"]
- [Assíncronas: ex.: "usar try/catch, sem cadeias .catch()"]

**Tipos de Erro:**
- [Quando lançar: ex.: "entrada inválida, dependências ausentes"]
- [Quando retornar: ex.: "falhas esperadas retornam Result<T, E>"]
- [Logging: ex.: "registrar erro com contexto antes de lançar"]

## Logging

**Framework:**
- [Ferramenta: ex.: "console.log, pino, winston"]
- [Níveis: ex.: "debug, info, warn, error"]

**Padrões:**
- [Formato: ex.: "logging estruturado com objeto de contexto"]
- [Quando: ex.: "registrar transições de estado, chamadas externas"]
- [Onde: ex.: "registrar nas fronteiras de serviço, não nos utilitários"]

## Comentários

**Quando Comentar:**
- [ex.: "explicar o por quê, não o o quê"]
- [ex.: "documentar lógica de negócios, algoritmos, casos extremos"]
- [ex.: "evitar comentários óbvios como // incrementar contador"]

**JSDoc/TSDoc:**
- [Uso: ex.: "obrigatório para APIs públicas, opcional para internas"]
- [Formato: ex.: "usar tags @param, @returns, @throws"]

**Comentários TODO:**
- [Padrão: ex.: "// TODO(nome): descrição"]
- [Rastreamento: ex.: "vincular ao número da issue se disponível"]

## Design de Funções

**Tamanho:**
- [ex.: "manter abaixo de 50 linhas, extrair helpers"]

**Parâmetros:**
- [ex.: "máximo de 3 parâmetros, usar objeto para mais"]
- [ex.: "desestruturar objetos na lista de parâmetros"]

**Valores de Retorno:**
- [ex.: "retornos explícitos, sem undefined implícito"]
- [ex.: "retornar cedo para cláusulas guard"]

## Design de Módulos

**Exports:**
- [ex.: "named exports preferidos, default exports para componentes React"]
- [ex.: "exportar de index.ts para a API pública"]

**Barrel Files:**
- [ex.: "usar index.ts para re-exportar a API pública"]
- [ex.: "evitar dependências circulares"]

---

*Análise de convenções: [data]*
*Atualize quando os padrões mudarem*
```

<good_examples>
```markdown
# Convenções de Código

**Data da Análise:** 2025-01-20

## Padrões de Nomenclatura

**Arquivos:**
- kebab-case para todos os arquivos (command-handler.ts, user-service.ts)
- *.test.ts ao lado dos arquivos de código-fonte
- index.ts para barrel exports

**Funções:**
- camelCase para todas as funções
- Sem prefixo especial para funções assíncronas
- handleNomeDoEvento para handlers de eventos (handleClick, handleSubmit)

**Variáveis:**
- camelCase para variáveis
- UPPER_SNAKE_CASE para constantes (MAX_RETRIES, API_BASE_URL)
- Sem prefixo underscore (sem marcador de privado em TS)

**Tipos:**
- PascalCase para interfaces, sem prefixo I (User, não IUser)
- PascalCase para aliases de tipo (UserConfig, ResponseData)
- PascalCase para nomes de enum, UPPER_CASE para valores (Status.PENDING)

## Estilo de Código

**Formatação:**
- Prettier com .prettierrc
- 100 caracteres de comprimento de linha
- Aspas simples para strings
- Ponto e vírgula obrigatório
- Indentação de 2 espaços

**Linting:**
- ESLint com eslint.config.js
- Extends @typescript-eslint/recommended
- Sem console.log no código de produção (use o logger)
- Executar: npm run lint

## Organização de Imports

**Ordem:**
1. Pacotes externos (react, express, commander)
2. Módulos internos (@/lib, @/services)
3. Imports relativos (./utils, ../types)
4. Imports de tipo (import type { User })

**Agrupamento:**
- Linha em branco entre grupos
- Alfabética dentro de cada grupo
- Imports de tipo por último em cada grupo

**Aliases de Caminho:**
- @/ mapeia para src/
- Sem outros aliases definidos

## Tratamento de Erros

**Padrões:**
- Lançar erros, capturar nas fronteiras (handlers de rota, funções main)
- Estender a classe Error para erros customizados (ValidationError, NotFoundError)
- Funções assíncronas usam try/catch, sem cadeias .catch()

**Tipos de Erro:**
- Lançar em entrada inválida, dependências ausentes, violações de invariante
- Registrar erro com contexto antes de lançar: logger.error({ err, userId }, 'Falha ao processar')
- Incluir causa na mensagem de erro: new Error('Falha ao X', { cause: erroOriginal })

## Logging

**Framework:**
- Instância do logger pino exportada de lib/logger.ts
- Níveis: debug, info, warn, error (sem trace)

**Padrões:**
- Logging estruturado com contexto: logger.info({ userId, action }, 'Ação do usuário')
- Registrar nas fronteiras de serviço, não nas funções utilitárias
- Registrar transições de estado, chamadas de API externas, erros
- Sem console.log no código commitado

## Comentários

**Quando Comentar:**
- Explicar o por quê, não o o quê: // Tentar 3 vezes porque a API tem falhas transitórias
- Documentar regras de negócios: // Usuários devem verificar o e-mail em 24 horas
- Explicar algoritmos não óbvios ou soluções alternativas
- Evitar comentários óbvios: // definir contagem para 0

**JSDoc/TSDoc:**
- Obrigatório para funções de API pública
- Opcional para funções internas se a assinatura for autoexplicativa
- Usar tags @param, @returns, @throws

**Comentários TODO:**
- Formato: // TODO: descrição (sem nome de usuário, usando git blame)
- Vincular à issue se existir: // TODO: Corrigir race condition (issue #123)

## Design de Funções

**Tamanho:**
- Manter abaixo de 50 linhas
- Extrair helpers para lógica complexa
- Um nível de abstração por função

**Parâmetros:**
- Máximo de 3 parâmetros
- Usar objeto de opções para 4+ parâmetros: function create(options: CreateOptions)
- Desestruturar na lista de parâmetros: function process({ id, name }: ProcessParams)

**Valores de Retorno:**
- Declarações de retorno explícitas
- Retornar cedo para cláusulas guard
- Usar tipo Result<T, E> para falhas esperadas

## Design de Módulos

**Exports:**
- Named exports preferidos
- Default exports apenas para componentes React
- Exportar API pública de barrel files index.ts

**Barrel Files:**
- index.ts re-exporta a API pública
- Manter helpers internos privados (não exportar do index)
- Evitar dependências circulares (importar de arquivos específicos se necessário)

---

*Análise de convenções: 2025-01-20*
*Atualize quando os padrões mudarem*
```
</good_examples>

<guidelines>
**O que pertence ao CONVENTIONS.md:**
- Padrões de nomenclatura observados no codebase
- Regras de formatação (configuração do Prettier, regras de linting)
- Padrões de organização de imports
- Estratégia de tratamento de erros
- Abordagem de logging
- Convenções de comentários
- Padrões de design de funções e módulos

**O que NÃO pertence aqui:**
- Decisões de arquitetura (isso é o ARCHITECTURE.md)
- Escolhas tecnológicas (isso é o STACK.md)
- Padrões de teste (isso é o TESTING.md)
- Organização de arquivos (isso é o STRUCTURE.md)

**Ao preencher este template:**
- Verifique arquivos .prettierrc, .eslintrc ou similares
- Examine 5-10 arquivos-fonte representativos para padrões
- Procure consistência: se 80%+ segue um padrão, documente-o
- Seja prescritivo: "Use X" e não "Às vezes Y é usado"
- Anote desvios: "Código legado usa Y, novo código deve usar X"
- Mantenha abaixo de ~150 linhas no total

**Útil para planejamento de fase quando:**
- Escrevendo novo código (corresponder ao estilo existente)
- Adicionando funcionalidades (seguir padrões de nomenclatura)
- Refatorando (aplicar convenções consistentes)
- Revisão de código (verificar em relação aos padrões documentados)
- Integração (entender as expectativas de estilo)

**Abordagem de análise:**
- Escanear o diretório src/ para padrões de nomenclatura de arquivos
- Verificar scripts do package.json para comandos de lint/format
- Ler 5-10 arquivos para identificar nomenclatura de funções, tratamento de erros
- Procurar arquivos de configuração (.prettierrc, eslint.config.js)
- Anotar padrões em imports, comentários, assinaturas de funções
</guidelines>
