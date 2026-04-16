# Template de Padrões de Teste

Template para `.planning/codebase/TESTING.md` - captura o framework e os padrões de teste.

**Propósito:** Documentar como os testes são escritos e executados. Guia para adicionar testes que correspondam aos padrões existentes.

---

## Template do Arquivo

```markdown
# Padrões de Teste

**Data da Análise:** [YYYY-MM-DD]

## Framework de Teste

**Runner:**
- [Framework: ex.: "Jest 29.x", "Vitest 1.x"]
- [Configuração: ex.: "jest.config.js na raiz do projeto"]

**Biblioteca de Asserção:**
- [Biblioteca: ex.: "expect built-in", "chai"]
- [Matchers: ex.: "toBe, toEqual, toThrow"]

**Comandos de Execução:**
```bash
[ex.: "npm test" ou "npm run test"]              # Executar todos os testes
[ex.: "npm test -- --watch"]                     # Modo watch
[ex.: "npm test -- path/to/file.test.ts"]       # Arquivo único
[ex.: "npm run test:coverage"]                   # Relatório de cobertura
```

## Organização dos Arquivos de Teste

**Local:**
- [Padrão: ex.: "*.test.ts ao lado dos arquivos de código-fonte"]
- [Alternativa: ex.: "diretório __tests__/" ou "árvore tests/ separada"]

**Nomenclatura:**
- [Testes unitários: ex.: "nome-do-módulo.test.ts"]
- [Integração: ex.: "nome-da-funcionalidade.integration.test.ts"]
- [E2E: ex.: "fluxo-do-usuário.e2e.test.ts"]

**Estrutura:**
```
[Mostrar o padrão de diretório real, ex.:
src/
  lib/
    utils.ts
    utils.test.ts
  services/
    user-service.ts
    user-service.test.ts
]
```

## Estrutura dos Testes

**Organização dos Suites:**
```typescript
[Mostrar o padrão real usado, ex.:

describe('NomeDoMódulo', () => {
  describe('nomeDaFunção', () => {
    it('deve lidar com o caso de sucesso', () => {
      // arrange
      // act
      // assert
    });

    it('deve lidar com o caso de erro', () => {
      // código do teste
    });
  });
});
]
```

**Padrões:**
- [Setup: ex.: "beforeEach para setup compartilhado, evitar beforeAll"]
- [Teardown: ex.: "afterEach para limpar, restaurar mocks"]
- [Estrutura: ex.: "padrão arrange/act/assert obrigatório"]

## Mocking

**Framework:**
- [Ferramenta: ex.: "Mocking built-in do Jest", "Vitest vi", "Sinon"]
- [Import mocking: ex.: "vi.mock() no topo do arquivo"]

**Padrões:**
```typescript
[Mostrar o padrão real de mocking, ex.:

// Mock de dependência externa
vi.mock('./external-service', () => ({
  fetchData: vi.fn()
}));

// Mock no teste
const mockFetch = vi.mocked(fetchData);
mockFetch.mockResolvedValue({ data: 'test' });
]
```

**O que Mockar:**
- [ex.: "APIs externas, sistema de arquivos, banco de dados"]
- [ex.: "Tempo/datas (usar vi.useFakeTimers)"]
- [ex.: "Chamadas de rede (usar mock fetch)"]

**O que NÃO Mockar:**
- [ex.: "Funções puras, utilitários"]
- [ex.: "Lógica de negócios interna"]

## Fixtures e Factories

**Dados de Teste:**
```typescript
[Mostrar padrão para criação de dados de teste, ex.:

// Padrão factory
function createTestUser(overrides?: Partial<User>): User {
  return {
    id: 'test-id',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  };
}

// Arquivo de fixture
// tests/fixtures/users.ts
export const mockUsers = [/* ... */];
]
```

**Local:**
- [ex.: "tests/fixtures/ para fixtures compartilhadas"]
- [ex.: "Funções factory no arquivo de teste ou tests/factories/"]

## Cobertura

**Requisitos:**
- [Meta: ex.: "80% de cobertura de linha", "sem meta específica"]
- [Enforcement: ex.: "CI bloqueia <80%", "cobertura apenas para referência"]

**Configuração:**
- [Ferramenta: ex.: "cobertura built-in via flag --coverage"]
- [Exclusões: ex.: "excluir *.test.ts, arquivos de configuração"]

**Ver Cobertura:**
```bash
[ex.: "npm run test:coverage"]
[ex.: "open coverage/index.html"]
```

## Tipos de Teste

**Testes Unitários:**
- [Escopo: ex.: "testar função/classe única em isolamento"]
- [Mocking: ex.: "mockar todas as dependências externas"]
- [Velocidade: ex.: "deve rodar em <1s por teste"]

**Testes de Integração:**
- [Escopo: ex.: "testar múltiplos módulos juntos"]
- [Mocking: ex.: "mockar serviços externos, usar módulos internos reais"]
- [Setup: ex.: "usar banco de dados de teste, seed de dados"]

**Testes E2E:**
- [Framework: ex.: "Playwright para E2E"]
- [Escopo: ex.: "testar fluxos completos do usuário"]
- [Local: ex.: "diretório e2e/ separado dos testes unitários"]

## Padrões Comuns

**Teste Assíncrono:**
```typescript
[Mostrar padrão, ex.:

it('deve lidar com operação assíncrona', async () => {
  const result = await asyncFunction();
  expect(result).toBe('expected');
});
]
```

**Teste de Erro:**
```typescript
[Mostrar padrão, ex.:

it('deve lançar em entrada inválida', () => {
  expect(() => functionCall()).toThrow('mensagem de erro');
});

// Erro assíncrono
it('deve rejeitar em caso de falha', async () => {
  await expect(asyncCall()).rejects.toThrow('mensagem de erro');
});
]
```

**Teste de Snapshot:**
- [Uso: ex.: "apenas para componentes React" ou "não usado"]
- [Local: ex.: "diretório __snapshots__/"]

---

*Análise de testes: [data]*
*Atualize quando os padrões de teste mudarem*
```

