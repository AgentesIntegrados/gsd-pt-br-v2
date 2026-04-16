# Template de Estrutura

Template para `.planning/codebase/STRUCTURE.md` - captura a organização física dos arquivos.

**Propósito:** Documentar onde as coisas estão fisicamente no codebase. Responde "onde coloco X?"

---

## Template do Arquivo

```markdown
# Estrutura do Codebase

**Data da Análise:** [YYYY-MM-DD]

## Layout de Diretórios

[Árvore ASCII com os diretórios de nível superior e seus propósitos - use os caracteres ├── └── │ apenas para a estrutura em árvore]

```
[raiz-do-projeto]/
├── [dir]/          # [Propósito]
├── [dir]/          # [Propósito]
├── [dir]/          # [Propósito]
└── [arquivo]       # [Propósito]
```

## Propósitos dos Diretórios

**[Nome do Diretório]:**
- Propósito: [O que reside aqui]
- Contém: [Tipos de arquivos: ex.: "arquivos-fonte *.ts", "diretórios de componentes"]
- Arquivos-chave: [Arquivos importantes neste diretório]
- Subdiretórios: [Se aninhados, descreva a estrutura]

**[Nome do Diretório]:**
- Propósito: [O que reside aqui]
- Contém: [Tipos de arquivos]
- Arquivos-chave: [Arquivos importantes]
- Subdiretórios: [Estrutura]

## Locais de Arquivos-Chave

**Pontos de Entrada:**
- [Caminho]: [Propósito: ex.: "ponto de entrada da CLI"]
- [Caminho]: [Propósito: ex.: "inicialização do servidor"]

**Configuração:**
- [Caminho]: [Propósito: ex.: "configuração TypeScript"]
- [Caminho]: [Propósito: ex.: "configuração de build"]
- [Caminho]: [Propósito: ex.: "variáveis de ambiente"]

**Lógica Principal:**
- [Caminho]: [Propósito: ex.: "serviços de negócios"]
- [Caminho]: [Propósito: ex.: "models do banco de dados"]
- [Caminho]: [Propósito: ex.: "rotas de API"]

**Testes:**
- [Caminho]: [Propósito: ex.: "testes unitários"]
- [Caminho]: [Propósito: ex.: "fixtures de teste"]

**Documentação:**
- [Caminho]: [Propósito: ex.: "documentação voltada ao usuário"]
- [Caminho]: [Propósito: ex.: "guia do desenvolvedor"]

## Convenções de Nomenclatura

**Arquivos:**
- [Padrão]: [Exemplo: ex.: "kebab-case.ts para módulos"]
- [Padrão]: [Exemplo: ex.: "PascalCase.tsx para componentes React"]
- [Padrão]: [Exemplo: ex.: "*.test.ts para arquivos de teste"]

**Diretórios:**
- [Padrão]: [Exemplo: ex.: "kebab-case para diretórios de funcionalidades"]
- [Padrão]: [Exemplo: ex.: "nomes no plural para coleções"]

**Padrões Especiais:**
- [Padrão]: [Exemplo: ex.: "index.ts para exports de diretório"]
- [Padrão]: [Exemplo: ex.: "__tests__ para diretórios de teste"]

## Onde Adicionar Novo Código

**Nova Funcionalidade:**
- Código principal: [Caminho do diretório]
- Testes: [Caminho do diretório]
- Configuração, se necessário: [Caminho do diretório]

**Novo Componente/Módulo:**
- Implementação: [Caminho do diretório]
- Tipos: [Caminho do diretório]
- Testes: [Caminho do diretório]

**Nova Rota/Comando:**
- Definição: [Caminho do diretório]
- Handler: [Caminho do diretório]
- Testes: [Caminho do diretório]

**Utilitários:**
- Helpers compartilhados: [Caminho do diretório]
- Definições de tipos: [Caminho do diretório]

## Diretórios Especiais

[Quaisquer diretórios com significado especial ou geração]

**[Diretório]:**
- Propósito: [ex.: "Código gerado", "Saída de build"]
- Fonte: [ex.: "Auto-gerado por X", "Artefatos de build"]
- Commitado: [Sim/Não - no .gitignore?]

---

*Análise de estrutura: [data]*
*Atualize quando a estrutura de diretórios mudar*
```

