# Overrides de Verificação

Mecanismo para aceitar intencionalmente falhas de must-have quando o desvio é conhecido e aceitável. Evita loops de verificação em itens que nunca passarão como especificados originalmente.

<override_format>

## Formato de Override

Overrides são declarados no frontmatter do VERIFICATION.md sob uma chave `overrides:`:

```yaml
---
phase: 03-authentication
verified: 2026-04-05T12:00:00Z
status: passed
score: 5/5
overrides_applied: 2
overrides:
  - must_have: "OAuth2 PKCE flow implemented"
    reason: "Using session-based auth instead — PKCE unnecessary for server-rendered app"
    accepted_by: "dave"
    accepted_at: "2026-04-04T15:30:00Z"
  - must_have: "Rate limiting on login endpoint"
    reason: "Deferred to Phase 5 (infrastructure) — tracked in ROADMAP.md"
    accepted_by: "dave"
    accepted_at: "2026-04-04T15:30:00Z"
---
```

### Campos Obrigatórios

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `must_have` | string | A verdade must-have, descrição do artefato ou link-chave sendo sobreposto. Não precisa ser uma correspondência exata — correspondência fuzzy é aplicada. |
| `reason` | string | Por que este desvio é aceitável. Deve ser específico — não apenas "não necessário". |
| `accepted_by` | string | Quem aceitou o override (nome de usuário ou função). Obrigatório. |
| `accepted_at` | string | Timestamp ISO de quando o override foi aceito. Obrigatório. |

</override_format>

## Quando Usar

Os overrides se aplicam quando uma fase desviou intencionalmente do plano original durante a execução — por exemplo, um requisito foi removido do escopo, uma abordagem alternativa foi escolhida ou uma dependência mudou.

Sem overrides, o verificador reporta esses itens como FAIL mesmo que o desvio tenha sido intencional. Os overrides permitem que o desenvolvedor marque itens específicos como `PASSED (override)` com um motivo documentado.

Os overrides são apropriados quando:
- Um requisito mudou após o planejamento, mas o ROADMAP.md ainda não foi atualizado
- Uma implementação alternativa satisfaz a intenção, mas não a redação literal
- Um must-have foi adiado para uma fase posterior com rastreamento explícito
- Restrições externas tornam o must-have original impossível ou desnecessário

## Quando NÃO Usar

Os overrides NÃO são apropriados quando:
- A implementação está simplesmente incompleta — corrija-a
- O must-have está pouco claro — esclareça-o
- O desenvolvedor quer pular a verificação — isso compromete o processo
- Múltiplos must-haves estão falhando para a mesma fase — se mais de 2-3 itens precisam de overrides, revise o plano em vez de fazer override em massa

<matching_rules>

## Regras de Correspondência

A correspondência de override usa **correspondência fuzzy**, não comparação exata de strings. Isso acomoda pequenas diferenças de redação entre como os must-haves são formulados no ROADMAP.md, no frontmatter do PLAN.md e na entrada de override.

### Algoritmo de Correspondência

1. **Normalize ambas as strings:** comparação sem distinção de maiúsculas/minúsculas — converta ambas as strings para minúsculas, remova pontuação, colapse espaços em branco
2. **Sobreposição de tokens:** divida em palavras, calcule a interseção
3. **Limiar de correspondência:** 80% de sobreposição de tokens em QUALQUER direção (tokens de override encontrados no must-have, OU tokens do must-have encontrados no override)
4. **Prioridade de substantivos-chave:** substantivos e termos técnicos (caminhos de arquivo, nomes de componentes, endpoints de API) são ponderados mais alto do que palavras comuns

### Exemplos

| Must-Have | Override `must_have` | Correspondência? | Motivo |
|-----------|---------------------|-----------------|--------|
| "User can authenticate via OAuth2 PKCE" | "OAuth2 PKCE flow implemented" | Sim | Termos-chave `OAuth2` e `PKCE` se sobrepõem, limiar de 80% atingido |
| "Rate limiting on /api/auth/login" | "Rate limiting on login endpoint" | Sim | Sobreposição de `rate limiting` + `login` |
| "Chat component renders messages" | "OAuth2 PKCE flow implemented" | Não | Sem sobreposição significativa de tokens |
| "src/components/Chat.tsx provides message list" | "Chat.tsx message list rendering" | Sim | Sobreposição de `Chat.tsx` + `message` + `list` |

### Resolução de Ambiguidade

Se um override corresponder a múltiplos must-haves, aplique-o à **correspondência mais específica** (maior percentual de sobreposição de tokens). Se ainda ambíguo, aplique à primeira correspondência e registre um aviso.

</matching_rules>

<verifier_behavior>

## Comportamento do Verificador com Overrides

### Ordem de Verificação

