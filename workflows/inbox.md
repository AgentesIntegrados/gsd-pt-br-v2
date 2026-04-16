<purpose>
Triagem e revisão de todas as issues e PRs abertas do GitHub em relação aos templates de contribuição do projeto.
Produz um relatório estruturado mostrando o status de conformidade de cada item, sinaliza campos
obrigatórios ausentes, identifica lacunas de labels e opcionalmente toma ações (label, comentar, fechar).
</purpose>

<required_reading>
Antes de começar, leia estes arquivos do projeto para entender os critérios de revisão:
- `.github/ISSUE_TEMPLATE/feature_request.yml` — campos obrigatórios para issues de feature
- `.github/ISSUE_TEMPLATE/enhancement.yml` — campos obrigatórios para issues de enhancement
- `.github/ISSUE_TEMPLATE/chore.yml` — campos obrigatórios para issues de chore
- `.github/ISSUE_TEMPLATE/bug_report.yml` — campos obrigatórios para relatórios de bug
- `.github/PULL_REQUEST_TEMPLATE/feature.md` — checklist obrigatório para PRs de feature
- `.github/PULL_REQUEST_TEMPLATE/enhancement.md` — checklist obrigatório para PRs de enhancement
- `.github/PULL_REQUEST_TEMPLATE/fix.md` — checklist obrigatório para PRs de fix
- `CONTRIBUTING.md` — a regra issue-first e os gates de aprovação
</required_reading>

<process>

<step name="preflight">
Verifique os pré-requisitos:

1. **`gh` CLI disponível e autenticado?**
   ```bash
   which gh && gh auth status 2>&1
   ```
   Se não disponível: imprima as instruções de configuração e saia.

2. **Detectar repositório:**
   Se a flag `--repo` for fornecida, use-a. Caso contrário:
   ```bash
   gh repo view --json nameWithOwner -q '.nameWithOwner' 2>/dev/null
   ```
   Se nenhum repo for detectado: erro — deve estar em um repo git com um remote GitHub.

3. **Analisar flags:**
   - `--issues` → defina REVIEW_ISSUES=true, REVIEW_PRS=false
   - `--prs` → defina REVIEW_ISSUES=false, REVIEW_PRS=true
   - `--label` → defina AUTO_LABEL=true
   - `--close-incomplete` → defina AUTO_CLOSE=true
   - Padrão (sem flags): revise tanto issues quanto PRs, apenas reporte (sem ações automáticas)
</step>

<step name="fetch_issues">
Ignore se REVIEW_ISSUES=false.

Busque todas as issues abertas:
```bash
gh issue list --state open --json number,title,labels,body,author,createdAt,updatedAt --limit 100
```

Para cada issue, classifique por labels e conteúdo do body:

| Label/Padrão | Tipo | Template |
|---|---|---|
| `feature-request` | Feature | feature_request.yml |
| `enhancement` | Enhancement | enhancement.yml |
| `bug` | Bug | bug_report.yml |
| `type: chore` | Chore | chore.yml |
| Sem label correspondente | Desconhecido | Sinalize para triagem manual |

Se uma issue não tiver label de tipo, tente classificar pelo conteúdo do body:
- Contém "### Feature name" → provavelmente Feature
- Contém "### What existing feature" → provavelmente Enhancement
- Contém "### What happened?" → provavelmente Bug
- Contém "### What is the maintenance task?" → provavelmente Chore
- Não é possível determinar → marque como `needs-triage`
</step>

<step name="review_issues">
Ignore se REVIEW_ISSUES=false.

Para cada issue classificada, revise em relação aos requisitos do seu template.

**Checklist de Revisão de Feature Request:**
- [ ] Checklist de pré-submissão presente (4 checkboxes)
- [ ] Nome da feature fornecido
- [ ] Tipo de adição selecionado
- [ ] Declaração do problema preenchida (sem texto de placeholder)
- [ ] O que está sendo adicionado descrito com exemplos
- [ ] Escopo completo das mudanças listado (arquivos criados/modificados/sistemas)
- [ ] User stories presentes (mínimo 2)
- [ ] Critérios de aceitação presentes (condições testáveis)
- [ ] Runtimes aplicáveis selecionados
- [ ] Avaliação de breaking changes presente
- [ ] Carga de manutenção descrita
- [ ] Alternativas consideradas (não vazio)
- **Verificação de label:** Tem label `needs-review`? Tem label `approved-feature`?
- **Verificação de gate:** Se um PR existir linkando esta issue, a issue tem `approved-feature`?

