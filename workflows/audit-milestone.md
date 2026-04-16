<purpose>
Verifica se o milestone atingiu sua definição de pronto ao agregar verificações de fase, verificar integração entre fases e avaliar cobertura de requisitos. Lê os arquivos VERIFICATION.md existentes (fases já verificadas durante execute-phase), agrega dívida técnica e lacunas adiadas e então inicializa o verificador de integração para fiação entre fases.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<available_agent_types>
Tipos válidos de subagente GSD (use nomes exatos — não use 'general-purpose' como fallback):
- gsd-integration-checker — Verifica integração entre fases
</available_agent_types>

<process>

## 0. Inicializar Contexto do Milestone

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init milestone-op)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS_CHECKER=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-integration-checker 2>/dev/null)
```

Extraia do JSON de inicialização: `milestone_version`, `milestone_name`, `phase_count`, `completed_phases`, `commit_docs`.

Resolva o modelo do verificador de integração:
```bash
integration_checker_model=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-integration-checker --raw)
```

## 1. Determinar Escopo do Milestone

```bash
# Obtenha as fases no milestone (ordenadas numericamente, suporta decimais)
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases list
```

- Analise a versão dos argumentos ou detecte a atual a partir do ROADMAP.md
- Identifique todos os diretórios de fase no escopo
- Extraia a definição de pronto do milestone do ROADMAP.md
- Extraia os requisitos mapeados para este milestone do REQUIREMENTS.md

## 2. Ler Todas as Verificações de Fase

Para cada diretório de fase, leia o VERIFICATION.md:

```bash
# Para cada fase, use find-phase para resolver o diretório (suporta fases arquivadas)
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase 01 --raw)
# Extraia o diretório do JSON, então leia o VERIFICATION.md desse diretório
# Repita para cada número de fase do ROADMAP.md
```

De cada VERIFICATION.md, extraia:
- **Status:** passed | gaps_found
- **Lacunas críticas:** (se houver — são bloqueadores)
- **Lacunas não críticas:** dívida técnica, itens adiados, avisos
- **Anti-padrões encontrados:** TODOs, stubs, placeholders
- **Cobertura de requisitos:** quais requisitos foram satisfeitos/bloqueados

Se uma fase não tiver VERIFICATION.md, sinalize como "fase não verificada" — isso é um bloqueador.

## 3. Inicializar Verificador de Integração

Com o contexto de fase coletado:

Extraia `MILESTONE_REQ_IDS` da tabela de rastreabilidade do REQUIREMENTS.md — todos os REQ-IDs atribuídos às fases neste milestone.

```
Task(
  prompt="Check cross-phase integration and E2E flows.

Phases: {phase_dirs}
Phase exports: {from SUMMARYs}
API routes: {routes created}

Milestone Requirements:
{MILESTONE_REQ_IDS — list each REQ-ID with description and assigned phase}

MUST map each integration finding to affected requirement IDs where applicable.

Verify cross-phase wiring and E2E user flows.
${AGENT_SKILLS_CHECKER}",
  subagent_type="gsd-integration-checker",
  model="{integration_checker_model}"
)
```

## 4. Coletar Resultados

Combine:
- Lacunas e dívida técnica em nível de fase (da etapa 2)
- Relatório do verificador de integração (lacunas de fiação, fluxos quebrados)

## 5. Verificar Cobertura de Requisitos (Referência Cruzada de 3 Fontes)

DEVE fazer referência cruzada de três fontes independentes para cada requisito:

### 5a. Analisar a Tabela de Rastreabilidade do REQUIREMENTS.md

Extraia todos os REQ-IDs mapeados para as fases do milestone a partir da tabela de rastreabilidade:
- ID do requisito, descrição, fase atribuída, status atual, estado marcado (`[x]` vs `[ ]`)

### 5b. Analisar Tabelas de Requisitos dos VERIFICATION.md das Fases

Para o VERIFICATION.md de cada fase, extraia a tabela expandida de requisitos:
- Requisito | Plano Fonte | Descrição | Status | Evidência
- Mapeie cada entrada de volta ao seu REQ-ID

### 5c. Extrair Verificação Cruzada do Frontmatter do SUMMARY.md

Para o SUMMARY.md de cada fase, extraia `requirements-completed` do frontmatter YAML:
```bash
for summary in .planning/phases/*-*/*-SUMMARY.md; do
  [ -e "$summary" ] || continue
  node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" summary-extract "$summary" --fields requirements_completed --pick requirements_completed
