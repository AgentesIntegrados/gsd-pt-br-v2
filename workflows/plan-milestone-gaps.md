<purpose>
Criar todas as fases necessárias para fechar lacunas identificadas por `/gsd-audit-milestone`. Lê MILESTONE-AUDIT.md, agrupa lacunas em fases lógicas, cria entradas de fase em ROADMAP.md e oferece planejar cada fase. Um comando cria todas as fases de correção — sem `/gsd-add-phase` manual por lacuna.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

## 1. Carregar Resultados da Auditoria

```bash
# Encontrar o arquivo de auditoria mais recente
(ls -t .planning/v*-MILESTONE-AUDIT.md 2>/dev/null || true) | head -1
```

Analise o frontmatter YAML para extrair lacunas estruturadas:
- `gaps.requirements` — requisitos não satisfeitos
- `gaps.integration` — conexões entre fases ausentes
- `gaps.flows` — fluxos E2E quebrados

Se nenhum arquivo de auditoria existir ou não tiver lacunas, erro:
```
Nenhuma lacuna de auditoria encontrada. Execute `/gsd-audit-milestone` primeiro.
```

## 2. Priorizar Lacunas

Agrupe lacunas por prioridade do REQUIREMENTS.md:

| Prioridade | Ação |
|------------|------|
| `must` | Criar fase, bloqueia milestone |
| `should` | Criar fase, recomendado |
| `nice` | Pergunte ao usuário: incluir ou deferir? |

Para lacunas de integração/fluxo, infira a prioridade dos requisitos afetados.

## 3. Agrupar Lacunas em Fases

Agrupe lacunas relacionadas em fases lógicas:

**Regras de agrupamento:**
- Mesma fase afetada → combine em uma fase de correção
- Mesmo subsistema (auth, API, UI) → combine
- Ordem de dependência (corrija stubs antes de conectar)
- Mantenha as fases focadas: 2-4 tarefas cada

**Exemplo de agrupamento:**
```
Lacuna: DASH-01 não satisfeita (Dashboard não busca dados)
Lacuna: Integração Fase 1→3 (Auth não passado para chamadas de API)
Lacuna: Fluxo "Ver dashboard" quebrado na busca de dados

→ Fase 6: "Conectar Dashboard à API"
  - Adicionar fetch ao Dashboard.tsx
  - Incluir cabeçalho auth no fetch
  - Tratar resposta, atualizar estado
  - Renderizar dados do usuário
```

## 4. Determinar Números de Fase

Encontre a fase mais alta existente:
```bash
# Obter lista de fases ordenada, extrair a última
HIGHEST=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" phases list --pick directories[-1])
```

As novas fases continuam a partir daí:
- Se a Fase 5 for a mais alta, as lacunas se tornam Fase 6, 7, 8...

## 5. Apresentar Plano de Fechamento de Lacunas

```markdown
## Plano de Fechamento de Lacunas

**Milestone:** {version}
**Lacunas a fechar:** {N} requisitos, {M} integração, {K} fluxos

### Fases Propostas

**Fase {N}: {Nome}**
Fecha:
- {REQ-ID}: {descrição}
- Integração: {from} → {to}
Tarefas: {count}

**Fase {N+1}: {Nome}**
Fecha:
- {REQ-ID}: {descrição}
- Fluxo: {nome do fluxo}
Tarefas: {count}

{Se existirem lacunas nice-to-have:}

### Diferidas (nice-to-have)

Estas lacunas são opcionais. Incluí-las?
- {descrição da lacuna}
- {descrição da lacuna}

---

Criar estas {X} fases? (sim / ajustar / diferir todas as opcionais)
```

Aguarde confirmação do usuário.

## 6. Atualizar ROADMAP.md

Adicione novas fases ao milestone atual:

```markdown
### Fase {N}: {Nome}
**Objetivo:** {derivado das lacunas sendo fechadas}
**Requisitos:** {REQ-IDs sendo satisfeitos}
**Fechamento de Lacunas:** Fecha lacunas da auditoria

### Fase {N+1}: {Nome}
...
```

## 7. Atualizar Tabela de Rastreabilidade de REQUIREMENTS.md (OBRIGATÓRIO)

Para cada REQ-ID atribuído a uma fase de fechamento de lacunas:
- Atualize a coluna Phase para refletir a nova fase de fechamento de lacunas
- Redefina Status para `Pending`

Redefina requisitos marcados que a auditoria encontrou não satisfeitos:
- Altere `[x]` → `[ ]` para qualquer requisito marcado como não satisfeito na auditoria
- Atualize a contagem de cobertura no topo de REQUIREMENTS.md

