# Modo de Revisão — Referência do Planejador

Ativado quando o orquestrador fornece `<revision_context>` com problemas do checker. NÃO é um recomeço do zero — são atualizações pontuais nos planos existentes.

**Mentalidade:** Cirurgião, não arquiteto. Mudanças mínimas para problemas específicos.

### Passo 1: Carregar Planos Existentes

```bash
cat .planning/phases/$PHASE-*/$PHASE-*-PLAN.md
```

Construa um modelo mental da estrutura atual do plano, tarefas existentes e must_haves.

### Passo 2: Analisar os Problemas do Checker

Os problemas chegam em formato estruturado:

```yaml
issues:
  - plan: "16-01"
    dimension: "task_completeness"
    severity: "blocker"
    description: "Task 2 missing <verify> element"
    fix_hint: "Add verification command for build output"
```

Agrupe por plano, dimensão e severidade.

### Passo 3: Estratégia de Revisão

| Dimensão | Estratégia |
|----------|------------|
| requirement_coverage | Adicionar tarefa(s) para requisito ausente |
| task_completeness | Adicionar elementos ausentes à tarefa existente |
| dependency_correctness | Corrigir depends_on, recalcular waves |
| key_links_planned | Adicionar tarefa de conexão ou atualizar action |
| scope_sanity | Dividir em múltiplos planos |
| must_haves_derivation | Derivar e adicionar must_haves ao frontmatter |

### Passo 4: Fazer Atualizações Pontuais

**FAÇA:** Edite seções específicas sinalizadas, preserve partes que funcionam, atualize waves se as dependências mudarem.

**NÃO FAÇA:** Reescreva planos inteiros para problemas menores, adicione tarefas desnecessárias, quebre planos que estão funcionando.

### Passo 5: Validar as Alterações

- [ ] Todos os problemas sinalizados foram endereçados
- [ ] Nenhum novo problema foi introduzido
- [ ] Números de wave ainda são válidos
- [ ] Dependências ainda estão corretas
- [ ] Arquivos em disco foram atualizados

### Passo 6: Commit

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "fix($PHASE): revise plans based on checker feedback" --files .planning/phases/$PHASE-*/$PHASE-*-PLAN.md
```

### Passo 7: Retornar Resumo da Revisão

```markdown
## REVISÃO CONCLUÍDA

**Problemas endereçados:** {N}/{M}

### Mudanças Realizadas

| Plano | Mudança | Problema Endereçado |
|-------|---------|---------------------|
| 16-01 | Adicionado <verify> à Tarefa 2 | task_completeness |
| 16-02 | Adicionada tarefa de logout | requirement_coverage (AUTH-02) |

### Arquivos Atualizados

- .planning/phases/16-xxx/16-01-PLAN.md
- .planning/phases/16-xxx/16-02-PLAN.md

{Se algum problema NÃO foi endereçado:}

### Problemas Não Endereçados

| Problema | Motivo |
|----------|--------|
| {issue} | {por quê — requer entrada do usuário, mudança arquitetural, etc.} |
```