<good_examples>
```markdown
# Estrutura do Codebase

**Data da Análise:** 2025-01-20

## Layout de Diretórios

```
get-shit-done/
├── bin/                # Pontos de entrada executáveis
├── commands/           # Definições de comandos slash
│   └── gsd/           # Comandos específicos do GSD
├── get-shit-done/     # Recursos de habilidade
│   ├── references/    # Documentos de princípios
│   ├── templates/     # Templates de arquivo
│   └── workflows/     # Procedimentos de múltiplas etapas
├── src/               # Código-fonte (se aplicável)
├── tests/             # Arquivos de teste
├── package.json       # Manifesto do projeto
└── README.md          # Documentação para o usuário
```

## Propósitos dos Diretórios

**bin/**
- Propósito: Pontos de entrada da CLI
- Contém: install.js (script de instalação)
- Arquivos-chave: install.js - lida com a instalação via npx
- Subdiretórios: Nenhum

**commands/gsd/**
- Propósito: Definições de comandos slash para o Claude Code
- Contém: arquivos *.md (um por comando)
- Arquivos-chave: new-project.md, plan-phase.md, execute-plan.md
- Subdiretórios: Nenhum (estrutura plana)

**get-shit-done/references/**
- Propósito: Documentos de filosofia e orientação principal
- Contém: principles.md, questioning.md, plan-format.md
- Arquivos-chave: principles.md - filosofia do sistema
- Subdiretórios: Nenhum

**get-shit-done/templates/**
- Propósito: Templates de documento para arquivos .planning/
- Contém: Definições de template com frontmatter
- Arquivos-chave: project.md, roadmap.md, plan.md, summary.md
- Subdiretórios: codebase/ (novo - para templates de stack/arquitetura/estrutura)

**get-shit-done/workflows/**
- Propósito: Procedimentos reutilizáveis de múltiplas etapas
- Contém: Definições de workflow chamadas pelos comandos
- Arquivos-chave: execute-plan.md, research-phase.md
- Subdiretórios: Nenhum

## Locais de Arquivos-Chave

**Pontos de Entrada:**
- `bin/install.js` - Script de instalação (ponto de entrada npx)

**Configuração:**
- `package.json` - Metadados do projeto, dependências, entrada bin
- `.gitignore` - Arquivos excluídos

**Lógica Principal:**
- `bin/install.js` - Toda a lógica de instalação (cópia de arquivo, substituição de caminho)

**Testes:**
- `tests/` - Arquivos de teste (se presentes)

**Documentação:**
- `README.md` - Guia de instalação e uso voltado ao usuário
- `CLAUDE.md` - Instruções para o Claude Code ao trabalhar neste repositório

## Convenções de Nomenclatura

**Arquivos:**
- kebab-case.md: Documentos Markdown
- kebab-case.js: Arquivos-fonte JavaScript
- MAIUSCULAS.md: Arquivos importantes do projeto (README, CLAUDE, CHANGELOG)

**Diretórios:**
- kebab-case: Todos os diretórios
- Plural para coleções: templates/, commands/, workflows/

**Padrões Especiais:**
- {nome-do-comando}.md: Definição de comando slash
- *-template.md: Poderia ser usado, mas o diretório templates/ é preferido

## Onde Adicionar Novo Código

**Novo Comando Slash:**
- Código principal: `commands/gsd/{nome-do-comando}.md`
- Testes: `tests/commands/{nome-do-comando}.test.js` (se o teste estiver implementado)
- Documentação: Atualizar `README.md` com o novo comando

**Novo Template:**
- Implementação: `get-shit-done/templates/{nome}.md`
- Documentação: O template é autodocumentado (inclui diretrizes)

**Novo Workflow:**
- Implementação: `get-shit-done/workflows/{nome}.md`
- Uso: Referenciar do comando com `@$HOME/.claude/get-shit-done/workflows/{nome}.md`

**Novo Documento de Referência:**
- Implementação: `get-shit-done/references/{nome}.md`
- Uso: Referenciar dos comandos/workflows conforme necessário

**Utilitários:**
- Sem utilitários ainda (`install.js` é monolítico)
- Se extraído: `src/utils/`

## Diretórios Especiais

**get-shit-done/**
- Propósito: Recursos instalados em $HOME/.claude/
- Fonte: Copiado por bin/install.js durante a instalação
- Commitado: Sim (fonte da verdade)

**commands/**
- Propósito: Comandos slash instalados em $HOME/.claude/commands/
- Fonte: Copiado por bin/install.js durante a instalação
- Commitado: Sim (fonte da verdade)

---

*Análise de estrutura: 2025-01-20*
*Atualize quando a estrutura de diretórios mudar*
```
</good_examples>

<guidelines>
**O que pertence ao STRUCTURE.md:**
- Layout de diretórios (árvore ASCII para visualização da estrutura)
- Propósito de cada diretório
- Locais de arquivos-chave (pontos de entrada, configs, lógica principal)
- Convenções de nomenclatura
- Onde adicionar novo código (por tipo)
- Diretórios especiais/gerados

**O que NÃO pertence aqui:**
- Arquitetura conceitual (isso é o ARCHITECTURE.md)
- Stack tecnológica (isso é o STACK.md)
- Detalhes de implementação de código (deixar para leitura do código)
- Todo arquivo individual (focar nos diretórios e arquivos-chave)

**Ao preencher este template:**
- Use `tree -L 2` ou similar para visualizar a estrutura
- Identifique os diretórios de nível superior e seus propósitos
- Anote padrões de nomenclatura observando os arquivos existentes
- Localize pontos de entrada, configs e áreas de lógica principal
- Mantenha a árvore de diretórios concisa (máximo 2-3 níveis)

**Formato da árvore (caracteres ASCII box-drawing apenas para estrutura):**
```
raiz/
├── dir1/           # Propósito
│   ├── subdir/    # Propósito
│   └── arquivo.ts # Propósito
├── dir2/          # Propósito
└── arquivo.ts     # Propósito
```

**Útil para planejamento de fase quando:**
- Adicionando novas funcionalidades (onde os arquivos devem ir?)
- Entendendo a organização do projeto
- Encontrando onde uma lógica específica reside
- Seguindo convenções existentes
</guidelines>
