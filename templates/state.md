# Template de Estado

Template para `.planning/STATE.md` — a memória viva do projeto.

---

## Template do Arquivo

```markdown
# Estado do Projeto

## Referência do Projeto

Veja: .planning/PROJECT.md (atualizado em [data])

**Valor principal:** [Frase única da seção Valor Principal do PROJECT.md]
**Foco atual:** [Nome da fase atual]

## Posição Atual

Fase: [X] de [Y] ([Nome da Fase])
Plano: [A] de [B] na fase atual
Status: [Pronto para planejar / Planejando / Pronto para executar / Em progresso / Fase concluída]
Última atividade: [YYYY-MM-DD] — [O que aconteceu]

Progresso: [░░░░░░░░░░] 0%

## Métricas de Performance

**Velocidade:**
- Total de planos concluídos: [N]
- Duração média: [X] min
- Tempo total de execução: [X.X] horas

**Por Fase:**

| Fase | Planos | Total | Média/Plano |
|------|--------|-------|-------------|
| - | - | - | - |

**Tendência Recente:**
- Últimos 5 planos: [durações]
- Tendência: [Melhorando / Estável / Piorando]

*Atualizado após a conclusão de cada plano*

## Contexto Acumulado

### Decisões

As decisões são registradas na tabela Decisões-Chave do PROJECT.md.
Decisões recentes que afetam o trabalho atual:

- [Fase X]: [Resumo da decisão]
- [Fase Y]: [Resumo da decisão]

### Todos Pendentes

[De .planning/todos/pending/ — ideias capturadas durante as sessões]

Nenhum ainda.

### Bloqueios/Preocupações

[Problemas que afetam trabalhos futuros]

Nenhum ainda.

## Itens Adiados

Itens reconhecidos e carregados do fechamento do milestone anterior:

| Categoria | Item | Status | Adiado Em |
|-----------|------|--------|-----------|
| *(nenhum)* | | | |

## Continuidade de Sessão

Última sessão: [YYYY-MM-DD HH:MM]
Parado em: [Descrição da última ação concluída]
Arquivo de retomada: [Caminho para .continue-here*.md se existir, caso contrário "Nenhum"]
```

<purpose>

O STATE.md é a memória de curto prazo do projeto abrangendo todas as fases e sessões.

**Problema que resolve:** Informações são capturadas em summaries, issues e decisões, mas não são consumidas sistematicamente. As sessões começam sem contexto.

**Solução:** Um único arquivo pequeno que é:
- Lido primeiro em todo fluxo de trabalho
- Atualizado após toda ação significativa
- Contém um resumo do contexto acumulado
- Habilita restauração instantânea de sessão

</purpose>

<lifecycle>

**Criação:** Após o ROADMAP.md ser criado (durante o init)
- Referencie o PROJECT.md (leia-o para o contexto atual)
- Inicialize seções de contexto acumulado vazias
- Defina a posição como "Fase 1 pronta para planejar"

**Leitura:** Primeiro passo de todo fluxo de trabalho
- progress: Apresentar o status ao usuário
- plan: Informar as decisões de planejamento
- execute: Conhecer a posição atual
- transition: Saber o que está completo

**Escrita:** Após toda ação significativa
- execute: Após o SUMMARY.md ser criado
  - Atualizar posição (fase, plano, status)
  - Anotar novas decisões (detalhar no PROJECT.md)
  - Adicionar bloqueios/preocupações
- transition: Após a fase ser marcada como completa
  - Atualizar barra de progresso
  - Limpar bloqueios resolvidos
  - Atualizar a data de referência do Projeto

</lifecycle>

<sections>

### Referência do Projeto
Aponta para o PROJECT.md para o contexto completo. Inclui:
- Valor principal (a UMA coisa que importa)
- Foco atual (qual fase)
- Data da última atualização (aciona releitura se desatualizado)

Claude lê o PROJECT.md diretamente para requisitos, restrições e decisões.

### Posição Atual
Onde estamos agora:
- Fase X de Y — qual fase
- Plano A de B — qual plano dentro da fase
- Status — estado atual
- Última atividade — o que aconteceu mais recentemente
- Barra de progresso — indicador visual do progresso geral

Cálculo do progresso: (planos concluídos) / (total de planos em todas as fases) × 100%

### Métricas de Performance
Rastreie a velocidade para entender os padrões de execução:
- Total de planos concluídos
- Duração média por plano
- Detalhamento por fase
- Tendência recente (melhorando/estável/piorando)

Atualizado após a conclusão de cada plano.

### Contexto Acumulado

**Decisões:** Referência à tabela Decisões-Chave do PROJECT.md, além de um resumo das decisões recentes para acesso rápido. O log completo de decisões vive no PROJECT.md.

**Todos Pendentes:** Ideias capturadas via /gsd-add-todo
- Contagem de todos pendentes
- Referência para .planning/todos/pending/
- Lista breve se poucos, contagem se muitos (ex.: "5 todos pendentes — veja /gsd-check-todos")

**Bloqueios/Preocupações:** Das seções "Prontidão para Próxima Fase"
- Problemas que afetam trabalhos futuros
- Prefixe com a fase de origem
- Limpos quando resolvidos

### Continuidade de Sessão
Habilita retomada instantânea:
- Quando foi a última sessão
- O que foi concluído por último
- Se há um arquivo .continue-here para retomar

</sections>

<size_constraint>

Mantenha o STATE.md com menos de 100 linhas.

É um RESUMO, não um arquivo. Se o contexto acumulado crescer demais:
- Mantenha apenas 3-5 decisões recentes no resumo (log completo no PROJECT.md)
- Mantenha apenas os bloqueios ativos, remova os resolvidos

O objetivo é "leia uma vez, saiba onde estamos" — se for muito longo, isso falha.

</size_constraint>
