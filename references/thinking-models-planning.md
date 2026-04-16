# Modelos de Raciocínio: Cluster de Planejamento

Modelos de raciocínio estruturado para os agentes **planner** e **roadmapper**. Aplique-os em pontos de decisão durante a criação do plano, não continuamente. Cada modelo combate um modo de falha específico documentado.

Fonte: Curado a partir do catálogo de modelos [thinking-partner](https://github.com/mattnowdev/thinking-partner) (150+ modelos). Selecionados por aplicabilidade direta ao fluxo de trabalho de planejamento do GSD.

## Resolução de Conflitos

Pré-Morte e Análise de Restrição ambos analisam risco em diferentes granularidades. Execute Análise de Restrição PRIMEIRO (identifique a restrição mais difícil), depois Pré-Morte (enumere os modos de falha em torno dessa restrição e o restante do plano).

## 1. Análise de Pré-Morte

**Combate:** Decomposição otimista do plano que ignora os modos de falha.

Antes de finalizar este plano, assuma que ele já falhou. Liste as 3 razões mais prováveis para falha — dependência ausente, decomposição errada, complexidade subestimada — e adicione etapas de mitigação ou critérios de aceitação que detectariam cada falha cedo.

## 2. Decomposição MECE

**Combate:** Tarefas sobrepostas (conflitos de merge) ou tarefas com lacunas (requisitos ausentes).

Verifique se esta divisão de tarefas é MECE no nível de REQUISITO: (1) liste cada requisito do objetivo da fase, (2) confirme que cada um mapeia para o `<done>` de exatamente uma tarefa, (3) se duas tarefas modificam o mesmo arquivo, confirme que modificam SEÇÕES DIFERENTES ou atendem REQUISITOS DIFERENTES, (4) sinalize qualquer requisito não coberto por nenhuma tarefa.

## 3. Análise de Restrição

**Combate:** Adiar a restrição mais difícil para a última tarefa, causando falhas em etapas tardias.

Identifique a restrição mais difícil nesta fase — a única coisa que, se não funcionar, torna todo o resto irrelevante. Agende essa restrição como Tarefa 1 ou 2, não como a última. Se a restrição envolve uma API externa ou biblioteca não familiar, adicione uma tarefa de spike/prova de conceito antes da implementação principal.

## 4. Teste de Reversibilidade

**Combate:** Analisar excessivamente decisões baratas, subanalisar as custosas.

Para cada decisão significativa neste plano, classifique como REVERSÍVEL (pode mudar depois com baixo custo) ou IRREVERSÍVEL (mudar depois requer migração, mudanças quebradas ou retrabalho significativo). Gaste tempo de análise proporcional à irreversibilidade. Para decisões irreversíveis, documente a justificativa no plano.

## 5. Contador da Maldição do Conhecimento

**Combate:** Ambiguidade do plano para o executor devido a instruções comprimidas.

Para cada etapa `<action>`, releia-a como se NUNCA tivesse visto esta base de código. Cada substantivo é inequívoco (qual arquivo? qual função? qual endpoint?)? Cada verbo é específico (adicionar ONDE? modificar COMO?)? Se uma etapa pode ser interpretada de duas formas, reescreva-a. Inclua caminhos de arquivo, nomes de função e comportamento esperado em cada etapa de ação.

## 6. Contador de Negligência da Taxa Base

**Combate:** Planejadores ignorando ressalvas de pesquisa de baixa confiança.

Antes de finalizar o plano, leia TODOS os itens `[NEEDS DECISION]` e recomendações de BAIXA confiança do SUMMARY.md. Para cada: ou (a) crie uma tarefa `checkpoint:decision` para resolvê-lo, ou (b) documente por que o risco é aceitável nas notas de desvio do plano. Itens de BAIXA confiança aceitos silenciosamente se tornam débito técnico não documentado.

## Modo de Fechamento de Lacunas: Verificação de Causa Raiz

**Aplica-se apenas quando:** O planejador entra no modo de fechamento de lacunas (disparado por `gaps_found` em VERIFICATION.md).

Antes de escrever o plano de correção, aplique uma única rodada de "por quê": Por que esta lacuna ocorreu? Foi uma deficiência de plano (tarefa errada), uma falha de execução (tarefa correta, implementação errada) ou uma suposição alterada (mudança de ambiente/dependência)? O plano de correção deve abordar a categoria de causa raiz, não apenas o sintoma.

---

## Quando NÃO Usar o Raciocínio

Pule modelos de raciocínio estruturado quando a situação não se beneficia deles:

- **Planos de tarefa única** — Se a fase tem um requisito claro e uma tarefa óbvia, não execute Pré-Morte ou análise MECE. Escreva a tarefa diretamente.
- **Fases bem pesquisadas** — Se RESEARCH.md tem recomendações de ALTA confiança para cada decisão e nenhum item `[NEEDS DECISION]`, pule o Contador de Negligência da Taxa Base. A pesquisa já resolveu a incerteza.
- **Iterações de revisão** — Ao revisar um plano com base no feedback do verificador, concentre-se em corrigir os problemas sinalizados. Não reexecute o conjunto completo de modelos em cada passagem de revisão — aplique apenas o modelo relevante para o problema específico (ex.: MECE se o verificador encontrou uma lacuna de cobertura).
- **Planos de boilerplate** — Mudanças de configuração, bumps de versão, atualizações de documentação. Estes não têm modos de falha que valham a análise de pré-morte.
