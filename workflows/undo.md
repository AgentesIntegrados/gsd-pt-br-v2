<purpose>
Fluxo seguro de reversão git. Reverte commits de fase ou plano GSD usando o manifesto de fase com verificações de dependência e um gate de confirmação. Usa git revert --no-commit (NUNCA git reset) para preservar o histórico.
</purpose>

<required_reading>
@$HOME/.claude/get-shit-done/references/ui-brand.md
@$HOME/.claude/get-shit-done/references/gate-prompts.md
</required_reading>

<process>

<step name="banner" priority="first">
Exibir o banner de etapa:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► UNDO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="parse_arguments">
Analise $ARGUMENTS para o modo de desfazer:

- `--last N` → MODE=last, COUNT=N (inteiro, padrão 10 se N ausente)
- `--phase NN` → MODE=phase, TARGET_PHASE=NN (número de fase com dois dígitos)
- `--plan NN-MM` → MODE=plan, TARGET_PLAN=NN-MM (ID de fase-plano)

Se nenhum argumento válido fornecido, exibir uso e sair:

```
Uso: /gsd-undo --last N | --phase NN | --plan NN-MM

Modos:
  --last N      Mostrar os últimos N commits GSD para seleção interativa
  --phase NN    Reverter todos os commits da fase NN
  --plan NN-MM  Reverter todos os commits do plano NN-MM

Exemplos:
  /gsd-undo --last 5
  /gsd-undo --phase 03
  /gsd-undo --plan 03-02
```
</step>

<step name="gather_commits">
Com base no MODE, reunir commits candidatos.

**MODE=last:**

Executar:
```bash
git log --oneline --no-merges -${COUNT}
```

Filtrar commits convencionais GSD que correspondam ao padrão `type(scope): message` (ex.: `feat(04-01):`, `docs(03):`, `fix(02-03):`).

Exibir lista numerada de commits correspondentes:
```
Commits GSD recentes:
  1. abc1234 feat(04-01): implement auth endpoint
  2. def5678 docs(03-02): complete plan summary
  3. ghi9012 fix(02-03): correct validation logic
```


**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário para digitar o número de sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Usar AskUserQuestion para perguntar:
- question: "Quais commits reverter? Digite os números (ex.: 1,3) ou 'todos'"
- header: "Selecionar"

Analisar a seleção do usuário em uma lista COMMITS.

---

**MODE=phase:**

Ler `.planning/.phase-manifest.json` se existir.

Se o arquivo existir e `manifest.phases?.[TARGET_PHASE]?.commits` for um array não-vazio:
  - Usar as entradas de `manifest.phases[TARGET_PHASE].commits` como COMMITS (cada entrada é um hash de commit)

Se o arquivo não existir, ou `manifest.phases?.[TARGET_PHASE]` estiver ausente:
  - Exibir: "Manifesto não tem entrada para a fase ${TARGET_PHASE} (ou arquivo ausente), usando fallback de busca no git log"
  - Fallback: executar git log e filtrar pelo escopo da fase alvo:
    ```bash
    git log --oneline --no-merges --all | grep -E "\(0*${TARGET_PHASE}(-[0-9]+)?\):" | head -50
    ```
  - Usar commits correspondentes como COMMITS

---

**MODE=plan:**

Executar:
```bash
git log --oneline --no-merges --all | grep -E "\(${TARGET_PLAN}\)" | head -50
```

Usar commits correspondentes como COMMITS.

---

**Verificação de lista vazia:**

Se COMMITS estiver vazio após a coleta:
```
Nenhum commit encontrado para ${MODE} ${TARGET}. Nada a reverter.
```
Sair normalmente.
</step>

<step name="dependency_check">
**Aplicável quando MODE=phase ou MODE=plan.**

Pular esta etapa completamente para MODE=last.

---

**MODE=phase:**

Ler `.planning/ROADMAP.md` inline.

Buscar fases que listam dependência da fase alvo. Procurar padrões como:
- "Depends on: Phase ${TARGET_PHASE}"
- "Depends on: ${TARGET_PHASE}"
- "depends_on: [${TARGET_PHASE}]"

Para cada fase N dependente encontrada:
1. Verificar se o diretório `.planning/phases/${N}-*/` existe
2. Se existir, verificar se há arquivos PLAN.md ou SUMMARY.md dentro

Se alguma fase downstream tiver trabalho iniciado, coletar avisos:
```
⚠  Dependência downstream detectada:
   A Fase ${N} depende da Fase ${TARGET_PHASE} e tem trabalho iniciado.
```

---

**MODE=plan:**

Extrair o número de fase de TARGET_PLAN (a parte NN de NN-MM). Extrair o número do plano (a parte MM).

Procurar planos posteriores no mesmo diretório de fase (`.planning/phases/${NN}-*/`). Para cada plano posterior (planos com número > MM):
1. Ler o PLAN.md do plano posterior
2. Verificar se suas seções `<files>` ou campos `consumes` referenciam saídas do plano alvo

Se algum plano posterior referenciar as saídas do plano alvo, coletar avisos:
```
⚠  Dependência intra-fase detectada:
   O Plano ${LATER_PLAN} na fase ${NN} referencia saídas do plano ${TARGET_PLAN}.
```

---

