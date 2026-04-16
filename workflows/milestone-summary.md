# Workflow de Resumo de Milestone

Gera um resumo completo e amigável do projeto a partir dos artefatos de um milestone concluído.
Projetado para integração de equipe — um novo colaborador pode ler a saída e entender o projeto inteiro.

---

## Passo 1: Resolver Versão

```bash
VERSION="$ARGUMENTS"
```

Se `$ARGUMENTS` estiver vazio:
1. Verifique `.planning/STATE.md` para a versão do milestone atual
2. Verifique `.planning/milestones/` para a versão arquivada mais recente
3. Se nenhum for encontrado, verifique se `.planning/ROADMAP.md` existe (o projeto pode estar no meio de um milestone)
4. Se nada for encontrado: erro "Nenhum milestone encontrado. Execute /gsd-new-project ou /gsd-new-milestone primeiro."

Defina `VERSION` para a versão resolvida (ex: "1.0").

## Passo 2: Localizar Artefatos

Determine se o milestone está **arquivado** ou **atual**:

**Milestone arquivado** (`.planning/milestones/v{VERSION}-ROADMAP.md` existe):
```
ROADMAP_PATH=".planning/milestones/v${VERSION}-ROADMAP.md"
REQUIREMENTS_PATH=".planning/milestones/v${VERSION}-REQUIREMENTS.md"
AUDIT_PATH=".planning/milestones/v${VERSION}-MILESTONE-AUDIT.md"
```

**Milestone atual/em andamento** (sem arquivo ainda):
```
ROADMAP_PATH=".planning/ROADMAP.md"
REQUIREMENTS_PATH=".planning/REQUIREMENTS.md"
AUDIT_PATH=".planning/v${VERSION}-MILESTONE-AUDIT.md"
```

Nota: O arquivo de auditoria é movido para `.planning/milestones/` no arquivamento (conforme o workflow `complete-milestone`). Verifique ambos os locais como alternativa.

**Sempre disponíveis:**
```
PROJECT_PATH=".planning/PROJECT.md"
RETRO_PATH=".planning/RETROSPECTIVE.md"
STATE_PATH=".planning/STATE.md"
```

Leia todos os arquivos que existirem. Arquivos faltando são aceitáveis — o resumo se adapta ao que estiver disponível.

## Passo 3: Descobrir Artefatos de Fase

Encontre todos os diretórios de fase:

```bash
gsd-tools.cjs init progress
```

Isso retorna metadados de fase. Para cada fase no escopo do milestone:

- Leia `{phase_dir}/{padded}-SUMMARY.md` se existir — extraia `one_liner`, `accomplishments`, `decisions`
- Leia `{phase_dir}/{padded}-VERIFICATION.md` se existir — extraia status, lacunas, itens adiados
- Leia `{phase_dir}/{padded}-CONTEXT.md` se existir — extraia decisões-chave da seção `<decisions>`
- Leia `{phase_dir}/{padded}-RESEARCH.md` se existir — registre o que foi pesquisado

Rastreie quais fases têm quais artefatos.

**Se não existirem diretórios de fase** (milestone vazio ou estado pré-build): pule para o Passo 5 e gere um resumo mínimo indicando "Nenhuma fase foi executada ainda." Não gere erro — o resumo ainda deve capturar o conteúdo de PROJECT.md e ROADMAP.md.

## Passo 4: Coletar Estatísticas do Git

Tente cada método na ordem até que um tenha sucesso:

**Método 1 — Milestone com tag** (verificar primeiro):
```bash
git tag -l "v${VERSION}" | head -1
```
Se a tag existir:
```bash
git log v${VERSION} --oneline | wc -l
git diff --stat $(git log --format=%H --reverse v${VERSION} | head -1)..v${VERSION}
```

**Método 2 — Intervalo de datas do STATE.md** (se não houver tag):
Leia STATE.md e extraia `started_at` ou a data de sessão mais antiga. Use como limite `--since`:
```bash
git log --oneline --since="<started_at_date>" | wc -l
```

**Método 3 — Commit mais antigo de fase** (se STATE.md não tiver data):
Encontre o commit mais antigo de `.planning/phases/`:
```bash
git log --oneline --diff-filter=A -- ".planning/phases/" | tail -1
```
Use a data desse commit como início.

**Método 4 — Pular estatísticas** (se nenhum dos acima funcionar):
Informe "Estatísticas do Git indisponíveis — nenhuma tag ou intervalo de datas pôde ser determinado." Isso não é um erro — o resumo continua sem a seção de Estatísticas.

Extraia (quando disponível):
- Total de commits no milestone
- Arquivos alterados, inserções, deleções
- Linha do tempo (data de início → data de fim)
- Colaboradores (dos autores do git log)

