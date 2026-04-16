<purpose>
Analise texto livre do usuário e direcione para o comando GSD mais adequado. Este é um despachante — nunca faz o trabalho diretamente. Identifique a intenção do usuário no melhor comando correspondente, confirme o roteamento e transfira.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt de invocação antes de começar.
</required_reading>

<process>

<step name="validate">
**Verifique se há input.**


**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON init for `true`. Quando TEXT_MODE estiver ativo, substitua toda chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Se `$ARGUMENTS` estiver vazio, pergunte via AskUserQuestion:

```
O que você gostaria de fazer? Descreva a tarefa, o bug ou a ideia e eu direcionarei para o comando GSD correto.
```

Aguarde a resposta antes de continuar.
</step>

<step name="check_project">
**Verifique se o projeto existe.**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state load 2>/dev/null)
```

Rastreie se `.planning/` existe — algumas rotas precisam disso, outras não.
</step>

<step name="route">
**Identifique a intenção no comando.**

Avalie `$ARGUMENTS` contra estas regras de roteamento. Aplique a **primeira regra que corresponder**:

| Se o texto descreve... | Direcione para | Motivo |
|------------------------|----------------|--------|
| Iniciar um novo projeto, "configurar", "inicializar" | `/gsd-new-project` | Precisa de inicialização completa do projeto |
| Mapear ou analisar um código-base existente | `/gsd-map-codebase` | Descoberta do código-base |
| Um bug, erro, falha, ou algo quebrado | `/gsd-debug` | Precisa de investigação sistemática |
| Explorar, pesquisar, comparar, ou "como funciona X" | `/gsd-research-phase` | Pesquisa de domínio antes do planejamento |
| Discutir visão, "como X deve parecer", brainstorming | `/gsd-discuss-phase` | Precisa de coleta de contexto |
| Uma tarefa complexa: refatoração, migração, arquitetura multi-arquivo, redesign de sistema | `/gsd-add-phase` | Precisa de uma fase completa com ciclo de plano/construção |
| Planejar uma fase específica ou "planejar fase N" | `/gsd-plan-phase` | Solicitação direta de planejamento |
| Executar uma fase ou "construir fase N", "executar fase N" | `/gsd-execute-phase` | Solicitação direta de execução |
| Executar todas as fases restantes automaticamente | `/gsd-autonomous` | Execução autônoma completa |
| Uma revisão ou preocupação de qualidade sobre trabalho existente | `/gsd-verify-work` | Precisa de verificação |
| Verificar progresso, status, "onde estou" | `/gsd-progress` | Verificação de status |
| Retomar trabalho, "continuar de onde parei" | `/gsd-resume-work` | Restauração de sessão |
| Uma nota, ideia, ou "lembrar de..." | `/gsd-add-todo` | Capturar para depois |
| Adicionar testes, "escrever testes", "cobertura de testes" | `/gsd-add-tests` | Geração de testes |
| Completar um marco, lançar, liberar | `/gsd-complete-milestone` | Ciclo de vida do marco |
| Uma tarefa pequena, específica e acionável (adicionar funcionalidade, corrigir typo, atualizar config) | `/gsd-quick` | Autocontido, executor único |

**Requer diretório `.planning/`:** Todas as rotas exceto `/gsd-new-project`, `/gsd-map-codebase`, `/gsd-help`, e `/gsd-join-discord`. Se o projeto não existir e a rota precisar dele, sugira `/gsd-new-project` primeiro.

**Tratamento de ambiguidade:** Se o texto puder razoavelmente corresponder a múltiplas rotas, pergunte ao usuário via AskUserQuestion com as 2-3 melhores opções. Por exemplo:

```
"Refatorar o sistema de autenticação" pode ser:
1. /gsd-add-phase — Ciclo completo de planejamento (recomendado para refatorações multi-arquivo)
2. /gsd-quick — Execução rápida (se o escopo for pequeno e claro)

Qual abordagem é mais adequada?
```
</step>

<step name="display">
**Mostre a decisão de roteamento.**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► ROTEAMENTO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Input:** {primeiros 80 caracteres de $ARGUMENTS}
**Direcionando para:** {comando escolhido}
**Motivo:** {explicação em uma linha}
```
</step>

<step name="dispatch">
**Invoque o comando escolhido.**

Execute o comando `/gsd-*` selecionado, passando `$ARGUMENTS` como args.

Se o comando escolhido espera um número de fase e um não foi fornecido no texto, extraia do contexto ou pergunte via AskUserQuestion.

Após invocar o comando, pare. O comando despachado cuida de tudo a partir daqui.
</step>

</process>

<success_criteria>
- [ ] Input validado (não vazio)
- [ ] Intenção identificada em exatamente um comando GSD
- [ ] Ambiguidade resolvida via pergunta ao usuário (se necessário)
- [ ] Existência do projeto verificada para rotas que precisam dela
- [ ] Decisão de roteamento exibida antes do despacho
- [ ] Comando invocado com argumentos apropriados
- [ ] Nenhum trabalho feito diretamente — apenas despachante
</success_criteria>
