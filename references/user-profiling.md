# Perfilamento de Usuário: Referência de Heurísticas de Detecção

Este documento de referência define heurísticas de detecção para perfilamento comportamental em 8 dimensões. O agente gsd-user-profiler aplica essas regras ao analisar mensagens extraídas de sessão. Não invente dimensões ou regras de pontuação além das definidas aqui.

## Como Usar Este Documento

1. O agente gsd-user-profiler lê este documento antes de analisar qualquer mensagem
2. Para cada dimensão, o agente varre as mensagens em busca dos padrões de sinal definidos abaixo
3. O agente aplica as heurísticas de detecção para classificar o padrão do desenvolvedor
4. A confiança é pontuada usando os limiares definidos por dimensão
5. Citações de evidência são curadas usando as regras na seção de Curadoria de Evidências
6. A saída deve estar em conformidade com o esquema JSON na seção de Esquema de Saída

---

## Dimensões

### 1. Estilo de Comunicação

`dimension_id: communication_style`

**O que estamos medindo:** Como o desenvolvedor formula requisições, instruções e feedback — o padrão estrutural de suas mensagens para o Claude.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `terse-direct` | Mensagens curtas e imperativas com contexto mínimo. Vai direto ao ponto. |
| `conversational` | Mensagens de tamanho médio misturando instruções com perguntas e pensamento em voz alta. Tom natural e informal. |
| `detailed-structured` | Mensagens longas com estrutura explícita — cabeçalhos, listas numeradas, declarações de problema, pré-análise. |
| `mixed` | Sem padrão dominante; o estilo muda conforme o tipo de tarefa ou contexto do projeto. |

**Padrões de sinal:**

1. **Distribuição do comprimento das mensagens** — Contagem média de palavras entre as mensagens. Terse < 50 palavras, conversational 50-200 palavras, detailed > 200 palavras.
2. **Proporção imperativo-para-interrogativo** — Proporção de comandos ("corrija isso", "adicione X") para perguntas ("o que você acha?", "devemos?"). Alta proporção imperativa sugere terse-direct.
3. **Formatação estrutural** — Presença de cabeçalhos markdown, listas numeradas, blocos de código ou marcadores dentro das mensagens. Formatação frequente sugere detailed-structured.
4. **Preâmbulos de contexto** — Se o desenvolvedor fornece contexto antes de fazer um pedido. Preâmbulos sugerem conversational ou detailed-structured.
5. **Completude das frases** — Se as mensagens usam frases completas ou fragmentos/abreviações. Fragmentos sugerem terse-direct.
6. **Padrão de acompanhamento** — Se o desenvolvedor fornece contexto adicional em mensagens subsequentes (requisições de múltiplas mensagens sugerem conversational).

**Heurísticas de detecção:**

1. Se comprimento médio da mensagem < 50 palavras E predominantemente imperativo E formatação mínima --> `terse-direct`
2. Se comprimento médio da mensagem 50-200 palavras E mistura de imperativo e interrogativo E formatação ocasional --> `conversational`
3. Se comprimento médio da mensagem > 200 palavras E formatação estrutural frequente E preâmbulos de contexto presentes --> `detailed-structured`
4. Se variância do comprimento da mensagem é alta (desvio padrão > 60% da média) E nenhum padrão único domina (< 60% das mensagens correspondem a um estilo) --> `mixed`
5. Se o padrão varia sistematicamente por tipo de projeto (ex.: terse em projetos CLI, detailed em frontend) --> `mixed` com nota dependente de contexto

**Pontuação de confiança:**

- **HIGH:** 10+ mensagens mostrando padrão consistente (> 70% de correspondência), mesmo padrão observado em 2+ projetos
- **MEDIUM:** 5-9 mensagens mostrando padrão, OU padrão consistente dentro de 1 projeto apenas
- **LOW:** < 5 mensagens com sinais relevantes, OU sinais mistos (padrões contraditórios observados em contextos similares)
- **UNSCORED:** 0 mensagens com sinais relevantes para esta dimensão

**Citações de exemplo:**

- **terse-direct:** "fix the auth bug" / "add pagination to the list endpoint" / "this test is failing, make it pass"
- **conversational:** "I'm thinking we should probably handle the error case here. What do you think about returning a 422 instead of a 500? The client needs to know it was a validation issue."
- **detailed-structured:** "## Context\nThe auth flow currently uses session cookies but we need to migrate to JWT.\n\n## Requirements\n1. Access tokens (15min expiry)\n2. Refresh tokens (7-day)\n3. httpOnly cookies\n\n## What I've tried\nI looked at jose and jsonwebtoken..."

**Padrões dependentes de contexto:**

