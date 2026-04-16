---
component: verifier
version: 2
last_calibrated: 2026-03-25
calibration_source: "Adapted from thinking-partner verification corpus"
---

# Exemplos Few-Shot do Verificador

## Exemplos Positivos

### Exemplo 1: Verificação de artefato multi-nível completa com descoberta real de lacuna

**Entrada:** Artefato must-have: `get-shit-done/workflows/context-bridge.md: >50 lines`

**Saída:**

> **L1 (Existência):** EXISTS — `get-shit-done/workflows/context-bridge.md` encontrado, 187 linhas.
> **L2 (Substancial):** SUBSTANTIVE — Sem marcadores TODO/FIXME, 4 seções definidas (`updateContextTier`, `getContextBudget`, `injectContextWarning`, `resetTierState`), lógica não trivial com leituras de configuração e cálculos de limiar.
> **L3 (Conectado):** WIRED — Referenciado por `get-shit-done/workflows/execute-phase.md` no passo 3. Chamado por `agents/gsd-executor.md` via include por referência @. Argumentos verificados: recebe o caminho do diretório de planejamento correspondendo ao parâmetro esperado.
> **Status: PASSED**

**Por que é bom:** O verificador verificou todos os três níveis de forma independente com evidências específicas em cada nível — caminho do arquivo, contagem de linhas, símbolos exportados, localização da importação e verificação de argumento. Não parou em "arquivo existe", mas rastreou a conexão até o uso real com argumentos corretos.

### Exemplo 2: Classificação FAILED correta com evidência de grep

**Entrada:** Verdade must-have: `validate-commit.js rejects commits missing scope parentheses`

**Saída:**

> **Método:** Executou `grep -n "scope" get-shit-done/bin/validate-commit.js` para localizar a lógica de validação de escopo.
> **Evidência:** Linhas 34-41 mostram regex `/^(feat|fix|refactor|test|docs|chore|wip)\(/` que requer parêntese de abertura após o tipo. Porém, nenhum teste para parêntese de FECHAMENTO AUSENTE existe — o regex aceita `feat(: description` (sem conteúdo de escopo e parêntese de fechamento).
> **Executou:** `echo "feat(: test" | node get-shit-done/bin/validate-commit.js` — código de saída 0 (aceito).
> **Status: FAILED** — Mensagens de commit com escopo vazio `feat(: desc)` são aceitas. A verdade "rejects commits missing scope parentheses" não se sustenta para o caso extremo de escopo vazio.

**Por que é bom:** O verificador não confiou na intenção do código — executou um teste real com um caso extremo e reportou a falha específica. A evidência inclui o regex, o comando de teste, o código de saída e por que a verdade não se sustenta.

### Exemplo 3: Capturando conexão ausente (padrão de lacuna derivado de calibração)

**Entrada:** Chave must-have key_link: `check-subagent-output.js imports and calls feedback-loop.js extractFeedback()`

**Saída:**

> **Método:** Executou `grep -n "feedback-loop\|extractFeedback" get-shit-done/bin/check-subagent-output.js`.
> **Evidência:** Zero correspondências. O arquivo `feedback-loop.js` existe (172 linhas, substancial), e `tests/feedback-loop.test.js` passa (229 linhas). Mas `check-subagent-output.js` nunca importa ou chama ele.
> **Verificação cruzada:** `grep -rn "require.*feedback-loop" get-shit-done/bin/` retorna zero resultados em todos os scripts.
> **Status: FAILED** — O artefato existe e é testado, mas está completamente órfão. O key_link "imports and calls" não se sustenta. Este é um gap de **conexão ausente**.

**Por que é bom:** O verificador capturou um padrão comum de calibração: código que existe, é testado, mas nunca é realmente conectado ao sistema. L1 e L2 passam, mas L3 (conexão) falha. Este padrão representa 37% de todas as lacunas encontradas no corpus de calibração.

### Exemplo 4: Capturando entradas de schema de configuração ausentes (padrão de lacuna derivado de calibração)

**Entrada:** Verdade must-have: `Config toggles features.architecture_graph and features.architecture_guard exist with defaults`

**Saída:**

