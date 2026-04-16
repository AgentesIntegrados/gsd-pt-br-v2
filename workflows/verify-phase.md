<purpose>
Verificar o cumprimento do objetivo da fase por meio de análise retroativa a partir do objetivo. Checar se a base de código entrega o que a fase prometeu, não apenas se as tarefas foram concluídas.

Executado por um subagente de verificação gerado pelo execute-phase.md.
</purpose>

<core_principle>
**Conclusão de tarefa ≠ Alcance do objetivo**

Uma tarefa "criar componente de chat" pode ser marcada como concluída quando o componente é um placeholder. A tarefa foi feita — mas o objetivo "interface de chat funcional" não foi alcançado.

Verificação retroativa a partir do objetivo:
1. O que precisa ser VERDADEIRO para o objetivo ser alcançado?
2. O que precisa EXISTIR para que essas verdades se sustentem?
3. O que precisa estar CONECTADO para que esses artefatos funcionem?
4. O que os TESTES precisam PROVAR para que essas verdades sejam evidenciadas?

Em seguida, verificar cada nível em relação à base de código real.
</core_principle>

<required_reading>
@$HOME/.claude/get-shit-done/references/verification-patterns.md
@$HOME/.claude/get-shit-done/templates/verification-report.md
</required_reading>

<process>

<step name="load_context" priority="first">
Carregar contexto da operação de fase:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `phase_dir`, `phase_number`, `phase_name`, `has_plans`, `plan_count`.

