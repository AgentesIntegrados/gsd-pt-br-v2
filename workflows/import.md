# Fluxo de Importação

Ingestão de planos externos com detecção de conflitos e delegação para agentes.

- **--from**: Importa plano externo → detecção de conflitos → escreve PLAN.md → valida via gsd-plan-checker

Futuro: o modo `--prd` (extração de PRD em PROJECT.md + REQUIREMENTS.md + ROADMAP.md) está planejado para um PR de acompanhamento.

---

<step name="banner">

Exiba o banner de estágio:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► IMPORTAÇÃO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

</step>

<step name="parse_arguments">

Analise `$ARGUMENTS` para determinar o modo de execução:

- Se `--from` estiver presente: extraia FILEPATH (o próximo token após `--from`), defina MODE=plan
- Se `--prd` estiver presente: exiba mensagem de que `--prd` ainda não está implementado e saia:
  ```
  GSD > o modo --prd está planejado para uma versão futura. Use --from para importar arquivos de plano.
  ```
- Se nenhuma flag for encontrada: exiba o uso e saia:

```
Uso: /gsd-import --from <path>

  --from <path>   Importa um arquivo de plano externo no formato GSD
```

**Valide o caminho do arquivo:**

Verifique se o caminho não contém sequências de traversal e se o arquivo existe:

```bash
case "{FILEPATH}" in
  *..* ) echo "SECURITY_ERROR: path contains traversal sequence"; exit 1 ;;
esac
test -f "{FILEPATH}" || echo "FILE_NOT_FOUND"
```

Se FILE_NOT_FOUND: exiba erro e saia:

```
╔══════════════════════════════════════════════════════════════╗
║  ERRO                                                        ║
╚══════════════════════════════════════════════════════════════╝

Arquivo não encontrado: {FILEPATH}

**Para corrigir:** Verifique o caminho do arquivo e tente novamente.
```

</step>

---

## Caminho A: MODE=plan (--from)

<step name="plan_load_context">

Carregue o contexto do projeto para detecção de conflitos:

1. Leia `.planning/ROADMAP.md` — extraia estrutura de fases, números de fase, dependências
2. Leia `.planning/PROJECT.md` — extraia restrições do projeto, stack tecnológica, limites de escopo.
   **Se PROJECT.md não existir:** pule as verificações de restrição que dependem dele e exiba:
   ```
   GSD > Nota: Nenhum PROJECT.md encontrado. As verificações de conflito contra as restrições do projeto serão puladas.
   ```
3. Leia `.planning/REQUIREMENTS.md` — extraia requisitos existentes para verificações de sobreposição e contradição.
   **Se REQUIREMENTS.md não existir:** pule as verificações de conflito de requisitos e continue.
4. Glob para todos os arquivos CONTEXT.md nos diretórios de fase:
   ```bash
   find .planning/phases/ -name "*-CONTEXT.md" -o -name "CONTEXT.md" 2>/dev/null
   ```
   Leia cada CONTEXT.md encontrado — extraia decisões bloqueadas (qualquer decisão em um bloco `<decisions>`)

Armazene o contexto carregado para detecção de conflitos no próximo passo.

</step>

<step name="plan_read_input">

Leia o arquivo importado em FILEPATH.

Determine o formato:
- **Formato GSD PLAN.md**: Tem frontmatter YAML com campos `phase:`, `plan:`, `type:`
- **Documento de forma livre**: Qualquer outro formato (spec em markdown, doc de design, lista de tarefas, etc.)

Extraia do conteúdo importado:
- **Fase alvo**: A qual fase este plano pertence (do frontmatter ou inferido do conteúdo)
- **Objetivos do plano**: O que o plano pretende realizar
- **Tarefas listadas**: Itens de trabalho individuais descritos no plano
- **Arquivos modificados**: Quaisquer arquivos mencionados como alvos
- **Dependências**: Quaisquer pré-requisitos referenciados

</step>

<step name="plan_conflict_detection">

Execute verificações de conflito em relação ao contexto do projeto carregado. Saída como relatório de conflito em texto simples usando labels [BLOCKER], [WARNING] e [INFO]. NÃO use tabelas markdown (sem formato `|---|`).

### Verificações BLOCKER (qualquer uma impede a importação):

- O plano alva um número de fase que não existe no ROADMAP.md → [BLOCKER]
- O plano especifica uma stack tecnológica que contradiz as restrições do PROJECT.md → [BLOCKER]
- O plano contradiz uma decisão bloqueada em qualquer bloco `<decisions>` do CONTEXT.md → [BLOCKER]
- O plano contradiz um requisito existente no REQUIREMENTS.md → [BLOCKER]

### Verificações WARNING (confirmação do usuário necessária):

- O plano parcialmente sobrepõe cobertura de requisito existente no REQUIREMENTS.md → [WARNING]
- O plano tem `depends_on` referenciando planos que ainda não estão completos → [WARNING]
- O plano modifica arquivos que se sobrepõem com planos incompletos existentes → [WARNING]
- O número de fase do plano conflita com a numeração de fase existente no ROADMAP.md → [WARNING]

### Verificações INFO (informacionais, sem ação necessária):

