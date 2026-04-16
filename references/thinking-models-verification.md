# Modelos de Raciocínio: Cluster de Verificação

Modelos de raciocínio estruturado para os agentes **verifier** e **plan-checker**. Aplique-os durante as passagens de verificação, não continuamente. Cada modelo combate um modo de falha específico documentado.

Fonte: Curado a partir do catálogo de modelos [thinking-partner](https://github.com/mattnowdev/thinking-partner) (150+ modelos). Selecionados por aplicabilidade direta ao fluxo de trabalho de verificação do GSD.

## Resolução de Conflitos

**Inversão** e **Contador de Viés de Confirmação** ambos buscam falhas, mas servem a propósitos diferentes. Execute-os em sequência:

1. **Inversão PRIMEIRO** (brainstorm): gere 3 formas pelas quais isso poderia estar errado
2. **Contador de Viés de Confirmação SEGUNDO** (verificação estruturada): encontre um requisito parcialmente atendido, um teste enganoso, um caminho de erro não coberto

A Inversão gera a lista; o Contador de Viés de Confirmação é a disciplina para verificar os itens nela.

## 1. Inversão

**Combate:** Verificadores confirmando sucesso em vez de encontrar falhas.

Em vez de verificar o que ESTÁ correto, liste 3 formas específicas pelas quais esta implementação pode estar ERRADA apesar de passar nos testes: casos extremos ausentes, perda silenciosa de dados, condições de corrida, caminhos de erro não tratados. Para cada um, escreva uma verificação concreta (grep por padrão, testar com entrada específica, verificar se o tratamento de erros existe). Adicionalmente, verifique se algum DESVIO documentado no SUMMARY.md muda o significado ou a aplicabilidade de um must-have. Se um must-have foi escrito assumindo a abordagem A, mas o executor usou a abordagem B, o must-have pode precisar de reinterpretação, não de verificação literal.

## 2. Cerca de Chesterton

**Combate:** Sinalizar código proposital como morto ou desnecessário.

Antes de sinalizar qualquer código existente como morto, redundante ou excessivamente complicado, determine POR QUE foi escrito dessa forma. Verifique git blame, comentários, casos de teste e o PLAN.md que o criou. Se o motivo não estiver claro, sinalize como "propósito desconhecido — recomendo manter com WARNING, não remover" e inclua o hash do git blame para o commit que o introduziu.

## 3. Contador de Viés de Confirmação

**Combate:** Verificadores preparados pelas afirmações do SUMMARY.md para ver sucesso.

Após sua passagem inicial de verificação, faça uma passagem de DESCONFIRMAÇÃO: (1) encontre um requisito que está apenas parcialmente atendido, (2) encontre um teste que passa mas não testa realmente o comportamento declarado, (3) encontre um caminho de erro sem cobertura de teste. Reporte esses itens mesmo se a verificação geral passar.

## 4. Calibração de Falácia do Planejamento

**Combate:** Aceitar planos excessivamente abrangentes como razoáveis (plan-checker).

Para cada tarefa estimada como "simples" ou "pequena", verifique: ela toca mais de 2 arquivos? Requer entendimento de uma API não familiar? Modifica infraestrutura compartilhada? Se sim para qualquer um, sinalize como provavelmente subestimada. Planos com mais de 5 tarefas ou tarefas tocando mais de 4 arquivos por tarefa estão excessivamente abrangentes.

## 5. Pensamento Contrafactual

**Combate:** Planos que assumem sucesso em cada etapa sem recuperação de erro (plan-checker).

Para cada plano, pergunte: "O que aconteceria se o executor seguisse este plano EXATAMENTE como escrito mas encontrasse uma falha comum: incompatibilidade de versão de dependência, API retornando formato inesperado, arquivo já modificado pelo plano anterior?" Se o plano não tem caminho de contingência e as etapas `<action>` assumem sucesso em cada ponto, sinalize como WARNING: "Nenhum caminho de recuperação de erro para a tarefa T{n}."

---

## Quando NÃO Usar o Raciocínio

Pule modelos de raciocínio estruturado quando a situação não se beneficia deles:

- **Re-verificação de itens previamente aprovados** — Quando em modo de re-verificação, itens que passaram na verificação inicial precisam apenas de uma verificação rápida de regressão (existência + sanidade básica), não o tratamento completo de Inversão + Contador de Viés de Confirmação.
- **Verificações binárias de existência** — Se um must-have é "arquivo X existe com mais de N linhas" e o arquivo claramente existe com conteúdo substancial, não execute Pensamento Contrafactual sobre ele. Reserve modelos para must-haves ambíguos ou dependentes de wiring.
- **Resultados de testes diretos** — Se os comandos `<verify>` produzem saída clara de aprovação/reprovação (ex.: a suite de testes termina com 0 e todos os testes passando), aceite o resultado. Invoque modelos apenas quando os resultados dos testes forem ambíguos ou quando suspeitar que os testes não testam realmente o que afirmam.
- **Problemas de nível INFO** — Não aplique raciocínio estruturado para decidir se uma observação de nível INFO é na verdade um BLOCKER. Itens INFO são informativos por definição e nunca disparam gates.