done
```

### 5d. Matriz de Determinação de Status

Para cada REQ-ID, determine o status usando todas as três fontes:

| Status no VERIFICATION.md | Frontmatter SUMMARY | REQUIREMENTS.md | → Status Final |
|------------------------|---------------------|-----------------|----------------|
| passed                 | listado             | `[x]`           | **satisfeito** |
| passed                 | listado             | `[ ]`           | **satisfeito** (atualize o checkbox) |
| passed                 | ausente             | qualquer        | **parcial** (verifique manualmente) |
| gaps_found             | qualquer            | qualquer        | **não satisfeito** |
| ausente                | listado             | qualquer        | **parcial** (lacuna de verificação) |
| ausente                | ausente             | qualquer        | **não satisfeito** |

### 5e. Gate de FALHA e Detecção de Órfãos

**OBRIGATÓRIO:** Qualquer requisito `não satisfeito` DEVE forçar o status `gaps_found` na auditoria do milestone.

**Detecção de órfãos:** Requisitos presentes na tabela de rastreabilidade do REQUIREMENTS.md, mas ausentes em TODOS os arquivos VERIFICATION.md de fases DEVEM ser sinalizados como órfãos. Requisitos órfãos são tratados como `não satisfeitos` — foram atribuídos, mas nunca verificados por nenhuma fase.

## 5.5. Descoberta de Conformidade Nyquist

Pule se `workflow.nyquist_validation` for explicitamente `false` (ausente = habilitado).

```bash
NYQUIST_CONFIG=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-get workflow.nyquist_validation --raw 2>/dev/null)
```

Se `false`: pule completamente.

Para cada diretório de fase, verifique `*-VALIDATION.md`. Se existir, analise o frontmatter (`nyquist_compliant`, `wave_0_complete`).

Classifique por fase:

| Status | Condição |
|--------|-----------|
| COMPLIANT | `nyquist_compliant: true` e todas as tarefas verdes |
| PARTIAL | VALIDATION.md existe, `nyquist_compliant: false` ou vermelho/pendente |
| MISSING | Sem VALIDATION.md |

Adicione ao YAML da auditoria: `nyquist: { compliant_phases, partial_phases, missing_phases, overall }`

Somente descoberta — nunca chama `/gsd-validate-phase` automaticamente.

## 6. Agregar em v{version}-MILESTONE-AUDIT.md

Crie `.planning/v{version}-v{version}-MILESTONE-AUDIT.md` com:

```yaml
---
milestone: {version}
audited: {timestamp}
status: passed | gaps_found | tech_debt
scores:
  requirements: N/M
  phases: N/M
  integration: N/M
  flows: N/M
gaps:  # Bloqueadores críticos
  requirements:
    - id: "{REQ-ID}"
      status: "unsatisfied | partial | orphaned"
      phase: "{fase atribuída}"
      claimed_by_plans: ["{arquivos de plano que referenciam este requisito}"]
      completed_by_plans: ["{arquivos de plano cujo SUMMARY marca como completo}"]
      verification_status: "passed | gaps_found | missing | orphaned"
      evidence: "{evidência específica ou falta dela}"
  integration: [...]
  flows: [...]
tech_debt:  # Não crítico, adiado
  - phase: 01-auth
    items:
      - "TODO: adicionar rate limiting"
      - "Aviso: sem validação de força de senha"
  - phase: 03-dashboard
    items:
      - "Adiado: layout responsivo para mobile"