Se houver avisos (de qualquer modo):
- Exibir todos os avisos
- Usar AskUserQuestion com padrão aprovar-revisar-abortar:
  - question: "Trabalho downstream depende do alvo que será revertido. Prosseguir mesmo assim?"
  - header: "Confirmar"
  - options: Prosseguir | Abortar

Se o usuário selecionar "Abortar": sair com "Reversão cancelada. Nenhuma alteração feita."
</step>

<step name="confirm_revert">
Exibir o gate de confirmação usando o padrão aprovar-revisar-abortar de gate-prompts.md.

Mostrar:
```
Os seguintes commits serão revertidos (em ordem cronológica inversa):

  {hash} — {mensagem}
  {hash} — {mensagem}
  ...

Total: {N} commit(s) a reverter
```

Usar AskUserQuestion:
- question: "Prosseguir com a reversão?"
- header: "Aprovar?"
- options: Aprovar | Abortar

Se "Abortar": exibir "Reversão cancelada. Nenhuma alteração feita." e sair.
Se "Aprovar": pedir um motivo:

```
AskUserQuestion(
  header: "Motivo",
  question: "Breve motivo para a reversão (usado na mensagem do commit):",
  options: []
)
```

Armazenar a resposta como REVERT_REASON. Continuar para execute_revert.
</step>

<step name="execute_revert">
**RESTRIÇÃO ABSOLUTA: Usar git revert --no-commit. NUNCA usar git reset (exceto para limpeza de conflito conforme documentado abaixo).**

**Guarda de árvore suja (executar primeiro, antes de qualquer reversão):**

Executar `git status --porcelain`. Se a saída for não-vazia, exibir os arquivos sujos e abortar:
```
A árvore de trabalho tem alterações não commitadas. Commite ou dê stash nelas antes de executar /gsd-undo.
```
Sair imediatamente — não prosseguir para nenhuma operação de reversão.

---

Ordenar COMMITS em ordem cronológica inversa (mais recente primeiro). Se os commits vieram do git log (já mais recentes primeiro), já estão na ordem correta.

Para cada hash de commit em COMMITS:
```bash
git revert --no-commit ${HASH}
```

Se alguma reversão falhar (conflito de merge ou erro):
1. Exibir a mensagem de erro
2. Executar limpeza — lidar com casos de primeira chamada e de sequência intermediária:
   ```bash
   # Tentar git revert --abort primeiro (funciona se esta for a primeira reversão com falha)
   git revert --abort 2>/dev/null
   # Se reversões anteriores com --no-commit já foram staged com sucesso antes desta falha,
   # revert --abort pode ser um no-op. Limpar alterações staged e na árvore de trabalho:
   git reset HEAD 2>/dev/null
   git restore . 2>/dev/null
   ```
3. Exibir:
   ```
   ╔══════════════════════════════════════════════════════════════╗
   ║  ERRO                                                        ║
   ╚══════════════════════════════════════════════════════════════╝

   Reversão falhou no commit ${HASH}.
   Causa provável: conflito de merge com alterações subsequentes.

   **Para corrigir:** Resolva o conflito manualmente ou reverta commits individualmente.
   Todas as reversões pendentes foram abortadas — árvore de trabalho está limpa.
   ```
4. Sair com erro.

Após todas as reversões serem staged com sucesso, criar um único commit:

Para MODE=phase:
```bash
git commit -m "revert(${TARGET_PHASE}): undo phase ${TARGET_PHASE} — ${REVERT_REASON}"
```

Para MODE=plan:
```bash
git commit -m "revert(${TARGET_PLAN}): undo plan ${TARGET_PLAN} — ${REVERT_REASON}"
```

Para MODE=last:
```bash
git commit -m "revert: undo ${N} selected commits — ${REVERT_REASON}"
```
</step>

<step name="summary">
Exibir o banner de conclusão:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► UNDO CONCLUÍDO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Mostrar resumo:
```
  ✓ ${N} commit(s) revertido(s)
  ✓ Commit de reversão único criado: ${REVERT_HASH}
```

Mostrar próximos passos:
```
───────────────────────────────────────────────────────────────

## ▶ Próximo Passo

**Revisar estado** — verificar se o projeto está no estado esperado após a reversão

/clear então:

/gsd-progress

───────────────────────────────────────────────────────────────

**Também disponível:**
- `/gsd-execute-phase ${PHASE}` — re-executar se necessário
- `/gsd-undo --last 1` — desfazer a própria reversão se algo deu errado

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Argumentos analisados corretamente para os três modos
- [ ] Modo --phase lê .planning/.phase-manifest.json usando manifest.phases[TARGET_PHASE].commits
- [ ] Modo --phase usa fallback de git log se a entrada do manifesto estiver ausente
- [ ] Verificação de dependência avisa quando fases downstream tiveram trabalho iniciado (MODE=phase)
- [ ] Verificação de dependência avisa quando planos posteriores referenciam saídas do plano alvo (MODE=plan)
- [ ] Guarda de árvore suja aborta se a árvore de trabalho tiver alterações não commitadas
- [ ] Gate de confirmação exibido antes de qualquer execução de reversão
- [ ] Reversões usam git revert --no-commit em ordem cronológica inversa
- [ ] Commit único criado após todas as reversões staged
- [ ] Tratamento de erros limpa casos de primeira chamada e de sequência intermediária
- [ ] git reset --hard NUNCA é usado em nenhum ponto deste fluxo de trabalho
</success_criteria>
</output>