<good_examples>
```markdown
# Padrões de Teste

**Data da Análise:** 2025-01-20

## Framework de Teste

**Runner:**
- Vitest 1.0.4
- Configuração: vitest.config.ts na raiz do projeto

**Biblioteca de Asserção:**
- expect built-in do Vitest
- Matchers: toBe, toEqual, toThrow, toMatchObject

**Comandos de Execução:**
```bash
npm test                              # Executar todos os testes
npm test -- --watch                   # Modo watch
npm test -- path/to/file.test.ts     # Arquivo único
npm run test:coverage                 # Relatório de cobertura
```

## Organização dos Arquivos de Teste

**Local:**
- *.test.ts ao lado dos arquivos de código-fonte
- Sem diretório tests/ separado

**Nomenclatura:**
- nome-unitario.test.ts para todos os testes
- Sem distinção entre unitário/integração no nome do arquivo

**Estrutura:**
```
src/
  lib/
    parser.ts
    parser.test.ts
  services/
    install-service.ts
    install-service.test.ts
  bin/
    install.ts
    (sem teste - testado via CLI de integração)
```

## Estrutura dos Testes

**Organização dos Suites:**
```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';

describe('NomeDoMódulo', () => {
  describe('nomeDaFunção', () => {
    beforeEach(() => {
      // resetar estado
    });

    it('deve lidar com entrada válida', () => {
      // arrange
      const input = createTestInput();

      // act
      const result = functionName(input);

      // assert
      expect(result).toEqual(expectedOutput);
    });

    it('deve lançar em entrada inválida', () => {
      expect(() => functionName(null)).toThrow('Entrada inválida');
    });
  });
});
```

**Padrões:**
- Usar beforeEach para setup por teste, evitar beforeAll
- Usar afterEach para restaurar mocks: vi.restoreAllMocks()
- Comentários explícitos de arrange/act/assert em testes complexos
- Um foco de asserção por teste (mas múltiplos expects são aceitáveis)

## Mocking

**Framework:**
- Mocking built-in do Vitest (vi)
- Mock de módulo via vi.mock() no topo do arquivo de teste

**Padrões:**
```typescript
import { vi } from 'vitest';
import { externalFunction } from './external';

// Mock do módulo
vi.mock('./external', () => ({
  externalFunction: vi.fn()
}));

describe('suite de teste', () => {
  it('mocka função', () => {
    const mockFn = vi.mocked(externalFunction);
    mockFn.mockReturnValue('resultado mockado');

    // código de teste usando função mockada

    expect(mockFn).toHaveBeenCalledWith('argumento esperado');
  });
});
```

**O que Mockar:**
- Operações do sistema de arquivos (fs-extra)
- Execução de processo filho (child_process.exec)
- Chamadas de API externas
- Variáveis de ambiente (process.env)

**O que NÃO Mockar:**
- Funções puras internas
- Utilitários simples (manipulação de strings, helpers de array)
- Tipos TypeScript

## Fixtures e Factories

**Dados de Teste:**
```typescript
// Funções factory no arquivo de teste
function createTestConfig(overrides?: Partial<Config>): Config {
  return {
    targetDir: '/tmp/test',
    global: false,
    ...overrides
  };
}

