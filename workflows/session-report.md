<purpose>
Gerar um documento de resumo pós-sessão capturando o trabalho realizado, resultados alcançados e estimativa de uso de recursos. Escreve SESSION_REPORT.md em .planning/reports/ para revisão humana e compartilhamento com partes interessadas.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt que invocou este fluxo antes de começar.
</required_reading>

<process>

<step name="gather_session_data">
Coletar dados da sessão das fontes disponíveis:

1. **STATE.md** — fase atual, marco, progresso, bloqueios, decisões
2. **Log do Git** — commits feitos durante esta sessão (últimas 24h ou desde o último relatório)
3. **Arquivos de Plano/Resumo** — planos executados, resumos escritos
4. **ROADMAP.md** — contexto do marco e metas da fase

```bash
# Obter commits recentes (últimas 24 horas)
git log --oneline --since="24 hours ago" --no-merges 2>/dev/null || echo "No recent commits"

# Contar arquivos alterados
git diff --stat HEAD~10 HEAD 2>/dev/null | tail -1 || echo "No diff available"
```

Ler `.planning/STATE.md` para obter:
- Marco e fase atuais
- Percentual de progresso
- Bloqueios ativos
- Decisões recentes

Ler `.planning/ROADMAP.md` para obter nome e objetivos do marco.

Verificar relatórios existentes:
```bash
ls -la .planning/reports/SESSION_REPORT*.md 2>/dev/null || echo "No previous reports"
```
</step>

<step name="estimate_usage">
Estimar uso de tokens a partir de sinais observáveis:

- A contagem de chamadas de ferramentas não está diretamente disponível, então estimar a partir da atividade git e operações de arquivo
- Nota: Esta é uma **estimativa** — contagens exatas de tokens requerem instrumentação em nível de API não disponível para hooks

Heurísticas de estimativa:
- Cada commit ≈ 1 ciclo de plano (pesquisa + plano + execução + verificação)
- Cada arquivo de plano ≈ 2.000-5.000 tokens de contexto do agente
- Cada arquivo de resumo ≈ 1.000-2.000 tokens gerados
- Instanciações de subagentes multiplicam por ~1,5x por tipo de agente utilizado
</step>

<step name="generate_report">
Criar o diretório e arquivo de relatório:

```bash
mkdir -p .planning/reports
```

Escrever `.planning/reports/SESSION_REPORT.md` (ou `.planning/reports/AAAAMMDD-session-report.md` se relatórios anteriores existirem):

```markdown
# Relatório de Sessão GSD

**Gerado em:** [timestamp]
**Projeto:** [do título do PROJECT.md ou nome do diretório]
**Marco:** [N] — [nome do marco do ROADMAP.md]

---

## Resumo da Sessão

**Duração:** [estimado do primeiro ao último timestamp de commit, ou "Sessão única"]
**Progresso da Fase:** [do STATE.md]
**Planos Executados:** [contagem de resumos escritos nesta sessão]
**Commits Realizados:** [contagem do log do git]

## Trabalho Realizado

### Fases Trabalhadas
[Listar fases trabalhadas com breve descrição do que foi feito]

### Resultados Principais
[Lista de entregas concretas: arquivos criados, funcionalidades implementadas, bugs corrigidos]

### Decisões Tomadas
[Da tabela de decisões do STATE.md, se alguma foi adicionada nesta sessão]

## Arquivos Alterados

[Resumo de arquivos modificados, criados, excluídos — do git diff stat]

## Bloqueios e Itens em Aberto

[Bloqueios ativos do STATE.md]
[Quaisquer itens TODO criados durante a sessão]

## Estimativa de Uso de Recursos

| Métrica | Estimativa |
|---------|------------|
| Commits | [N] |
| Arquivos alterados | [N] |
| Planos executados | [N] |
| Subagentes instanciados | [estimado] |

> **Nota:** Estimativas de tokens e custo requerem instrumentação em nível de API.
> Estas métricas refletem apenas a atividade observável da sessão.

---

*Gerado por `/gsd-session-report`*
```
</step>

<step name="display_result">
Mostrar ao usuário:

```
## Relatório de Sessão Gerado

📄 `.planning/reports/[filename].md`

### Destaques
- **Commits:** [N]
- **Arquivos alterados:** [N]  
- **Progresso da fase:** [X]%
- **Planos executados:** [N]
```

Se este for o primeiro relatório, mencionar:
```
💡 Execute `/gsd-session-report` ao final de cada sessão para construir um histórico de atividades do projeto.
```
</step>

</process>

<success_criteria>
- [ ] Dados da sessão coletados do STATE.md, log git e arquivos de plano
- [ ] Relatório escrito em .planning/reports/
- [ ] Relatório inclui resumo do trabalho, resultados e alterações de arquivos
- [ ] Nome do arquivo inclui data para evitar sobreescritas
- [ ] Resumo do resultado exibido ao usuário
</success_criteria>
