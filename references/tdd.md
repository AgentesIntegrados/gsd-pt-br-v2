<overview>
TDD é sobre qualidade de design, não métricas de cobertura. O ciclo vermelho-verde-refatorar força você a pensar sobre o comportamento antes da implementação, produzindo interfaces mais limpas e código mais testável.

**Princípio:** Se você pode descrever o comportamento como `expect(fn(input)).toBe(output)` antes de escrever `fn`, o TDD melhora o resultado.

**Insight fundamental:** O trabalho TDD é fundamentalmente mais pesado do que tarefas padrão — requer 2-3 ciclos de execução (RED → GREEN → REFACTOR), cada um com leituras de arquivo, execuções de teste e depuração potencial. Funcionalidades TDD recebem planos dedicados para garantir que o contexto completo esteja disponível durante todo o ciclo.
</overview>

<when_to_use_tdd>
## Quando o TDD Melhora a Qualidade

**Candidatos ao TDD (criar um plano TDD):**
- Lógica de negócios com entradas/saídas definidas
- Endpoints de API com contratos de requisição/resposta
- Transformações de dados, parsing, formatação
- Regras de validação e restrições
- Algoritmos com comportamento testável
- Máquinas de estado e fluxos de trabalho
- Funções utilitárias com especificações claras

**Pule o TDD (use plano padrão com tarefas `type="auto"`):**
- Layout de UI, estilização, componentes visuais
- Alterações de configuração
- Código de cola conectando componentes existentes
- Scripts e migrações pontuais
- CRUD simples sem lógica de negócios
- Prototipagem exploratória

**Heurística:** Você consegue escrever `expect(fn(input)).toBe(output)` antes de escrever `fn`?
→ Sim: Crie um plano TDD
→ Não: Use plano padrão, adicione testes depois se necessário
</when_to_use_tdd>

<tdd_plan_structure>
## Estrutura do Plano TDD

Cada plano TDD implementa **uma funcionalidade** através do ciclo completo RED-GREEN-REFACTOR.

```markdown
---
phase: XX-name
plan: NN
type: tdd
---

<objective>
[Qual funcionalidade e por quê]
Purpose: [Benefício de design do TDD para esta funcionalidade]
Output: [Funcionalidade funcionando e testada]
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@relevant/source/files.ts
</context>

<feature>
  <name>[Nome da funcionalidade]</name>
  <files>[arquivo fonte, arquivo de teste]</files>
  <behavior>
    [Comportamento esperado em termos testáveis]
    Cases: input → expected output
  </behavior>
  <implementation>[Como implementar assim que os testes passarem]</implementation>
</feature>

<verification>
[Comando de teste que prova que a funcionalidade funciona]
</verification>

<success_criteria>
- Teste com falha escrito e commitado
- Implementação passa no teste
- Refatoração completa (se necessário)
- Todos os 2-3 commits presentes
</success_criteria>

<output>
Após a conclusão, criar SUMMARY.md com:
- RED: Qual teste foi escrito, por que falhou
- GREEN: Qual implementação fez passar
- REFACTOR: Qual limpeza foi feita (se houver)
- Commits: Lista de commits produzidos
</output>
```

**Uma funcionalidade por plano TDD.** Se as funcionalidades são triviais o suficiente para agrupar, são triviais o suficiente para pular o TDD — use um plano padrão e adicione testes depois.
</tdd_plan_structure>

<execution_flow>
## Ciclo Vermelho-Verde-Refatorar

**RED - Escrever teste com falha:**
1. Criar arquivo de teste seguindo as convenções do projeto
2. Escrever teste descrevendo o comportamento esperado (do elemento `<behavior>`)
3. Executar o teste — DEVE falhar
4. Se o teste passar: a funcionalidade existe ou o teste está errado. Investigue.
5. Commit: `test({phase}-{plan}): add failing test for [feature]`

**GREEN - Implementar para passar:**
1. Escrever o código mínimo para fazer o teste passar
2. Sem esperteza, sem otimização — apenas faça funcionar
3. Executar o teste — DEVE passar
4. Commit: `feat({phase}-{plan}): implement [feature]`