Quando o estilo de comunicação varia sistematicamente por projeto ou tipo de tarefa, reporte a divisão em vez de forçar uma única avaliação. Exemplo: "context-dependent: terse-direct for bug fixes and CLI tooling, detailed-structured for architecture and frontend work." A orquestração da Fase 3 resolve divisões dependentes de contexto apresentando a divisão ao usuário.

---

### 2. Velocidade de Decisão

`dimension_id: decision_speed`

**O que estamos medindo:** Com que rapidez o desenvolvedor faz escolhas quando o Claude apresenta opções, alternativas ou trade-offs.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `fast-intuitive` | Decide imediatamente com base na experiência ou intuição. Deliberação mínima. |
| `deliberate-informed` | Solicita comparação ou resumo antes de decidir. Quer entender os trade-offs. |
| `research-first` | Atrasa a decisão para pesquisar de forma independente. Pode sair e voltar com descobertas. |
| `delegator` | Delega à recomendação do Claude. Confia na sugestão. |

**Padrões de sinal:**

1. **Latência de resposta a opções** — Quantas mensagens entre o Claude apresentar opções e o desenvolvedor escolher. Imediato (mesma mensagem ou próxima) sugere fast-intuitive.
2. **Solicitações de comparação** — Presença de "compare these", "what are the trade-offs?", "pros and cons?" sugere deliberate-informed.
3. **Indicadores de pesquisa externa** — Mensagens como "I looked into X and...", "according to the docs...", "I read that..." sugerem research-first.
4. **Linguagem de delegação** — "just pick one", "whatever you recommend", "your call", "go with the best option" sugere delegator.
5. **Frequência de reversão de decisão** — Com que frequência o desenvolvedor muda uma decisão após tomá-la. Reversões frequentes podem indicar fast-intuitive com baixa confiança.

**Heurísticas de detecção:**

1. Se o desenvolvedor seleciona opções em 1-2 mensagens da apresentação E usa linguagem decisiva ("use X", "go with A") E raramente pede comparações --> `fast-intuitive`
2. Se o desenvolvedor solicita análise de trade-off ou tabelas de comparação E decide após receber comparação E faz perguntas de esclarecimento --> `deliberate-informed`
3. Se o desenvolvedor adia decisões com "let me look into this" E retorna com informações externas E cita documentação ou artigos --> `research-first`
4. Se o desenvolvedor usa linguagem de delegação (> 3 instâncias) E raramente substitui as escolhas do Claude E diz "sounds good" ou "your call" --> `delegator`
5. Se não há padrão claro OU evidência está dividida entre múltiplos estilos --> classifique como o estilo dominante com nota dependente de contexto

**Pontuação de confiança:**

- **HIGH:** 10+ pontos de decisão observados mostrando padrão consistente, mesmo padrão em 2+ projetos
- **MEDIUM:** 5-9 pontos de decisão, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 pontos de decisão observados, OU estilos de tomada de decisão mistos
- **UNSCORED:** 0 mensagens contendo sinais relevantes para decisão

**Citações de exemplo:**

- **fast-intuitive:** "Use Tailwind. Next question." / "Option B, let's move on"
- **deliberate-informed:** "Can you compare Prisma vs Drizzle for this use case? I want to understand the migration story and type safety differences before I pick."
- **research-first:** "Hold off on the DB choice -- I want to read the Drizzle docs and check their GitHub issues first. I'll come back with a decision."
- **delegator:** "You know more about this than me. Whatever you recommend, go with it."

**Padrões dependentes de contexto:**

A velocidade de decisão frequentemente varia por importância. Um desenvolvedor pode ser fast-intuitive para escolhas de estilo, mas research-first para decisões de banco de dados ou autenticação. Quando esse padrão é claro, reporte a divisão: "context-dependent: fast-intuitive for low-stakes (styling, naming), deliberate-informed for high-stakes (architecture, security)."

---

### 3. Profundidade de Explicação

`dimension_id: explanation_depth`

**O que estamos medindo:** Quanta explicação o desenvolvedor quer junto com o código — sua preferência por entendimento versus velocidade.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `code-only` | Quer código funcionando com explicação mínima ou nenhuma. Lê e entende o código diretamente. |
| `concise` | Quer breve explicação da abordagem com o código. Decisões-chave anotadas, não exaustivas. |
| `detailed` | Quer um detalhamento completo da abordagem, raciocínio e código. Aprecia estrutura. |
| `educational` | Quer explicação conceitual profunda. Trata as interações como oportunidades de aprendizado. |

**Padrões de sinal:**

1. **Solicitações explícitas de profundidade** — "just show me the code", "explain why", "teach me about X", "skip the explanation"
2. **Reação às explicações** — O desenvolvedor pula as explicações? Pede mais detalhes? Diz "too much"?
3. **Profundidade das perguntas de acompanhamento** — Acompanhamento superficial ("does it work?") vs. conceitual ("why this pattern over X?")
4. **Sinais de compreensão de código** — O desenvolvedor referencia detalhes de implementação em suas mensagens? Isso sugere que ele lê e entende o código diretamente.
5. **Sinais de "eu sei disso"** — Mensagens como "I'm familiar with X", "skip the basics", "I know how hooks work" indicam menor preferência por explicação.

