---
phase: {N}
slug: {phase-slug}
status: draft
threats_open: 0
asvs_level: 1
created: {date}
---

# Fase {N} — Segurança

> Contrato de segurança por fase: registro de ameaças, riscos aceitos e trilha de auditoria.

---

## Fronteiras de Confiança

| Fronteira | Descrição | Dados em Trânsito |
|-----------|-----------|-------------------|
| {boundary} | {description} | {tipo de dado / sensibilidade} |

---

## Registro de Ameaças

| ID da Ameaça | Categoria | Componente | Disposição | Mitigação | Status |
|--------------|-----------|------------|------------|-----------|--------|
| T-{N}-01 | {categoria STRIDE} | {component} | {mitigar / aceitar / transferir} | {controle ou referência} | aberto |

*Status: aberto · fechado*
*Disposição: mitigar (implementação necessária) · aceitar (risco documentado) · transferir (terceiro)*

---

## Log de Riscos Aceitos

| ID do Risco | Ref. Ameaça | Justificativa | Aceito Por | Data |
|-------------|-------------|---------------|------------|------|

*Riscos aceitos não ressurgem em execuções futuras de auditoria.*

*Se nenhum: "Nenhum risco aceito."*

---

## Trilha de Auditoria de Segurança

| Data da Auditoria | Total de Ameaças | Fechadas | Abertas | Executada Por |
|-------------------|------------------|----------|---------|---------------|
| {YYYY-MM-DD} | {N} | {N} | {N} | {nome / agente} |

---

## Aprovação

- [ ] Todas as ameaças têm uma disposição (mitigar / aceitar / transferir)
- [ ] Riscos aceitos documentados no Log de Riscos Aceitos
- [ ] `threats_open: 0` confirmado
- [ ] `status: verified` definido no frontmatter

**Aprovação:** {pendente / verificado em YYYY-MM-DD}
