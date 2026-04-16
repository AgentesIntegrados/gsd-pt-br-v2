# Template de UAT

Template para `.planning/phases/XX-name/{phase_num}-UAT.md` — rastreamento persistente de sessão de UAT.

---

## Template do Arquivo

```markdown
---
status: testing | partial | complete | diagnosed
phase: XX-name
source: [lista de arquivos SUMMARY.md testados]
started: [timestamp ISO]
updated: [timestamp ISO]
---

## Teste Atual
<!-- SOBRESCREVA a cada teste - mostra onde estamos -->

number: [N]
name: [nome do teste]
expected: |
  [o que o usuário deve observar]
awaiting: resposta do usuário

## Testes

### 1. [Nome do Teste]
expected: [comportamento observável - o que o usuário deve ver]
result: [pendente]

### 2. [Nome do Teste]
expected: [comportamento observável]
result: pass

### 3. [Nome do Teste]
expected: [comportamento observável]
result: issue
reported: "[resposta verbatim do usuário]"
severity: major

### 4. [Nome do Teste]
expected: [comportamento observável]
result: skipped
reason: [por que pulado]

### 5. [Nome do Teste]
expected: [comportamento observável]
result: blocked
blocked_by: server | physical-device | release-build | third-party | prior-phase
reason: [por que bloqueado]

...

## Resumo

total: [N]
passed: [N]
issues: [N]
pending: [N]
skipped: [N]
blocked: [N]

## Lacunas

<!-- Formato YAML para consumo pelo plan-phase --gaps -->
- truth: "[comportamento esperado do teste]"
  status: failed
  reason: "Usuário relatou: [resposta verbatim]"
  severity: blocker | major | minor | cosmetic
  test: [N]
  root_cause: ""     # Preenchido pelo diagnóstico
  artifacts: []      # Preenchido pelo diagnóstico
  missing: []        # Preenchido pelo diagnóstico
  debug_session: ""  # Preenchido pelo diagnóstico
```

---

<section_rules>

**Frontmatter:**
- `status`: SOBRESCREVA - "testing", "partial" ou "complete"
- `phase`: IMUTÁVEL - definido na criação
- `source`: IMUTÁVEL - arquivos SUMMARY sendo testados
- `started`: IMUTÁVEL - definido na criação
- `updated`: SOBRESCREVA - atualize a cada mudança

**Teste Atual:**
- SOBRESCREVA completamente a cada transição de teste
- Mostra qual teste está ativo e o que é aguardado
- Na conclusão: "[testes concluídos]"

**Testes:**
- Cada teste: SOBRESCREVA o campo result quando o usuário responder
- Valores de `result`: [pendente], pass, issue, skipped, blocked
- Se issue: adicione `reported` (verbatim) e `severity` (inferida)
- Se skipped: adicione `reason` se fornecido
- Se blocked: adicione `blocked_by` (tag) e `reason` (se fornecido)

**Resumo:**
- SOBRESCREVA as contagens após cada resposta
- Rastreia: total, passed, issues, pending, skipped

**Lacunas:**
- APENAS ACRESCENTE quando um problema for encontrado (formato YAML)
- Após diagnóstico: preencha `root_cause`, `artifacts`, `missing`, `debug_session`
- Esta seção alimenta diretamente o /gsd-plan-phase --gaps

</section_rules>

<diagnosis_lifecycle>

**Após a conclusão dos testes (status: complete), se houver lacunas:**

1. Usuário executa o diagnóstico (a partir da oferta do verify-work ou manualmente)
2. O fluxo de trabalho diagnose-issues spawna agentes de debug paralelos
3. Cada agente investiga uma lacuna, retorna a causa raiz
4. A seção Lacunas do UAT.md é atualizada com o diagnóstico:
   - Cada lacuna recebe `root_cause`, `artifacts`, `missing`, `debug_session` preenchidos
5. status → "diagnosed"
6. Pronto para /gsd-plan-phase --gaps com as causas raiz

**Após o diagnóstico:**
```yaml
## Lacunas

- truth: "Comentário aparece imediatamente após o envio"
  status: failed
  reason: "Usuário relatou: funciona, mas não aparece até eu atualizar a página"
  severity: major
  test: 2
  root_cause: "useEffect em CommentList.tsx com dependência commentCount ausente"
  artifacts:
    - path: "src/components/CommentList.tsx"
      issue: "useEffect com dependência ausente"
  missing:
    - "Adicionar commentCount ao array de dependências do useEffect"
  debug_session: ".planning/debug/comment-not-refreshing.md"
```