Em seguida, carregar detalhes da fase e listar planos/resumos:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}"
grep -E "^| ${phase_number}" .planning/REQUIREMENTS.md 2>/dev/null || true
ls "$phase_dir"/*-SUMMARY.md "$phase_dir"/*-PLAN.md 2>/dev/null || true
```

Carregar todas as fases do milestone para filtragem de itens adiados (Passo 9b):
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze
```

Extrair o **objetivo da fase** do ROADMAP.md (o resultado a verificar, não as tarefas), os **requisitos** do REQUIREMENTS.md se existir, e **todas as fases do milestone** do roadmap analyze (para cruzar lacunas com fases posteriores).
</step>

<step name="establish_must_haves">
**Opção A: Requisitos obrigatórios no frontmatter do PLAN**

Usar gsd-tools para extrair must_haves de cada PLAN:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  MUST_HAVES=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" frontmatter get "$plan" --field must_haves)
  echo "=== $plan ===" && echo "$MUST_HAVES"
done
```

Retorna JSON: `{ truths: [...], artifacts: [...], key_links: [...] }`

Agregar todos os must_haves entre os planos para verificação em nível de fase.

**Opção B: Usar Critérios de Sucesso do ROADMAP.md**

Se não houver must_haves no frontmatter (MUST_HAVES retorna erro ou está vazio), verificar Critérios de Sucesso:

```bash
PHASE_DATA=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${phase_number}" --raw)
```

Analisar o array `success_criteria` da saída JSON. Se não estiver vazio:
1. Usar cada Critério de Sucesso diretamente como uma **verdade** (já estão escritos como comportamentos observáveis e verificáveis)
2. Derivar **artefatos** (caminhos de arquivo concretos para cada verdade)
3. Derivar **conexões-chave** (ligações críticas onde stubs se escondem)
4. Documentar os requisitos obrigatórios antes de prosseguir

Os Critérios de Sucesso do ROADMAP.md são o contrato — eles substituem os must_haves em nível de PLAN quando ambos existem.

**Opção C: Derivar do objetivo da fase (fallback)**

Se não houver must_haves no frontmatter E não houver Critérios de Sucesso no ROADMAP:
1. Declarar o objetivo do ROADMAP.md
2. Derivar **verdades** (3-7 comportamentos observáveis, cada um verificável)
3. Derivar **artefatos** (caminhos de arquivo concretos para cada verdade)
4. Derivar **conexões-chave** (ligações críticas onde stubs se escondem)
5. Documentar os requisitos obrigatórios derivados antes de prosseguir
</step>

<step name="verify_truths">
Para cada verdade observável, determinar se a base de código a viabiliza.

**Status:** ✓ VERIFICADO (todos os artefatos de suporte passam) | ✗ FALHOU (artefato ausente/stub/desconectado) | ? INCERTO (requer humano)

Para cada verdade: identificar artefatos de suporte → verificar status do artefato → verificar conexão → determinar status da verdade.

**Exemplo:** A verdade "Usuário pode ver mensagens existentes" depende de Chat.tsx (renderiza), /api/chat GET (fornece), modelo Message (esquema). Se Chat.tsx for um stub ou a API retornar [] fixo → FALHOU. Se todos existem, têm conteúdo substancial e estão conectados → VERIFICADO.
</step>

<step name="verify_artifacts">
Usar gsd-tools para verificação de artefatos em relação aos must_haves de cada PLAN:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  ARTIFACT_RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" verify artifacts "$plan")
  echo "=== $plan ===" && echo "$ARTIFACT_RESULT"
done
```

Analisar resultado JSON: `{ all_passed, passed, total, artifacts: [{path, exists, issues, passed}] }`

**Status do artefato a partir do resultado:**
- `exists=false` → AUSENTE
- `issues` não vazio → STUB (verificar issues para "Only N lines" ou "Missing pattern")
- `passed=true` → VERIFICADO (Níveis 1-2 passam)

**Nível 3 — Conectado (verificação manual para artefatos que passam nos Níveis 1-2):**
```bash
grep -r "import.*$artifact_name" src/ --include="*.ts" --include="*.tsx"  # IMPORTADO
grep -r "$artifact_name" src/ --include="*.ts" --include="*.tsx" | grep -v "import"  # USADO
```
CONECTADO = importado E usado. ÓRFÃO = existe mas não é importado/usado.

| Existe | Substancial | Conectado | Status |
|--------|-------------|-----------|--------|
| ✓ | ✓ | ✓ | ✓ VERIFICADO |
| ✓ | ✓ | ✗ | ⚠️ ÓRFÃO |
| ✓ | ✗ | - | ✗ STUB |
| ✗ | - | - | ✗ AUSENTE |

**Verificação pontual de exportações (severidade AVISO):**

Para artefatos que passam no Nível 3, verificar pontualmente exportações individuais:
- Extrair símbolos exportados principais (funções, constantes, classes — ignorar types/interfaces)
- Para cada um, buscar uso fora do arquivo que o define
- Sinalizar exportações com zero sites de chamada externos como "exportado mas não utilizado"

Isso captura armazenamentos mortos como `setPlan()` que existem em um arquivo conectado mas nunca são realmente chamados. Reportar como AVISO — pode indicar conexão incompleta entre planos ou código residual de revisões de plano.
</step>

<step name="verify_wiring">
Usar gsd-tools para verificação de conexões-chave em relação aos must_haves de cada PLAN:

```bash
for plan in "$PHASE_DIR"/*-PLAN.md; do
  LINKS_RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" verify key-links "$plan")
  echo "=== $plan ===" && echo "$LINKS_RESULT"
done
```

Analisar resultado JSON: `{ all_verified, verified, total, links: [{from, to, via, verified, detail}] }`

**Status da conexão a partir do resultado:**
- `verified=true` → CONECTADO
- `verified=false` com "not found" → NÃO_CONECTADO
- `verified=false` com "Pattern not found" → PARCIAL

**Padrões de fallback (se key_links não estiver nos must_haves):**

| Padrão | Verificação | Status |
|--------|-------------|--------|
| Componente → API | chamada fetch/axios ao caminho da API, resposta usada (await/.then/setState) | CONECTADO / PARCIAL (chamada sem uso da resposta) / NÃO_CONECTADO |
| API → Banco de dados | query Prisma/DB no model, resultado retornado via res.json() | CONECTADO / PARCIAL (query sem retorno) / NÃO_CONECTADO |
| Formulário → Handler | onSubmit com implementação real (fetch/axios/mutate/dispatch), não console.log/vazio | CONECTADO / STUB (apenas log/vazio) / NÃO_CONECTADO |
| Estado → Renderização | variável useState aparece no JSX (`{stateVar}` ou `{stateVar.property}`) | CONECTADO / NÃO_CONECTADO |

Registrar status e evidências para cada conexão-chave.
</step>

<step name="verify_requirements">
Se REQUIREMENTS.md existir:
```bash
grep -E "Phase ${PHASE_NUM}" .planning/REQUIREMENTS.md 2>/dev/null || true
```

Para cada requisito: analisar descrição → identificar verdades/artefatos de suporte → status: ✓ SATISFEITO / ✗ BLOQUEADO / ? REQUER HUMANO.
</step>

<step name="behavioral_verification">
**Executar a suíte de testes do projeto e comandos CLI para verificar comportamento, não apenas estrutura.**

Verificações estáticas (grep, existência de arquivos, conexões) detectam lacunas estruturais mas não falhas em tempo de execução. Este passo executa testes reais e comandos do projeto para verificar se o objetivo da fase foi alcançado comportamentalmente.

Isso segue o princípio de engenharia de harness da Anthropic: separar geração de avaliação, com o avaliador interagindo com o sistema em execução em vez de inspecionar artefatos estáticos.

**Passo 1: Executar suíte de testes**

```bash
# Detectar executor de testes e rodar todos os testes (timeout: 5 minutos)
TEST_EXIT=0
timeout 300 bash -c '
if [ -f "package.json" ]; then
  npm test 2>&1
elif [ -f "Cargo.toml" ]; then
  cargo test 2>&1
elif [ -f "go.mod" ]; then
  go test ./... 2>&1
elif [ -f "pyproject.toml" ] || [ -f "requirements.txt" ]; then
  python -m pytest -q --tb=short 2>&1 || uv run python -m pytest -q --tb=short 2>&1
else
  echo "⚠ Nenhum executor de testes detectado — pulando suíte de testes"
  exit 1
fi
'
TEST_EXIT=$?
if [ "${TEST_EXIT}" -eq 0 ]; then
  echo "✓ Suíte de testes passou"
elif [ "${TEST_EXIT}" -eq 124 ]; then
  echo "⚠ Suíte de testes expirou após 5 minutos"
else
  echo "✗ Suíte de testes falhou (código de saída ${TEST_EXIT})"
fi
```

Registrar: total de testes, aprovados, reprovados, cobertura (se disponível).

**Se algum teste falhar:** Marcar como `behavioral_failures` — severidade BLOQUEADOR independentemente de as verificações estáticas terem passado. Uma fase não pode ser verificada se os testes falharem.

**Passo 2: Executar comandos CLI/do projeto a partir dos critérios de sucesso (se testáveis)**

Para cada critério de sucesso que descreve um comando de usuário (ex.: "Usuário pode executar `mixtiq validate`", "Usuário pode executar `npm start`"):

1. Verificar se o comando existe e se os inputs necessários estão disponíveis:
   - Procurar arquivos de exemplo em `templates/`, `fixtures/`, `test/`, `examples/`, ou `testdata/`
   - Verificar se o binário/script CLI existe no PATH ou no projeto
2. **Se não houver inputs ou fixtures adequados:** Marcar como `? REQUER HUMANO` com motivo "Nenhuma fixture de teste disponível — requer verificação manual" e prosseguir.
   Não inventar inputs de exemplo.
3. Se os inputs estiverem disponíveis: executar o comando e verificar se sai com sucesso.

```bash
# Executar apenas se tanto o comando quanto o input existirem
if command -v {project_cli} &>/dev/null && [ -f "{example_input}" ]; then
  {project_cli} {example_input} 2>&1
fi
```

Registrar: comando, código de saída, resumo da saída, aprovado/reprovado (ou IGNORADO se não houver fixtures).

**Passo 3: Relatório**

```
## Verificação Comportamental

| Verificação | Resultado | Detalhe |
|-------------|-----------|---------|
| Suíte de testes | {N} aprovados, {M} reprovados | {primeira falha, se houver} |
| {comando CLI 1} | ✓ / ✗ | {resumo da saída} |
| {comando CLI 2} | ✓ / ✗ | {resumo da saída} |
```

**Se todas as verificações comportamentais passarem:** Continuar para scan_antipatterns.
**Se alguma falhar:** Adicionar às lacunas de verificação com severidade BLOQUEADOR.
</step>

<step name="scan_antipatterns">
Extrair arquivos modificados nesta fase do SUMMARY.md, escanear cada um:

| Padrão | Busca | Severidade |
|--------|-------|------------|
| TODO/FIXME/XXX/HACK | `grep -n -E "TODO\|FIXME\|XXX\|HACK"` | ⚠️ Aviso |
| Conteúdo placeholder | `grep -n -iE "placeholder\|coming soon\|will be here"` | 🛑 Bloqueador |
| Retornos vazios | `grep -n -E "return null\|return \{\}\|return \[\]\|=> \{\}"` | ⚠️ Aviso |
| Funções só com log | Funções contendo apenas console.log | ⚠️ Aviso |

Categorizar: 🛑 Bloqueador (impede o objetivo) | ⚠️ Aviso (incompleto) | ℹ️ Info (notável).
</step>

<step name="audit_test_quality">
**Verificar se os testes PROVAM o que afirmam provar.**

Este passo captura enganos no nível de testes que passam em todas as verificações anteriores: arquivos existem, têm conteúdo substancial, estão conectados, e os testes passam — mas os testes não validam realmente o requisito.

**1. Identificar arquivos de teste vinculados a requisitos**

A partir dos arquivos PLAN e SUMMARY, mapear cada requisito para os arquivos de teste que deveriam prová-lo.

**2. Varredura de testes desabilitados**

Para TODOS os arquivos de teste vinculados a requisitos, buscar padrões desabilitados/ignorados:

```bash
grep -rn -E "it\.skip|describe\.skip|test\.skip|xit\(|xdescribe\(|xtest\(|@pytest\.mark\.skip|@unittest\.skip|#\[ignore\]|\.pending|it\.todo|test\.todo" "$TEST_FILE"
```

**Regra:** Um teste desabilitado vinculado a um requisito = requisito NÃO testado.
- 🛑 BLOQUEADOR se o teste desabilitado for o único que prova aquele requisito
- ⚠️ AVISO se outros testes ativos também cobrem o requisito

**3. Detecção de testes circulares**

Buscar scripts/utilitários que geram valores esperados executando o sistema sob teste:

```bash
grep -rn -E "writeFileSync|writeFile|fs\.write|open\(.*w\)" "$TEST_DIRS"
```

Para cada ocorrência, verificar se também importa o sistema/serviço/módulo sendo testado. Se um script importa o sistema sob teste E escreve valores de saída esperados → CIRCULAR.

**Indicadores de teste circular:**
- Script importa um serviço E escreve em arquivos de fixture
- Valores esperados têm comentários como "computed from engine", "captured from baseline"
- Nome do script contém "capture", "baseline", "generate", "snapshot" em contexto de teste
- Valores esperados foram adicionados no mesmo commit que as asserções de teste

**Regra:** Um teste que compara a saída do sistema com valores gerados pelo mesmo sistema é circular. Ele prova consistência, não correção.

**4. Proveniência dos valores esperados** (para requisitos de comparação/paridade/migração)

Quando um requisito exige comparação com uma fonte externa ("idêntico a X", "corresponde a Y", "mesma saída que Z"):

- A fonte externa é realmente invocada ou referenciada no pipeline de testes?
- Os arquivos de fixture contêm dados provenientes do sistema externo?
- Ou todos os valores esperados vêm do próprio sistema novo ou de fórmulas matemáticas?

**Classificação de proveniência:**
- VÁLIDA: Valor esperado proveniente de saída de sistema externo/legado, captura manual ou oráculo independente
- PARCIAL: Valor esperado de derivação matemática (prova a fórmula, não a correspondência com o sistema)
- CIRCULAR: Valor esperado proveniente do sistema sendo testado
- DESCONHECIDA: Nenhuma informação de proveniência — tratar como SUSPEITO

**5. Força das asserções**

Para cada teste vinculado a um requisito, classificar a asserção mais forte:

| Nível | Exemplos | Prova |
|-------|---------|-------|
| Existência | `toBeDefined()`, `!= null` | Algo foi retornado |
| Tipo | `typeof x === 'number'` | Formato correto |
| Status | `code === 200` | Sem erro |
| Valor | `toEqual(expected)`, `toBeCloseTo(x)` | Valor específico |
| Comportamental | Asserções de fluxo multi-etapa | Correção ponta a ponta |

Se um requisito exige prova em nível de valor ou comportamental e o teste tem apenas asserções de existência/tipo/status → INSUFICIENTE.

**6. Quantidade de cobertura**

Se um requisito especifica uma quantidade de casos de teste (ex.: "30 cálculos"), verificar se o número real de casos de teste ativos (não ignorados) atende ao requisito.

**Relatório — adicionar ao VERIFICATION.md:**

```markdown
### Auditoria de Qualidade dos Testes

| Arquivo de Teste | Req. Vinculado | Ativos | Ignorados | Circular | Nível de Asserção | Veredicto |
|-----------------|---------------|--------|-----------|----------|-------------------|-----------|

**Testes desabilitados em requisitos:** {N} → {BLOQUEADOR se algum req tiver APENAS testes desabilitados}
**Padrões circulares detectados:** {N} → {BLOQUEADOR se algum}
**Asserções insuficientes:** {N} → {AVISO}
```

**Impacto no status:** Qualquer BLOQUEADOR da auditoria de qualidade dos testes → status geral = `gaps_found`, independentemente de outros checks passarem.
</step>

<step name="identify_human_verification">
**Sempre requer humano:** Aparência visual, conclusão de fluxo do usuário, comportamento em tempo real (WebSocket/SSE), integração com serviço externo, sensação de desempenho, clareza das mensagens de erro.

**Requer humano se incerto:** Conexões complexas que o grep não consegue rastrear, comportamento dinâmico dependente de estado, casos de borda.

Formatar cada item como: Nome do Teste → O que fazer → Resultado esperado → Por que não é possível verificar programaticamente.
</step>

<step name="determine_status">
Classificar o status usando esta árvore de decisão EM ORDEM (mais restritivo primeiro):

1. SE qualquer verdade FALHOU, artefato AUSENTE/STUB, conexão-chave NÃO_CONECTADA, bloqueador encontrado, **ou auditoria de qualidade dos testes encontrou bloqueadores (testes de requisito desabilitados, testes circulares)**:
   → **gaps_found**

2. SE o passo anterior produziu QUALQUER item de verificação humana:
   → **human_needed** (mesmo que todas as verdades estejam VERIFICADAS e a pontuação seja N/N)

3. SE todos os checks passam E não há itens de verificação humana:
   → **passed**

**passed é VÁLIDO APENAS quando não existem itens de verificação humana.**

**Pontuação:** `verified_truths / total_truths`
</step>

<step name="filter_deferred_items">
Antes de reportar lacunas, cruzar cada lacuna com fases posteriores do milestone usando os dados completos do roadmap carregados em load_context (de `roadmap analyze`).

Para cada lacuna potencial identificada em determine_status:
1. Verificar se a verdade falha da lacuna ou o item ausente está coberto pelo objetivo ou critérios de sucesso de uma fase posterior
2. **Critérios de correspondência:** A preocupação da lacuna aparece no texto do objetivo de uma fase posterior, no texto dos critérios de sucesso, ou o nome da fase posterior claramente sugere que cobre esta área
3. Se uma correspondência clara for encontrada → mover a lacuna para uma lista `deferred` com a referência da fase correspondente e o texto de evidência
4. Se nenhuma correspondência em fases posteriores → manter como lacuna real

**Importante:** Seja conservador. Adiar uma lacuna apenas quando há evidência clara e específica em uma fase posterior. Correspondências vagas ou tangenciais NÃO devem causar adiamento — em caso de dúvida, manter como lacuna real.

**Itens adiados NÃO afetam a determinação de status.** Recalcular após filtrar:
- Se a lista de lacunas agora está vazia e não há itens humanos → `passed`
- Se a lista de lacunas está vazia mas há itens humanos → `human_needed`
- Se a lista de lacunas ainda tem itens → `gaps_found`

Incluir itens adiados no frontmatter do VERIFICATION.md (seção `deferred:`) e no corpo (tabela de Itens Adiados) para transparência. Se não houver itens adiados, omitir essas seções.
</step>

<step name="generate_fix_plans">
Se gaps_found:

1. **Agrupar lacunas relacionadas:** Stub de API + componente desconectado → "Conectar frontend ao backend". Múltiplos ausentes → "Concluir implementação central". Apenas conexão → "Conectar componentes existentes".

2. **Gerar plano por grupo:** Objetivo, 2-3 tarefas (arquivos/ação/verificação para cada), passo de re-verificação. Manter focado: uma preocupação por plano.

3. **Ordenar por dependência:** Corrigir ausentes → corrigir stubs → corrigir conexões → **corrigir evidências de teste** → verificar.
</step>

<step name="create_report">
```bash
REPORT_PATH="$PHASE_DIR/${PHASE_NUM}-VERIFICATION.md"
```

Preencher seções do template: frontmatter (fase/timestamp/status/pontuação), alcance do objetivo, tabela de artefatos, tabela de conexões, cobertura de requisitos, anti-padrões, verificação humana, resumo de lacunas, planos de correção (se gaps_found), metadados.

Ver $HOME/.claude/get-shit-done/templates/verification-report.md para o template completo.
</step>

<step name="return_to_orchestrator">
Retornar status (`passed` | `gaps_found` | `human_needed`), pontuação (N/M requisitos obrigatórios), caminho do relatório.

Se gaps_found: listar lacunas + nomes dos planos de correção recomendados.
Se human_needed: listar itens que requerem teste humano.

O orquestrador roteia: `passed` → update_roadmap | `gaps_found` → criar/executar correções, re-verificar | `human_needed` → apresentar ao usuário.
</step>

</process>

<success_criteria>
- [ ] Requisitos obrigatórios estabelecidos (do frontmatter ou derivados)
- [ ] Todas as verdades verificadas com status e evidência
- [ ] Todos os artefatos verificados nos três níveis
- [ ] Todas as conexões-chave verificadas
- [ ] Cobertura de requisitos avaliada (se aplicável)
- [ ] Anti-padrões escaneados e categorizados
- [ ] Qualidade dos testes auditada (testes desabilitados, padrões circulares, força das asserções, proveniência)
- [ ] Itens de verificação humana identificados
- [ ] Status geral determinado
- [ ] Itens adiados filtrados em relação a fases posteriores do milestone (se lacunas encontradas)
- [ ] Planos de correção gerados (se gaps_found após filtragem)
- [ ] VERIFICATION.md criado com relatório completo
- [ ] Resultados retornados ao orquestrador
</success_criteria>