**Heurísticas de detecção:**

1. Se o desenvolvedor diz "just the code" ou "skip the explanation" E raramente faz perguntas conceituais de acompanhamento E referencia detalhes do código diretamente --> `code-only`
2. Se o desenvolvedor aceita explicações breves sem pedir mais E faz acompanhamentos focados sobre decisões específicas --> `concise`
3. Se o desenvolvedor faz perguntas "por quê" E solicita detalhamentos E aprecia explicações estruturadas --> `detailed`
4. Se o desenvolvedor faz perguntas conceituais além da tarefa imediata E usa linguagem de aprendizado ("I want to understand", "teach me") --> `educational`

**Pontuação de confiança:**

- **HIGH:** 10+ mensagens mostrando preferência consistente, mesma preferência em 2+ projetos
- **MEDIUM:** 5-9 mensagens, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 mensagens relevantes, OU preferências mudam entre interações
- **UNSCORED:** 0 mensagens com sinais relevantes

**Citações de exemplo:**

- **code-only:** "Just give me the implementation. I'll read through it." / "Skip the explanation, show the code."
- **concise:** "Quick summary of the approach, then the code please." / "Why did you use a Map here instead of an object?"
- **detailed:** "Walk me through this step by step. I want to understand the auth flow before we implement it."
- **educational:** "Can you explain how JWT refresh token rotation works conceptually? I want to understand the security model, not just implement it."

**Padrões dependentes de contexto:**

A profundidade de explicação frequentemente correlaciona com familiaridade com o domínio. Um desenvolvedor pode querer code-only para tecnologias conhecidas, mas educational para novos domínios. Reporte divisões quando observadas: "context-dependent: code-only for React/TypeScript, detailed for database optimization."

---

### 4. Abordagem de Depuração

`dimension_id: debugging_approach`

**O que estamos medindo:** Como o desenvolvedor aborda problemas, erros e comportamento inesperado ao trabalhar com o Claude.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `fix-first` | Cola o erro, quer que seja corrigido. Interesse mínimo em diagnóstico. Orientado a resultados. |
| `diagnostic` | Compartilha o erro com contexto, quer entender a causa antes de corrigir. |
| `hypothesis-driven` | Investiga de forma independente primeiro, traz teorias específicas para o Claude validar. |
| `collaborative` | Quer trabalhar no problema passo a passo com o Claude como parceiro. |

**Padrões de sinal:**

1. **Estilo de apresentação do erro** — Apenas colar o erro (fix-first) vs. erro + "I think it might be..." (hypothesis-driven) vs. "Can you help me understand why..." (diagnostic)
2. **Indicadores de pré-investigação** — O desenvolvedor compartilha o que já tentou? Menciona ter lido logs, verificado estado ou isolado o problema?
3. **Interesse pela causa raiz** — Após uma correção, o desenvolvedor pergunta "why did that happen?" ou apenas segue em frente?
4. **Linguagem passo a passo** — "Let's check X first", "what should we look at next?", "walk me through the debugging"
5. **Padrão de aceitação de correção** — O desenvolvedor aplica correções imediatamente ou as questiona primeiro?

**Heurísticas de detecção:**

1. Se o desenvolvedor cola erros sem contexto E aceita correções sem perguntas sobre causa raiz E segue em frente imediatamente --> `fix-first`
2. Se o desenvolvedor fornece contexto do erro E pergunta "why is this happening?" E quer explicação com a correção --> `diagnostic`
3. Se o desenvolvedor compartilha sua própria análise E propõe teorias ("I think the issue is X because...") E pede ao Claude para confirmar ou refutar --> `hypothesis-driven`
4. Se o desenvolvedor usa linguagem colaborativa ("let's", "what should we check?") E prefere diagnóstico incremental E percorre problemas juntos --> `collaborative`

**Pontuação de confiança:**

- **HIGH:** 10+ interações de depuração mostrando abordagem consistente, mesma abordagem em 2+ projetos
- **MEDIUM:** 5-9 interações de depuração, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 interações de depuração, OU abordagem varia significativamente
- **UNSCORED:** 0 mensagens com sinais relevantes para depuração

**Citações de exemplo:**

- **fix-first:** "Getting this error: TypeError: Cannot read properties of undefined. Fix it."
- **diagnostic:** "The API returns 500 when I send a POST to /users. Here's the request body and the server log. What's causing this?"
- **hypothesis-driven:** "I think the race condition is in the useEffect cleanup. I checked and the subscription isn't being cancelled on unmount. Can you confirm?"
- **collaborative:** "Let's debug this together. The test passes locally but fails in CI. What should we check first?"