**Checklist de Revisão de Enhancement:**
- [ ] Checklist de pré-submissão presente (4 checkboxes)
- [ ] O que está sendo melhorado identificado
- [ ] Comportamento atual descrito com exemplos
- [ ] Comportamento proposto descrito com exemplos
- [ ] Razão e benefício articulados (não vagos)
- [ ] Escopo das mudanças listado
- [ ] Breaking changes avaliados
- [ ] Alternativas consideradas
- [ ] Área afetada selecionada
- **Verificação de label:** Tem label `needs-review`? Tem label `approved-enhancement`?
- **Verificação de gate:** Se um PR existir linkando esta issue, a issue tem `approved-enhancement`?

**Checklist de Revisão de Bug Report:**
- [ ] Versão do GSD fornecida
- [ ] Runtime selecionado
- [ ] SO selecionado
- [ ] Versão do Node.js fornecida
- [ ] Descrição do que aconteceu
- [ ] Comportamento esperado descrito
- [ ] Passos para reproduzir fornecidos
- [ ] Frequência selecionada
- [ ] Severidade/impacto selecionados
- [ ] Checklist de PII confirmado
- **Verificação de label:** Tem label `needs-triage` ou `confirmed-bug`?

**Checklist de Revisão de Chore:**
- [ ] Checklist de pré-submissão confirmado (sem mudanças voltadas ao usuário)
- [ ] Tarefa de manutenção descrita
- [ ] Tipo de manutenção selecionado
- [ ] Estado atual descrito com detalhes
- [ ] Trabalho proposto listado
- [ ] Critérios de aceitação presentes
- [ ] Área afetada selecionada
- **Verificação de label:** Tem label `needs-triage`?

**Pontuação:** Para cada issue, calcule um percentual de completude:
- Conte os campos obrigatórios presentes vs. total de campos obrigatórios
- Pontuação = (presentes / total) * 100
- Status: COMPLETO (100%), MAJORITARIAMENTE COMPLETO (75-99%), INCOMPLETO (50-74%), REJEITAR (<50%)
</step>

<step name="fetch_prs">
Ignore se REVIEW_PRS=false.

Busque todos os PRs abertos:
```bash
gh pr list --state open --json number,title,labels,body,author,headRefName,baseRefName,isDraft,createdAt,reviewDecision,statusCheckRollup --limit 100
```

Para cada PR, classifique pelo conteúdo do body e issue linkada:

| Padrão do Body | Tipo | Template |
|---|---|---|
| Contém "## Feature PR" ou "## Feature summary" | PR de Feature | feature.md |
| Contém "## Enhancement PR" ou "## What this enhancement improves" | PR de Enhancement | enhancement.md |
| Contém "## Fix PR" ou "## What was broken" | PR de Fix | fix.md |
| Usa template padrão | Template errado | Sinalize — deve usar template tipado |
| Não é possível determinar | Desconhecido | Sinalize para revisão manual |

Também verifique issues linkadas:
```bash
gh pr view {number} --json body -q '.body' | grep -oE '(Closes|Fixes|Resolves) #[0-9]+'
```
</step>

<step name="review_prs">
Ignore se REVIEW_PRS=false.

Para cada PR classificado, revise em relação aos requisitos do seu template.

**Checklist de Revisão de PR de Feature:**
- [ ] Usa template de PR de feature (não o padrão)
- [ ] Issue linkada com `Closes #NNN`
- [ ] Issue linkada existe e tem label `approved-feature`
- [ ] Resumo da feature presente
- [ ] Tabela de arquivos novos preenchida
- [ ] Tabela de arquivos modificados preenchida
- [ ] Notas de implementação presentes
- [ ] Checklist de conformidade com spec presente (critérios de aceitação da issue)
- [ ] Cobertura de testes descrita
- [ ] Plataformas testadas marcadas (macOS, Windows, Linux)
- [ ] Runtimes testados marcados
- [ ] Confirmação de escopo marcada
- [ ] Checklist completo preenchido
- [ ] Seção de breaking changes preenchida
- **Verificação de CI:** Todos os status checks passando?
- **Verificação de review:** Tem aprovação de review?

