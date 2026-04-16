# Template CLAUDE.md

Template para o `CLAUDE.md` na raiz do projeto — gerado automaticamente por `gsd-tools generate-claude-md`.

Contém 7 seções delimitadas por marcadores. Cada seção pode ser atualizada independentemente.
O subcomando `generate-claude-md` gerencia 6 seções (projeto, stack, convenções, arquitetura, skills, aplicação do workflow).
A seção de perfil é gerenciada exclusivamente por `generate-claude-profile`.

---

## Templates de Seção

### Seção de Projeto
```
<!-- GSD:project-start source:PROJECT.md -->
## Projeto

{{project_content}}
<!-- GSD:project-end -->
```

**Texto de fallback:**
```
Projeto ainda não inicializado. Execute /gsd-new-project para configurar.
```

### Seção de Stack
```
<!-- GSD:stack-start source:STACK.md -->
## Stack de Tecnologia

{{stack_content}}
<!-- GSD:stack-end -->
```

**Texto de fallback:**
```
Stack de tecnologia ainda não documentada. Será preenchida após o mapeamento do codebase ou na primeira fase.
```

### Seção de Convenções
```
<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Convenções

{{conventions_content}}
<!-- GSD:conventions-end -->
```

**Texto de fallback:**
```
Convenções ainda não estabelecidas. Serão preenchidas conforme padrões emergem durante o desenvolvimento.
```

### Seção de Arquitetura
```
<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Arquitetura

{{architecture_content}}
<!-- GSD:architecture-end -->
```

**Texto de fallback:**
```
Arquitetura ainda não mapeada. Siga os padrões existentes encontrados no codebase.
```

### Seção de Skills
```
<!-- GSD:skills-start source:skills/ -->
## Skills do Projeto

| Skill          | Descrição             | Caminho                   |
| -------------- | --------------------- | ------------------------- |
| {{skill_name}} | {{skill_description}} | `{{skill_path}}/SKILL.md` |
<!-- GSD:skills-end -->
```

**Texto de fallback:**
```
Nenhuma skill de projeto encontrada. Adicione skills em qualquer um dos diretórios: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/` ou `.github/skills/` com um arquivo de índice `SKILL.md`.
```

**Comportamento de descoberta:**
- Varre `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/` em busca de subdiretórios contendo `SKILL.md`
- Extrai `name` e `description` do frontmatter YAML (suporta descrições multi-linha)
- Ignora as skills instaladas pelo próprio GSD (diretórios começando com `gsd-`)
- Remove duplicatas pelo nome da skill entre os diretórios

### Seção de Aplicação do Workflow
```
<!-- GSD:workflow-start source:GSD defaults -->
## Aplicação do Workflow GSD

Antes de usar Edit, Write ou outras ferramentas que modificam arquivos, inicie o trabalho por um comando GSD para manter os artefatos de planejamento e o contexto de execução sincronizados.

Use estes pontos de entrada:
- `/gsd-quick` para pequenos ajustes, atualizações de documentação e tarefas ad-hoc
- `/gsd-debug` para investigação e correção de bugs
- `/gsd-execute-phase` para trabalho de fase planejada

Não faça edições diretas no repositório fora de um workflow GSD, a menos que o usuário solicite explicitamente ignorar essa regra.
<!-- GSD:workflow-end -->
```

### Seção de Perfil (Apenas Placeholder)
```
<!-- GSD:profile-start -->
## Perfil do Desenvolvedor

> Perfil ainda não configurado. Execute `/gsd-profile-user` para gerar seu perfil de desenvolvedor.
> Esta seção é gerenciada por `generate-claude-profile` — não edite manualmente.
<!-- GSD:profile-end -->
```

**Nota:** Esta seção NÃO é gerenciada por `generate-claude-md`. É gerenciada exclusivamente
por `generate-claude-profile`. O placeholder acima é usado apenas ao criar um novo
arquivo CLAUDE.md quando ainda não existe uma seção de perfil.

---

## Ordenação das Seções

1. **Projeto** — Identidade e propósito (o que é este projeto)
2. **Stack** — Escolhas de tecnologia (quais ferramentas são utilizadas)
3. **Convenções** — Padrões e regras de código (como o código é escrito)
4. **Arquitetura** — Estrutura do sistema (como os componentes se encaixam)
5. **Skills** — Skills descobertas do projeto com nome e descrição (qual conhecimento de domínio está disponível)
6. **Aplicação do Workflow** — Pontos de entrada padrão do GSD para trabalho que modifica arquivos
7. **Perfil** — Preferências comportamentais do desenvolvedor (como interagir)

## Formato dos Marcadores

- Início: `<!-- GSD:{name}-start source:{file} -->`
- Fim: `<!-- GSD:{name}-end -->`
- O atributo source permite atualizações direcionadas quando os arquivos fonte mudam
- Correspondência parcial no marcador de início (sem o `-->` de fechamento) para detecção

## Comportamento de Fallback

Quando um arquivo fonte está ausente, o texto de fallback fornece orientação acionável para o Claude:
- Guia o comportamento do Claude na ausência de dados
- Não são placeholders de anúncio ou avisos de "ausente"
- Cada fallback diz ao Claude o que fazer, não apenas o que está faltando
