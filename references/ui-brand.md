<ui_patterns>

Padrões visuais para saída do GSD voltada ao usuário. Orquestradores fazem referência a este arquivo via @.

## Banners de Estágio

Use para grandes transições de fluxo de trabalho.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► {NOME DO ESTÁGIO}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Nomes de estágio (maiúsculas):**
- `QUESTIONING`
- `RESEARCHING`
- `DEFINING REQUIREMENTS`
- `CREATING ROADMAP`
- `PLANNING PHASE {N}`
- `EXECUTING WAVE {N}`
- `VERIFYING`
- `PHASE {N} COMPLETE ✓`
- `MILESTONE COMPLETE 🎉`

---

## Caixas de Checkpoint

Ação do usuário necessária. Largura de 62 caracteres.

```
╔══════════════════════════════════════════════════════════════╗
║  CHECKPOINT: {Tipo}                                          ║
╚══════════════════════════════════════════════════════════════╝

{Conteúdo}

──────────────────────────────────────────────────────────────
→ {PROMPT DE AÇÃO}
──────────────────────────────────────────────────────────────
```

**Tipos:**
- `CHECKPOINT: Verification Required` → `→ Type "approved" or describe issues`
- `CHECKPOINT: Decision Required` → `→ Select: option-a / option-b`
- `CHECKPOINT: Action Required` → `→ Type "done" when complete`

---

## Símbolos de Status

```
✓  Completo / Aprovado / Verificado
✗  Falhou / Ausente / Bloqueado
◆  Em Progresso
○  Pendente
⚡ Auto-aprovado
⚠  Aviso
🎉 Marco completo (apenas em banner)
```

---

## Exibição de Progresso

**Nível de fase/marco:**
```
Progress: ████████░░ 80%
```

**Nível de tarefa:**
```
Tasks: 2/4 complete
```

**Nível de plano:**
```
Plans: 3/5 complete
```

---

## Indicadores de Spawn

```
◆ Spawning researcher...

◆ Spawning 4 researchers in parallel...
  → Stack research
  → Features research
  → Architecture research
  → Pitfalls research

✓ Researcher complete: STACK.md written
```

---

## Bloco "A Seguir"

Sempre ao final de grandes conclusões.

```
───────────────────────────────────────────────────────────────

## ▶ Next Up

**{Identificador}: {Nome}** — {descrição em uma linha}

`/clear` então:

`{comando para copiar e colar}`

───────────────────────────────────────────────────────────────

**Also available:**
- `/gsd-alternative-1` — descrição
- `/gsd-alternative-2` — descrição

───────────────────────────────────────────────────────────────
```

---

## Caixa de Erro

```
╔══════════════════════════════════════════════════════════════╗
║  ERROR                                                       ║
╚══════════════════════════════════════════════════════════════╝

{Descrição do erro}

**To fix:** {Passos de resolução}
```

---

## Tabelas

```
| Phase | Status | Plans | Progress |
|-------|--------|-------|----------|
| 1     | ✓      | 3/3   | 100%     |
| 2     | ◆      | 1/4   | 25%      |
| 3     | ○      | 0/2   | 0%       |
```

---

## Anti-Padrões

- Variar larguras de caixas/banners
- Misturar estilos de banner (`===`, `---`, `***`)
- Omitir o prefixo `GSD ►` nos banners
- Emojis aleatórios (`🚀`, `✨`, `💫`)
- Bloco "A Seguir" ausente após conclusões

</ui_patterns>