// Fixtures compartilhadas em tests/fixtures/
// tests/fixtures/sample-command.md
export const sampleCommand = `---
description: Comando de teste
---
Conteúdo aqui`;
```

**Local:**
- Funções factory: definidas no arquivo de teste perto do uso
- Fixtures compartilhadas: tests/fixtures/ (para dados de teste em múltiplos arquivos)
- Dados mock: inline no teste quando simples, factory quando complexo

## Cobertura

**Requisitos:**
- Sem meta de cobertura obrigatória
- Cobertura rastreada para referência
- Foco em caminhos críticos (parsers, lógica de serviço)

**Configuração:**
- Cobertura do Vitest via c8 (built-in)
- Exclui: *.test.ts, bin/install.ts, arquivos de configuração

**Ver Cobertura:**
```bash
npm run test:coverage
open coverage/index.html
```

## Tipos de Teste

**Testes Unitários:**
- Testar função única em isolamento
- Mockar todas as dependências externas (fs, child_process)
- Rápidos: cada teste <100ms
- Exemplos: parser.test.ts, validator.test.ts

**Testes de Integração:**
- Testar múltiplos módulos juntos
- Mockar apenas fronteiras externas (sistema de arquivos, processo)
- Exemplos: install-service.test.ts (testa service + parser)

**Testes E2E:**
- Não usados atualmente
- Integração CLI testada manualmente

## Padrões Comuns

**Teste Assíncrono:**
```typescript
it('deve lidar com operação assíncrona', async () => {
  const result = await asyncFunction();
  expect(result).toBe('expected');
});
```

**Teste de Erro:**
```typescript
it('deve lançar em entrada inválida', () => {
  expect(() => parse(null)).toThrow('Não é possível analisar null');
});

// Erro assíncrono
it('deve rejeitar se arquivo não encontrado', async () => {
  await expect(readConfig('invalid.txt')).rejects.toThrow('ENOENT');
});
```

**Mock do Sistema de Arquivos:**
```typescript
import { vi } from 'vitest';
import * as fs from 'fs-extra';

vi.mock('fs-extra');

it('mocka o sistema de arquivos', () => {
  vi.mocked(fs.readFile).mockResolvedValue('conteúdo do arquivo');
  // código de teste
});
```

**Teste de Snapshot:**
- Não usado neste codebase
- Preferência por asserções explícitas para clareza

---

*Análise de testes: 2025-01-20*
*Atualize quando os padrões de teste mudarem*
```
</good_examples>

<guidelines>
**O que pertence ao TESTING.md:**
- Framework de teste e configuração do runner
- Local e padrões de nomenclatura dos arquivos de teste
- Estrutura dos testes (padrões describe/it, beforeEach)
- Abordagem de mocking e exemplos
- Padrões de fixture/factory
- Requisitos de cobertura
- Como executar os testes (comandos)
- Padrões de teste comuns no código real

**O que NÃO pertence aqui:**
- Casos de teste específicos (deixar para os arquivos de teste reais)
- Escolhas tecnológicas (isso é o STACK.md)
- Configuração de CI/CD (isso é a documentação de deploy)

**Ao preencher este template:**
- Verifique os scripts do package.json para comandos de teste
- Encontre o arquivo de configuração de teste (jest.config.js, vitest.config.ts)
- Leia 3-5 arquivos de teste existentes para identificar os padrões
- Procure utilitários de teste em tests/ ou test-utils/
- Verifique a configuração de cobertura
- Documente os padrões reais usados, não os ideais

**Útil para planejamento de fase quando:**
- Adicionando novas funcionalidades (escrever testes correspondentes)
- Refatorando (manter os padrões de teste)
- Corrigindo bugs (adicionar testes de regressão)
- Entendendo a abordagem de verificação
- Configurando a infraestrutura de teste

**Abordagem de análise:**
- Verificar package.json para framework de teste e scripts
- Ler arquivo de configuração de teste para cobertura, setup
- Examinar organização dos arquivos de teste (colocados vs separados)
- Revisar 5 arquivos de teste para padrões (mocking, estrutura, asserções)
- Procurar utilitários de teste, fixtures, factories
- Anotar tipos de teste (unitário, integração, e2e)
- Documentar comandos para executar os testes
</guidelines>
