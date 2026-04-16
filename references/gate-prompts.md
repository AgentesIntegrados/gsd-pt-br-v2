# Padrões de Prompts de Portão

Padrões de prompt reutilizáveis para verificações estruturadas de portão em workflows e agentes.

**Para detalhes sobre o formato da caixa de checkpoint, consulte `references/ui-brand.md`** -- as caixas de checkpoint usam bordas com traços duplos e largura interna de 62 caracteres.

## Regras

- `header` deve ter no máximo 12 caracteres
- `multiSelect` é sempre `false` para verificações de portão
- Sempre trate o caso "Outro" (usuário digitou uma resposta freeform em vez de selecionar)
- Máximo de 4 opções por prompt -- se mais forem necessárias, use um fluxo em 2 etapas

---

## Padrão: aprovar-revisar-abortar
Portão de 3 opções para aprovação de plano, aprovação de fechamento de lacunas.
- question: "Aprovar estes {substantivo}?"
- header: "Aprovar?"
- options: Aprovar | Solicitar alterações | Abortar

## Padrão: sim-não
Confirmação simples de 2 opções para replanejar, reconstruir, substituir planos, commit.
- question: "{Pergunta específica sobre a ação}"
- header: "Confirmar"
- options: Sim | Não

## Padrão: desatualizado-continuar
Portão de atualização de 2 opções para avisos de desatualização, frescor de timestamp.
- question: "{Artefato} pode estar desatualizado. Atualizar ou continuar?"
- header: "Desatualizado"
- options: Atualizar | Continuar mesmo assim

## Padrão: sim-não-escolher
Seleção de 3 opções para seleção de seed, inclusão de itens.
- question: "Incluir {itens} no planejamento?"
- header: "Incluir?"
- options: Sim, todos | Deixar eu escolher | Não

## Padrão: falha-multiplas-opcoes
Tratador de falha de 4 opções para falhas de build.
- question: "O plano {id} falhou. Como devemos prosseguir?"
- header: "Falhou"
- options: Tentar novamente | Pular | Reverter | Abortar

## Padrão: escalada-multiplas-opcoes
Escalada de 4 opções para escalada de revisão (máximo de tentativas excedido).
- question: "A Fase {N} falhou na verificação {tentativa} vezes. Como devemos prosseguir?"
- header: "Escalar"
- options: Aceitar lacunas | Replanejar (via /gsd-plan-phase) | Depurar (via /gsd-debug) | Tentar novamente

## Padrão: lacunas-multiplas-opcoes
Tratador de lacunas de 4 opções para lacunas encontradas na revisão.
- question: "{contagem} lacunas de verificação precisam de atenção. Como devemos prosseguir?"
- header: "Lacunas"
- options: Corrigir automaticamente | Substituir | Manual | Pular

## Padrão: prioridade-multiplas-opcoes
Seleção de prioridade de 4 opções para prioridade de lacuna de marco.
- question: "Quais lacunas devemos abordar?"
- header: "Prioridade"
- options: Somente obrigatórias | Obrigatórias + desejáveis | Tudo | Deixar eu escolher

## Padrão: alternar-confirmar
Confirmação de 2 opções para habilitar/desabilitar funcionalidades booleanas.
- question: "Habilitar {nome_da_funcionalidade}?"
- header: "Alternar"
- options: Habilitar | Desabilitar

## Padrão: roteamento-de-acao
Até 4 próximas ações sugeridas com seleção (status, workflows de retomada).
- question: "O que você gostaria de fazer a seguir?"
- header: "Próximo Passo"
- options: {ação principal} | {alternativa 1} | {alternativa 2} | Outra coisa
- Nota: Gere as opções dinamicamente a partir do estado do workflow. Sempre inclua "Outra coisa" como última opção.

## Padrão: confirmar-escopo
Confirmação de 3 opções para validação de escopo de tarefa rápida.
- question: "Esta tarefa parece complexa. Prosseguir como tarefa rápida ou usar planejamento completo?"
- header: "Escopo"
- options: Tarefa rápida | Plano completo (via /gsd-plan-phase) | Revisar

## Padrão: selecionar-profundidade
Seleção de profundidade de 3 opções para preferências de workflow de planejamento.
- question: "Quão detalhado deve ser o planejamento?"
- header: "Profundidade"
- options: Rápido (3-5 fases, pular pesquisa) | Padrão (5-8 fases, padrão) | Abrangente (8-12 fases, pesquisa profunda)

## Padrão: tratamento-de-contexto
Tratador de 3 opções para CONTEXT.md existente no workflow de discuss.
- question: "A Fase {N} já tem um CONTEXT.md. Como devemos tratá-lo?"
- header: "Contexto"
- options: Sobrescrever | Acrescentar | Cancelar

## Padrão: opcao-area-cinza
Template dinâmico para apresentar escolhas em área cinza no workflow de discuss.
- question: "{Título da área cinza}"
- header: "Decisão"
- options: {Opção 1} | {Opção 2} | Deixar Claude decidir
- Nota: Opções geradas em tempo de execução. Sempre inclua "Deixar Claude decidir" como última opção.
