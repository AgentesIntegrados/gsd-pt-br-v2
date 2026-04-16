---
phase: {N}
slug: {phase-slug}
status: draft
shadcn_initialized: false
preset: none
created: {date}
---

# Fase {N} — Contrato de Design de UI

> Contrato visual e de interação para fases de frontend. Gerado pelo gsd-ui-researcher, verificado pelo gsd-ui-checker.

---

## Sistema de Design

| Propriedade | Valor |
|-------------|-------|
| Ferramenta | {shadcn / none} |
| Preset | {string do preset ou "não aplicável"} |
| Biblioteca de componentes | {radix / base-ui / none} |
| Biblioteca de ícones | {biblioteca} |
| Fonte | {fonte} |

---

## Escala de Espaçamento

Valores declarados (devem ser múltiplos de 4):

| Token | Valor | Uso |
|-------|-------|-----|
| xs | 4px | Espaços entre ícones, padding inline |
| sm | 8px | Espaçamento de elemento compacto |
| md | 16px | Espaçamento padrão de elemento |
| lg | 24px | Padding de seção |
| xl | 32px | Espaços de layout |
| 2xl | 48px | Quebras de seção principais |
| 3xl | 64px | Espaçamento de nível de página |

Exceções: {liste quaisquer, ou "nenhuma"}

---

## Tipografia

| Papel | Tamanho | Peso | Altura de Linha |
|-------|---------|------|-----------------|
| Corpo | {px} | {peso} | {proporção} |
| Rótulo | {px} | {peso} | {proporção} |
| Título | {px} | {peso} | {proporção} |
| Display | {px} | {peso} | {proporção} |

---

## Cores

| Papel | Valor | Uso |
|-------|-------|-----|
| Dominante (60%) | {hex} | Plano de fundo, superfícies |
| Secundária (30%) | {hex} | Cards, sidebar, nav |
| Destaque (10%) | {hex} | {liste apenas elementos específicos} |
| Destrutiva | {hex} | Apenas ações destrutivas |

Destaque reservado para: {lista explícita — nunca "todos os elementos interativos"}

---

## Contrato de Copywriting

| Elemento | Texto |
|----------|-------|
| CTA Principal | {verbo + substantivo específico} |
| Título de estado vazio | {texto} |
| Corpo de estado vazio | {texto + próximo passo} |
| Estado de erro | {problema + caminho de solução} |
| Confirmação destrutiva | {nome da ação}: {texto de confirmação} |

---

## Segurança do Registry

| Registry | Blocos Usados | Portão de Segurança |
|----------|---------------|---------------------|
| shadcn oficial | {lista} | não necessário |
| {nome de terceiro} | {lista} | visualização shadcn + diff necessário |

---

## Aprovação do Checker

- [ ] Dimensão 1 Copywriting: APROVADO
- [ ] Dimensão 2 Visuais: APROVADO
- [ ] Dimensão 3 Cores: APROVADO
- [ ] Dimensão 4 Tipografia: APROVADO
- [ ] Dimensão 5 Espaçamento: APROVADO
- [ ] Dimensão 6 Segurança do Registry: APROVADO

**Aprovação:** {pendente / aprovado em YYYY-MM-DD}
