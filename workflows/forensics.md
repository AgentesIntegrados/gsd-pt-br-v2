# Fluxo Forensics

Investigação post-mortem para fluxos GSD que falharam ou travaram. Analisa o histórico do git,
artefatos de `.planning/` e o estado do sistema de arquivos para detectar anomalias e gerar um
relatório diagnóstico estruturado.

**Princípio:** Esta é uma investigação somente leitura. Não modifique arquivos do projeto.
Apenas escreva o relatório forense.

---

## Passo 1: Obter Descrição do Problema

```bash
PROBLEM="$ARGUMENTS"
```

Se `$ARGUMENTS` estiver vazio, pergunte ao usuário:
> "O que deu errado? Descreva o problema — ex.: 'o modo autônomo travou na fase 3',
> 'o execute-phase falhou silenciosamente', 'os custos parecem incomumente altos'."

Registre a descrição do problema para o relatório.

## Passo 2: Coletar Evidências

Colete dados de todas as fontes disponíveis. Fontes ausentes são aceitáveis — adapte-se ao que existe.

### 2a. Histórico do Git

```bash
# Commits recentes (últimos 30)
git log --oneline -30

# Commits com timestamps para análise de lacunas
git log --format="%H %ai %s" -30

# Arquivos alterados em commits recentes (detecta edições repetidas)
git log --name-only --format="" -20 | sort | uniq -c | sort -rn | head -20

# Trabalho não commitado
git status --short
git diff --stat
```

Registre:
- Linha do tempo de commits (datas, mensagens, frequência)
- Arquivos mais editados (potencial indicador de loop travado)
- Mudanças não commitadas (potencial indicador de crash/interrupção)

### 2b. Estado do Planejamento

Leia estes arquivos se existirem:
- `.planning/STATE.md` — marco atual, fase, progresso, bloqueadores, última sessão
- `.planning/ROADMAP.md` — lista de fases com status
- `.planning/config.json` — configuração do fluxo de trabalho

Extraia:
- Fase atual e seu status
- Último ponto de parada registrado na sessão
- Quaisquer bloqueadores ou flags

### 2c. Artefatos de Fase

Para cada diretório de fase em `.planning/phases/*/`:

```bash
ls .planning/phases/*/
```

Para cada fase, verifique quais artefatos existem:
- `{padded}-PLAN.md` ou `{padded}-PLAN-*.md` (planos de execução)
- `{padded}-SUMMARY.md` (resumo de conclusão)
- `{padded}-VERIFICATION.md` (verificação de qualidade)
- `{padded}-CONTEXT.md` (decisões de design)
- `{padded}-RESEARCH.md` (pesquisa pré-planejamento)

Rastreie: quais fases têm conjuntos completos de artefatos vs. lacunas.

### 2d. Relatórios de Sessão

Leia `.planning/reports/SESSION_REPORT.md` se existir — extraia resultados da última sessão,
trabalho concluído, estimativas de tokens.

### 2e. Estado de Git Worktree

```bash
git worktree list
```

Verifique worktrees órfãos (de agentes que crasharam).

## Passo 3: Detectar Anomalias

Avalie as evidências coletadas em relação a estes padrões de anomalia:

### Detecção de Loop Travado

**Sinal:** O mesmo arquivo aparece em 3+ commits consecutivos dentro de uma janela de tempo curta.

```bash
# Procure por arquivos commitados repetidamente em sequência
git log --name-only --format="---COMMIT---" -20
```

Analise os limites dos commits. Se algum arquivo aparecer em 3+ commits consecutivos, sinalize como:
- **Confiança ALTA** se as mensagens de commit forem similares (ex.: "fix:", "fix:", "fix:" no mesmo arquivo)
- **Confiança MÉDIA** se o arquivo aparecer frequentemente mas as mensagens de commit variarem

### Detecção de Artefato Ausente

**Sinal:** A fase parece completa (tem commits, está passada no roadmap) mas carece de artefatos esperados.

Para cada fase que deveria estar completa:
- PLAN.md ausente → o passo de planejamento foi pulado
- SUMMARY.md ausente → a fase não foi fechada corretamente
- VERIFICATION.md ausente → a verificação de qualidade foi pulada

### Detecção de Trabalho Abandonado

**Sinal:** Grande lacuna entre o último commit e o tempo atual, com STATE.md mostrando mid-execution.

```bash
# Tempo desde o último commit
git log -1 --format="%ai"
```

Se STATE.md mostrar uma fase ativa mas o último commit for >2 horas atrás e houver
mudanças não commitadas, sinalize como potencial abandono ou crash.

### Detecção de Crash/Interrupção