**Padrões dependentes de contexto:**

A abordagem de depuração pode variar por urgência. Um desenvolvedor pode ser fix-first sob pressão de prazo, mas hypothesis-driven durante o desenvolvimento regular. Anote padrões temporais se detectados.

---

### 5. Filosofia de UX

`dimension_id: ux_philosophy`

**O que estamos medindo:** Como o desenvolvedor prioriza a experiência do usuário, o design e a qualidade visual em relação à funcionalidade.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `function-first` | Faça funcionar, polir depois. Preocupação mínima com UX durante a implementação. |
| `pragmatic` | Usabilidade básica desde o início. Nada feio ou quebrado, mas sem obsessão com design. |
| `design-conscious` | Design e UX são tratados tão importantes quanto funcionalidade. Atenção aos detalhes visuais. |
| `backend-focused` | Constrói principalmente backend/CLI. Exposição ou interesse mínimo em frontend. |

**Padrões de sinal:**

1. **Solicitações relacionadas a design** — Menções de estilização, layout, responsividade, animações, esquemas de cores, espaçamento
2. **Timing de polimento** — O desenvolvedor pede polimento visual durante a implementação ou o adia?
3. **Especificidade do feedback de UI** — Vaga ("make it look better") vs. específica ("increase the padding to 16px, change the font weight to 600")
4. **Distribuição frontend vs. backend** — Proporção de solicitações focadas em frontend para solicitações focadas em backend
5. **Menções de acessibilidade** — Referências a a11y, leitores de tela, navegação por teclado, labels ARIA

**Heurísticas de detecção:**

1. Se o desenvolvedor raramente menciona UI/UX E se concentra em lógica, APIs, dados E adia estilização ("we'll make it pretty later") --> `function-first`
2. Se o desenvolvedor inclui requisitos básicos de UX E menciona usabilidade, mas não perfeição de pixel E equilibra forma com função --> `pragmatic`
3. Se o desenvolvedor fornece requisitos de design específicos E menciona polimento, animações, espaçamento E trata bugs de UI tão seriamente quanto bugs de lógica --> `design-conscious`
4. Se o desenvolvedor trabalha principalmente em ferramentas CLI, APIs ou sistemas backend E raramente ou nunca trabalha em frontend E mensagens se concentram em dados, performance, infraestrutura --> `backend-focused`

**Pontuação de confiança:**

- **HIGH:** 10+ mensagens com sinais relevantes de UX, mesmo padrão em 2+ projetos
- **MEDIUM:** 5-9 mensagens, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 mensagens relevantes, OU filosofia varia por tipo de projeto
- **UNSCORED:** 0 mensagens com sinais relevantes de UX

**Citações de exemplo:**

- **function-first:** "Just get the form working. We'll style it later." / "I don't care how it looks, I need the data flowing."
- **pragmatic:** "Make sure the loading state is visible and the error messages are clear. Standard styling is fine."
- **design-conscious:** "The button needs more breathing room -- add 12px vertical padding and make the hover state transition 200ms. Also check the contrast ratio."
- **backend-focused:** "I'm building a CLI tool. No UI needed." / "Add the REST endpoint, I'll handle the frontend separately."

**Padrões dependentes de contexto:**

A filosofia de UX é inerentemente dependente do projeto. Um desenvolvedor construindo uma ferramenta CLI é necessariamente backend-focused para aquele projeto. Quando possível, distinga entre padrões orientados ao projeto e orientados à preferência. Se o desenvolvedor só tem projetos backend, anote que a avaliação reflete os dados disponíveis: "backend-focused (note: all analyzed projects are backend/CLI -- may not reflect frontend preferences)."

---

### 6. Filosofia de Fornecedor

`dimension_id: vendor_philosophy`

**O que estamos medindo:** Como o desenvolvedor aborda a escolha e avaliação de bibliotecas, frameworks e serviços externos.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `pragmatic-fast` | Usa o que funciona, o que o Claude sugere ou o que é mais rápido. Avaliação mínima. |
| `conservative` | Prefere opções conhecidas, testadas em batalha e amplamente adotadas. Avesso a riscos. |
| `thorough-evaluator` | Pesquisa alternativas, lê documentação, compara funcionalidades e trade-offs antes de comprometer. |
| `opinionated` | Tem preferências fortes e pré-existentes por ferramentas específicas. Sabe o que gosta. |

**Padrões de sinal:**

1. **Linguagem de seleção de biblioteca** — "just use whatever", "is X the standard?", "I want to compare A vs B", "we're using X, period"
2. **Profundidade de avaliação** — O desenvolvedor aceita a primeira sugestão ou pede alternativas?
3. **Preferências declaradas** — Menções explícitas de ferramentas preferidas, experiência passada ou filosofia de ferramentas
4. **Padrões de rejeição** — O desenvolvedor rejeita sugestões do Claude? Em que base (popularidade, experiência pessoal, qualidade da documentação)?
5. **Atitude em relação a dependências** — "minimize dependencies", "no external deps", "add whatever we need" — revela filosofia sobre código externo

