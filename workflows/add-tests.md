<purpose>
Gera testes unitários e E2E para uma fase concluída com base em seu SUMMARY.md, CONTEXT.md e implementação. Classifica cada arquivo alterado em categorias TDD (unitário), E2E (navegador) ou Skip, apresenta um plano de testes para aprovação do usuário e então gera os testes seguindo as convenções RED-GREEN.

Atualmente os usuários criam prompts `/gsd-quick` manualmente para geração de testes após cada fase. Este workflow padroniza o processo com classificação adequada, gates de qualidade e relatório de lacunas.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analise `$ARGUMENTS` para:
- Número da fase (inteiro, decimal ou com sufixo de letra) → armazene como `$PHASE_ARG`
- Texto restante após o número da fase → armazene como `$EXTRA_INSTRUCTIONS` (opcional)

Exemplo: `/gsd-add-tests 12 foco nos casos extremos` → `$PHASE_ARG=12`, `$EXTRA_INSTRUCTIONS="foco nos casos extremos"`

Se nenhum argumento de fase for fornecido:

```
ERRO: Número de fase obrigatório
Uso: /gsd-add-tests <fase> [instruções adicionais]
Exemplo: /gsd-add-tests 12
Exemplo: /gsd-add-tests 12 foco nos casos extremos no módulo de preços
```

Encerrar.
</step>

<step name="init_context">
Carregue o contexto da operação de fase:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extraia do JSON de inicialização: `phase_dir`, `phase_number`, `phase_name`.

Verifique se o diretório da fase existe. Se não:
```
ERRO: Diretório da fase não encontrado para a fase ${PHASE_ARG}
Verifique se a fase existe em .planning/phases/
```
Encerrar.

Leia os artefatos da fase (em ordem de prioridade):
1. `${phase_dir}/*-SUMMARY.md` — o que foi implementado, arquivos alterados
2. `${phase_dir}/CONTEXT.md` — critérios de aceitação, decisões
3. `${phase_dir}/*-VERIFICATION.md` — cenários verificados pelo usuário (se UAT foi feito)

Se nenhum SUMMARY.md existir:
```
ERRO: Nenhum SUMMARY.md encontrado para a fase ${PHASE_ARG}
Este comando funciona em fases concluídas. Execute /gsd-execute-phase primeiro.
```
Encerrar.