**Checklist de Revisão de PR de Enhancement:**
- [ ] Usa template de PR de enhancement (não o padrão)
- [ ] Issue linkada com `Closes #NNN`
- [ ] Issue linkada existe e tem label `approved-enhancement`
- [ ] O que foi melhorado descrito
- [ ] Antes/depois fornecido
- [ ] Abordagem de implementação descrita
- [ ] Método de verificação descrito
- [ ] Plataformas testadas marcadas
- [ ] Runtimes testados marcados
- [ ] Confirmação de escopo marcada
- [ ] Checklist completo preenchido
- [ ] Seção de breaking changes preenchida
- **Verificação de CI:** Todos os status checks passando?

**Checklist de Revisão de PR de Fix:**
- [ ] Usa template de PR de fix (não o padrão)
- [ ] Issue linkada com `Fixes #NNN`
- [ ] Issue linkada existe e tem label `confirmed-bug`
- [ ] O que estava quebrado descrito
- [ ] O que o fix faz descrito
- [ ] Causa raiz explicada
- [ ] Método de verificação descrito
- [ ] Teste de regressão adicionado (ou explicado por que não)
- [ ] Plataformas testadas marcadas
- [ ] Runtimes testados marcados
- [ ] Checklist completo preenchido
- [ ] Seção de breaking changes preenchida
- **Verificação de CI:** Todos os status checks passando?

**Verificações Cross-cutting de PR (todos os tipos):**
- [ ] Título do PR é descritivo (não apenas "fix" ou "update")
- [ ] Uma preocupação por PR (não misturando fix + enhancement)
- [ ] Sem mudanças de formatação não relacionadas visíveis no diff
- [ ] CHANGELOG.md atualizado
- [ ] Não usa `--no-verify` ou pula hooks

**Pontuação:** Igual às issues — percentual de completude por PR.
</step>

<step name="check_gates">
Faça referência cruzada entre issues e PRs para aplicar a regra issue-first:

Para cada PR aberto:
1. Extraia o número da issue linkada do body
2. Se não houver issue linkada: **VIOLAÇÃO DE GATE** — PR sem issue
3. Se a issue linkada existir, verifique suas labels:
   - PR de Feature → issue deve ter `approved-feature`
   - PR de Enhancement → issue deve ter `approved-enhancement`
   - PR de Fix → issue deve ter `confirmed-bug`
4. Se o label estiver ausente: **VIOLAÇÃO DE GATE** — PR aberto antes da aprovação

Reporte violações de gate com destaque — estas são as descobertas mais importantes porque
o projeto fecha automaticamente PRs sem os gates de aprovação adequados.
</step>

<step name="generate_report">
Produza um relatório de triagem estruturado:

```
===================================================================
  GSD INBOX TRIAGE — {repo} — {date}
===================================================================

RESUMO
-------
Issues abertas: {count}    PRs abertos: {count}
  Features:       {n}        PRs de Feature:      {n}
  Enhancements:   {n}        PRs de Enhancement:  {n}
  Bugs:           {n}        PRs de Fix:          {n}
  Chores:         {n}        Template errado:     {n}
  Não classificados: {n}     Sem issue linkada:   {n}

VIOLAÇÕES DE GATE (ação necessária)
---------------------------------
{Para cada violação:}
  PR #{number}: {title}
    Problema: {descrição — ex.: "Sem label approved-feature na issue linkada #45"}
    Ação:     {o que fazer — ex.: "Feche o PR ou aprove a issue #45 primeiro"}

ISSUES QUE PRECISAM DE ATENÇÃO
------------------------
{Para cada issue ordenada por pontuação de completude, menor primeiro:}
  #{number} [{type}] {title}
    Pontuação: {percentage}% completo
    Ausentes: {lista de campos obrigatórios ausentes}
    Labels: {labels atuais} → Sugerido: {labels recomendados}
    Idade: {dias desde a criação}

PRS QUE PRECISAM DE ATENÇÃO
---------------------
{Para cada PR ordenado por pontuação de completude, menor primeiro:}
  #{number} [{type}] {title}
    Pontuação: {percentage}% completo
    Ausentes: {lista de itens do checklist ausentes}
    CI: {passando/falhando/pendente}
    Review: {aprovado/mudanças solicitadas/nenhum}
    Issue linkada: #{issue_number} ({issue_status})
    Idade: {dias desde a criação}

PRONTOS PARA MERGE
--------------
{PRs que estão 100% completos, CI passando, aprovados:}
  #{number} {title} — pronto

ITENS OBSOLETOS (>30 dias, sem atividade)
------------------------------------
{Issues e PRs sem atualizações em 30+ dias}

===================================================================
```

