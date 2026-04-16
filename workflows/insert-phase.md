<purpose>
Insere uma fase decimal para trabalho urgente descoberto no meio de um marco, entre fases inteiras existentes. Usa numeração decimal (72.1, 72.2, etc.) para preservar a sequência lógica das fases planejadas enquanto acomoda inserções urgentes sem renumerar todo o roadmap.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt de invocação antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analise os argumentos do comando:
- Primeiro argumento: número inteiro da fase após a qual inserir
- Argumentos restantes: descrição da fase

Exemplo: `/gsd-insert-phase 72 Corrigir bug crítico de auth`
-> after = 72
-> description = "Corrigir bug crítico de auth"

Se os argumentos estiverem ausentes:

```
ERRO: Número de fase e descrição são obrigatórios
Uso: /gsd-insert-phase <after> <description>
Exemplo: /gsd-insert-phase 72 Corrigir bug crítico de auth
```

Encerre.

Valide que o primeiro argumento é um inteiro.
</step>

<step name="init_context">
Carregue o contexto de operação de fase:

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${after_phase}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Verifique `roadmap_exists` no JSON init. Se false:
```
ERRO: Nenhum roadmap encontrado (.planning/ROADMAP.md)
```
Encerre.
</step>

<step name="insert_phase">
**Delegue a inserção da fase ao gsd-tools:**

```bash
RESULT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phase insert "${after_phase}" "${description}")
```

O CLI gerencia:
- Verificar se a fase alvo existe em ROADMAP.md
- Calcular o próximo número de fase decimal (verificando decimais existentes no disco)
- Gerar slug a partir da descrição
- Criar o diretório da fase (`.planning/phases/{N.M}-{slug}/`)
- Inserir a entrada da fase no ROADMAP.md após a fase alvo com o marcador (INSERTED)

Extraia do resultado: `phase_number`, `after_phase`, `name`, `slug`, `directory`.
</step>

<step name="update_project_state">
Atualize STATE.md para refletir a fase inserida:

1. Leia `.planning/STATE.md`
2. Em "## Accumulated Context" → "### Roadmap Evolution" adicione a entrada:
   ```
   - Fase {decimal_phase} inserida após a Fase {after_phase}: {description} (URGENTE)
   ```

Se a seção "Roadmap Evolution" não existir, crie-a.
</step>

<step name="completion">
Apresente o resumo de conclusão:

```
Fase {decimal_phase} inserida após a Fase {after_phase}:
- Descrição: {description}
- Diretório: .planning/phases/{decimal-phase}-{slug}/
- Status: Ainda não planejada
- Marcador: (INSERTED) - indica trabalho urgente

Roadmap atualizado: .planning/ROADMAP.md
Estado do projeto atualizado: .planning/STATE.md

---

## Próxima Etapa

**Fase {decimal_phase}: {description}** -- inserção urgente

`/clear` e então:

`/gsd-plan-phase {decimal_phase}`

---

**Também disponível:**
- Revisar o impacto da inserção: Verifique se as dependências da Fase {next_integer} ainda fazem sentido
- Revisar o roadmap

---
```
</step>

</process>

<anti_patterns>

- Não use isto para trabalho planejado no final do marco (use /gsd-add-phase)
- Não insira antes da Fase 1 (decimal 0.1 não faz sentido)
- Não renumere fases existentes
- Não modifique o conteúdo da fase alvo
- Não crie planos ainda (isso é /gsd-plan-phase)
- Não faça commit das mudanças (o usuário decide quando fazer commit)
</anti_patterns>

<success_criteria>
A inserção da fase está completa quando:

- [ ] `gsd-tools phase insert` executado com sucesso
- [ ] Diretório da fase criado
- [ ] Roadmap atualizado com nova entrada de fase (inclui marcador "(INSERTED)")
- [ ] STATE.md atualizado com nota de evolução do roadmap
- [ ] Usuário informado sobre os próximos passos e implicações de dependência
</success_criteria>