Apresente o banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► ADICIONAR TESTES — Fase ${phase_number}: ${phase_name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="analyze_implementation">
Extraia a lista de arquivos modificados pela fase do SUMMARY.md (seção "Files Changed" ou equivalente).

Para cada arquivo, classifique em uma das três categorias:

| Categoria | Critério | Tipo de Teste |
|----------|----------|-----------|
| **TDD** | Funções puras onde `expect(fn(input)).toBe(output)` pode ser escrito | Testes unitários |
| **E2E** | Comportamento de UI verificável por automação de navegador | Testes Playwright/E2E |
| **Skip** | Não testável de forma significativa ou já coberto | Nenhum |

**Classificação TDD — aplique quando:**
- Lógica de negócio: cálculos, preços, regras fiscais, validação
- Transformações de dados: mapeamento, filtragem, agregação, formatação
- Parsers: CSV, JSON, XML, parsing de formato customizado
- Validadores: validação de entrada, validação de schema, regras de negócio
- Máquinas de estado: transições de status, etapas de fluxo de trabalho
- Utilitários: manipulação de strings, tratamento de datas, formatação de números

**Classificação E2E — aplique quando:**
- Atalhos de teclado: key bindings, teclas modificadoras, sequências de acordes
- Navegação: transições de página, roteamento, breadcrumbs, voltar/avançar
- Interações de formulário: envio, erros de validação, foco em campo, autocomplete
- Seleção: seleção de linha, multi-seleção, intervalos por shift-click
- Arrastar e soltar: reordenação, mover entre contêineres
- Diálogos modais: abrir, fechar, confirmar, cancelar
- Grids de dados: ordenação, filtragem, edição inline, redimensionamento de coluna

**Classificação Skip — aplique quando:**
- Layout/estilo de UI: classes CSS, aparência visual, breakpoints responsivos
- Configuração: arquivos de configuração, variáveis de ambiente, feature flags
- Código de ligação: configuração de injeção de dependência, registro de middleware, tabelas de roteamento
- Migrações: migrações de banco de dados, alterações de schema
- CRUD simples: criar/ler/atualizar/deletar básico sem lógica de negócio
- Definições de tipo: registros, DTOs, interfaces sem lógica

Leia cada arquivo para verificar a classificação. Não classifique apenas pelo nome do arquivo.
</step>

<step name="present_classification">
Apresente a classificação ao usuário para confirmação antes de prosseguir:


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON de inicialização for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada AskUserQuestion por uma lista numerada em texto simples e peça ao usuário que digite o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.

```
AskUserQuestion(
  header: "Classificação de Testes",
  question: |
    ## Arquivos classificados para testes

    ### TDD (Testes Unitários) — {N} arquivos
    {lista de arquivos com breve justificativa}

    ### E2E (Testes de Navegador) — {M} arquivos
    {lista de arquivos com breve justificativa}

    ### Skip — {K} arquivos
    {lista de arquivos com breve justificativa}

    {se $EXTRA_INSTRUCTIONS: "Instruções adicionais: ${EXTRA_INSTRUCTIONS}"}

    Como você gostaria de prosseguir?
  options:
    - "Aprovar e gerar plano de testes"
    - "Ajustar classificação (vou especificar as mudanças)"
    - "Cancelar"
)
```

Se o usuário selecionar "Ajustar classificação": aplique as mudanças e reapresente.
Se o usuário selecionar "Cancelar": encerre com elegância.
</step>

<step name="discover_test_structure">
Antes de gerar o plano de testes, descubra a estrutura de testes existente do projeto:

```bash
# Encontre diretórios de testes existentes
find . -type d -name "*test*" -o -name "*spec*" -o -name "*__tests__*" 2>/dev/null | head -20
# Encontre arquivos de teste existentes para correspondência de convenções
find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "*Tests.fs" -o -name "*Test.fs" \) 2>/dev/null | head -20
# Verifique executores de teste
ls package.json *.sln 2>/dev/null || true
```

Identifique:
- Estrutura de diretório de testes (onde ficam os testes unitários, onde ficam os E2E)
- Convenções de nomenclatura (`.test.ts`, `.spec.ts`, `*Tests.fs`, etc.)
- Comandos de execução de testes (como executar testes unitários, como executar testes E2E)
- Framework de testes (xUnit, NUnit, Jest, Playwright, etc.)

Se a estrutura de testes for ambígua, pergunte ao usuário:
```
AskUserQuestion(
  header: "Estrutura de Testes",
  question: "Encontrei múltiplas localizações de testes. Onde devo criar os testes?",
  options: [lista de localizações descobertas]
)
```
</step>

<step name="generate_test_plan">
Para cada arquivo aprovado, crie um plano de testes detalhado.

**Para arquivos TDD**, planeje testes seguindo RED-GREEN-REFACTOR:
1. Identifique funções/métodos testáveis no arquivo
2. Para cada função: liste cenários de entrada, saídas esperadas, casos extremos
3. Nota: como o código já existe, os testes podem passar imediatamente — tudo bem, mas verifique se testam o comportamento CORRETO

**Para arquivos E2E**, planeje testes seguindo os gates RED-GREEN:
1. Identifique cenários de usuário do CONTEXT.md/VERIFICATION.md
2. Para cada cenário: descreva a ação do usuário, resultado esperado, asserções
3. Nota: o gate RED significa confirmar que o teste falharia se a funcionalidade estivesse quebrada

Apresente o plano de testes completo:

```
AskUserQuestion(
  header: "Plano de Testes",
  question: |
    ## Plano de Geração de Testes

    ### Testes Unitários ({N} testes em {M} arquivos)
    {para cada arquivo: caminho do arquivo de teste, lista de casos de teste}

    ### Testes E2E ({P} testes em {Q} arquivos)
    {para cada arquivo: caminho do arquivo de teste, lista de cenários}

    ### Comandos de Teste
    - Unitário: {comando descoberto}
    - E2E: {comando E2E descoberto}

    Pronto para gerar?
  options:
    - "Gerar todos"
    - "Escolher a dedo (vou especificar quais)"
    - "Ajustar plano"
)
```

Se "Escolher a dedo": pergunte ao usuário quais testes incluir.
Se "Ajustar plano": aplique as mudanças e reapresente.
</step>

<step name="execute_tdd_generation">
Para cada teste TDD aprovado:

1. **Crie o arquivo de teste** seguindo as convenções do projeto descobertas (diretório, nomenclatura, imports)

2. **Escreva o teste** com estrutura clara de arrange/act/assert:
   ```
   // Arrange — configure entradas e saídas esperadas
   // Act — chame a função sendo testada
   // Assert — verifique se a saída corresponde às expectativas
   ```

3. **Execute o teste**:
   ```bash
   {comando de teste descoberto}
   ```

4. **Avalie o resultado:**
   - **Teste passa**: Ótimo — a implementação satisfaz o teste. Verifique se o teste verifica comportamento significativo (não apenas se compila).
   - **Teste falha com erro de asserção**: Pode ser um bug genuíno descoberto pelo teste. Sinalize:
     ```
     ⚠️ Possível bug encontrado: {nome do teste}
     Esperado: {expected}
     Atual: {actual}
     Arquivo: {arquivo de implementação}
     ```
     NÃO corrija a implementação — este é um comando de geração de testes, não de correção. Registre o achado.
   - **Teste falha com erro (import, sintaxe, etc.)**: Este é um erro de teste. Corrija o teste e re-execute.
</step>

<step name="execute_e2e_generation">
Para cada teste E2E aprovado:

1. **Verifique se há testes existentes** cobrindo o mesmo cenário:
   ```bash
   grep -r "{palavra-chave do cenário}" {diretório de testes E2E} 2>/dev/null || true
   ```
   Se encontrado, estenda em vez de duplicar.

2. **Crie o arquivo de teste** visando o cenário de usuário do CONTEXT.md/VERIFICATION.md

3. **Execute o teste E2E**:
   ```bash
   {comando E2E descoberto}
   ```

4. **Avalie o resultado:**
   - **VERDE (passa)**: Registre o sucesso
   - **VERMELHO (falha)**: Determine se é um problema de teste ou um bug genuíno da aplicação. Sinalize bugs:
     ```
     ⚠️ Falha E2E: {nome do teste}
     Cenário: {descrição}
     Erro: {mensagem de erro}
     ```
   - **Não pode executar**: Reporte o bloqueador. NÃO marque como completo.
     ```
     🛑 Bloqueador E2E: {razão pela qual os testes não podem executar}
     ```

**Regra sem-pular:** Se os testes E2E não puderem ser executados (dependências faltando, problemas de ambiente), reporte o bloqueador e marque o teste como incompleto. Nunca marque sucesso sem realmente executar o teste.
</step>

<step name="summary_and_commit">
Crie um relatório de cobertura de testes e apresente ao usuário:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► GERAÇÃO DE TESTES CONCLUÍDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Resultados

| Categoria | Gerados | Passando | Falhando | Bloqueados |
|----------|-----------|---------|---------|---------|
| Unitário  | {N}       | {n1}    | {n2}    | {n3}    |
| E2E       | {M}       | {m1}    | {m2}    | {m3}    |

## Arquivos Criados/Modificados
{lista de arquivos de teste com caminhos}

## Lacunas de Cobertura
{áreas que não puderam ser testadas e por quê}

## Bugs Descobertos
{quaisquer falhas de asserção que indicam bugs de implementação}
```

Registre a geração de testes no estado do projeto:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state-snapshot
```

Se houver testes passando para fazer commit:

```bash
git add {arquivos de teste}
git commit -m "test(phase-${phase_number}): add unit and E2E tests from add-tests command"
```

Apresente os próximos passos:

```
---

## ▶ Próximo Passo

{se bugs foram descobertos:}
**Corrigir bugs descobertos:** `/gsd-quick fix the {N} test failures discovered in phase ${phase_number}`

{se há testes bloqueados:}
**Resolver bloqueadores de teste:** {descrição do que é necessário}

{caso contrário:}
**Todos os testes passando!** A fase ${phase_number} está totalmente testada.

---

**Também disponível:**
- `/gsd-add-tests {próxima_fase}` — testar outra fase
- `/gsd-verify-work {phase_number}` — executar verificação UAT

---
```
</step>

</process>

<success_criteria>
- [ ] Artefatos da fase carregados (SUMMARY.md, CONTEXT.md, opcionalmente VERIFICATION.md)
- [ ] Todos os arquivos alterados classificados em categorias TDD/E2E/Skip
- [ ] Classificação apresentada ao usuário e aprovada
- [ ] Estrutura de testes do projeto descoberta (diretórios, convenções, executores)
- [ ] Plano de testes apresentado ao usuário e aprovado
- [ ] Testes TDD gerados com estrutura arrange/act/assert
- [ ] Testes E2E gerados visando cenários de usuário
- [ ] Todos os testes executados — nenhum teste não executado marcado como passando
- [ ] Bugs descobertos pelos testes sinalizados (não corrigidos)
- [ ] Arquivos de teste commitados com mensagem adequada
- [ ] Lacunas de cobertura documentadas
- [ ] Próximos passos apresentados ao usuário
</success_criteria>
