# Template de Stack Tecnológica

Template para `.planning/codebase/STACK.md` - captura a base tecnológica.

**Propósito:** Documentar quais tecnologias executam este codebase. Focado em "o que é executado quando você roda o código."

---

## Template do Arquivo

```markdown
# Stack Tecnológica

**Data da Análise:** [YYYY-MM-DD]

## Linguagens

**Principal:**
- [Linguagem] [Versão] - [Onde é usada: ex.: "todo o código da aplicação"]

**Secundária:**
- [Linguagem] [Versão] - [Onde é usada: ex.: "scripts de build, ferramentas"]

## Runtime

**Ambiente:**
- [Runtime] [Versão] - [ex.: "Node.js 20.x"]
- [Requisitos adicionais, se houver]

**Gerenciador de Pacotes:**
- [Gerenciador] [Versão] - [ex.: "npm 10.x"]
- Lockfile: [ex.: "package-lock.json presente"]

## Frameworks

**Principal:**
- [Framework] [Versão] - [Propósito: ex.: "servidor web", "framework de UI"]

**Testes:**
- [Framework] [Versão] - [ex.: "Jest para testes unitários"]
- [Framework] [Versão] - [ex.: "Playwright para E2E"]

**Build/Dev:**
- [Ferramenta] [Versão] - [ex.: "Vite para bundling"]
- [Ferramenta] [Versão] - [ex.: "compilador TypeScript"]

## Dependências-Chave

[Inclua apenas dependências críticas para entender a stack - limite a 5-10 mais importantes]

**Críticas:**
- [Pacote] [Versão] - [Por que importa: ex.: "autenticação", "acesso ao banco de dados"]
- [Pacote] [Versão] - [Por que importa]

**Infraestrutura:**
- [Pacote] [Versão] - [ex.: "Express para roteamento HTTP"]
- [Pacote] [Versão] - [ex.: "cliente PostgreSQL"]

## Configuração

**Ambiente:**
- [Como configurado: ex.: "arquivos .env", "variáveis de ambiente"]
- [Configurações-chave: ex.: "DATABASE_URL, API_KEY necessários"]

**Build:**
- [Arquivos de configuração de build: ex.: "vite.config.ts, tsconfig.json"]

## Requisitos de Plataforma

**Desenvolvimento:**
- [Requisitos de SO ou "qualquer plataforma"]
- [Ferramentas adicionais: ex.: "Docker para BD local"]

**Produção:**
- [Destino de deploy: ex.: "Vercel", "AWS Lambda", "contêiner Docker"]
- [Requisitos de versão]

---

*Análise de stack: [data]*
*Atualize após mudanças importantes nas dependências*
```

<good_examples>
```markdown
# Stack Tecnológica

**Data da Análise:** 2025-01-20

## Linguagens

**Principal:**
- TypeScript 5.3 - Todo o código da aplicação

**Secundária:**
- JavaScript - Scripts de build, arquivos de configuração

## Runtime

**Ambiente:**
- Node.js 20.x (LTS)
- Sem runtime de navegador (ferramenta CLI apenas)

**Gerenciador de Pacotes:**
- npm 10.x
- Lockfile: `package-lock.json` presente

## Frameworks

**Principal:**
- Nenhum (CLI Node.js puro)

**Testes:**
- Vitest 1.0 - Testes unitários
- tsx - Execução TypeScript sem etapa de build

**Build/Dev:**
- TypeScript 5.3 - Compilação para JavaScript
- esbuild - Usado pelo Vitest para transformações rápidas

## Dependências-Chave

**Críticas:**
- commander 11.x - Análise de argumentos CLI e estrutura de comandos
- chalk 5.x - Estilização de saída no terminal
- fs-extra 11.x - Operações estendidas do sistema de arquivos

**Infraestrutura:**
- Built-ins do Node.js - fs, path, child_process para operações de arquivo

## Configuração

**Ambiente:**
- Sem variáveis de ambiente necessárias
- Configuração via flags da CLI apenas

**Build:**
- `tsconfig.json` - Opções do compilador TypeScript
- `vitest.config.ts` - Configuração do runner de testes

## Requisitos de Plataforma

**Desenvolvimento:**
- macOS/Linux/Windows (qualquer plataforma com Node.js)
- Sem dependências externas

**Produção:**
- Distribuído como pacote npm
- Instalado globalmente via npm install -g
- Roda na instalação do Node.js do usuário

---

*Análise de stack: 2025-01-20*
*Atualize após mudanças importantes nas dependências*
```
</good_examples>

<guidelines>
**O que pertence ao STACK.md:**
- Linguagens e versões
- Requisitos de runtime (Node, Bun, Deno, navegador)
- Gerenciador de pacotes e lockfile
- Escolhas de framework
- Dependências críticas (limite a 5-10 mais importantes)
- Ferramentas de build
- Requisitos de plataforma/deploy

**O que NÃO pertence aqui:**
- Estrutura de arquivos (isso é o STRUCTURE.md)
- Padrões arquiteturais (isso é o ARCHITECTURE.md)
- Toda dependência do package.json (apenas as críticas)
- Detalhes de implementação (deixar para o código)

**Ao preencher este template:**
- Verifique o package.json para as dependências
- Anote a versão do runtime do .nvmrc ou engines do package.json
- Inclua apenas dependências que afetam a compreensão (não todo utilitário)
- Especifique versões apenas quando a versão importa (mudanças disruptivas, compatibilidade)

**Útil para planejamento de fase quando:**
- Adicionando novas dependências (verificar compatibilidade)
- Atualizando frameworks (saber o que está em uso)
- Escolhendo a abordagem de implementação (deve funcionar com a stack existente)
- Entendendo os requisitos de build
</guidelines>
