# Modelos de Raciocínio: Cluster de Pesquisa

Modelos de raciocínio estruturado para os agentes **researcher** e **synthesizer**. Aplique-os em pontos de decisão durante a pesquisa e síntese, não continuamente. Cada modelo combate um modo de falha específico documentado.

Fonte: Curado a partir do catálogo de modelos [thinking-partner](https://github.com/mattnowdev/thinking-partner) (150+ modelos). Selecionados por aplicabilidade direta ao fluxo de trabalho de pesquisa do GSD.

## Resolução de Conflitos

**Primeiros Princípios e Argumento do Homem de Palha ambos expandem o escopo** — execute Primeiros Princípios PRIMEIRO (decomponha o problema), depois Argumento do Homem de Palha (fortaleça as alternativas). Não execute simultaneamente.

## 1. Pensamento de Primeiros Princípios

**Combate:** Aceitar explicações superficiais sem decompor em componentes fundamentais.

Antes de aceitar qualquer recomendação tecnológica ou padrão arquitetural, decomponha-o em suas restrições fundamentais: Qual problema isso resolve? Quais são os requisitos inegociáveis? Quais são os limites físicos/lógicos? Construa sua recomendação DE BAIXO para CIMA a partir dessas restrições, em vez de DE CIMA para BAIXO a partir da sabedoria convencional. Se você não consegue explicar POR QUE uma recomendação está correta a partir dos primeiros princípios, sinalize-a como `[LOW]` independentemente da contagem de fontes.

## 2. Consciência do Paradoxo de Simpson

**Combate:** O sintetizador agrega pesquisa conflitante sem verificar divisões por variáveis de confundimento.

Ao combinar descobertas de múltiplos documentos de pesquisa que mostram resultados contraditórios, verifique se a contradição desaparece quando você divide por uma variável oculta: versão do framework, alvo de implantação, escala do projeto ou categoria de caso de uso. Uma biblioteca que apresenta benchmark mais rápido no geral pode ser mais lenta para SUA carga de trabalho específica. Antes de resolver contradições por voto majoritário, pergunte: "Existe uma divisão de subgrupo que explica por que ambas as descobertas são corretas em seu próprio contexto?"

## 3. Viés de Sobrevivência

**Combate:** Encontrar apenas exemplos bem-sucedidos enquanto perde falhas e abordagens abandonadas.

Após reunir evidências FAVORÁVEIS a uma abordagem recomendada, pesquise ativamente por projetos que a ABANDONARAM. Verifique issues do GitHub por "migrated away from", "replaced X with" ou "problems with X at scale". Uma tecnologia com 10 histórias de sucesso e 100 falhas silenciosas parece ótima até você verificar o cemitério. Pese evidências negativas (histórias de migração para longe, avisos de depreciação, issues não resolvidas) com MAIS peso do que evidências positivas — falhas são sub-reportadas.

## 4. Contador de Viés de Confirmação

**Combate:** Buscar evidências que confirmam a hipótese inicial enquanto ignora evidências desconfirmatórias.

Após formar sua recomendação inicial, gaste um ciclo completo de pesquisa buscando CONTRA ela. Use termos de busca como "{technology} problems", "{technology} alternatives", "why not {technology}", "{technology} vs {competitor}". Para cada evidência desconfirmatória encontrada, ou (a) refute-a com fontes de maior confiança, ou (b) adicione-a como ressalva à sua recomendação. Se você não consegue encontrar NENHUMA crítica à sua recomendação, sua busca foi muito estreita — amplie-a.

## 5. Argumento do Homem de Palha

**Combate:** Descartar abordagens alternativas sem dar a elas a sua forma mais forte possível.

Antes de recomendar contra uma tecnologia ou abordagem alternativa, construa o CASO MAIS FORTE possível para ela. O que um defensor apaixonado diria? Para quais casos de uso ela é melhor do que sua recomendação? Quais trade-offs a favorecem? Apresente a alternativa com o argumento mais forte ao lado de sua recomendação com uma comparação honesta. Se a alternativa com o argumento mais forte for competitiva, sinalize a decisão como `[NEEDS DECISION]` em vez de fazer uma recomendação unilateral.

---

## Quando NÃO Usar o Raciocínio

Pule modelos de raciocínio estruturado quando a situação não se beneficia deles:

- **Decisões bloqueadas no CONTEXT.md** — Se o usuário já decidiu "usar a biblioteca X", não execute análise de Argumento do Homem de Palha sobre alternativas ou decomposição de Primeiros Princípios da escolha. Pesquise como usar X bem, não se X é a escolha certa.
- **Consultas de stack padrão** — Se você está simplesmente verificando a versão mais recente de uma biblioteca bem conhecida ou lendo sua documentação de API, não invoque Viés de Sobrevivência ou Contador de Viés de Confirmação. Esses modelos são para avaliar recomendações contestadas, não para consultas factuais.
- **Fases de tecnologia única** — Se a fase envolve uma tecnologia sem alternativas a avaliar (ex.: "adicionar regra ESLint X"), pule modelos comparativos (Argumento do Homem de Palha, Contador de Viés de Confirmação). Apenas pesquise a implementação.
- **Pesquisa somente interna** — Se a pesquisa é puramente interna (entender padrões de código existentes, encontrar onde uma função é chamada), modelos de raciocínio estruturado não agregam valor. Use grep e leia o código.