**Heurísticas de detecção:**

1. Se o desenvolvedor aceita sugestões de biblioteca sem resistência E usa frases como "sounds good" ou "go with that" E raramente pede alternativas --> `pragmatic-fast`
2. Se o desenvolvedor pergunta sobre popularidade, manutenção, comunidade E prefere "industry standard" ou "battle-tested" E evita novidades/experimentais --> `conservative`
3. Se o desenvolvedor solicita comparações E lê documentação antes de decidir E pergunta sobre casos extremos, licença, tamanho do bundle --> `thorough-evaluator`
4. Se o desenvolvedor nomeia bibliotecas específicas sem ser solicitado E substitui sugestões do Claude E expressa preferências fortes --> `opinionated`

**Pontuação de confiança:**

- **HIGH:** 10+ decisões de fornecedor/biblioteca observadas, mesmo padrão em 2+ projetos
- **MEDIUM:** 5-9 decisões, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 decisões de fornecedor observadas, OU padrão varia
- **UNSCORED:** 0 mensagens com sinais de seleção de fornecedor

**Citações de exemplo:**

- **pragmatic-fast:** "Use whatever ORM you recommend. I just need it working." / "Sure, Tailwind is fine."
- **conservative:** "Is Prisma the most widely used ORM for this? I want something with a large community." / "Let's stick with what most teams use."
- **thorough-evaluator:** "Before we pick a state management library, can you compare Zustand vs Jotai vs Redux Toolkit? I want to understand bundle size, API surface, and TypeScript support."
- **opinionated:** "We're using Drizzle, not Prisma. I've used both and Drizzle's SQL-like API is better for complex queries."

**Padrões dependentes de contexto:**

A filosofia de fornecedor pode mudar com base na importância do projeto ou domínio. Projetos pessoais podem usar pragmatic-fast enquanto projetos profissionais usam thorough-evaluator. Reporte a divisão se detectada.

---

### 7. Gatilhos de Frustração

`dimension_id: frustration_triggers`

**O que estamos medindo:** O que causa frustração visível, correção ou sinais emocionais negativos nas mensagens do desenvolvedor para o Claude.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `scope-creep` | Frustrado quando o Claude faz coisas que não foram pedidas. Quer execução delimitada. |
| `instruction-adherence` | Frustrado quando o Claude não segue as instruções com precisão. Valoriza exatidão. |
| `verbosity` | Frustrado quando o Claude explica demais ou é muito prolixo. Quer concisão. |
| `regression` | Frustrado quando o Claude quebra código funcionando enquanto corrige outra coisa. Valoriza estabilidade. |

**Padrões de sinal:**

1. **Linguagem de correção** — "I didn't ask for that", "don't do X", "I said Y not Z", "why did you change this?"
2. **Padrões de repetição** — Repetir a mesma instrução com ênfase sugere frustração com instruction-adherence
3. **Mudanças de tom emocional** — Mudança de neutro para lacônico, uso de maiúsculas, pontos de exclamação, palavras de frustração explícitas
4. **Declarações "Don't"** — "don't add extra features", "don't explain so much", "don't touch that file" — o que proíbem revela o que os frustra
5. **Recuperação de frustração** — Com que rapidez o desenvolvedor retorna ao tom neutro após um evento de frustração

**Heurísticas de detecção:**

1. Se o desenvolvedor corrige o Claude por fazer trabalho não solicitado E usa linguagem como "I only asked for X", "stop adding things", "stick to what I asked" --> `scope-creep`
2. Se o desenvolvedor repete instruções E corrige desvios específicos dos requisitos declarados E enfatiza precisão ("I specifically said...") --> `instruction-adherence`
3. Se o desenvolvedor pede ao Claude para ser mais curto E pula explicações E expressa irritação com o tamanho ("too much", "just the answer") --> `verbosity`
4. Se o desenvolvedor expressa frustração com funcionalidade quebrada E verifica regressões E diz "you broke X while fixing Y" --> `regression`

**Pontuação de confiança:**

- **HIGH:** 10+ eventos de frustração mostrando padrão de gatilho consistente, mesmo gatilho em 2+ projetos
- **MEDIUM:** 5-9 eventos de frustração, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 eventos de frustração observados (nota: baixa contagem de frustração é POSITIVA — significa que o desenvolvedor está geralmente satisfeito, não que os dados são insuficientes)
- **UNSCORED:** 0 mensagens com sinais de frustração (nota: "nenhuma frustração detectada" é uma descoberta válida)

**Citações de exemplo:**

