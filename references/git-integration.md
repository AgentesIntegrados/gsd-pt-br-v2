<overview>
Integração Git para o framework GSD.
</overview>

<core_principle>

**Faça commit de resultados, não do processo.**

O log do git deve parecer um changelog do que foi entregue, não um diário de atividades de planejamento.
</core_principle>

<commit_points>

| Evento                   | Commit? | Por quê                                              |
| ----------------------- | ------- | ------------------------------------------------ |
| BRIEF + ROADMAP criados | SIM     | Inicialização do projeto                           |
| PLAN.md criado          | NÃO     | Intermediário - fazer commit junto com a conclusão do plano       |
| RESEARCH.md criado      | NÃO     | Intermediário                                     |
| DISCOVERY.md criado     | NÃO     | Intermediário                                     |
| **Tarefa concluída**    | SIM     | Unidade atômica de trabalho (1 commit por tarefa)         |
| **Plano concluído**     | SIM     | Commit de metadados (SUMMARY + STATE + ROADMAP)     |
| Handoff criado          | SIM     | Estado WIP preservado                              |

</commit_points>

<git_check>

```bash
[ -d .git ] && echo "GIT_EXISTS" || echo "NO_GIT"
```

Se NO_GIT: Execute `git init` silenciosamente. Projetos GSD sempre têm seu próprio repositório.
</git_check>

<commit_formats>

<format name="initialization">
## Inicialização do Projeto (brief + roadmap juntos)

```
docs: initialize [project-name] ([N] phases)

[Uma linha do PROJECT.md]

Phases:
1. [phase-name]: [goal]
2. [phase-name]: [goal]
3. [phase-name]: [goal]
```

O que fazer commit:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: initialize [project-name] ([N] phases)" --files .planning/
```

</format>

<format name="task-completion">
## Conclusão de Tarefa (Durante a Execução do Plano)

Cada tarefa recebe seu próprio commit imediatamente após a conclusão.

> **Agentes paralelos:** Ao executar como um executor paralelo (gerado por execute-phase),
> use `--no-verify` em todos os commits para evitar contenda de bloqueio de pre-commit hook.
> O orquestrador valida os hooks uma vez após todos os agentes concluírem.

```
{type}({phase}-{plan}): {task-name}

- [Mudança principal 1]
- [Mudança principal 2]
- [Mudança principal 3]
```

**Tipos de commit:**
- `feat` - Nova funcionalidade
- `fix` - Correção de bug
- `test` - Somente testes (fase RED do TDD)
- `refactor` - Limpeza de código (fase REFACTOR do TDD)
- `perf` - Melhoria de performance
- `chore` - Dependências, configuração, ferramentas

**Exemplos:**

```bash
# Tarefa padrão
git add src/api/auth.ts src/types/user.ts
git commit -m "feat(08-02): create user registration endpoint

- POST /auth/register validates email and password
- Checks for duplicate users
- Returns JWT token on success
"

# Tarefa TDD - fase RED
git add src/__tests__/jwt.test.ts
git commit -m "test(07-02): add failing test for JWT generation

- Tests token contains user ID claim
- Tests token expires in 1 hour
- Tests signature verification
"

# Tarefa TDD - fase GREEN
git add src/utils/jwt.ts
git commit -m "feat(07-02): implement JWT generation

- Uses jose library for signing
- Includes user ID and expiry claims
- Signs with HS256 algorithm
"
```

</format>

<format name="plan-completion">
## Conclusão do Plano (Após Todas as Tarefas Concluídas)

Após todos os commits de tarefas, um commit final de metadados registra a conclusão do plano.

```
docs({phase}-{plan}): complete [plan-name] plan

Tasks completed: [N]/[N]
- [Task 1 name]
- [Task 2 name]
- [Task 3 name]

SUMMARY: .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md
```

O que fazer commit:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}-{plan}): complete [plan-name] plan" --files .planning/phases/XX-name/{phase}-{plan}-PLAN.md .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md
```

**Nota:** Arquivos de código NÃO incluídos — já foram commitados por tarefa.

</format>

<format name="handoff">
## Handoff (WIP)

```
wip: [phase-name] paused at task [X]/[Y]

Current: [task name]
[If blocked:] Blocked: [reason]
```

O que fazer commit:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "wip: [phase-name] paused at task [X]/[Y]" --files .planning/
```

</format>
</commit_formats>

<example_log>

**Abordagem antiga (commits por plano):**
```
a7f2d1 feat(checkout): Stripe payments with webhook verification
3e9c4b feat(products): catalog with search, filters, and pagination
8a1b2c feat(auth): JWT with refresh rotation using jose
5c3d7e feat(foundation): Next.js 15 + Prisma + Tailwind scaffold
2f4a8d docs: initialize ecommerce-app (5 phases)
```

**Nova abordagem (commits por tarefa):**
```
# Phase 04 - Checkout
1a2b3c docs(04-01): complete checkout flow plan
4d5e6f feat(04-01): add webhook signature verification
7g8h9i feat(04-01): implement payment session creation
0j1k2l feat(04-01): create checkout page component