```bash
# Verificar se a tabela de rastreabilidade reflete as atribuições de fechamento de lacunas
grep -c "Pending" .planning/REQUIREMENTS.md
```

## 8. Criar Diretórios de Fase

```bash
mkdir -p ".planning/phases/{NN}-{name}"
```

## 9. Commitar Atualização de Roadmap e Requisitos

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" commit "docs(roadmap): adicionar fases de fechamento de lacunas {N}-{M}" --files .planning/ROADMAP.md .planning/REQUIREMENTS.md
```

## 10. Oferecer Próximos Passos

```markdown
## ✓ Fases de Fechamento de Lacunas Criadas

**Fases adicionadas:** {N} - {M}
**Lacunas endereçadas:** {count} requisitos, {count} integração, {count} fluxos

---

## ▶ Próximo Passo

**Planejar a primeira fase de fechamento de lacunas**

`/clear` então:

`/gsd-plan-phase {N}`

---

**Também disponível:**
- `/gsd-execute-phase {N}` — se os planos já existirem
- `cat .planning/ROADMAP.md` — ver roadmap atualizado

---

**Após todas as fases de lacunas concluídas:**

`/gsd-audit-milestone` — re-auditar para verificar lacunas fechadas
`/gsd-complete-milestone {version}` — arquivar quando a auditoria passar
```

</process>

<gap_to_phase_mapping>

## Como Lacunas Se Tornam Tarefas

**Lacuna de requisito → Tarefas:**
```yaml
gap:
  id: DASH-01
  description: "Usuário vê seus dados"
  reason: "Dashboard existe mas não busca da API"
  missing:
    - "useEffect com fetch para /api/user/data"
    - "State para dados do usuário"
    - "Renderizar dados do usuário no JSX"

torna-se:

phase: "Conectar Dados do Dashboard"
tasks:
  - name: "Adicionar busca de dados"
    files: [src/components/Dashboard.tsx]
    action: "Adicionar useEffect que busca /api/user/data na montagem"

  - name: "Adicionar gerenciamento de state"
    files: [src/components/Dashboard.tsx]
    action: "Adicionar useState para userData, loading, estados de erro"

  - name: "Renderizar dados do usuário"
    files: [src/components/Dashboard.tsx]
    action: "Substituir placeholder com renderização userData.map"
```

**Lacuna de integração → Tarefas:**
```yaml
gap:
  from_phase: 1
  to_phase: 3
  connection: "Token de auth → chamadas de API"
  reason: "Chamadas de API do Dashboard não incluem cabeçalho auth"
  missing:
    - "Cabeçalho auth nas chamadas fetch"
    - "Renovação de token no 401"

torna-se:

phase: "Adicionar Auth às Chamadas de API do Dashboard"
tasks:
  - name: "Adicionar cabeçalho auth aos fetches"
    files: [src/components/Dashboard.tsx, src/lib/api.ts]
    action: "Incluir cabeçalho Authorization com token em todas as chamadas de API"

  - name: "Tratar respostas 401"
    files: [src/lib/api.ts]
    action: "Adicionar interceptor para renovar token ou redirecionar para login no 401"
```

**Lacuna de fluxo → Tarefas:**
```yaml
gap:
  name: "Usuário vê dashboard após login"
  broken_at: "Carregamento de dados do Dashboard"
  reason: "Sem chamada fetch"
  missing:
    - "Buscar dados do usuário na montagem"
    - "Exibir estado de carregamento"
    - "Renderizar dados do usuário"

torna-se:

# Geralmente a mesma fase que a lacuna de requisito/integração
# Lacunas de fluxo frequentemente se sobrepõem com outros tipos de lacuna
```

</gap_to_phase_mapping>

<success_criteria>
- [ ] MILESTONE-AUDIT.md carregado e lacunas analisadas
- [ ] Lacunas priorizadas (must/should/nice)
- [ ] Lacunas agrupadas em fases lógicas
- [ ] Usuário confirmou plano de fases
- [ ] ROADMAP.md atualizado com novas fases
- [ ] Tabela de rastreabilidade de REQUIREMENTS.md atualizada com atribuições de fase de fechamento de lacunas
- [ ] Checkboxes de requisitos não satisfeitos redefinidos (`[x]` → `[ ]`)
- [ ] Contagem de cobertura atualizada em REQUIREMENTS.md
- [ ] Diretórios de fase criados
- [ ] Alterações commitadas (inclui REQUIREMENTS.md)
- [ ] Usuário sabe executar `/gsd-plan-phase` em seguida
</success_criteria>
</output>