- **scope-creep:** "I asked you to fix the login bug, not refactor the entire auth module. Revert everything except the bug fix."
- **instruction-adherence:** "I said to use a Map, not an object. I was specific about this. Please redo it with a Map."
- **verbosity:** "Way too much explanation. Just show me the code change, nothing else."
- **regression:** "The search was working fine before. Now after your 'fix' to the filter, search results are empty. Don't touch things I didn't ask you to change."

**Padrões dependentes de contexto:**

Os gatilhos de frustração tendem a ser consistentes entre projetos (orientados pela personalidade, não pelo projeto). No entanto, sua intensidade pode variar com a importância do projeto. Se múltiplos gatilhos de frustração forem observados, reporte o primário (mais frequente) e anote os secundários.

---

### 8. Estilo de Aprendizado

`dimension_id: learning_style`

**O que estamos medindo:** Como o desenvolvedor prefere entender novos conceitos, ferramentas ou padrões que encontra.

**Espectro de avaliação:**

| Avaliação | Descrição |
|-----------|-----------|
| `self-directed` | Lê código diretamente, descobre coisas de forma independente. Faz perguntas específicas ao Claude. |
| `guided` | Pede ao Claude para explicar partes relevantes. Prefere entendimento guiado. |
| `documentation-first` | Lê documentações e tutoriais oficiais antes de mergulhar. Faz referência à documentação. |
| `example-driven` | Quer exemplos funcionando para modificar e aprender. Aprende por correspondência de padrões. |

**Padrões de sinal:**

1. **Iniciação do aprendizado** — O desenvolvedor começa lendo código, pedindo explicação, solicitando documentação ou pedindo exemplos?
2. **Referência a fontes externas** — Menções de documentação, tutoriais, Stack Overflow, posts de blog sugerem documentation-first
3. **Solicitações de exemplo** — "show me an example", "can you give me a sample?", "let me see how this looks in practice"
4. **Indicadores de leitura de código** — "I looked at the implementation", "I see that X calls Y", "from reading the code..."
5. **Solicitações de explicação vs. solicitações de código** — Proporção de mensagens "explain X" para "show me X"

**Heurísticas de detecção:**

1. Se o desenvolvedor referencia leitura de código diretamente E faz perguntas específicas e direcionadas E demonstra investigação independente --> `self-directed`
2. Se o desenvolvedor pede ao Claude para explicar conceitos E solicita detalhamentos E prefere entendimento mediado pelo Claude --> `guided`
3. Se o desenvolvedor cita documentação E pede links de documentação E menciona leitura de tutoriais ou guias oficiais --> `documentation-first`
4. Se o desenvolvedor solicita exemplos E modifica exemplos fornecidos E aprende por correspondência de padrões --> `example-driven`

**Pontuação de confiança:**

- **HIGH:** 10+ interações de aprendizado mostrando preferência consistente, mesma preferência em 2+ projetos
- **MEDIUM:** 5-9 interações de aprendizado, OU consistente dentro de 1 projeto apenas
- **LOW:** < 5 interações de aprendizado, OU preferência varia por familiaridade com o tópico
- **UNSCORED:** 0 mensagens com sinais relevantes para aprendizado

**Citações de exemplo:**

- **self-directed:** "I read through the middleware code. The issue is that the token check happens after the rate limiter. Should those be swapped?"
- **guided:** "Can you walk me through how the auth flow works in this codebase? Start from the login request."
- **documentation-first:** "I read the Prisma docs on relations. Can you help me apply the many-to-many pattern from their guide to our schema?"
- **example-driven:** "Show me a working example of a protected API route with JWT validation. I'll adapt it for our endpoints."

**Padrões dependentes de contexto:**

O estilo de aprendizado frequentemente varia com a expertise no domínio. Um desenvolvedor pode ser self-directed em domínios familiares, mas guided ou example-driven em novos. Reporte a divisão se detectada: "context-dependent: self-directed for TypeScript/Node, example-driven for Rust/systems programming."

---

## Curadoria de Evidências

### Formato de Evidência

Use o formato combinado para cada entrada de evidência:

**Signal:** [interpretação do padrão — o que a citação demonstra] / **Example:** "[citação reduzida, ~100 caracteres]" -- project: [nome do projeto]

### Metas de Evidência

- **3 citações de evidência por dimensão** (24 no total em todas as 8 dimensões)
- Selecione citações que melhor ilustrem o padrão avaliado
- Prefira citações de projetos diferentes para demonstrar consistência entre projetos
- Quando menos de 3 citações relevantes existirem, inclua o que estiver disponível e anote a contagem de evidências

### Truncamento de Citações

- Reduza as citações ao sinal comportamental — a parte que demonstra o padrão
- Alvo de aproximadamente 100 caracteres por citação
- Preserve o fragmento significativo, não a mensagem completa
- Se o sinal estiver no meio de uma mensagem longa, use "..." para indicar truncamento
- Nunca inclua a mensagem completa de 500 caracteres quando 50 capturam o sinal

