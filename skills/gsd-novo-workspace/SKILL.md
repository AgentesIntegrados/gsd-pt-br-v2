---
name: gsd-novo-workspace
description: "Criar um workspace isolado com cópias de repositórios e .planning/ independente"
argument-hint: "--name <nome> [--repos repo1,repo2] [--path /destino] [--strategy worktree|clone] [--branch nome] [--auto]"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---

<context>
**Flags:**
- `--name` (obrigatório) — Nome do workspace
- `--repos` — Caminhos ou nomes de repositórios separados por vírgula. Se omitido, seleção interativa dos repositórios git filhos no diretório atual
- `--path` — Diretório de destino. Padrão: `~/gsd-workspaces/<nome>`
- `--strategy` — `worktree` (padrão, leve) ou `clone` (totalmente independente)
- `--branch` — Branch para checkout. Padrão: `workspace/<nome>`
- `--auto` — Pular perguntas interativas, usar padrões
</context>

<objective>
Criar um diretório de workspace físico contendo cópias dos repositórios git especificados (como worktrees ou clones) com um diretório `.planning/` independente para sessões GSD isoladas.

**Casos de uso:**
- Orquestração multi-repositório: trabalhar em um subconjunto de repositórios em paralelo com estado GSD isolado
- Isolamento de feature branch: criar uma worktree do repositório atual com seu próprio `.planning/`

**Cria:**
- `<path>/WORKSPACE.md` — manifesto do workspace
- `<path>/.planning/` — diretório de planejamento independente
- `<path>/<repo>/` — git worktree ou clone para cada repositório especificado

**Após este comando:** Entre no workspace com `cd` e execute `/gsd-new-project` para inicializar o GSD.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/new-workspace.md
@$HOME/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<process>
Executar o workflow new-workspace de @$HOME/.claude/get-shit-done/workflows/new-workspace.md do início ao fim.
Preservar todos os pontos de controle do workflow (validação, aprovações, commits, roteamento).
</process>
</content>
