<purpose>
Adiciona uma nova fase inteira ao final do milestone atual no roadmap. Calcula automaticamente o próximo número de fase, cria o diretório da fase e atualiza a estrutura do roadmap.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analise os argumentos do comando:
- Todos os argumentos se tornam a descrição da fase
- Exemplo: `/gsd-add-phase Adicionar autenticação` → description = "Adicionar autenticação"
- Exemplo: `/gsd-add-phase Corrigir problemas críticos de performance` → description = "Corrigir problemas críticos de performance"

Se nenhum argumento for fornecido:

```
ERRO: Descrição da fase obrigatória
Uso: /gsd-add-phase <descrição>
Exemplo: /gsd-add-phase Adicionar sistema de autenticação
```

Encerrar.
</step>

<step name="init_context">
Carregue o contexto da operação de fase:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "0")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Verifique `roadmap_exists` no JSON de inicialização. Se false:
```
ERRO: Nenhum roadmap encontrado (.planning/ROADMAP.md)
Execute /gsd-new-project para inicializar.
```
Encerrar.
</step>

<step name="add_phase">
**Delegue a adição da fase ao gsd-tools:**

```bash
RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase add "${description}")
```

O CLI cuida de:
- Encontrar o maior número de fase inteiro existente
- Calcular o próximo número de fase (máx + 1)
- Gerar o slug a partir da descrição
- Criar o diretório da fase (`.planning/phases/{NN}-{slug}/`)
- Inserir a entrada da fase no ROADMAP.md com as seções Goal, Depends on e Plans

Extraia do resultado: `phase_number`, `padded`, `name`, `slug`, `directory`.
</step>

<step name="update_project_state">
Atualize STATE.md para refletir a nova fase:

1. Leia `.planning/STATE.md`
2. Em "## Accumulated Context" → "### Roadmap Evolution" adicione a entrada:
   ```
   - Phase {N} added: {description}
   ```

Se a seção "Roadmap Evolution" não existir, crie-a.
</step>

<step name="completion">
Apresente o resumo de conclusão:

```
Fase {N} adicionada ao milestone atual:
- Descrição: {description}
- Diretório: .planning/phases/{phase-num}-{slug}/
- Status: Ainda não planejada

Roadmap atualizado: .planning/ROADMAP.md

---

## ▶ Próximo Passo

**Fase {N}: {description}**

`/clear` e depois:

`/gsd-plan-phase {N}`

---

**Também disponível:**
- `/gsd-add-phase <descrição>` — adicionar outra fase
- Revisar o roadmap

---
```
</step>

</process>

<success_criteria>
- [ ] `gsd-tools phase add` executado com sucesso
- [ ] Diretório da fase criado
- [ ] Roadmap atualizado com nova entrada de fase
- [ ] STATE.md atualizado com nota de evolução do roadmap
- [ ] Usuário informado sobre os próximos passos
</success_criteria>