### Atribuição de Projeto

- Toda citação de evidência deve incluir o nome do projeto
- A atribuição de projeto permite verificação e mostra padrões entre projetos
- Formato: `-- project: [name]`

### Exclusão de Conteúdo Sensível (Camada 1)

O agente profiler nunca deve selecionar citações contendo qualquer um dos seguintes padrões:

- `sk-` (prefixos de chave de API)
- `Bearer ` (tokens de autenticação)
- `password` (credenciais)
- `secret` (segredos)
- `token` (quando usado como valor de credencial, não discussão conceitual)
- `api_key` ou `API_KEY` (referências a chaves de API)
- Caminhos de arquivo absolutos completos contendo nomes de usuário (ex.: `/Users/john/...`, `/home/john/...`)

**Quando conteúdo sensível for encontrado e excluído**, reporte como metadados na saída de análise:

```json
{
  "sensitive_excluded": [
    { "type": "api_key_pattern", "count": 2 },
    { "type": "file_path_with_username", "count": 1 }
  ]
}
```

Esses metadados permitem auditoria de defesa em profundidade. A Camada 2 (filtro de regex na etapa de gravação de perfil) fornece uma segunda passagem, mas o profiler ainda deve evitar selecionar citações sensíveis.

### Prioridade de Linguagem Natural

Pese mensagens em linguagem natural mais alto do que:
- Saída de log colada (detectada por timestamps, strings de formato repetidas, `[DEBUG]`, `[INFO]`, `[ERROR]`)
- Dumps de contexto de sessão (mensagens começando com "This session is being continued from a previous conversation")
- Grandes colagens de código (mensagens onde > 80% do conteúdo está dentro de cercas de código)

Esses tipos de mensagem são genuínos, mas carregam menos sinal comportamental. Despriorize-os ao selecionar citações de evidência.

---

## Ponderação por Recência

### Diretriz

Sessões recentes (últimos 30 dias) devem ser ponderadas aproximadamente 3x em comparação com sessões mais antigas ao analisar padrões.

### Justificativa

Os estilos dos desenvolvedores evoluem. Um desenvolvedor que era lacônico há seis meses pode agora fornecer contexto estruturado detalhado. O comportamento recente é uma reflexão mais precisa do estilo de trabalho atual.

### Aplicação

1. Ao contar sinais para pontuação de confiança, sinais recentes contam 3x (ex.: 4 sinais recentes = 12 sinais ponderados)
2. Ao selecionar citações de evidência, prefira citações recentes em detrimento de mais antigas quando ambas demonstram o mesmo padrão
3. Quando padrões conflitam entre sessões recentes e mais antigas, o padrão recente tem precedência para a avaliação, mas anote a evolução: "recently shifted from terse-direct to conversational"
4. A janela de 30 dias é relativa à data de análise, não a uma data fixa

### Casos Extremos

- Se TODAS as sessões forem mais antigas que 30 dias, não aplique ponderação (todas as sessões são igualmente desatualizadas)
- Se TODAS as sessões estiverem dentro dos últimos 30 dias, não aplique ponderação (todas as sessões são igualmente recentes)
- O peso de 3x é uma diretriz, não um multiplicador rígido — use julgamento quando a contagem ponderada muda um limiar de confiança

---

## Tratamento de Dados Escassos

### Limiares de Mensagem

| Total de Mensagens Genuínas | Modo | Comportamento |
|----------------------------|------|---------------|
| > 50 | `full` | Análise completa em todas as 8 dimensões. Questionário opcional (o usuário pode escolher complementar). |
| 20-50 | `hybrid` | Analise as mensagens disponíveis. Pontue cada dimensão com confiança. Complemente com questionário para dimensões LOW/UNSCORED. |
| < 20 | `insufficient` | Todas as dimensões pontuadas como LOW ou UNSCORED. Recomende questionário de fallback como fonte principal de perfil. Nota: "dados de sessão insuficientes para análise comportamental." |

### Tratando Dimensões Insuficientes

Quando uma dimensão específica tem dados insuficientes (mesmo que o total de mensagens exceda os limiares):

- Defina a confiança como `UNSCORED`
- Defina o resumo como: "Dados insuficientes — nenhum sinal claro detectado para esta dimensão."
- Defina claude_instruction para um fallback neutro: "No strong preference detected. Ask the developer when this dimension is relevant."
- Defina evidence_quotes como array vazio `[]`
- Defina evidence_count como `0`

### Suplemento por Questionário

Quando operando no modo `hybrid`, o questionário preenche lacunas para dimensões onde a análise de sessão produziu confiança LOW ou UNSCORED. As avaliações derivadas do questionário usam:
- Confiança **MEDIUM** para escolhas fortes e definitivas
- Confiança **LOW** para "varia" ou seleções ambíguas

