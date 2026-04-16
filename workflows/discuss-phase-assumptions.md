<purpose>
Surfaçar suposições do Claude sobre uma fase analisando primeiro a base de código, depois apresentando suposições de forma conversacional para confirmação ou correção. Abordagem codebase-first em vez de questionamentos especulativos.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<available_agent_types>
Tipos de subagentes GSD válidos (use nomes exatos — não use substitutos genéricos):
- gsd-assumptions-analyzer — Analisa base de código para extrair suposições implícitas
</available_agent_types>

<process>

<step name="initialize">
Analisar $ARGUMENTS para fase alvo e flags:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
AGENT_SKILLS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-assumptions-analyzer 2>/dev/null)
ANALYZER_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-assumptions-analyzer --raw)
```

Extrair do JSON: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `has_context`, `has_research`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SUPOSIÇÕES DA FASE {N}: {nome}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Abordagem: Analisar base de código → surfaçar suposições → confirmar com você
```
</step>

<step name="spawn_analyzer">
Iniciar analisador de suposições para varrer a base de código:

```
◆ Analisando base de código para suposições implícitas...
```

```
Task(
  prompt="""
Leia $HOME/.claude/agents/gsd-assumptions-analyzer.md para instruções.

<objetivo>
Analisar a base de código e artefatos de planejamento para extrair suposições implícitas
sobre como a Fase {phase_number}: {phase_name} deve ser implementada.
</objetivo>

<arquivos_para_ler>
- .planning/ROADMAP.md (definição da fase)
- .planning/STATE.md (estado atual do projeto)
- .planning/REQUIREMENTS.md (requisitos)
- {context_path} (se existir: decisões de usuário anteriores)
- {research_path} (se existir: pesquisa técnica)
- Arquivos de código-fonte relevantes para o escopo da fase
</arquivos_para_ler>

${AGENT_SKILLS}

<saida>
Retornar suposições estruturadas como JSON:
{
  "assumptions": [
    {
      "category": "technical|product|process|integration",
      "assumption": "Texto da suposição",
      "basis": "Por que assumimos isso (evidência da base de código)",
      "risk": "high|medium|low",
      "question": "Como verificar/confirmar isso com o usuário"
    }
  ]
}
</saida>
""",
  subagent_type="gsd-assumptions-analyzer",
  model="{ANALYZER_MODEL}",
  description="Analisar suposições Fase {N}"
)
```

Analisar JSON de retorno. Agrupar suposições por categoria.
</step>

<step name="present_assumptions_conversationally">
Apresentar suposições de forma conversacional — uma categoria por vez.

Para cada categoria (técnico, produto, processo, integração):

Exibir:
```
## Suposições {categoria}

Baseado em minha análise da base de código:

{Para cada suposição na categoria:}
**{N}. {assumption}**
   ↳ Baseado em: {basis}
   ↳ Risco se errado: {risk}

Essas suposições estão corretas? Alguma coisa diferente do que você espera?
```

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Aguardar resposta do usuário em texto simples (sem AskUserQuestion — este é um diálogo conversacional).

Processar resposta:
- "sim"/"correto"/"ok" → marcar suposições como confirmadas
- Correção fornecida → capturar a correção, marcar como revisada
- Informação nova fornecida → adicionar como nova decisão
</step>

<step name="write_context">
Criar ou atualizar CONTEXT.md com suposições confirmadas/corrigidas:

```bash
CONTEXT_FILE="${PHASE_DIR}/${PADDED_PHASE}-CONTEXT.md"
```

Formato do arquivo CONTEXT.md:
```markdown
---
phase: "{phase_number}"
type: "assumptions"
created: "{data}"
---

## Decisões e Suposições Confirmadas

### Técnico
{suposições confirmadas com status: confirmado/corrigido}

### Produto
{suposições confirmadas com status: confirmado/corrigido}

### Processo
{suposições confirmadas com status: confirmado/corrigido}

### Integração
{suposições confirmadas com status: confirmado/corrigido}

## Correções do Usuário

{suposições onde o usuário forneceu informações diferentes}

## Perguntas Abertas

{quaisquer suposições de alto risco não ainda confirmadas}
```

Commitar se `commit_docs` for true:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(${padded_phase}): suposições da fase capturadas" --files "${CONTEXT_FILE}"
```
</step>

<step name="report">
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► SUPOSIÇÕES CAPTURADAS ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Fase {N}: {Nome}
Suposições confirmadas: {N}
Correções capturadas: {N}
Perguntas abertas: {N}

Salvo em: {caminho do CONTEXT.md}

───────────────────────────────────────────────────────────────

## ▶ Próximos passos

`/clear` então: `/gsd-plan-phase {N}`

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] gsd-assumptions-analyzer iniciado com contexto de base de código
- [ ] Suposições extraídas e agrupadas por categoria
- [ ] Suposições apresentadas conversacionalmente
- [ ] Respostas do usuário processadas (confirmado/corrigido)
- [ ] CONTEXT.md criado/atualizado com suposições
- [ ] Usuário sabe os próximos passos
</success_criteria>
