# Template do Log de Discussão

Template para `.planning/phases/XX-name/{phase_num}-DISCUSSION-LOG.md` — trilha de auditoria das sessões de perguntas e respostas do discuss-phase.

**Propósito:** Trilha de auditoria de software para tomada de decisões. Captura todas as opções consideradas, não apenas a selecionada. Separado do CONTEXT.md, que é o artefato de implementação consumido pelos agentes downstream.

**NÃO destinado ao consumo por LLM.** Este arquivo nunca deve ser referenciado em blocos `<files_to_read>` ou prompts de agentes.

## Formato

```markdown
# Fase [X]: [Nome] - Log de Discussão

> **Somente trilha de auditoria.** Não utilize como entrada para agentes de planejamento, pesquisa ou execução.
> As decisões são capturadas no CONTEXT.md — este log preserva as alternativas consideradas.

**Data:** [data ISO]
**Fase:** [número da fase]-[nome da fase]
**Áreas discutidas:** [lista separada por vírgulas]

---

## [Nome da Área 1]

| Opção | Descrição | Selecionada |
|-------|-----------|-------------|
| [Opção 1] | [Descrição breve] | |
| [Opção 2] | [Descrição breve] | ✓ |
| [Opção 3] | [Descrição breve] | |

**Escolha do usuário:** [Opção selecionada ou resposta em texto livre, verbatim]
**Observações:** [Quaisquer esclarecimentos ou justificativas fornecidos durante a discussão]

---

## [Nome da Área 2]

...

---

## Critério do Claude

[Áreas delegadas ao julgamento do Claude — lista o que foi adiado e por quê]

## Ideias Adiadas

[Ideias mencionadas, mas fora do escopo desta fase]

---

*Fase: XX-name*
*Log de discussão gerado em: [data]*
```

## Regras

- Gerado automaticamente ao final de cada sessão de discuss-phase
- Inclui TODAS as opções consideradas, não apenas a selecionada
- Inclui notas e esclarecimentos em texto livre do usuário
- Marcado claramente como somente para auditoria, não como artefato de implementação
- NÃO interfere na geração do CONTEXT.md nem no comportamento dos agentes downstream
- Commitado junto com o CONTEXT.md no mesmo commit git