Se a análise de sessão e o questionário concordam em uma dimensão, a confiança pode ser elevada (ex.: sessão LOW + questionário MEDIUM em acordo = MEDIUM).

---

## Esquema de Saída

O agente profiler deve retornar JSON correspondendo a este esquema exato, envolvido em tags `<analysis>`.

```json
{
  "profile_version": "1.0",
  "analyzed_at": "ISO-8601 timestamp",
  "data_source": "session_analysis",
  "projects_analyzed": ["project-name-1", "project-name-2"],
  "messages_analyzed": 0,
  "message_threshold": "full|hybrid|insufficient",
  "sensitive_excluded": [
    { "type": "string", "count": 0 }
  ],
  "dimensions": {
    "communication_style": {
      "rating": "terse-direct|conversational|detailed-structured|mixed",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [
        {
          "signal": "Interpretação do padrão descrevendo o que a citação demonstra",
          "quote": "Citação reduzida, aproximadamente 100 caracteres",
          "project": "project-name"
        }
      ],
      "summary": "Descrição de uma a duas frases do padrão observado",
      "claude_instruction": "Diretiva imperativa para o Claude: 'Match structured communication style' not 'You tend to provide structured context'"
    },
    "decision_speed": {
      "rating": "fast-intuitive|deliberate-informed|research-first|delegator",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "explanation_depth": {
      "rating": "code-only|concise|detailed|educational",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "debugging_approach": {
      "rating": "fix-first|diagnostic|hypothesis-driven|collaborative",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "ux_philosophy": {
      "rating": "function-first|pragmatic|design-conscious|backend-focused",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "vendor_philosophy": {
      "rating": "pragmatic-fast|conservative|thorough-evaluator|opinionated",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "frustration_triggers": {
      "rating": "scope-creep|instruction-adherence|verbosity|regression",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    },
    "learning_style": {
      "rating": "self-directed|guided|documentation-first|example-driven",
      "confidence": "HIGH|MEDIUM|LOW|UNSCORED",
      "evidence_count": 0,
      "cross_project_consistent": true,
      "evidence_quotes": [],
      "summary": "string",
      "claude_instruction": "string"
    }
  }
}
```

### Notas do Esquema

- **`profile_version`**: Sempre `"1.0"` para esta versão do esquema
- **`analyzed_at`**: Timestamp ISO-8601 de quando a análise foi realizada
- **`data_source`**: `"session_analysis"` para perfilamento baseado em sessão, `"questionnaire"` para somente questionário, `"hybrid"` para combinado
- **`projects_analyzed`**: Lista de nomes de projetos que contribuíram com mensagens
- **`messages_analyzed`**: Número total de mensagens genuínas do usuário processadas
- **`message_threshold`**: Qual modo de limiar foi ativado (`full`, `hybrid`, `insufficient`)
- **`sensitive_excluded`**: Array de tipos de conteúdo sensível excluídos com contagens (array vazio se nenhum encontrado)
- **`claude_instruction`**: Deve ser escrito em forma imperativa direcionada ao Claude. Este campo é como o perfil se torna acionável.
  - Bom: "Provide structured responses with headers and numbered lists to match this developer's communication style."
  - Ruim: "You tend to like structured responses."
  - Bom: "Ask before making changes beyond the stated request -- this developer values bounded execution."
  - Ruim: "The developer gets frustrated when you do extra work."

---

## Consistência Entre Projetos

### Avaliação

Para cada dimensão, avalie se o padrão observado é consistente entre os projetos analisados:

- **`cross_project_consistent: true`** — A mesma avaliação se aplicaria independentemente de qual projeto é analisado. Evidência de 2+ projetos mostra o mesmo padrão.
- **`cross_project_consistent: false`** — O padrão varia por projeto. Inclua uma nota dependente de contexto no resumo.

### Reportando Divisões

Quando `cross_project_consistent` é false, o resumo deve descrever a divisão:

- "Context-dependent: terse-direct for CLI/backend projects (gsd-tools, api-server), detailed-structured for frontend projects (dashboard, landing-page)."
- "Context-dependent: fast-intuitive for familiar tech (React, Node), research-first for new domains (Rust, ML)."

O campo de avaliação deve refletir o padrão **dominante** (mais evidências). O resumo descreve a nuance.

### Resolução da Fase 3

Divisões dependentes de contexto são resolvidas durante a orquestração da Fase 3. O orquestrador apresenta a divisão ao desenvolvedor e pergunta qual padrão representa sua preferência geral. Até ser resolvido, o Claude usa o padrão dominante com consciência da variação dependente de contexto.

---

*Versão do documento de referência: 1.0*
*Dimensões: 8*
*Esquema: profile_version 1.0*