</diagnosis_lifecycle>

<lifecycle>

**Criação:** Quando /gsd-verify-work inicia uma nova sessão
- Extraia os testes dos arquivos SUMMARY.md
- Defina o status como "testing"
- Teste Atual aponta para o teste 1
- Todos os testes têm result: [pendente]

**Durante os testes:**
- Apresente o teste da seção Teste Atual
- Usuário responde com confirmação de pass ou descrição do problema
- Atualize o result do teste (pass/issue/skipped)
- Atualize as contagens do Resumo
- Se issue: acrescente à seção Lacunas (formato YAML), infira a severidade
- Mova o Teste Atual para o próximo teste pendente

**Na conclusão:**
- status → "complete"
- Teste Atual → "[testes concluídos]"
- Commite o arquivo
- Apresente o resumo com os próximos passos

**Conclusão parcial:**
- status → "partial" (se houver testes pendentes, bloqueados ou pulados não resolvidos)
- Teste Atual → "[testes pausados — {N} itens pendentes]"
- Commite o arquivo
- Apresente o resumo com os itens pendentes em destaque

**Retomada de sessão parcial:**
- `/gsd-verify-work {phase}` retoma a partir do primeiro teste pendente/bloqueado
- Quando todos os itens forem resolvidos, o status avança para "complete"

**Retomada após /clear:**
1. Leia o frontmatter → conheça a fase e o status
2. Leia o Teste Atual → saiba onde estamos
3. Encontre o primeiro result [pendente] → continue a partir daí
4. O Resumo mostra o progresso até agora

</lifecycle>

<severity_guide>

A severidade é INFERIDA a partir da linguagem natural do usuário, nunca perguntada.

| O usuário descreve | Inferir |
|--------------------|---------|
| Crash, erro, exceção, falha completa, inutilizável | blocker |
| Não funciona, nada acontece, comportamento errado, ausente | major |
| Funciona, mas..., lento, estranho, menor, pequeno problema | minor |
| Cor, fonte, espaçamento, alinhamento, visual, aparência estranha | cosmetic |

Padrão: **major** (padrão seguro, o usuário pode esclarecer se estiver errado)

</severity_guide>

<good_example>
```markdown
---
status: diagnosed
phase: 04-comments
source: 04-01-SUMMARY.md, 04-02-SUMMARY.md
started: 2025-01-15T10:30:00Z
updated: 2025-01-15T10:45:00Z
---

## Teste Atual

[testes concluídos]

## Testes

### 1. Visualizar Comentários em Postagem
expected: Seção de comentários expande, mostra contagem e lista de comentários
result: pass

### 2. Criar Comentário de Nível Superior
expected: Enviar comentário via editor de rich text, aparece na lista com informações do autor
result: issue
reported: "funciona, mas não aparece até eu atualizar a página"
severity: major

### 3. Responder a um Comentário
expected: Clique em Responder, editor inline aparece, envio mostra resposta aninhada
result: pass

### 4. Aninhamento Visual
expected: Thread de 3+ níveis mostra recuo, bordas esquerdas, limitado em profundidade razoável
result: pass

### 5. Excluir Próprio Comentário
expected: Clique em excluir no próprio comentário, removido ou mostra [excluído] se tiver respostas
result: pass

### 6. Contagem de Comentários
expected: Postagem mostra contagem precisa, incrementa ao adicionar comentário
result: pass

## Resumo

total: 6
passed: 5
issues: 1
pending: 0
skipped: 0

## Lacunas

- truth: "Comentário aparece imediatamente após o envio na lista"
  status: failed
  reason: "Usuário relatou: funciona, mas não aparece até eu atualizar a página"
  severity: major
  test: 2
  root_cause: "useEffect em CommentList.tsx com dependência commentCount ausente"
  artifacts:
    - path: "src/components/CommentList.tsx"
      issue: "useEffect com dependência ausente"
  missing:
    - "Adicionar commentCount ao array de dependências do useEffect"
  debug_session: ".planning/debug/comment-not-refreshing.md"
```
</good_example>
