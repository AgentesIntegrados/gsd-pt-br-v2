<purpose>

Arquiva diretórios de fase acumulados de milestones concluídos em `.planning/milestones/v{X.Y}-phases/`. Identifica quais fases pertencem a cada milestone concluído, mostra um resumo de simulação e move os diretórios após confirmação.

</purpose>

<required_reading>

1. `.planning/MILESTONES.md`
2. Listagem do diretório `.planning/milestones/`
3. Listagem do diretório `.planning/phases/`

</required_reading>

<process>

<step name="identify_completed_milestones">

Leia `.planning/MILESTONES.md` para identificar milestones concluídos e suas versões.

```bash
cat .planning/MILESTONES.md
```

Extraia cada versão de milestone (ex: v1.0, v1.1, v2.0).

Verifique quais diretórios de arquivo de milestone já existem:

```bash
ls -d .planning/milestones/v*-phases 2>/dev/null || true
```

Filtre os milestones que NÃO têm ainda um diretório de arquivo `-phases`.

Se todos os milestones já tiverem diretórios de fase arquivados:

```
Todos os milestones concluídos já têm diretórios de fase arquivados. Nada para limpar.
```

Pare aqui.

</step>

<step name="determine_phase_membership">

Para cada milestone concluído sem arquivo `-phases`, leia o snapshot do ROADMAP arquivado para determinar quais fases pertencem a ele:

```bash
cat .planning/milestones/v{X.Y}-ROADMAP.md
```

Extraia os números e nomes das fases do roadmap arquivado (ex: Fase 1: Foundation, Fase 2: Auth).

Verifique quais desses diretórios de fase ainda existem em `.planning/phases/`:

```bash
ls -d .planning/phases/*/ 2>/dev/null || true
```

Corresponda os diretórios de fase à membresia do milestone. Inclua apenas os diretórios que ainda existem em `.planning/phases/`.

</step>

<step name="show_dry_run">

Apresente um resumo de simulação para cada milestone:

```
## Resumo de Limpeza

### v{X.Y} — {Nome do Milestone}
Estes diretórios de fase serão arquivados:
- 01-foundation/
- 02-auth/
- 03-core-features/

Destino: .planning/milestones/v{X.Y}-phases/

### v{X.Z} — {Nome do Milestone}
Estes diretórios de fase serão arquivados:
- 04-security/
- 05-hardening/

Destino: .planning/milestones/v{X.Z}-phases/
```

Se nenhum diretório de fase restar para arquivar (todos já movidos ou deletados):

```
Nenhum diretório de fase encontrado para arquivar. As fases podem ter sido removidas ou arquivadas anteriormente.
```

Pare aqui.


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU se `text_mode` do JSON de inicialização for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada AskUserQuestion por uma lista numerada em texto simples e peça ao usuário que digite o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
AskUserQuestion: "Prosseguir com o arquivamento?" com opções: "Sim — arquivar fases listadas" | "Cancelar"

Se "Cancelar": Pare.

</step>

<step name="archive_phases">

Para cada milestone, mova os diretórios de fase:

```bash
mkdir -p .planning/milestones/v{X.Y}-phases
```

Para cada diretório de fase pertencente a este milestone:

```bash
mv .planning/phases/{dir} .planning/milestones/v{X.Y}-phases/
```

Repita para todos os milestones no conjunto de limpeza.

</step>

<step name="commit">

Faça commit das mudanças:

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "chore: archive phase directories from completed milestones" --files .planning/milestones/ .planning/phases/
```

</step>

<step name="report">

```
Arquivado:
{Para cada milestone}
- v{X.Y}: {N} diretórios de fase → .planning/milestones/v{X.Y}-phases/

.planning/phases/ limpo.
```

</step>

</process>

<success_criteria>

- [ ] Todos os milestones concluídos sem arquivos de fase existentes identificados
- [ ] Membresia de fase determinada a partir de snapshots de ROADMAP arquivados
- [ ] Resumo de simulação mostrado e confirmado pelo usuário
- [ ] Diretórios de fase movidos para `.planning/milestones/v{X.Y}-phases/`
- [ ] Mudanças commitadas

</success_criteria>
