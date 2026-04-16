<purpose>
Surfaçar as suposições do Claude sobre uma fase de forma conversacional. Diferente do discuss-phase, não cria artefatos — apenas expõe o que Claude está assumindo para que o usuário possa corrigir conforme necessário.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Carregar contexto da fase:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init plan-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON: `phase_dir`, `phase_number`, `phase_name`, `has_context`, `has_research`.

Ler artefatos da fase:
```bash
cat .planning/ROADMAP.md
cat .planning/STATE.md 2>/dev/null || true
cat "${phase_dir}"/*-CONTEXT.md 2>/dev/null || true
cat "${phase_dir}"/*-RESEARCH.md 2>/dev/null || true
```

Exibir:
```
## Suposições para Fase {N}: {nome}

Com base nos artefatos do projeto, aqui está o que estou assumindo:
```
</step>

<step name="surface_assumptions">
Analisar os artefatos e surfaçar suposições explicitamente em categorias:

**Suposições Técnicas:**
Liste cada suposição técnica no formato:
- "Assumo que {suposição técnica}. {Correto?}"

**Suposições de Produto:**
- "Assumo que {suposição de produto}. {Correto?}"

**Suposições de Integração:**
- "Assumo que {suposição de integração}. {Correto?}"

**Suposições de Escopo:**
- "Assumo que {suposição de escopo}. {Correto?}"

Usar linguagem natural — isso é conversa, não documentação.

Encerrar com:
```
Alguma dessas suposições está errada? Sinta-se à vontade para me corrigir e incorporarei 
suas correções antes de planejar.
```

Aguardar resposta do usuário (texto simples, sem AskUserQuestion).
</step>

<step name="process_corrections">
Processar quaisquer correções fornecidas pelo usuário:

- Reconhecer cada correção
- Confirmar o entendimento correto
- Observar como isso muda a abordagem

Exibir:
```
✓ Entendido. Atualizando meu entendimento:

{Para cada correção:}
- Antes: {suposição original}
- Depois: {entendimento corrigido}

Alguma outra coisa a ajustar?
```

Continuar até o usuário indicar satisfação ou nenhuma mais correções.
</step>

<step name="offer_next">
Após as correções serem processadas:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Suposições {N} esclarecidas, {M} corrigidas.

**Modo de texto (`workflow.text_mode: true` na config ou flag `--text`):** Definir `TEXT_MODE=true` se `--text` estiver em `$ARGUMENTS` OU `text_mode` for `true` no JSON de inicialização. Quando TEXT_MODE estiver ativo, substituir cada chamada `AskUserQuestion` por uma lista numerada em texto simples e pedir ao usuário que digite o número de sua escolha.

Próximos passos:
- `/gsd-plan-phase {N}` — planejar com entendimento atualizado
- `/gsd-discuss-phase {N}` — discussão mais detalhada com captura de artefatos
- `/gsd-discuss-phase-assumptions {N}` — análise de suposições de base de código primeiro

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

</process>

<success_criteria>
- [ ] Contexto da fase carregado (artefatos de planejamento)
- [ ] Suposições surfaçadas em categorias
- [ ] Linguagem conversacional usada (não documentação)
- [ ] Correções do usuário processadas e confirmadas
- [ ] Próximos passos oferecidos
</success_criteria>
