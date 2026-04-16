# Modelos de Raciocínio: Cluster de Depuração

Modelos de raciocínio estruturado para o agente **debugger**. Aplique-os em pontos de decisão durante a investigação, não continuamente. Cada modelo combate um modo de falha específico documentado.

Fonte: Curado a partir do catálogo de modelos [thinking-partner](https://github.com/mattnowdev/thinking-partner) (150+ modelos). Selecionados por aplicabilidade direta ao fluxo de trabalho de depuração do GSD.

## Resolução de Conflitos

**Análise de Árvore de Falhas e Investigação por Hipótese são sequenciais:** Árvore de Falhas PRIMEIRO (gerar a árvore de possíveis causas), Investigação por Hipótese SEGUNDO (testar cada ramo sistematicamente). A Árvore de Falhas fornece o mapa; a Investigação por Hipótese fornece a disciplina para percorrê-lo.

## 1. Análise de Árvore de Falhas

**Combate:** Saltar para conclusões sem mapear sistematicamente os caminhos de falha.

Antes de testar qualquer hipótese, construa uma árvore de falhas: comece com o sintoma observado como nó raiz, depois ramifique em todas as possíveis causas em cada nível (hardware, software, configuração, dados, ambiente). Use portas AND/OR — algumas falhas requerem múltiplas condições (AND), outras têm gatilhos independentes (OR). Esta árvore se torna seu roteiro de investigação. Priorize ramos por probabilidade e testabilidade, mas NÃO pode ramos apenas porque parecem improváveis — causas improváveis que são fáceis de testar devem ser testadas cedo.

## 2. Investigação por Hipótese

**Combate:** Fazer mudanças aleatórias e esperar que algo funcione — o anti-padrão de "depuração por espingarda".

Para cada hipótese da árvore de falhas, siga o protocolo estrito: PREVER ("Se a hipótese H estiver correta, então o teste T deve produzir o resultado R"), TESTAR (executar exatamente um teste), OBSERVAR (registrar o resultado real), CONCLUIR (correspondeu = APOIADO, falhou = ELIMINADO, inesperado = nova evidência). Nunca pule a etapa PREVER — sem uma previsão, você não pode distinguir um resultado significativo de ruído. Nunca mude mais de uma variável por teste — se você mudar duas coisas e o bug desaparecer, não sabe qual mudança o corrigiu.

## 3. Navalha de Occam

**Combate:** Perseguir explicações elaboradas quando as simples ainda não foram descartadas.

Antes de investigar bugs complexos de interação entre múltiplos componentes, condições de corrida ou problemas em nível de framework, verifique as explicações simples primeiro: typo no nome da variável, caminho de arquivo errado, importação ausente, valor de configuração incorreto, cache desatualizado, variável de ambiente errada. Essas causas "entediantes" são responsáveis pela maioria dos bugs. Só escale para hipóteses complexas APÓS as simples serem eliminadas. Se sua hipótese atual requer 3+ coisas dando errado simultaneamente, recue e procure uma falha de ponto único.

## 4. Pensamento Contrafactual

**Combate:** Não conseguir isolar a causalidade por não perguntar "e se mudássemos apenas esta coisa?"

Quando você tem uma hipótese sobre a causa raiz, construa um contrafactual: "Se eu mudar APENAS esta variável/configuração/linha, o bug deve desaparecer (ou aparecer)." Execute o teste contrafactual. Se o bug persistir após sua mudança direcionada, sua hipótese está errada — a causa está em outro lugar. Se o bug desaparecer, você tem forte evidência causal. Isso é mais poderoso que correlação ("o bug apareceu após o deploy X") porque testa o mecanismo, não apenas a cronologia.

---

## Quando NÃO Usar o Raciocínio

Pule modelos de raciocínio estruturado quando a situação não se beneficia deles:

- **Bugs óbvios de causa única** — Se a mensagem de erro nomeia o arquivo exato, a linha e a causa (ex.: `TypeError: Cannot read property 'x' of undefined at foo.js:42`), corrija diretamente. Não construa uma árvore de falhas para uma referência nula com stack trace.
- **Reproduzindo uma correção conhecida** — Se você já conhece a causa raiz de uma investigação anterior ou o usuário te disse exatamente o que está errado, pule a investigação por hipótese e vá diretamente para a correção.
- **Typos, importações ausentes, caminhos errados** — Se a Navalha de Occam resolveria imediatamente, aplique a correção sem invocar o modelo completo. O modelo existe para quando verificações simples falham, não para fazer checkpoint de verificações simples.
- **Lendo logs de erro** — Ler e entender a saída de erros é depuração normal, não um "ponto de decisão". Invoque modelos apenas quando tiver múltiplas hipóteses plausíveis e precisar escolher qual testar primeiro.
