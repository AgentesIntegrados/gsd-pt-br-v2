# Tipos de Artefatos GSD

Esta referência documenta todos os tipos de artefatos na taxonomia de planejamento GSD. Cada tipo tem uma
forma definida, ciclo de vida, localização e mecanismo de consumo. Um artefato bem formatado que nenhum workflow
lê é inerte — o mecanismo de consumo é o que dá significado a um artefato.

---

## Artefatos Principais

### ROADMAP.md
- **Forma**: Listagem de marcos + fases com objetivos e referências canônicas
- **Ciclo de vida**: Criado → Atualizado por marco → Arquivado
- **Localização**: `.planning/ROADMAP.md`
- **Consumido por**: Comandos `plan-phase`, `discuss-phase`, `execute-phase`, `progress`, `state`

### STATE.md
- **Forma**: Rastreador de posição atual (fase, plano, progresso, decisões)
- **Ciclo de vida**: Atualizado continuamente ao longo do projeto
- **Localização**: `.planning/STATE.md`
- **Consumido por**: Todos os workflows de orquestração; comandos `resume-project`, `progress`, `next`

### REQUIREMENTS.md
- **Forma**: Critérios de aceitação numerados com tabela de rastreabilidade
- **Ciclo de vida**: Criado no início do projeto → Atualizado conforme os requisitos são satisfeitos
- **Localização**: `.planning/REQUIREMENTS.md`
- **Consumido por**: `discuss-phase`, `plan-phase`, geração de CONTEXT.md; executor marca como concluído

### CONTEXT.md (por fase)
- **Forma**: Formato de 6 seções: domain, decisions, canonical_refs, code_context, specifics, deferred
- **Ciclo de vida**: Criado antes do planejamento → Usado durante planejamento e execução → Substituído pela próxima fase
- **Localização**: `.planning/phases/XX-name/XX-CONTEXT.md`
- **Consumido por**: `plan-phase` (lê decisions), `execute-phase` (lê code_context e canonical_refs)

### PLAN.md (por plano)
- **Forma**: Frontmatter + objetivo + tarefas com tipos + critérios de sucesso + especificação de saída
- **Ciclo de vida**: Criado pelo planejador → Executado → SUMMARY.md produzido
- **Localização**: `.planning/phases/XX-name/XX-YY-PLAN.md`
- **Consumido por**: Executor de `execute-phase`; commits de tarefas referenciam IDs de plano

### SUMMARY.md (por plano)
- **Forma**: Frontmatter com grafo de dependência + narrativa + desvios + self-check
- **Ciclo de vida**: Criado na conclusão do plano → Lido por planos subsequentes na mesma fase
- **Localização**: `.planning/phases/XX-name/XX-YY-SUMMARY.md`
- **Consumido por**: Orquestrador (progress), planejador (contexto para futuros planos), `milestone-summary`

### HANDOFF.json / .continue-here.md
- **Forma**: Estado de pausa estruturado (JSON legível por máquina + Markdown legível por humano)
- **Ciclo de vida**: Criado na pausa → Consumido na retomada → Substituído pela próxima pausa
- **Localização**: `.planning/HANDOFF.json` + `.planning/phases/XX-name/.continue-here.md` (ou caminho de spike/deliberação)
- **Consumido por**: Workflow `resume-project`

---

## Artefatos Estendidos

### DISCUSSION-LOG.md (por fase)
- **Forma**: Trilha de auditoria de suposições e correções da discuss-phase
- **Ciclo de vida**: Criado no momento da discussão → Registro de auditoria somente leitura
- **Localização**: `.planning/phases/XX-name/XX-DISCUSSION-LOG.md`
- **Consumido por**: Revisão humana; não lido por workflows automatizados

### USER-PROFILE.md
- **Forma**: Nível de calibração e perfil de preferências
- **Ciclo de vida**: Criado por `profile-user` → Atualizado conforme preferências são observadas
- **Localização**: `$HOME/.claude/get-shit-done/USER-PROFILE.md`
- **Consumido por**: `discuss-phase-assumptions` (nível de calibração), `plan-phase`

### SPIKE.md / DESIGN.md (por spike)
- **Forma**: Pergunta de pesquisa + metodologia + descobertas + recomendação
- **Ciclo de vida**: Criado → Investigado → Decidido → Arquivado
- **Localização**: `.planning/spikes/SPIKE-NNN/`
- **Consumido por**: Planejador quando o spike é referenciado; `pause-work` para handoff de contexto de spike

---

## Artefatos de Referência Permanente

### METHODOLOGY.md

- **Forma**: Referência permanente — frameworks interpretativos reutilizáveis (lentes) que se aplicam entre fases
- **Ciclo de vida**: Criado → Ativo → Substituído (quando uma lente é substituída por uma melhor)
- **Localização**: `.planning/METHODOLOGY.md` (escopo do projeto, não da fase)
- **Conteúdo**: Lentes nomeadas, cada uma documentando:
  - O que diagnostica (a classe de problema que detecta)
  - O que recomenda (a classe de resposta que prescreve)
  - Quando aplicar (condições de acionamento)
  - Exemplo: Atualização Bayesiana, modelagem de ameaças STRIDE, priorização de Custo de Atraso
- **Consumido por**:
  - `discuss-phase-assumptions` — lê METHODOLOGY.md (se existir) e aplica as lentes ativas
    à análise de suposições atual antes de apresentar descobertas ao usuário
  - `plan-phase` — lê METHODOLOGY.md para informar a seleção de metodologia para cada plano
  - `pause-work` — inclui METHODOLOGY.md na seção Leitura Obrigatória de `.continue-here.md`
    para que os agentes que retomam herdem a orientação analítica do projeto

**Por que o consumo importa:** Um METHODOLOGY.md que nenhum workflow lê é inerte. As lentes só
têm efeito quando um agente as carrega em seu contexto de raciocínio antes da análise. É por isso que
os workflows discuss-phase-assumptions e pause-work referenciam explicitamente este arquivo.

**Exemplo de entrada de lente:**

```markdown
## Atualização Bayesiana

**Diagnostica:** Decisões tomadas com priors desatualizados — suposições formadas cedo que as evidências
contradizem desde então, mas que permanecem incorporadas no plano.

**Recomenda:** Antes de confirmar uma suposição, pergunte: "Que evidência me faria mudar isso?"
Se nenhuma evidência poderia mudá-la, é uma crença, não uma suposição. Sinalize para revisão do usuário.

**Aplique quando:** Qualquer suposição carrega o rótulo Confident, mas foi formada antes de mudanças
arquitetônicas recentes, atualizações de biblioteca ou correções de escopo.
```
