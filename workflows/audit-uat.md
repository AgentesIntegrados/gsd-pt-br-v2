<purpose>
Auditoria entre fases de todos os arquivos UAT e de verificação. Encontra todos os itens pendentes (pending, skipped, blocked, human_needed), opcionalmente verifica na base de código para detectar docs desatualizados e produz um plano de testes humanos priorizado.
</purpose>

<process>

<step name="initialize">
Execute a auditoria via CLI:

```bash
AUDIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" audit-uat --raw)
```

Analise o JSON para o array `results` e o objeto `summary`.

Se `summary.total_items` for 0:
```
## Tudo Limpo

Nenhum item pendente de UAT ou verificação encontrado em todas as fases.
Todos os testes estão passando, resolvidos ou diagnosticados com planos de correção.
```
Pare aqui.
</step>

<step name="categorize">
Agrupe os itens pelo que é acionável AGORA vs. o que precisa de pré-requisitos:

**Testável Agora** (sem dependências externas):
- `pending` — testes que nunca foram executados
- `human_uat` — itens de verificação humana
- `skipped_unresolved` — pulados sem razão de bloqueio clara

**Precisa de Pré-requisitos:**
- `server_blocked` — precisa de servidor externo rodando
- `device_needed` — precisa de dispositivo físico (não simulador)
- `build_needed` — precisa de build de release/preview
- `third_party` — precisa de configuração de serviço externo

Para cada item em "Testável Agora", use Grep/Read para verificar se a funcionalidade subjacente ainda existe na base de código:
- Se o teste referencia um componente/função que não existe mais → marque como `stale`
- Se o teste referencia código que foi significativamente reescrito → marque como `needs_update`
- Caso contrário → marque como `active`
</step>

<step name="present">
Apresente o relatório de auditoria:

```
## Relatório de Auditoria UAT

**{total_items} itens pendentes em {total_files} arquivos em {phase_count} fases**

### Testável Agora ({count})

| # | Fase | Teste | Descrição | Status |
|---|-------|------|-------------|--------|
| 1 | {phase} | {test_name} | {expected} | {active/stale/needs_update} |
...

### Precisa de Pré-requisitos ({count})

| # | Fase | Teste | Bloqueado Por | Descrição |
|---|-------|------|------------|-------------|
| 1 | {phase} | {test_name} | {category} | {expected} |
...

### Desatualizado (pode ser fechado) ({count})

| # | Fase | Teste | Por que Desatualizado |
|---|-------|------|-----------|
| 1 | {phase} | {test_name} | {razão} |
...

---

## Ações Recomendadas

1. **Fechar itens desatualizados:** `/gsd-verify-work {phase}` — marcar testes desatualizados como resolvidos
2. **Executar testes ativos:** Plano de teste UAT humano abaixo
3. **Quando pré-requisitos forem atendidos:** Retestar itens bloqueados com `/gsd-verify-work {phase}`
```
</step>

<step name="test_plan">
Gere um plano de teste UAT humano somente para itens "Testável Agora" + "active":

Agrupe pelo que pode ser testado junto (mesma tela, mesma funcionalidade, mesmo pré-requisito):

```
## Plano de Teste UAT Humano

### Grupo 1: {categoria — ex: "Fluxo de Cobrança"}
Pré-requisitos: {o que precisa estar rodando/configurado}

1. **{Nome do teste}** (Fase {N})
   - Navegue para: {onde}
   - Faça: {ação}
   - Esperado: {comportamento esperado}

2. **{Nome do teste}** (Fase {N})
   ...

### Grupo 2: {categoria}
...
```
</step>

</process>
