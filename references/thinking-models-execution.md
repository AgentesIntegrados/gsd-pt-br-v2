# Modelos de Raciocínio: Cluster de Execução

Modelos de raciocínio estruturado para o agente **executor**. Aplique-os em pontos de decisão durante a execução de tarefas, não continuamente. Cada modelo combate um modo de falha específico documentado.

Fonte: Curado a partir do catálogo de modelos [thinking-partner](https://github.com/mattnowdev/thinking-partner) (150+ modelos). Selecionados por aplicabilidade direta ao fluxo de trabalho de execução do GSD.

## Resolução de Conflitos

**Função Forçadora e Primeiros Princípios ambos empurram para "faça agora".** Execute Primeiros Princípios PRIMEIRO (entenda a restrição), Função Forçadora SEGUNDO (crie o mecanismo). Sequenciais, não competitivos.

## 1. Círculo de Preocupação vs. Círculo de Controle

**Combate:** Executor tentando corrigir coisas fora do seu escopo — bugs upstream, débito técnico não relacionado, problemas de infraestrutura.

Antes de modificar qualquer código não listado explicitamente na seção `<files>` do plano, pergunte: Isso está no meu Círculo de Controle (escopo do plano) ou no meu Círculo de Preocupação (coisas que noto mas não devo corrigir)? Se for Círculo de Preocupação: documente como uma nota de desvio ou item adiado, NÃO corrija. O trabalho do executor é construir o que o plano diz, não melhorar a base de código. Expansão de escopo por correções "enquanto estou aqui" é a causa número 1 de estouro de execução.

## 2. Função Forçadora

**Combate:** Adiar decisões difíceis para o tempo de execução em vez de resolvê-las em tempo de build.

Quando você encontrar um requisito ambíguo ou ponto de integração pouco claro, crie uma função forçadora que torne a decisão explícita AGORA em vez de escondê-la por trás de um TODO ou verificação em tempo de execução. Exemplos: use o tipo `never` do TypeScript para forçar switches exaustivos, adicione uma asserção em tempo de build para valores de configuração obrigatórios, crie uma interface que force os chamadores a tratar casos de erro. Se uma decisão realmente não pode ser tomada em tempo de build, documente como um desvio `checkpoint:decision` — não adie silenciosamente.

## 3. Pensamento de Primeiros Princípios

**Combate:** Copiar padrões de código existente sem entender se eles se encaixam na tarefa atual.

Antes de copiar um padrão de outro arquivo ou fase, decomponha POR QUE esse padrão existe: Qual restrição ele satisfaz? Sua tarefa atual tem a mesma restrição? Se não, o padrão pode ser cargo cult. Construa sua implementação a partir dos requisitos reais da tarefa, não do exemplo existente mais próximo. Em caso de dúvida, as etapas `<action>` do plano definem o que construir — derive a implementação dessas etapas, não do código adjacente.

## 4. Navalha de Occam

**Combate:** Engenharia excessiva de tarefas simples com abstrações desnecessárias, genéricos ou preparação para o futuro.

Antes de adicionar uma camada de abstração, parâmetro de tipo genérico, padrão de fábrica ou opção de configuração, pergunte: O plano REQUER esta flexibilidade? Se o plano diz "criar uma função que faça X", crie uma função que faça X — não um framework configurável, extensível e plugável que teoricamente poderia fazer X através de Y através de Z. A implementação mais simples que satisfaz a condição `<done>` do plano é a correta. Adicione complexidade somente quando o plano explicitamente pedir.

## 5. Cerca de Chesterton

**Combate:** Remover ou modificar código existente sem entender por que foi escrito dessa forma.

Antes de remover, substituir ou modificar significativamente o código existente que o plano toca, determine POR QUE ele existe. Verifique: git blame para o commit que o introduziu, comentários explicando a justificativa, casos de teste que o exercitam, o PLAN.md ou SUMMARY.md que o criou. Se o propósito não estiver claro, mantenha-o e adicione um comentário notando a incerteza — NÃO remova código cujo propósito você não entende. Se o plano disser explicitamente para removê-lo, ainda documente o que fazia nas notas de desvio.

---

## Quando NÃO Usar o Raciocínio

Pule modelos de raciocínio estruturado quando a situação não se beneficia deles:

- **Ações de tarefa diretas** — Se o plano diz "criar arquivo X com conteúdo Y" e a ação é inequívoca, execute diretamente. Não invoque Primeiros Princípios para analisar por que você está criando um arquivo que o plano mandou criar.
- **Seguindo padrões estabelecidos do projeto** — Se a base de código tem um padrão claro e consistente (ex.: todo handler de rota segue a mesma estrutura) e o plano diz para adicionar outro, siga o padrão. A Cerca de Chesterton se aplica a remover padrões, não a segui-los.
- **Edições triviais de arquivo** — Adicionar uma importação, corrigir um typo, atualizar um número de versão. Estas são mudanças mecânicas que não envolvem decisões de design.
- **Executando comandos de verificação** — Executar as etapas `<verify>` do plano é procedural. Invoque modelos apenas se uma etapa de verificação falhar e você precisar decidir como responder.