**REFACTOR (se necessário):**
1. Limpar a implementação se melhorias óbvias existirem
2. Executar testes — DEVEM continuar passando
3. Fazer commit apenas se houver mudanças: `refactor({phase}-{plan}): clean up [feature]`

**Resultado:** Cada plano TDD produz 2-3 commits atômicos.
</execution_flow>

<test_quality>
## Testes Bons vs. Testes Ruins

**Teste comportamento, não implementação:**
- Bom: "retorna string de data formatada"
- Ruim: "chama o helper formatDate com os parâmetros corretos"
- Testes devem sobreviver a refatorações

**Um conceito por teste:**
- Bom: Testes separados para entrada válida, entrada vazia, entrada malformada
- Ruim: Teste único verificando todos os casos extremos com múltiplas asserções

**Nomes descritivos:**
- Bom: "should reject empty email", "returns null for invalid ID"
- Ruim: "test1", "handles error", "works correctly"

**Sem detalhes de implementação:**
- Bom: Testar API pública, comportamento observável
- Ruim: Mockar internos, testar métodos privados, assertar sobre estado interno
</test_quality>

<framework_setup>
## Configuração de Framework de Teste (Se Nenhum Existir)

Ao executar um plano TDD sem framework de teste configurado, configure-o como parte da fase RED:

**1. Detectar tipo de projeto:**
```bash
# JavaScript/TypeScript
if [ -f package.json ]; then echo "node"; fi

# Python
if [ -f requirements.txt ] || [ -f pyproject.toml ]; then echo "python"; fi

# Go
if [ -f go.mod ]; then echo "go"; fi

# Rust
if [ -f Cargo.toml ]; then echo "rust"; fi
```

**2. Instalar framework mínimo:**
| Projeto | Framework | Instalação |
|---------|-----------|------------|
| Node.js | Jest | `npm install -D jest @types/jest ts-jest` |
| Node.js (Vite) | Vitest | `npm install -D vitest` |
| Python | pytest | `pip install pytest` |
| Go | testing | Embutido |
| Rust | cargo test | Embutido |

**3. Criar configuração se necessário:**
- Jest: `jest.config.js` com preset ts-jest
- Vitest: `vitest.config.ts` com test globals
- pytest: `pytest.ini` ou seção `pyproject.toml`

**4. Verificar configuração:**
```bash
# Executar suite de testes vazia — deve passar com 0 testes
npm test  # Node
pytest    # Python
go test ./...  # Go
cargo test    # Rust
```

**5. Criar primeiro arquivo de teste:**
Siga as convenções do projeto para localização de testes:
- `*.test.ts` / `*.spec.ts` ao lado do fonte
- Diretório `__tests__/`
- Diretório `tests/` na raiz

A configuração do framework é um custo único incluído na fase RED do primeiro plano TDD.
</framework_setup>

<error_handling>
## Tratamento de Erros

**Teste não falha na fase RED:**
- A funcionalidade pode já existir — investigue
- O teste pode estar errado (não testando o que você acha)
- Corrija antes de prosseguir

**Teste não passa na fase GREEN:**
- Depure a implementação
- Não pule para refatorar
- Continue iterando até ficar verde

**Testes falham na fase REFACTOR:**
- Desfaça a refatoração
- O commit foi prematuro
- Refatore em passos menores

**Testes não relacionados quebram:**
- Pare e investigue
- Pode indicar problema de acoplamento
- Corrija antes de prosseguir
</error_handling>

<commit_pattern>
## Padrão de Commit para Planos TDD

Planos TDD produzem 2-3 commits atômicos (um por fase):

```
test(08-02): add failing test for email validation

- Tests valid email formats accepted
- Tests invalid formats rejected
- Tests empty input handling

feat(08-02): implement email validation

- Regex pattern matches RFC 5322
- Returns boolean for validity
- Handles edge cases (empty, null)

refactor(08-02): extract regex to constant (optional)

- Moved pattern to EMAIL_REGEX constant
- No behavior changes
- Tests still pass
```

**Comparação com planos padrão:**
- Planos padrão: 1 commit por tarefa, 2-4 commits por plano
- Planos TDD: 2-3 commits para uma única funcionalidade

