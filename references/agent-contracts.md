# Contratos de Agentes

Marcadores de conclusão e esquemas de handoff para todos os agentes GSD. Os workflows utilizam esses marcadores para detectar a conclusão dos agentes e rotear adequadamente.

Este documento descreve o que É, não o que deveria ser. Inconsistências de capitalização são documentadas conforme aparecem nos arquivos-fonte dos agentes.

---

## Registro de Agentes

| Agente | Função | Marcadores de Conclusão |
|-------|------|--------------------|
| gsd-planner | Criação de planos | `## PLANNING COMPLETE` |
| gsd-executor | Execução de planos | `## PLAN COMPLETE`, `## CHECKPOINT REACHED` |
| gsd-phase-researcher | Pesquisa com escopo de fase | `## RESEARCH COMPLETE`, `## RESEARCH BLOCKED` |
| gsd-project-researcher | Pesquisa de escopo do projeto | `## RESEARCH COMPLETE`, `## RESEARCH BLOCKED` |
| gsd-plan-checker | Validação de planos | `## VERIFICATION PASSED`, `## ISSUES FOUND` |
| gsd-research-synthesizer | Síntese de múltiplas pesquisas | `## SYNTHESIS COMPLETE`, `## SYNTHESIS BLOCKED` |
| gsd-debugger | Investigação de depuração | `## DEBUG COMPLETE`, `## ROOT CAUSE FOUND`, `## CHECKPOINT REACHED` |
| gsd-roadmapper | Criação/revisão de roadmap | `## ROADMAP CREATED`, `## ROADMAP REVISED`, `## ROADMAP BLOCKED` |
| gsd-ui-auditor | Revisão de UI | `## UI REVIEW COMPLETE` |
| gsd-ui-checker | Validação de UI | `## ISSUES FOUND` |
| gsd-ui-researcher | Criação de especificação de UI | `## UI-SPEC COMPLETE`, `## UI-SPEC BLOCKED` |
| gsd-verifier | Verificação pós-execução | `## Verification Complete` (title case) |
| gsd-integration-checker | Verificação de integração entre fases | `## Integration Check Complete` (title case) |
| gsd-nyquist-auditor | Auditoria por amostragem | `## PARTIAL`, `## ESCALATE` (não-padrão) |
| gsd-security-auditor | Auditoria de segurança | `## OPEN_THREATS`, `## ESCALATE` (não-padrão) |
| gsd-codebase-mapper | Análise da base de código | Sem marcador (escreve documentos diretamente) |
| gsd-assumptions-analyzer | Extração de suposições | Sem marcador (retorna seções `## Assumptions`) |
| gsd-doc-verifier | Validação de documentos | Sem marcador (escreve JSON em `.planning/tmp/`) |
| gsd-doc-writer | Geração de documentos | Sem marcador (escreve documentos diretamente) |
| gsd-advisor-researcher | Pesquisa consultiva | Sem marcador (agente utilitário) |
| gsd-user-profiler | Criação de perfil do usuário | Sem marcador (retorna JSON em tags de análise) |
| gsd-intel-updater | Análise de inteligência da base de código | `## INTEL UPDATE COMPLETE`, `## INTEL UPDATE FAILED` |

## Regras de Marcadores

1. **Marcadores em MAIÚSCULAS** (ex.: `## PLANNING COMPLETE`) são a convenção padrão
2. **Marcadores em Title Case** (ex.: `## Verification Complete`) existem no gsd-verifier e gsd-integration-checker — são intencionais, não bugs
3. **Marcadores não-padrão** (ex.: `## PARTIAL`, `## ESCALATE`) em agentes de auditoria indicam resultados parciais que requerem julgamento do orquestrador
4. **Agentes sem marcadores** escrevem artefatos diretamente no disco ou retornam dados estruturados (JSON/seções) que o chamador analisa
5. Os marcadores devem aparecer como cabeçalhos H2 (`## `) no início de uma linha na saída final do agente

## Contratos-Chave de Handoff

### Planner -> Executor (via PLAN.md)

| Campo | Obrigatório | Descrição |
|-------|----------|-------------|
| Frontmatter | Sim | phase, plan, type, wave, depends_on, files_modified, autonomous, requirements |
| `<objective>` | Sim | O que o plano alcança |
| `<tasks>` | Sim | Lista ordenada de tarefas com type, files, action, verify, acceptance_criteria |
| `<verification>` | Sim | Passos gerais de verificação |
| `<success_criteria>` | Sim | Critérios de conclusão mensuráveis |

### Executor -> Verifier (via SUMMARY.md)

| Campo | Obrigatório | Descrição |
|-------|----------|-------------|
| Frontmatter | Sim | phase, plan, subsystem, tags, key-files, metrics |
| Tabela de commits | Sim | Hashes de commit e descrições por tarefa |
| Seção de desvios | Sim | Problemas corrigidos automaticamente ou "None" |
| Self-Check | Sim | PASSED ou FAILED com detalhes |

## Padrões Regex dos Workflows

Os workflows correspondem a esses marcadores para detectar a conclusão dos agentes:

**plan-phase.md corresponde a:**
- `## RESEARCH COMPLETE` / `## RESEARCH BLOCKED` (saída do pesquisador)
- `## PLANNING COMPLETE` (saída do planejador)
- `## CHECKPOINT REACHED` (pausa do planejador/executor)
- `## VERIFICATION PASSED` / `## ISSUES FOUND` (saída do plan-checker)

**execute-phase.md corresponde a:**
- `## PHASE COMPLETE` (todos os planos da fase concluídos)
- `## Self-Check: FAILED` (self-check do summary)

> **NOTA:** `## PLAN COMPLETE` é o marcador de conclusão do gsd-executor, mas execute-phase.md não o corresponde por regex. Em vez disso, detecta a conclusão do executor via verificações pontuais (existência do SUMMARY.md, estado do commit git). Esse é o comportamento intencional, não uma inconsistência.
