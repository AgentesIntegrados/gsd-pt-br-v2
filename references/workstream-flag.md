# Flag de Workstream (`--ws`)

## Visão Geral

A flag `--ws <name>` delimita as operações do GSD a um workstream específico, permitindo
trabalho paralelo em marcos por múltiplas instâncias do Claude Code na mesma base de código.

## Prioridade de Resolução

1. Flag `--ws <name>` (explícita, maior prioridade)
2. Variável de ambiente `GSD_WORKSTREAM` (por instância)
3. Ponteiro de workstream ativo com escopo de sessão no armazenamento temporário (por sessão de runtime / terminal)
4. Arquivo `.planning/active-workstream` (fallback compartilhado legado quando não há chave de sessão)
5. `null` — modo plano (sem workstreams)

## Por que ponteiros com escopo de sessão existem

O arquivo compartilhado `.planning/active-workstream` é fundamentalmente inseguro quando múltiplas
instâncias do Claude/Codex estão ativas no mesmo repositório ao mesmo tempo. Uma sessão pode
silenciosamente redirecionar o `STATE.md`, `ROADMAP.md` e caminhos de fase de outra sessão.

O GSD agora prefere um ponteiro com escopo de sessão identificado pela identidade do runtime/sessão
(`GSD_SESSION_KEY`, `CODEX_THREAD_ID`, `CLAUDE_CODE_SSE_PORT`, IDs de sessão de terminal,
ou o TTY controlador). Isso mantém sessões concorrentes isoladas enquanto preserva
compatibilidade legada para runtimes que não expõem uma chave de sessão estável.

## Resolução de Identidade de Sessão

Quando o GSD resolve o ponteiro com escopo de sessão no passo 3 acima, ele usa esta ordem:

1. Variáveis de ambiente explícitas de runtime/sessão como `GSD_SESSION_KEY`, `CODEX_THREAD_ID`,
   `CLAUDE_SESSION_ID`, `CLAUDE_CODE_SSE_PORT`, `OPENCODE_SESSION_ID`,
   `GEMINI_SESSION_ID`, `CURSOR_SESSION_ID`, `WINDSURF_SESSION_ID`,
   `TERM_SESSION_ID`, `WT_SESSION`, `TMUX_PANE` e `ZELLIJ_SESSION_NAME`
2. `TTY` ou `SSH_TTY` se o shell/runtime já expõe o caminho do terminal
3. Uma única sondagem `tty` de melhor esforço, mas apenas quando stdin é interativo

Se nenhum desses produzir uma identidade estável, o GSD não continua sondando. Ele faz
fallback diretamente para o arquivo compartilhado legado `.planning/active-workstream`.

Isso importa em ambientes headless ou despidos: quando stdin já é
não-interativo, o GSD intencionalmente pula a execução de shell para `tty` porque esse caminho
não pode descobrir uma identidade de sessão estável e apenas adiciona falhas evitáveis no
hot path de roteamento.

## Ciclo de Vida do Ponteiro

Ponteiros com escopo de sessão são intencionalmente leves e de melhor esforço:

- Limpar um workstream para uma sessão remove apenas o arquivo de ponteiro daquela sessão
- Se esse foi o último ponteiro para o repositório, o GSD também remove o
  diretório temporário por projeto agora vazio
- Se ponteiros de sessão irmãs ainda existem, o diretório temporário é mantido no lugar
- Quando um ponteiro se refere a um diretório de workstream que não existe mais, o GSD
  trata como estado obsoleto: remove esse arquivo de ponteiro e resolve para `null`
  até que a sessão defina explicitamente um novo workstream ativo novamente

O GSD atualmente não executa um coletor de lixo em segundo plano para diretórios temporários históricos.
A limpeza é oportunista no ponteiro sendo limpo ou auto-curado,
e a higiene mais ampla de temporários é deixada para a limpeza de temporários do SO ou trabalho de manutenção futuro.

## Propagação de Roteamento

Todos os comandos de roteamento de fluxo de trabalho incluem `${GSD_WS}` que:
- Expande para `--ws <name>` quando um workstream está ativo
- Expande para string vazia no modo plano (compatível com versões anteriores)

Isso garante que o escopo do workstream se encadeie automaticamente pelo fluxo de trabalho:
`new-milestone → discuss-phase → plan-phase → execute-phase → transition`

## Estrutura de Diretório

```
.planning/
├── PROJECT.md          # Compartilhado
├── config.json         # Compartilhado
├── milestones/         # Compartilhado
├── codebase/           # Compartilhado
├── active-workstream   # Apenas fallback compartilhado legado
└── workstreams/
    ├── feature-a/      # Workstream A
    │   ├── STATE.md
    │   ├── ROADMAP.md
    │   ├── REQUIREMENTS.md
    │   └── phases/
    └── feature-b/      # Workstream B
        ├── STATE.md
        ├── ROADMAP.md
        ├── REQUIREMENTS.md
        └── phases/
```

## Uso da CLI

```bash
# Todos os comandos gsd-tools aceitam --ws
node gsd-tools.cjs state json --ws feature-a
node gsd-tools.cjs find-phase 3 --ws feature-b

# Alternância local de sessão sem --ws em cada comando
GSD_SESSION_KEY=my-terminal-a node gsd-tools.cjs workstream set feature-a
GSD_SESSION_KEY=my-terminal-a node gsd-tools.cjs state json
GSD_SESSION_KEY=my-terminal-b node gsd-tools.cjs workstream set feature-b
GSD_SESSION_KEY=my-terminal-b node gsd-tools.cjs state json

# CRUD de Workstream
node gsd-tools.cjs workstream create <name>
node gsd-tools.cjs workstream list
node gsd-tools.cjs workstream status <name>
node gsd-tools.cjs workstream complete <name>
```