- O plano usa uma biblioteca que não está atualmente na stack tecnológica do projeto → [INFO]
- O plano adiciona uma nova fase à estrutura do ROADMAP.md → [INFO]

Exiba o Relatório Completo de Detecção de Conflitos:

```
## Relatório de Detecção de Conflitos

### BLOCKERS ({N})

[BLOCKER] {Título curto}
  Encontrado: {o que o plano importado diz}
  Esperado: {o que o contexto do projeto requer}
  → {Ação específica para resolver}

### WARNINGS ({N})

[WARNING] {Título curto}
  Encontrado: {o que foi detectado}
  Impacto: {o que pode dar errado}
  → {Ação sugerida}

### INFO ({N})

[INFO] {Título curto}
  Nota: {informação relevante}
```

**Se qualquer [BLOCKER] existir:**

Exiba:
```
GSD > BLOQUEADO: {N} bloqueador(es) devem ser resolvidos antes que a importação possa prosseguir.
```

Saia SEM escrever nenhum arquivo. Este é o gate de segurança — nenhum PLAN.md é escrito quando bloqueadores existem.

**Se apenas WARNINGS e/ou INFO (sem bloqueadores):**


**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON init for `true`. Quando TEXT_MODE estiver ativo, substitua toda chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Pergunte via AskUserQuestion usando o padrão approve-revise-abort:
- question: "Revise os avisos acima. Prosseguir com a importação?"
- header: "Aprovar?"
- options: Aprovar | Abortar

Se o usuário selecionar "Abortar": saia limpo com a mensagem "Importação cancelada."

</step>

<step name="plan_convert">

Converta o conteúdo importado para o formato GSD PLAN.md.

Certifique-se de que o PLAN.md tenha todos os campos obrigatórios do frontmatter:
```yaml
---
phase: "{NN}-{slug}"
plan: "{NN}-{MM}"
type: "feature|refactor|config|test|docs"
wave: 1
depends_on: []
files_modified: []
autonomous: true
must_haves:
  truths: []
  artifacts: []
---
```

**Rejeite convenções de nomenclatura PBR no conteúdo de origem:**
Se o plano importado referenciar nomenclatura PBR de planos (ex.: `PLAN-01.md`, `plan-01.md`), renomeie todas as referências para a convenção GSD `{NN}-{MM}-PLAN.md` durante a conversão.

Aplique a convenção de nomenclatura GSD para o nome do arquivo de saída:
- Formato: `{NN}-{MM}-PLAN.md` (ex.: `04-01-PLAN.md`)
- NUNCA use `PLAN-01.md`, `plan-01.md` ou qualquer outro formato
- NN = número da fase (com zero à esquerda), MM = número do plano dentro da fase (com zero à esquerda)

Determine o diretório alvo:
```
.planning/phases/{NN}-{slug}/
```

Se o diretório não existir, crie-o:
```bash
mkdir -p ".planning/phases/{NN}-{slug}/"
```

Escreva o arquivo PLAN.md no diretório alvo.

</step>

<step name="plan_validate">

Delegue a validação ao gsd-plan-checker:

```
Task({
  subagent_type: "gsd-plan-checker",
  prompt: "Validate: .planning/phases/{phase}/{plan}-PLAN.md — check frontmatter completeness, task structure, and GSD conventions. Report any issues."
})
```

Se o checker retornar erros:
- Exiba os erros ao usuário
- Peça ao usuário para resolver os problemas antes que o plano seja considerado importado
- Não apague o arquivo escrito — o usuário pode corrigir e re-validar manualmente

Se o checker retornar limpo:
- Exiba: "Validação do plano passou"

</step>

<step name="plan_finalize">

Atualize `.planning/ROADMAP.md` para refletir o novo plano:
- Adicione o plano à lista de Planos na seção de fase correta
- Inclua o nome e descrição do plano

Atualize `.planning/STATE.md` se apropriado (ex.: incremente a contagem total de planos).

Faça commit do plano importado e dos arquivos atualizados:
```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs({phase}): import plan from {basename FILEPATH}" --files .planning/phases/{phase}/{plan}-PLAN.md .planning/ROADMAP.md
```

Exiba a conclusão:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► IMPORTAÇÃO CONCLUÍDA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Mostre: nome do arquivo de plano escrito, diretório da fase, resultado da validação, próximos passos.

</step>

---

## Anti-Padrões

NÃO:
- Use tabelas markdown (`|---|`) no relatório de detecção de conflitos — use labels em texto simples [BLOCKER]/[WARNING]/[INFO]
- Escreva arquivos PLAN.md como `PLAN-01.md` ou `plan-01.md` — sempre use `{NN}-{MM}-PLAN.md`
- Use `pbr:plan-checker` ou `pbr:planner` — use `gsd-plan-checker` e `gsd-planner`
- Escreva `.planning/.active-skill` — este é um padrão PBR sem equivalente GSD
- Referencie `pbr-tools`, `pbr:` ou `PLAN-BUILD-RUN` em qualquer lugar
- Escreva qualquer arquivo PLAN.md quando bloqueadores existirem — o gate de segurança deve se manter
- Pule a validação de caminho no argumento do arquivo --from