---
```

Mais relatório completo em markdown com tabelas para requisitos, fases, integração, dívida técnica.

**Valores de status:**
- `passed` — todos os requisitos atendidos, sem lacunas críticas, dívida técnica mínima
- `gaps_found` — bloqueadores críticos existem
- `tech_debt` — sem bloqueadores, mas itens adiados acumulados precisam de revisão

## 7. Apresentar Resultados

Roteie pelo status (veja `<offer_next>`).

</process>

<offer_next>
Gere este markdown diretamente (não como bloco de código). Roteie com base no status:

---

**Se passed:**

## ✓ Milestone {version} — Auditoria Aprovada

**Pontuação:** {N}/{M} requisitos satisfeitos
**Relatório:** .planning/v{version}-MILESTONE-AUDIT.md

Todos os requisitos cobertos. Integração entre fases verificada. Fluxos E2E completos.

───────────────────────────────────────────────────────────────

## ▶ Próximo Passo

**Completar milestone** — arquivar e taguear

/clear e depois:

/gsd-complete-milestone {version}

───────────────────────────────────────────────────────────────

---

**Se gaps_found:**

## ⚠ Milestone {version} — Lacunas Encontradas

**Pontuação:** {N}/{M} requisitos satisfeitos
**Relatório:** .planning/v{version}-MILESTONE-AUDIT.md

### Requisitos Não Satisfeitos

{Para cada requisito não satisfeito:}
- **{REQ-ID}: {descrição}** (Fase {X})
  - {razão}

### Problemas Entre Fases

{Para cada lacuna de integração:}
- **{de} → {para}:** {problema}

### Fluxos Quebrados

{Para cada lacuna de fluxo:}
- **{nome do fluxo}:** quebra em {etapa}

### Cobertura Nyquist

| Fase | VALIDATION.md | Conforme | Ação |
|-------|---------------|-----------|--------|
| {fase} | existe/ausente | true/false/parcial | `/gsd-validate-phase {N}` |

Fases que precisam de validação: execute `/gsd-validate-phase {N}` para cada fase sinalizada.

───────────────────────────────────────────────────────────────

## ▶ Próximo Passo

**Planejar fechamento de lacunas** — criar fases para completar o milestone

/clear e depois:

/gsd-plan-milestone-gaps

───────────────────────────────────────────────────────────────

**Também disponível:**
- cat .planning/v{version}-MILESTONE-AUDIT.md — ver relatório completo
- /gsd-complete-milestone {version} — prosseguir mesmo assim (aceitar dívida técnica)

───────────────────────────────────────────────────────────────

---

**Se tech_debt (sem bloqueadores, mas dívida acumulada):**

## ⚡ Milestone {version} — Revisão de Dívida Técnica

**Pontuação:** {N}/{M} requisitos satisfeitos
**Relatório:** .planning/v{version}-MILESTONE-AUDIT.md

Todos os requisitos atendidos. Sem bloqueadores críticos. Dívida técnica acumulada precisa de revisão.

### Dívida Técnica por Fase

{Para cada fase com dívida:}
**Fase {X}: {nome}**
- {item 1}
- {item 2}

### Total: {N} itens em {M} fases

───────────────────────────────────────────────────────────────

## ▶ Opções

**A. Completar milestone** — aceitar dívida, rastrear no backlog

/gsd-complete-milestone {version}

**B. Planejar fase de limpeza** — tratar dívida antes de completar

/clear e depois:

/gsd-plan-milestone-gaps

───────────────────────────────────────────────────────────────
</offer_next>

<success_criteria>
- [ ] Escopo do milestone identificado
- [ ] Todos os arquivos VERIFICATION.md das fases lidos
- [ ] Frontmatter `requirements-completed` do SUMMARY.md extraído para cada fase
- [ ] Tabela de rastreabilidade do REQUIREMENTS.md analisada para todos os REQ-IDs do milestone
- [ ] Referência cruzada de 3 fontes concluída (VERIFICATION + SUMMARY + rastreabilidade)
- [ ] Requisitos órfãos detectados (na rastreabilidade, mas ausentes em todos os VERIFICATIONs)
- [ ] Dívida técnica e lacunas adiadas agregadas
- [ ] Verificador de integração iniciado com IDs de requisitos do milestone
- [ ] v{version}-MILESTONE-AUDIT.md criado com objetos de lacuna de requisito estruturados
- [ ] Gate de FALHA aplicado — qualquer requisito não satisfeito força o status gaps_found
- [ ] Conformidade Nyquist verificada para todas as fases do milestone (se habilitado)
- [ ] Fases com VALIDATION.md faltando sinalizadas com sugestão de validate-phase
- [ ] Resultados apresentados com próximos passos acionáveis
</success_criteria>
