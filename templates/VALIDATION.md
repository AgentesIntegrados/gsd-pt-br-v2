---
phase: {N}
slug: {phase-slug}
status: draft
nyquist_compliant: false
wave_0_complete: false
created: {date}
---

# Fase {N} — Estratégia de Validação

> Contrato de validação por fase para amostragem de feedback durante a execução.

---

## Infraestrutura de Testes

| Propriedade | Valor |
|-------------|-------|
| **Framework** | {pytest 7.x / jest 29.x / vitest / go test / outro} |
| **Arquivo de configuração** | {caminho ou "nenhum — Wave 0 instala"} |
| **Comando de execução rápida** | `{comando rápido}` |
| **Comando de suite completa** | `{comando completo}` |
| **Tempo estimado de execução** | ~{N} segundos |

---

## Taxa de Amostragem

- **Após cada commit de tarefa:** Execute `{comando de execução rápida}`
- **Após cada wave de plano:** Execute `{comando de suite completa}`
- **Antes do `/gsd-verify-work`:** Suite completa deve estar verde
- **Latência máxima de feedback:** {N} segundos

---

## Mapa de Verificação por Tarefa

| ID da Tarefa | Plano | Wave | Requisito | Ref. Ameaça | Comportamento Seguro | Tipo de Teste | Comando Automatizado | Arquivo Existe | Status |
|--------------|-------|------|-----------|-------------|---------------------|---------------|---------------------|----------------|--------|
| {N}-01-01 | 01 | 1 | REQ-{XX} | T-{N}-01 / — | {comportamento seguro esperado ou "N/A"} | unit | `{comando}` | ✅ / ❌ W0 | ⬜ pendente |

*Status: ⬜ pendente · ✅ verde · ❌ vermelho · ⚠️ instável*

---

## Requisitos da Wave 0

- [ ] `{tests/test_file.py}` — stubs para REQ-{XX}
- [ ] `{tests/conftest.py}` — fixtures compartilhadas
- [ ] `{instalação do framework}` — se nenhum framework for detectado

*Se nenhum: "A infraestrutura existente cobre todos os requisitos da fase."*

---

## Verificações Apenas Manuais

| Comportamento | Requisito | Por Que Manual | Instruções de Teste |
|---------------|-----------|----------------|---------------------|
| {comportamento} | REQ-{XX} | {motivo} | {etapas} |

*Se nenhum: "Todos os comportamentos da fase possuem verificação automatizada."*

---

## Aprovação de Validação

- [ ] Todas as tarefas têm `<automated>` verify ou dependências da Wave 0
- [ ] Continuidade de amostragem: no máximo 3 tarefas consecutivas sem verificação automatizada
- [ ] Wave 0 cobre todas as referências MISSING
- [ ] Sem flags de modo watch
- [ ] Latência de feedback < {N}s
- [ ] `nyquist_compliant: true` definido no frontmatter

**Aprovação:** {pendente / aprovado em YYYY-MM-DD}