A verificação de override acontece **antes de marcar um must-have como FAIL**. O fluxo é:

1. Avalie o must-have em relação à base de código (Passos 3-5 do processo de verificação)
2. Se o resultado da avaliação for FAIL ou UNCERTAIN:
   a. Verifique o array `overrides:` no frontmatter do VERIFICATION.md por correspondência fuzzy
   b. Se override encontrado: marque como `PASSED (override)` em vez de FAIL
   c. Se nenhum override encontrado: marque como FAIL normalmente
3. Se o resultado da avaliação for PASS: marque como VERIFIED (overrides são irrelevantes)

### Formato de Saída

Itens sobrescritos aparecem com status distinto em todas as tabelas de verificação:

```markdown
| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | User can authenticate | VERIFIED | OAuth session flow working |
| 2 | OAuth2 PKCE flow | PASSED (override) | Override: Using session-based auth — accepted by dave on 2026-04-04 |
| 3 | Chat renders messages | FAILED | Component returns placeholder |
```

O status `PASSED (override)` deve ser visualmente distinto de `VERIFIED` e `FAILED`. Na coluna de evidência, inclua o motivo do override e quem o aceitou.

### Impacto no Status Geral

- Itens `PASSED (override)` contam para a pontuação de aprovação, não para a pontuação de reprovação
- Uma fase com todos os itens VERIFIED ou PASSED (override) pode ter status `passed`
- Overrides NÃO suprimem itens `human_needed` — esses ainda requerem teste humano

### Pontuação no Frontmatter

A pontuação e a contagem de overrides no frontmatter refletem os overrides aplicados:

```yaml
score: 5/5  # inclui 2 overrides
overrides_applied: 2
```

</verifier_behavior>

<creating_overrides>

## Criando Overrides

### Sugestão Interativa de Override

Quando o verificador marca um must-have como FAIL e a falha parece intencional (ex.: implementação alternativa existe, ou o código explicitamente trata o caso de forma diferente), o verificador deve sugerir criar um override:

```markdown
### F-002: OAuth2 PKCE flow

**Status:** FAILED
**Evidence:** No PKCE implementation found. Session-based auth used instead.

**This looks intentional.** The codebase uses session-based authentication which achieves the same goal differently. To accept this deviation, add an override to VERIFICATION.md frontmatter:

```yaml
overrides:
  - must_have: "OAuth2 PKCE flow implemented"
    reason: "Using session-based auth instead — PKCE unnecessary for server-rendered app"
    accepted_by: "{your name}"
    accepted_at: "{current ISO timestamp}"
```

Then re-run verification to apply.
```

### Override via gsd-tools

Overrides também podem ser gerenciados através do fluxo de trabalho de verificação:

1. Execute `/gsd-verify-work` — a verificação encontra lacunas
2. Revise as lacunas — determine quais são desvios intencionais
3. Adicione entradas de override ao frontmatter do VERIFICATION.md
4. Execute novamente `/gsd-verify-work` — os overrides são aplicados, lacunas restantes são mostradas

</creating_overrides>

<override_lifecycle>

## Ciclo de Vida do Override

### Durante a Re-verificação

Quando uma fase é re-verificada (ex.: após fechamento de lacunas):
- Overrides existentes são transferidos automaticamente
- Se o código subjacente agora satisfaz o must-have, o override torna-se desnecessário — marque como VERIFIED
- Overrides nunca são removidos automaticamente; eles persistem como documentação

### Na Conclusão do Marco

Durante `/gsd-audit-milestone`, os overrides são exibidos no relatório de auditoria:

```
### Verification Overrides ({count} across {phase_count} phases)

| Phase | Must-Have | Reason | Accepted By |
|-------|----------|--------|-------------|
| 03 | OAuth2 PKCE | Session-based auth used instead | dave |
```

Isso dá à equipe visibilidade de todos os desvios aceitos antes de fechar o marco.

### Limpeza

Overrides obsoletos (onde o must-have foi posteriormente implementado ou removido do ROADMAP.md) podem ser limpos durante a conclusão do marco. Eles são informativos — deixá-los não causa danos.

</override_lifecycle>

## Exemplo de VERIFICATION.md

```markdown
---
phase: 03-api-layer
verified: 2026-04-05T12:00:00Z
status: passed
score: 3/3
overrides_applied: 1
overrides:
  - must_have: "paginated API responses"
    reason: "Descoped — dataset under 100 items, pagination adds complexity without value"
    accepted_by: "dave"
    accepted_at: "2026-04-04T15:30:00Z"
---

## Phase 3: API Layer — Verification

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | REST endpoints return JSON | VERIFIED | curl tests confirm |
| 2 | Paginated API responses | PASSED (override) | Descoped — see override: dataset under 100 items |
| 3 | Authentication middleware | VERIFIED | JWT validation working |
```
