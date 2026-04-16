<purpose>
Capturar uma ideia prospectiva como um arquivo de seed estruturado com condições de gatilho.
Seeds são apresentadas automaticamente durante /gsd-new-milestone quando as condições de gatilho correspondem ao escopo do novo milestone.

Seeds superam itens diferidos porque:
- Preservam O PORQUÊ a ideia importa (não apenas O QUÊ)
- Definem QUANDO apresentar (condições de gatilho, não varredura manual)
- Rastreiam trilhas (referências de código, decisões relacionadas)
- São apresentadas automaticamente no momento certo via varredura new-milestone
</purpose>

<process>

<step name="parse_idea">
Analise `$ARGUMENTS` para o resumo da ideia.

Se vazio, pergunte:
```
Qual é a ideia? (uma frase)
```

Armazene como `$IDEA`.
</step>

<step name="create_seed_dir">
```bash
mkdir -p .planning/seeds
```
</step>

<step name="gather_context">
Faça perguntas focadas para construir uma seed completa:


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.

```
AskUserQuestion(
  header: "Gatilho",
  question: "Quando esta ideia deve ser apresentada? (ex: 'quando adicionarmos contas de usuário', 'próxima versão principal', 'quando performance se tornar prioridade')",
  options: []  // texto livre
)
```

Armazene como `$TRIGGER`.

```
AskUserQuestion(
  header: "Por que",
  question: "Por que isso importa? Que problema resolve ou que oportunidade cria?",
  options: []
)
```

Armazene como `$WHY`.

```
AskUserQuestion(
  header: "Escopo",
  question: "Qual é o tamanho disso? (estimativa aproximada)",
  options: [
    { label: "Pequeno", description: "Algumas horas — pode ser uma tarefa rápida" },
    { label: "Médio", description: "Uma ou duas fases — precisa de planejamento" },
    { label: "Grande", description: "Um milestone completo — esforço significativo" }
  ]
)
```

Armazene como `$SCOPE`.
</step>

<step name="collect_breadcrumbs">
Busque no código-fonte referências relevantes:

```bash
# Encontrar arquivos relacionados às palavras-chave da ideia
grep -rl "$KEYWORD" --include="*.ts" --include="*.js" --include="*.md" . 2>/dev/null | head -10
```

Verifique também:
- STATE.md atual para decisões relacionadas
- ROADMAP.md para fases relacionadas
- todos/ para ideias capturadas relacionadas

Armazene caminhos de arquivos relevantes como `$BREADCRUMBS`.
</step>

<step name="generate_seed_id">
```bash
# Encontrar o próximo número de seed
EXISTING=$( (ls .planning/seeds/SEED-*.md 2>/dev/null || true) | wc -l )
NEXT=$((EXISTING + 1))
PADDED=$(printf "%03d" $NEXT)
```

Gere slug a partir do resumo da ideia.
</step>

<step name="write_seed">
Escreva `.planning/seeds/SEED-{PADDED}-{slug}.md`:

```markdown
---
id: SEED-{PADDED}
status: dormant
planted: {data ISO}
planted_during: {milestone/fase atual de STATE.md}
trigger_when: {$TRIGGER}
scope: {$SCOPE}
---

# SEED-{PADDED}: {$IDEA}

## Por Que Isso Importa

{$WHY}

## Quando Apresentar

**Gatilho:** {$TRIGGER}

Esta seed deve ser apresentada durante `/gsd-new-milestone` quando o escopo
do milestone corresponder a qualquer uma destas condições:
- {condição de gatilho 1}
- {condição de gatilho 2}

## Estimativa de Escopo

**{$SCOPE}** — {elaboração baseada na escolha de escopo}

## Trilhas

Código e decisões relacionados encontrados no código-fonte atual:

{lista de $BREADCRUMBS com caminhos de arquivos}

## Notas

{qualquer contexto adicional da sessão atual}
```
</step>

<step name="commit_seed">
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: plantar seed — {$IDEA}" --files .planning/seeds/SEED-{PADDED}-{slug}.md
```
</step>

<step name="confirm">
```
✅ Seed plantada: SEED-{PADDED}

"{$IDEA}"
Gatilho: {$TRIGGER}
Escopo: {$SCOPE}
Arquivo: .planning/seeds/SEED-{PADDED}-{slug}.md

Esta seed será apresentada automaticamente quando você executar /gsd-new-milestone
e o escopo do milestone corresponder à condição de gatilho.
```
</step>

</process>

<success_criteria>
- [ ] Arquivo de seed criado em .planning/seeds/
- [ ] Frontmatter inclui status, gatilho, escopo
- [ ] Trilhas coletadas do código-fonte
- [ ] Commitado no git
- [ ] Usuário vê confirmação com informações do gatilho
</success_criteria>
</output>
