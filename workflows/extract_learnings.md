<purpose>
Extrai decisões, lições aprendidas, padrões descobertos e surpresas encontradas dos artefatos de fase concluídos em um arquivo LEARNINGS.md estruturado. Captura o conhecimento institucional que de outra forma seria perdido entre as fases.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt de invocação antes de começar.
</required_reading>

<objective>
Analise os artefatos de fase concluídos (PLAN.md, SUMMARY.md, VERIFICATION.md, UAT.md, STATE.md) e extraia aprendizados estruturados em 4 categorias: decisões, lições, padrões e surpresas. Cada item extraído inclui atribuição de fonte. A saída é um arquivo LEARNINGS.md com frontmatter YAML contendo metadados sobre a extração.
</objective>

<process>

<step name="initialize">
Analise os argumentos e carregue o estado do projeto:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Analise do JSON init: `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `padded_phase`.

Se a fase não for encontrada, encerre com erro: "Fase {PHASE_ARG} não encontrada."
</step>

<step name="collect_artifacts">
Leia os artefatos da fase. PLAN.md e SUMMARY.md são obrigatórios; VERIFICATION.md, UAT.md e STATE.md são opcionais.

**Artefatos obrigatórios:**
- `${PHASE_DIR}/*-PLAN.md` — todos os arquivos de plano da fase
- `${PHASE_DIR}/*-SUMMARY.md` — todos os arquivos de resumo da fase

Se os arquivos PLAN.md ou SUMMARY.md não forem encontrados ou estiverem ausentes, encerre com erro: "Artefatos obrigatórios ausentes. PLAN.md e SUMMARY.md são necessários para a extração de aprendizados."

**Artefatos opcionais (leia se disponíveis, ignore se não encontrados):**
- `${PHASE_DIR}/*-VERIFICATION.md` — resultados de verificação
- `${PHASE_DIR}/*-UAT.md` — resultados de testes de aceitação do usuário
- `.planning/STATE.md` — estado do projeto com decisões e bloqueadores

Registre quais artefatos opcionais estão ausentes para o campo `missing_artifacts` do frontmatter.
</step>

<step name="extract_learnings">
Analise todos os artefatos coletados e extraia os aprendizados em 4 categorias:

### 1. Decisões
Decisões técnicas e arquiteturais tomadas durante a fase. Procure por:
- Decisões explícitas documentadas em PLAN.md ou SUMMARY.md
- Escolhas de tecnologia e sua justificativa
- Trade-offs que foram avaliados
- Decisões de design registradas em STATE.md

Cada entrada de decisão deve incluir:
- **O que** foi decidido
- **Por que** foi decidido (justificativa)
- **Fonte:** atribuição ao artefato onde a decisão foi encontrada (ex.: "Fonte: 03-01-PLAN.md")

### 2. Lições
Coisas aprendidas durante a execução que não eram conhecidas previamente. Procure por:
- Complexidade inesperada em SUMMARY.md
- Problemas descobertos durante a verificação em VERIFICATION.md
- Abordagens malsucedidas documentadas em SUMMARY.md
- Feedback de UAT que revelou lacunas

Cada entrada de lição deve incluir:
- **O que** foi aprendido
- **Contexto** da lição
- **Fonte:** atribuição ao artefato de origem

### 3. Padrões
Padrões reutilizáveis, abordagens ou técnicas descobertas. Procure por:
- Padrões de implementação bem-sucedidos em SUMMARY.md
- Padrões de teste de VERIFICATION.md ou UAT.md
- Padrões de fluxo de trabalho que funcionaram bem
- Padrões de organização de código de PLAN.md

Cada entrada de padrão deve incluir:
- **Padrão** nome/descrição
- **Quando usar**
- **Fonte:** atribuição ao artefato de origem

### 4. Surpresas
Descobertas, comportamentos ou resultados inesperados. Procure por:
- Coisas que levaram mais ou menos tempo do que o estimado
- Dependências ou interações inesperadas
- Casos extremos não antecipados no planejamento
- Desempenho ou comportamento que diferiu das expectativas

Cada entrada de surpresa deve incluir:
- **O que** foi surpreendente
- **Impacto** da surpresa
- **Fonte:** atribuição ao artefato de origem
</step>

<step name="capture_thought_integration">
Se a ferramenta `capture_thought` estiver disponível na sessão atual, capture cada aprendizado extraído como um pensamento com metadados:

```
capture_thought({
  category: "decision" | "lesson" | "pattern" | "surprise",
  phase: PHASE_NUMBER,
  content: LEARNING_TEXT,
  source: ARTIFACT_NAME
})
```

Se `capture_thought` não estiver disponível (ex.: o runtime não suporta), ignore este passo graciosamente e continue. O arquivo LEARNINGS.md é a saída principal — capture_thought é uma integração suplementar que fornece um fallback para runtimes com suporte a captura de pensamentos. O fluxo não deve falhar ou avisar se capture_thought estiver indisponível.
</step>

<step name="write_learnings">
Escreva o arquivo LEARNINGS.md no diretório da fase. Se um LEARNINGS.md anterior existir, sobrescreva-o (substitua o arquivo completamente).

Caminho de saída: `${PHASE_DIR}/${PADDED_PHASE}-LEARNINGS.md`

O arquivo deve ter frontmatter YAML com estes campos:
```yaml
---
phase: {PHASE_NUMBER}
phase_name: "{PHASE_NAME}"
project: "{PROJECT_NAME}"
generated: "{ISO_DATE}"
counts:
  decisions: {N}
  lessons: {N}
  patterns: {N}
  surprises: {N}
missing_artifacts:
  - "{ARTIFACT_NAME}"
---
```

O corpo segue esta estrutura:
```markdown
# Phase {PHASE_NUMBER} Learnings: {PHASE_NAME}

## Decisões

### {Título da Decisão}
{O que foi decidido}

**Justificativa:** {Por que}
**Fonte:** {arquivo de artefato}

---

## Lições

### {Título da Lição}
{O que foi aprendido}

**Contexto:** {contexto}
**Fonte:** {arquivo de artefato}

---

## Padrões

### {Nome do Padrão}
{Descrição}

**Quando usar:** {aplicabilidade}
**Fonte:** {arquivo de artefato}

---

## Surpresas

### {Título da Surpresa}
{O que foi surpreendente}

**Impacto:** {descrição do impacto}
**Fonte:** {arquivo de artefato}
```
</step>

<step name="update_state">
Atualize STATE.md para refletir a extração de aprendizados:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" state update "Last Activity" "$(date +%Y-%m-%d)"
```
</step>

<step name="report">
```
---------------------------------------------------------------

## Aprendizados Extraídos: Fase {X} — {Name}

Decisões:  {N}
Lições:    {N}
Padrões:   {N}
Surpresas: {N}
Total:     {N}

Saída: {PHASE_DIR}/{PADDED_PHASE}-LEARNINGS.md

Artefatos ausentes: {lista ou "nenhum"}

Próximos passos:
- Revise os aprendizados extraídos para verificar precisão
- /gsd-progress — veja o estado geral do projeto
- /gsd-execute-phase {next} — continue para a próxima fase

---------------------------------------------------------------
```
</step>

</process>

<success_criteria>
- [ ] Artefatos da fase localizados e lidos com sucesso
- [ ] Todas as 4 categorias extraídas: decisões, lições, padrões, surpresas
- [ ] Cada item extraído tem atribuição de fonte
- [ ] LEARNINGS.md escrito com frontmatter YAML correto
- [ ] Artefatos opcionais ausentes rastreados no frontmatter
- [ ] Integração com capture_thought tentada se a ferramenta estiver disponível
- [ ] STATE.md atualizado com atividade de extração
- [ ] Usuário recebe relatório resumido
</success_criteria>

<critical_rules>
- PLAN.md e SUMMARY.md são obrigatórios — encerre com erro claro se estiverem ausentes
- VERIFICATION.md, UAT.md e STATE.md são opcionais — extraia deles se presentes, ignore graciosamente se não encontrados
- Todo aprendizado extraído deve ter atribuição de fonte de volta ao artefato de origem
- Executar extract-learnings duas vezes na mesma fase deve sobrescrever (substituir) o LEARNINGS.md anterior, não anexar
- Não invente aprendizados — extraia apenas o que está explicitamente documentado nos artefatos
- Se capture_thought estiver indisponível, o fluxo não deve falhar — degradação graciosa para saída somente em arquivo
- O frontmatter do LEARNINGS.md deve incluir contagens para todas as 4 categorias e listar quaisquer missing_artifacts
</critical_rules>