## Passo 5: Gerar Documento de Resumo

Escreva em `.planning/reports/MILESTONE_SUMMARY-v${VERSION}.md`:

```markdown
# Milestone v{VERSION} — Resumo do Projeto

**Gerado em:** {data}
**Finalidade:** Integração de equipe e revisão do projeto

---

## 1. Visão Geral do Projeto

{De PROJECT.md: "O Que É Isso", proposta de valor central, usuários-alvo}
{Se no meio do milestone: indique quais fases estão completas vs em andamento}

## 2. Arquitetura e Decisões Técnicas

{Dos arquivos CONTEXT.md das fases: escolhas técnicas-chave}
{Das decisões em SUMMARY.md: padrões, bibliotecas, frameworks escolhidos}
{De PROJECT.md: stack de tecnologia se documentada}

Apresente como lista de decisões com breve justificativa:
- **Decisão:** {o que foi escolhido}
  - **Por quê:** {justificativa do CONTEXT.md}
  - **Fase:** {qual fase tomou essa decisão}

## 3. Fases Entregues

| Fase | Nome | Status | Resumo em Uma Linha |
|------|------|--------|---------------------|
{Para cada fase: número, nome, status (completa/em andamento/planejada), one_liner do SUMMARY.md}

## 4. Cobertura de Requisitos

{De REQUIREMENTS.md: liste cada requisito com status}
- ✅ {Requisito atendido}
- ⚠️ {Requisito parcialmente atendido — anote a lacuna}
- ❌ {Requisito não atendido — anote o motivo}

{Se MILESTONE-AUDIT.md existir: inclua o veredito da auditoria}

## 5. Registro de Decisões-Chave

{Agregue das seções `<decisions>` de todos os CONTEXT.md}
{Cada decisão com: ID, descrição, fase, justificativa}

## 6. Dívida Técnica e Itens Adiados

{Dos arquivos VERIFICATION.md: lacunas encontradas, antipadrões observados}
{De RETROSPECTIVE.md: lições aprendidas, o que melhorar}
{Das seções `<deferred>` dos CONTEXT.md: ideias guardadas para depois}

## 7. Como Começar

{Pontos de entrada para novos colaboradores:}
- **Executar o projeto:** {de PROJECT.md ou SUMMARY.md}
- **Diretórios principais:** {da estrutura da codebase}
- **Testes:** {comando de teste de PROJECT.md ou CLAUDE.md}
- **Por onde começar:** {pontos de entrada principais, módulos centrais}

---

## Estatísticas

- **Linha do tempo:** {início} → {fim} ({duração})
- **Fases:** {contagem de completas} / {contagem total}
- **Commits:** {contagem}
- **Arquivos alterados:** {contagem} (+{inserções} / -{deleções})
- **Colaboradores:** {lista}
```

## Passo 6: Escrever e Fazer Commit

**Proteção contra sobrescrita:** Se `.planning/reports/MILESTONE_SUMMARY-v${VERSION}.md` já existir, pergunte ao usuário:
> "Um resumo de milestone para v{VERSION} já existe. Sobrescrever ou visualizar o existente?"
Se "visualizar": exiba o arquivo existente e pule para o Passo 8 (modo interativo). Se "sobrescrever": prossiga.

Crie o diretório de relatórios se necessário:
```bash
mkdir -p .planning/reports
```

Escreva o resumo e faça o commit:
```bash
gsd-tools.cjs commit "docs(v${VERSION}): generate milestone summary for onboarding" \
  --files ".planning/reports/MILESTONE_SUMMARY-v${VERSION}.md"
```

## Passo 7: Apresentar Resumo

Exiba o documento de resumo completo inline.

## Passo 8: Oferecer Modo Interativo

Após apresentar o resumo:

> "Resumo salvo em `.planning/reports/MILESTONE_SUMMARY-v{VERSION}.md`.
>
> Tenho contexto completo dos artefatos do build. Quer perguntar algo sobre o projeto?
> Decisões de arquitetura, fases específicas, requisitos, dívida técnica — pode perguntar à vontade."

Se o usuário fizer perguntas:
- Responda a partir dos artefatos já carregados (CONTEXT.md, SUMMARY.md, VERIFICATION.md, etc.)
- Referencie arquivos e decisões específicas
- Mantenha-se fiel ao que foi realmente construído (sem especulações)

Se o usuário terminar:
- Sugira próximos passos: `/gsd-new-milestone`, `/gsd-progress`, ou compartilhar o resumo com a equipe

## Passo 9: Atualizar STATE.md

```bash
gsd-tools.cjs state record-session \
  --stopped-at "Milestone v${VERSION} summary generated" \
  --resume-file ".planning/reports/MILESTONE_SUMMARY-v${VERSION}.md"
```
