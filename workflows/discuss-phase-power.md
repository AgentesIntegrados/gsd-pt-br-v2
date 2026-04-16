<purpose>
Modo power user para discuss-phase. Gera arquivos HTML+JSON de perguntas para resposta assíncrona. Em vez de um fluxo de perguntas interativas, produz uma interface onde o usuário pode responder todas as perguntas de uma vez.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `has_context`.

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DISCUSS FASE {N} (MODO POWER)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Modo: Resposta assíncrona — perguntas geradas em HTML+JSON
```
</step>

<step name="research_phase_scope">
Ler os artefatos de planejamento da fase:

```bash
cat .planning/ROADMAP.md
cat .planning/STATE.md 2>/dev/null || true
cat .planning/REQUIREMENTS.md 2>/dev/null || true
cat "${PHASE_DIR}"/*-CONTEXT.md 2>/dev/null || true
cat "${PHASE_DIR}"/*-RESEARCH.md 2>/dev/null || true
```

Identificar:
- Meta da fase e resultados esperados
- Requisitos em escopo
- Áreas de ambiguidade
- Decisões técnicas necessárias
- Riscos conhecidos
</step>

<step name="generate_questions">
Gerar conjunto abrangente de perguntas organizadas por categoria:

**Categorias:**
1. **Escopo** — O que está dentro/fora do escopo?
2. **Técnico** — Abordagem de implementação, stack, restrições
3. **Produto** — Comportamento esperado, casos extremos, UX
4. **Integrações** — Dependências de sistemas externos
5. **Qualidade** — Critérios de sucesso, metas de desempenho
6. **Prioridade** — Se o tempo apertar, o que vai primeiro?

Para cada pergunta gerar:
```json
{
  "id": "q{N}",
  "category": "{categoria}",
  "question": "{texto da pergunta}",
  "why": "{por que isso importa para o planejamento}",
  "options": ["{opção1}", "{opção2}"],
  "default": "{default sugerido se usuário não souber}",
  "required": true|false
}
```

Mirar em 8-15 perguntas no total — apenas as que realmente afetam o planejamento.
</step>

<step name="generate_html_interface">
Gerar arquivo HTML interativo para resposta de perguntas:

```bash
QUESTIONS_DIR="${PHASE_DIR}/questions"
mkdir -p "${QUESTIONS_DIR}"
QUESTIONS_JSON="${QUESTIONS_DIR}/${PADDED_PHASE}-questions.json"
QUESTIONS_HTML="${QUESTIONS_DIR}/${PADDED_PHASE}-questions.html"
ANSWERS_JSON="${QUESTIONS_DIR}/${PADDED_PHASE}-answers.json"
```

Escrever JSON de perguntas em `${QUESTIONS_JSON}`.

Gerar HTML com:
- Todas as perguntas renderizadas como campos de formulário
- Opções como botões de rádio quando fornecidas
- Campos de texto para respostas abertas
- Indicador de progresso
- Botão "Salvar Respostas" que exporta JSON para `${ANSWERS_JSON}`
- Seções dobrável por categoria

Template HTML (versão simplificada):
```html
<!DOCTYPE html>
<html>
<head>
  <title>Fase {N}: {nome} — Perguntas de Planejamento</title>
  <style>
    /* CSS de formulário limpo e responsivo */
  </style>
</head>
<body>
  <h1>Fase {N}: {nome}</h1>
  <p>Responda estas perguntas para guiar o planejamento. Salve quando terminar.</p>

  <form id="questions-form">
    {/* Perguntas renderizadas dinamicamente */}
  </form>

  <button onclick="saveAnswers()">Salvar Respostas</button>

  <script>
    const questions = {/* JSON de perguntas incorporado */};

    function saveAnswers() {
      const answers = {};
      // ... coletar respostas do formulário
      // ... baixar como answers.json
    }
  </script>
</body>
</html>
```
</step>

<step name="present_to_user">
Exibir instruções:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PERGUNTAS GERADAS ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Fase {N}: {Nome} — {N} perguntas geradas

Arquivos:
  HTML:      {caminho do HTML}
  JSON:      {caminho do questions.json}
  Respostas: {caminho do answers.json} (será criado por você)

## Como usar

1. Abra o arquivo HTML em seu navegador:
   open {caminho do HTML}

2. Responda as perguntas no formulário

3. Clique "Salvar Respostas" para gerar {caminho do answers.json}

4. Execute: /gsd-plan-phase {N} --answers {caminho do answers.json}
   O planejador lerá suas respostas e criará os planos.

## Resposta alternativa

Você também pode editar diretamente as perguntas e respostas:
  {caminho do questions.json}

Adicione um campo "answer" a cada pergunta e execute:
  /gsd-plan-phase {N} --answers {caminho do questions.json}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Exibir as 3 primeiras perguntas como prévia:
```
## Prévia das Perguntas

**1. {categoria}: {pergunta}**
   Por quê: {explicação}
   Padrão: {default}

**2. ...**

**3. ...**

(+ {N-3} mais no arquivo HTML)
```
</step>

<step name="handle_answers_flag">
**Se `--answers {caminho}` fornecido em $ARGUMENTS:**

Ler o arquivo de respostas:
```bash
ANSWERS=$(cat "${ANSWERS_FILE}")
```

Analisar respostas e gerar CONTEXT.md:

```bash
CONTEXT_FILE="${PHASE_DIR}/${PADDED_PHASE}-CONTEXT.md"
```

Escrever CONTEXT.md com respostas mapeadas para decisões.

Exibir confirmação:
```
✓ {N} respostas processadas
✓ CONTEXT.md criado em: {caminho}

Próximo: /gsd-plan-phase {N}
```
</step>

</process>

<success_criteria>
- [ ] Escopo da fase pesquisado a partir de artefatos de planejamento
- [ ] 8-15 perguntas geradas por categoria
- [ ] JSON de perguntas escrito
- [ ] Interface HTML interativa gerada
- [ ] Instruções de uso exibidas ao usuário
- [ ] Flag --answers processa respostas em CONTEXT.md
</success_criteria>