# Phase 03 - Products
3m4n5o docs(03-02): complete product listing plan
6p7q8r feat(03-02): add pagination controls
9s0t1u feat(03-02): implement search and filters
2v3w4x feat(03-01): create product catalog schema

# Phase 02 - Auth
5y6z7a docs(02-02): complete token refresh plan
8b9c0d feat(02-02): implement refresh token rotation
1e2f3g test(02-02): add failing test for token refresh
4h5i6j docs(02-01): complete JWT setup plan
7k8l9m feat(02-01): add JWT generation and validation
0n1o2p chore(02-01): install jose library

# Phase 01 - Foundation
3q4r5s docs(01-01): complete scaffold plan
6t7u8v feat(01-01): configure Tailwind and globals
9w0x1y feat(01-01): set up Prisma with database
2z3a4b feat(01-01): create Next.js 15 project

# Initialization
5c6d7e docs: initialize ecommerce-app (5 phases)
```

Cada plano produz 2-4 commits (tarefas + metadados). Claro, granular e rastreável.

</example_log>

<anti_patterns>

**Ainda não fazer commit (artefatos intermediários):**
- Criação de PLAN.md (fazer commit junto com a conclusão do plano)
- RESEARCH.md (intermediário)
- DISCOVERY.md (intermediário)
- Ajustes menores de planejamento
- "Corrigiu typo no roadmap"

**Fazer commit (resultados):**
- Cada conclusão de tarefa (feat/fix/test/refactor)
- Metadados de conclusão do plano (docs)
- Inicialização do projeto (docs)

**Princípio fundamental:** Faça commit de código funcionando e resultados entregues, não do processo de planejamento.

</anti_patterns>

<commit_strategy_rationale>

## Por que Commits por Tarefa?

**Engenharia de contexto para IA:**
- O histórico do Git se torna a principal fonte de contexto para sessões futuras do Claude
- `git log --grep="{phase}-{plan}"` mostra todo o trabalho de um plano
- `git diff <hash>^..<hash>` mostra as mudanças exatas por tarefa
- Menos dependência de parsear SUMMARY.md = mais contexto para o trabalho real

**Recuperação de falhas:**
- Tarefa 1 commitada ✅, Tarefa 2 falhou ❌
- Claude na próxima sessão: vê a tarefa 1 completa, pode tentar novamente a tarefa 2
- Pode usar `git reset --hard` para voltar à última tarefa bem-sucedida

**Depuração:**
- `git bisect` encontra a tarefa exata com falha, não apenas o plano com falha
- `git blame` rastreia a linha até o contexto da tarefa específica
- Cada commit é revertível de forma independente

**Observabilidade:**
- Fluxo de trabalho de desenvolvedor solo + Claude se beneficia de atribuição granular
- Commits atômicos são a melhor prática do Git
- "Ruído de commits" é irrelevante quando o consumidor é o Claude, não humanos

</commit_strategy_rationale>

<sub_repos_support>

## Suporte a Workspace Multi-Repositório (sub_repos)

Para workspaces com repositórios git separados (ex.: `backend/`, `frontend/`, `shared/`), o GSD encaminha commits para cada repositório de forma independente.

### Configuração

Em `.planning/config.json`, liste os diretórios dos sub-repositórios em `planning.sub_repos`:

```json
{
  "planning": {
    "commit_docs": false,
    "sub_repos": ["backend", "frontend", "shared"]
  }
}
```

Defina `commit_docs: false` para que os documentos de planejamento fiquem locais e não sejam commitados em nenhum sub-repositório.

### Como Funciona

1. **Auto-detecção:** Durante `/gsd-new-project`, diretórios com sua própria pasta `.git` são detectados e oferecidos para seleção como sub-repositórios. Em execuções subsequentes, `loadConfig` sincroniza automaticamente a lista `sub_repos` com o sistema de arquivos — adicionando repositórios recém-criados e removendo os excluídos. Isso significa que `config.json` pode ser reescrito automaticamente quando os repositórios mudam no disco.
2. **Agrupamento de arquivos:** Arquivos de código são agrupados pelo prefixo do sub-repositório (ex.: `backend/src/api/users.ts` pertence ao repositório `backend/`).
3. **Commits independentes:** Cada sub-repositório recebe seu próprio commit atômico via `gsd-tools.cjs commit-to-subrepo`. Os caminhos de arquivo são tornados relativos à raiz do sub-repositório antes do staging.
4. **Planejamento permanece local:** O diretório `.planning/` não é commitado; ele funciona como coordenação entre repositórios.

### Roteamento de Commits

Em vez do comando padrão `commit`, use `commit-to-subrepo` quando `sub_repos` estiver configurado:

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs commit-to-subrepo "feat(02-01): add user API" \
  --files backend/src/api/users.ts backend/src/types/user.ts frontend/src/components/UserForm.tsx
```

Isso faz o staging de `src/api/users.ts` e `src/types/user.ts` no repositório `backend/`, e `src/components/UserForm.tsx` no repositório `frontend/`, então commita cada um de forma independente com a mesma mensagem.

Arquivos que não correspondem a nenhum sub-repositório configurado são reportados como não correspondidos.

</sub_repos_support>
