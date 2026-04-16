<purpose>
Fluxo de ideação socrática. Guia o desenvolvedor na exploração de uma ideia por meio de perguntas instigantes, oferece pesquisa no meio da conversa quando útil e então roteia os resultados cristalizados para artefatos GSD.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt de invocação antes de começar.

@$HOME/.claude/get-shit-done/references/questioning.md
@$HOME/.claude/get-shit-done/references/domain-probes.md
</required_reading>

<available_agent_types>
Tipos válidos de subagente GSD (use nomes exatos — não recorra a 'general-purpose'):
- gsd-phase-researcher — Pesquisa perguntas específicas e retorna descobertas concisas
</available_agent_types>

<process>

## Passo 1: Abrir a conversa

Se um tópico foi fornecido, reconheça-o e comece a explorar:
```
## Explorar: {topic}

Vamos pensar juntos sobre isso. Vou fazer perguntas para ajudar a clarificar
a ideia antes de nos comprometermos com qualquer artefato.
```

Se nenhum tópico foi fornecido, pergunte:
```
## Explorar

O que está passando pela sua cabeça? Pode ser uma ideia de funcionalidade, uma
questão arquitetural, um problema que você está tentando resolver ou algo que
você ainda não tem certeza.
```

## Passo 2: Conversa socrática (2-5 trocas)

Guie a conversa usando os princípios de `questioning.md` e `domain-probes.md`:

- Faça **uma pergunta por vez** (nunca uma lista de perguntas)
- As perguntas devem explorar: restrições, trade-offs, usuários, escopo, dependências, riscos
- Use sondas específicas de domínio contextualmente quando o tópico tocar um domínio conhecido
- Ouça os sinais: "ou" / "versus" / "trade-off" indicam prioridades concorrentes que valem explorar
- Reflita o que você ouviu para confirmar o entendimento antes de avançar

**A conversa deve parecer natural, não formulaica.** Evite sequências rígidas. Siga a energia do desenvolvedor — se ele está animado com um aspecto, aprofunde-se nele.

## Passo 3: Oferta de pesquisa no meio da conversa (após 2-3 trocas)

Se a conversa levantar questões factuais, comparações de tecnologias ou incógnitas que uma pesquisa poderia resolver, ofereça:

```
Isso toca em [questão específica]. Quer que eu faça uma passagem rápida de
pesquisa antes de continuarmos? Levaria ~30 segundos e pode trazer contexto útil.

[Sim, pesquise isso] / [Não, vamos continuar explorando]
```

Se sim, spawne um agente de pesquisa:
```
Task(
  prompt="Quick research: {specific_question}. Return 3-5 key findings, no more than 200 words.",
  subagent_type="gsd-phase-researcher"
)
```

Compartilhe as descobertas e continue a conversa.

Se o tópico não justificar pesquisa, ignore completamente este passo. **Não force.**

## Passo 4: Cristalizar os resultados (após 3-6 trocas)

Quando a conversa atingir conclusões naturais ou o desenvolvedor sinalizar prontidão, proponha os resultados. Analise a conversa para identificar o que foi discutido e sugira **até 4 resultados** a partir de:

| Tipo | Destino | Quando sugerir |
|------|-------------|-----------------|
| Note | `.planning/notes/{slug}.md` | Observações, contexto, decisões que valem ser lembradas |
| Todo | `.planning/todos/pending/{slug}.md` | Tarefas concretas e acionáveis identificadas |
| Seed | `.planning/seeds/{slug}.md` | Ideias prospectivas com condições de ativação |
| Questão de pesquisa | `.planning/research/questions.md` (append) | Questões abertas que precisam de investigação mais profunda |
| Requirement | `REQUIREMENTS.md` (append) | Requisitos claros que emergiram da discussão |
| Nova fase | `ROADMAP.md` (append) | Escopo grande o suficiente para justificar sua própria fase |

Apresente as sugestões:
```
Com base em nossa conversa, sugiro capturar:

1. **Note:** "Decisões sobre estratégia de autenticação" — seu raciocínio sobre JWT vs sessions
2. **Todo:** "Avaliar Passport.js vs middleware customizado" — a comparação que você quer fazer
3. **Seed:** "Suporte a provedor OAuth2" — gatilho: quando a fase de gerenciamento de usuários começar

Criar estes? Você pode selecionar itens específicos ou modificá-los.

[Criar todos] / [Deixa eu escolher] / [Pular — só explorando]
```

**Nunca escreva artefatos sem seleção explícita do usuário.**

## Passo 5: Escrever os resultados selecionados

Para cada resultado selecionado, escreva o arquivo:

- **Notes:** Crie `.planning/notes/{slug}.md` com frontmatter (title, date, context)
- **Todos:** Crie `.planning/todos/pending/{slug}.md` com frontmatter (title, date, priority)
- **Seeds:** Crie `.planning/seeds/{slug}.md` com frontmatter (title, trigger_condition, planted_date)
- **Questões de pesquisa:** Anexe em `.planning/research/questions.md`
- **Requirements:** Anexe em `.planning/REQUIREMENTS.md` com o próximo REQ ID disponível
- **Phases:** Use o comando `/gsd-add-phase` existente via SlashCommand

Faça commit se `commit_docs` estiver habilitado:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs: capture exploration — {topic_slug}" --files {file_list}
```

## Passo 6: Encerrar

```
## Exploração Concluída

**Tópico:** {topic}
**Resultados:** {count} artefato(s) criado(s)
{lista de arquivos criados}

Continue explorando com `/gsd-explore` ou comece a trabalhar com `/gsd-next`.
```

</process>

<success_criteria>
- [ ] A conversa socrática segue os princípios de questioning.md
- [ ] Perguntas feitas uma por vez, não em lotes
- [ ] Pesquisa oferecida contextualmente (não forçada)
- [ ] Até 4 resultados propostos a partir da conversa
- [ ] Usuário seleciona explicitamente quais resultados criar
- [ ] Arquivos escritos nos destinos corretos
- [ ] Commit respeita a configuração commit_docs
</success_criteria>