**Sinal:** Mudanças não commitadas + STATE.md mostrando mid-execution + worktrees órfãos.

Combine:
- `git status` mostra arquivos modificados/staged
- STATE.md tem uma entrada de execução ativa
- `git worktree list` mostra worktrees além do principal

### Detecção de Desvio de Escopo

**Sinal:** Commits recentes tocam arquivos fora do escopo esperado da fase atual.

Leia o PLAN.md da fase atual para determinar os caminhos de arquivo esperados. Compare com
os arquivos realmente modificados em commits recentes. Sinalize qualquer arquivo que esteja claramente fora
do domínio da fase.

### Detecção de Regressão de Testes

**Sinal:** Mensagens de commit contendo "fix test", "revert" ou re-commits de arquivos de teste.

```bash
git log --oneline -20 | grep -iE "fix test|revert|broken|regression|fail"
```

## Passo 4: Gerar Relatório

Crie o diretório de forensics se necessário:
```bash
mkdir -p .planning/forensics
```

Escreva em `.planning/forensics/report-$(date +%Y%m%d-%H%M%S).md`:

```markdown
# Relatório Forense

**Gerado:** {timestamp ISO}
**Problema:** {descrição do usuário}

---

## Resumo de Evidências

### Atividade do Git
- **Último commit:** {data} — "{mensagem}"
- **Commits (últimos 30):** {count}
- **Intervalo de tempo:** {mais antigo} → {mais recente}
- **Mudanças não commitadas:** {sim/não — liste se sim}
- **Worktrees ativos:** {count — liste se >1}

### Estado do Planejamento
- **Marco atual:** {versão ou "nenhum"}
- **Fase atual:** {número — nome — status}
- **Última sessão:** {stopped_at do STATE.md}
- **Bloqueadores:** {quaisquer flags do STATE.md}

### Completude dos Artefatos
| Fase | PLAN | CONTEXT | RESEARCH | SUMMARY | VERIFICATION |
|-------|------|---------|----------|---------|-------------|
{para cada fase: nome | ✅/❌ por artefato}

## Anomalias Detectadas

### {Tipo de Anomalia} — {Confiança: ALTA/MÉDIA/BAIXA}
**Evidência:** {commits específicos, arquivos ou dados de estado}
**Interpretação:** {o que isso provavelmente significa}

{repita para cada anomalia encontrada}

## Hipótese de Causa Raiz

Com base nas evidências acima, a explicação mais provável é:

{hipótese de 1-3 frases baseada nas anomalias}

## Ações Recomendadas

1. {Passo de remediação específico e acionável}
2. {Outro passo se aplicável}
3. {Comando de recuperação se aplicável — ex.: `/gsd-resume-work`, `/gsd-execute-phase N`}

---

*Relatório gerado por `/gsd-forensics`. Todos os caminhos redatados para portabilidade.*
```

**Regras de redação:**
- Substitua caminhos absolutos por caminhos relativos (remova o prefixo `$HOME`)
- Remova quaisquer chaves de API, tokens ou credenciais encontradas na saída do git diff
- Truncue diffs grandes para as primeiras 50 linhas

## Passo 5: Apresentar Relatório

Exiba o relatório forense completo inline.

## Passo 6: Oferecer Investigação Interativa

> "Relatório salvo em `.planning/forensics/report-{timestamp}.md`.
>
> Posso aprofundar qualquer descoberta. Quer que eu:
> - Trace uma anomalia específica até sua causa raiz?
> - Leia arquivos específicos referenciados nas evidências?
> - Verifique se um problema similar foi reportado antes?"

Se o usuário fizer perguntas de acompanhamento, responda a partir das evidências já coletadas.
Leia arquivos adicionais apenas se especificamente necessário.

## Passo 7: Oferecer Criação de Issue

Se anomalias acionáveis foram encontradas (confiança ALTA ou MÉDIA):

> "Quer que eu crie um issue no GitHub para isso? Vou formatar as descobertas e redatar os caminhos."

Se confirmado:
```bash
# Verifique se o label "bug" existe antes de usá-lo
BUG_LABEL=$(gh label list --search "bug" --json name -q '.[0].name' 2>/dev/null)
LABEL_FLAG=""
if [ -n "$BUG_LABEL" ]; then
  LABEL_FLAG="--label bug"
fi

gh issue create \
  --title "bug: {descrição concisa da anomalia}" \
  $LABEL_FLAG \
  --body "{descobertas formatadas do relatório}"
```

## Passo 8: Atualizar STATE.md

```bash
gsd-tools.cjs state record-session \
  --stopped-at "Investigação forense concluída" \
  --resume-file ".planning/forensics/report-{timestamp}.md"
```