Ambos seguem o mesmo formato: `{type}({phase}-{plan}): {description}`

**Benefícios:**
- Cada commit é revertível de forma independente
- Git bisect funciona no nível de commit
- Histórico claro mostrando disciplina TDD
- Consistente com a estratégia geral de commits
</commit_pattern>

<gate_enforcement>
## Regras de Imposição de Gates

Quando `workflow.tdd_mode` está habilitado na configuração, a sequência de gates RED/GREEN/REFACTOR é imposta para todos os planos `type: tdd`.

### Definições de Gates

| Gate | Obrigatório | Padrão de Commit | Validação |
|------|-------------|-----------------|-----------|
| RED | Sim | `test({phase}-{plan}): ...` | Teste existe E falha antes da implementação |
| GREEN | Sim | `feat({phase}-{plan}): ...` | Teste passa após implementação |
| REFACTOR | Não | `refactor({phase}-{plan}): ...` | Testes ainda passam após limpeza |

### Regras de Falha Rápida

1. **GREEN inesperado na fase RED:** Se o teste passar antes que qualquer código de implementação seja escrito, PARE. A funcionalidade pode já existir ou o teste está errado. Investigue antes de prosseguir.
2. **Commit RED ausente:** Se nenhum commit `test(...)` preceder o commit `feat(...)`, a disciplina TDD foi violada. Sinalize em SUMMARY.md.
3. **REFACTOR quebra testes:** Desfaça a refatoração imediatamente. O commit foi prematuro — refatore em passos menores.

### Validação do Gate pelo Executor

Após concluir um plano `type: tdd`, o executor valida o log do git:
```bash
# Verificar commit do gate RED
git log --oneline --grep="^test(${PHASE}-${PLAN})" | head -1
# Verificar commit do gate GREEN  
git log --oneline --grep="^feat(${PHASE}-${PLAN})" | head -1
# Verificar commit opcional do gate REFACTOR
git log --oneline --grep="^refactor(${PHASE}-${PLAN})" | head -1
```

Se commits dos gates RED ou GREEN estiverem ausentes, adicione uma seção `## TDD Gate Compliance` ao SUMMARY.md com os detalhes da violação.
</gate_enforcement>

<end_of_phase_review>
## Checkpoint de Revisão TDD ao Final da Fase

Quando `workflow.tdd_mode` está habilitado, o orquestrador execute-phase insere um checkpoint de revisão colaborativo após a conclusão de todas as waves, mas antes da verificação da fase.

### Formato do Checkpoint de Revisão

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 TDD REVIEW — Phase {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TDD Plans: {count} | Gate violations: {count}

| Plan | RED | GREEN | REFACTOR | Status |
|------|-----|-------|----------|--------|
| {id} |  ✓  |   ✓   |    ✓     | Pass   |
| {id} |  ✓  |   ✗   |    —     | FAIL   |

{Se houver violações:}
⚠ Violações de gate são consultivas — revise antes de avançar.
```

### O que a Revisão Verifica

1. **Sequência de gates:** Cada plano TDD tem commits RED → GREEN em ordem
2. **Qualidade do teste:** Testes da fase RED falham pelo motivo correto (não erros de importação ou sintaxe)
3. **GREEN mínimo:** A implementação é mínima — sem otimização prematura na fase GREEN
4. **Disciplina de refatoração:** Se o commit REFACTOR existe, os testes ainda passam

Este checkpoint é consultivo — ele não bloqueia a conclusão da fase, mas expõe problemas de disciplina TDD para revisão humana.
</end_of_phase_review>

<context_budget>
## Orçamento de Contexto

Planos TDD visam **~40% de uso de contexto** (menor do que os ~50% dos planos padrão).

Por que menor:
- Fase RED: escrever teste, executar teste, potencialmente depurar por que não falhou
- Fase GREEN: implementar, executar teste, potencialmente iterar sobre falhas
- Fase REFACTOR: modificar código, executar testes, verificar ausência de regressões

Cada fase envolve leitura de arquivos, execução de comandos, análise de saída. A troca de mensagens é inerentemente mais pesada do que a execução linear de tarefas.

O foco em uma única funcionalidade garante qualidade total durante todo o ciclo.
</context_budget>