> **Método:** Executou `grep "architecture_graph\|architecture_guard" get-shit-done/bin/lib/config-schema.json`.
> **Evidência:** Zero correspondências no schema. Porém, `grep "architecture" get-shit-done/bin/lib/config.cjs` mostra ambas as flags em CONFIG_DEFAULTS (linhas 426, 428).
> **Lacuna:** Os padrões existem no código, mas a validação do schema não os conhece. `configValidate()` removeria esses campos no carregamento, tornando os padrões inalcançáveis.
> **Status: FAILED** — As flags de configuração funcionam no código, mas são invisíveis para a validação do schema. Este é um gap de **conexão ausente** entre camadas de configuração.

**Por que é bom:** O verificador verificou múltiplas camadas de configuração (schema vs padrões vs código) em vez de apenas uma. A conexão de configuração requer que schema + padrões + código concordem — verificar apenas uma camada perde bugs de incompatibilidade entre camadas.

### Exemplo 5: Distinguindo lacunas de planejamento de falhas de execução

**Entrada:** Verdade must-have: `All MSYS regex patterns consolidated into shared module`

**Saída:**

> **Método:** Executou `grep -rn "\/\[a-zA-Z\]\/" get-shit-done/bin/` para encontrar regex MSYS inline.
> **Evidência:** 2 correspondências em `commands/gsd/misc.md` (linhas 26, 487). Estes NÃO estavam no escopo do plano original — a Fase 148 extraiu código para misc.md APÓS o plano da Fase 149 ser escrito.
> **Avaliação:** O executor completou corretamente todas as tarefas planejadas. A lacuna é um **problema de ordenação de planejamento**, não uma falha de execução.
> **Status: PASS_WITH_NOTES** — O executor fez o que foi planejado. Regex não consolidado anotado como acompanhamento, não como falha desta fase.

**Por que é bom:** O verificador distinguiu entre o que foi planejado vs. o que existe agora. Código que mudou APÓS a criação do plano é uma lacuna de planejamento, não uma falha do executor. Isso evita veredictos FAIL falsos em trabalho corretamente executado.

## Padrões de Lacuna Derivados de Calibração

Com base na análise do corpus de verificação (taxa de aprovação de 80%, 8 lacunas):

| Padrão | Contagem | % das Lacunas | O que Verificar |
|--------|----------|---------------|-----------------|
| Conexão ausente | 3 | 37% | Arquivo existe + testado mas nunca importado/chamado pelo consumidor |
| Testes ausentes | 2 | 25% | Funcionalidade funciona mas arquivo de teste excluído do CI ou nunca executado |
| Outros | 3 | 38% | Incompatibilidades de camada de configuração, problemas de ordenação de planejamento, lacunas de schema |

## Exemplos Negativos

### Exemplo 1: Aprovação em branco sem evidência por critério

**Entrada:** Must-haves: 5 verdades, 3 artefatos, 2 key_links do frontmatter do plano.

**Saída:**

> All must-haves have been verified. The implementation matches the plan requirements. Files exist and are properly connected. Status: PASSED.
> must_haves_checked: 10, must_haves_passed: 10

**Por que é ruim:** Nenhum must-have individual é listado ou verificado. Sem caminhos de arquivo, sem números de linha, sem saída de grep, sem detalhamento nível por nível. O verificador reformulou os must-haves como aprovados sem mostrar nenhuma evidência. Isso perderia stubs, exportações órfãs e conexões quebradas.

### Exemplo 2: Verificação de existência de arquivo sem verificação de substância ou conexão

**Entrada:** Artefato must-have: `get-shit-done/bin/lib/audit-checks/index.cjs: >100 lines`

**Saída:**

> **L1 (Existência):** EXISTS — Arquivo encontrado em `get-shit-done/bin/lib/audit-checks/index.cjs`, 142 linhas.
> **Status: PASSED**

**Por que é ruim:** O verificador parou no Nível 1. O arquivo tem 142 linhas, mas poderia conter `// TODO: implement all checks` com funções stub retornando objetos vazios. O Nível 2 (substancial) e o Nível 3 (conectado) foram completamente ignorados. Um arquivo que existe mas nunca é importado ou contém apenas código placeholder não deveria passar.