Escreva este relatório em `.planning/INBOX-TRIAGE.md` se um diretório `.planning/` existir,
caso contrário imprima apenas no console.
</step>

<step name="auto_actions">
Execute apenas se as flags `--label` ou `--close-incomplete` foram definidas.

**Se --label:**
Para cada issue/PR onde labels estão ausentes ou incorretos:
```bash
gh issue edit {number} --add-label "{label}"
```
Ou:
```bash
gh pr edit {number} --add-label "{label}"
```

Recomendações de labels:
- Issues não classificadas → adicione `needs-triage`
- Issues de feature sem revisão → adicione `needs-review`
- Issues de enhancement sem revisão → adicione `needs-review`
- Relatórios de bug sem triagem → adicione `needs-triage`
- PRs com violações de gate → adicione `gate-violation`

**Se --close-incomplete:**
Para issues com pontuação abaixo de 50% de completude:
```bash
gh issue close {number} --comment "Fechado pela triagem do GSD inbox: esta issue está com campos obrigatórios ausentes conforme o template. Ausentes: {lista}. Por favor, reabra com uma submissão completa. Veja CONTRIBUTING.md para os requisitos."
```

Para PRs com violações de gate:
```bash
gh pr close {number} --comment "Fechado pela triagem do GSD inbox: este PR não atende ao requisito issue-first. {violação específica}. Veja CONTRIBUTING.md para o processo correto."
```

Sempre confirme com o usuário antes de fechar qualquer coisa:

**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON init for `true`. Quando TEXT_MODE estiver ativo, substitua toda chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.

```
AskUserQuestion:
  question: "Encontrei {N} itens para fechar. Revise a lista acima — prosseguir com o fechamento?"
  options:
    - label: "Fechar todos"
      description: "Fechar todos os {N} itens não conformes com comentários explicativos"
    - label: "Deixa eu escolher"
      description: "Vou escolher quais fechar"
    - label: "Pular"
      description: "Não feche nada — apenas reporte"
```
</step>

<step name="report">
```
───────────────────────────────────────────────────────────────

## Triagem do Inbox Concluída

Revisados: {issue_count} issues, {pr_count} PRs
Violações de gate: {violation_count}
Prontos para merge: {ready_count}
Precisam de atenção: {attention_count}
Obsoletos (30+ dias): {stale_count}
{Se relatório salvo: "Relatório salvo em .planning/INBOX-TRIAGE.md"}

Próximos passos:
- Revise as violações de gate primeiro — elas bloqueiam o pipeline de contribuição
- Trate submissões incompletas (comente ou feche)
- Faça merge de PRs prontos
- Faça triagem de issues não classificadas

───────────────────────────────────────────────────────────────
```
</step>

</process>

<offer_next>
Após a triagem:

- /gsd-review — Execute revisão cross-AI em um plano de fase específico
- /gsd-ship — Crie um PR a partir do trabalho concluído
- /gsd-progress — Veja o estado geral do projeto
- /gsd-inbox --label — Execute novamente com auto-labeling habilitado
</offer_next>

<success_criteria>
- [ ] Todas as issues abertas buscadas e classificadas por tipo
- [ ] Cada issue revisada em relação aos requisitos do seu template
- [ ] Todos os PRs abertos buscados e classificados por tipo
- [ ] Cada PR revisado em relação ao checklist do seu template
- [ ] Violações de gate issue-first identificadas
- [ ] Relatório estruturado gerado com pontuações e itens de ação
- [ ] Ações automáticas executadas apenas quando sinalizadas e confirmadas pelo usuário
</success_criteria>
