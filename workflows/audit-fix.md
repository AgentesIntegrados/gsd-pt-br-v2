<purpose>
Pipeline autônomo de auditoria para correção. Executa uma auditoria, analisa os achados, classifica cada um como corrigível automaticamente ou somente manual, inicializa agentes executores para problemas corrigíveis, executa testes após cada correção e faz commit atomicamente com IDs de achados para rastreabilidade.
</purpose>

<available_agent_types>
- gsd-executor — executes a specific, scoped code change
</available_agent_types>

<process>

<step name="parse-arguments">
Extraia os flags da invocação do usuário:

- `--max N` — número máximo de achados para corrigir (padrão: **5**)
- `--severity high|medium|all` — severidade mínima para processar (padrão: **medium**)
- `--dry-run` — classifique achados sem corrigir (mostra apenas a tabela de classificação)
- `--source <audit>` — qual auditoria executar (padrão: **audit-uat**)

Valide que `--source` é uma auditoria suportada. Atualmente suportado:
- `audit-uat`

Se `--source` não for suportado, pare com um erro:
```
Erro: Fonte de auditoria não suportada "{source}". Fontes suportadas: audit-uat
```
</step>

<step name="run-audit">
Invoque o comando de auditoria de origem e capture a saída.

Para a fonte `audit-uat`:
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init audit-uat 2>/dev/null || echo "{}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Leia os arquivos UAT e de verificação existentes para extrair achados:
- Glob: `.planning/phases/*/*-UAT.md`
- Glob: `.planning/phases/*/*-VERIFICATION.md`

Analise cada achado em um registro estruturado:
- **ID** — identificador sequencial (F-01, F-02, ...)
- **description** — resumo conciso do problema
- **severity** — high, medium ou low
- **file_refs** — caminhos de arquivo específicos referenciados no achado
</step>

<step name="classify-findings">
Para cada achado, classifique como um de:

- **auto-fixable** — mudança de código clara, arquivo específico referenciado, correção testável
- **manual-only** — requer decisões de design, escopo ambíguo, mudanças arquiteturais, entrada do usuário necessária
- **skip** — severidade abaixo do limiar `--severity`

**Heurísticas de classificação** (prefira manual-only quando incerto):

Sinais de auto-fixable:
- Referencia um caminho de arquivo específico + número de linha
- Descreve um teste ou asserção faltando
- Export faltando, caminho de import errado, typo em identificador
- Mudança clara em arquivo único com comportamento esperado óbvio

Sinais de manual-only:
- Usa palavras como "considerar", "avaliar", "projetar", "repensar"
- Requer nova arquitetura ou mudanças de API
- Escopo ambíguo ou múltiplas abordagens válidas
- Requer entrada do usuário ou decisões de design
- Preocupações transversais afetando múltiplos subsistemas
- Problemas de performance ou escalabilidade sem correção clara

**Quando incerto, sempre classifique como manual-only.**
</step>

<step name="present-classification">
Exiba a tabela de classificação:

```
## Classificação de Audit-Fix

| # | Achado | Severidade | Classificação | Razão |
|---|---------|----------|---------------|--------|
| F-01 | Export faltando em index.ts | high | auto-fixable | Arquivo específico, correção clara |
| F-02 | Sem tratamento de erro no fluxo de pagamento | high | manual-only | Requer decisões de design |
| F-03 | Stub de teste com 0 asserções | medium | auto-fixable | Lacuna de teste clara |
```

Se `--dry-run` foi especificado, **pare aqui e saia**. A tabela de classificação é a saída final — não prossiga para correção.
</step>

<step name="fix-loop">
Para cada achado **auto-fixable** (até `--max`, ordenado por severidade desc):

**a. Inicie o agente executor:**
```
Task(
  prompt="Fix finding {ID}: {description}. Files: {file_refs}. Make the minimal change to resolve this specific finding. Do not refactor surrounding code.",
  subagent_type="gsd-executor"
)
```

**b. Execute os testes:**
```bash
npm test 2>&1 | tail -20
```

**c. Se os testes passarem** — faça commit atomicamente:
```bash
git add {changed_files}
git commit -m "fix({scope}): resolve {ID} — {description}"
```
A mensagem de commit **deve** incluir o ID do achado (ex: F-01) para rastreabilidade.

**d. Se os testes falharem** — reverta as mudanças, marque o achado como `fix-failed` e **pare o pipeline**:
```bash
git checkout -- {changed_files} 2>/dev/null
```
Registre o motivo da falha e pare o processamento — não continue para o próximo achado.
Uma falha de teste indica que a base de código pode estar em um estado inesperado, então o pipeline deve parar para evitar problemas em cascata. Os achados auto-fixable restantes aparecerão no relatório como `not-attempted`.
</step>

<step name="report">
Apresente o resumo final:

```
## Audit-Fix Concluído

**Fonte:** {audit_command}
**Achados:** {total} total, {auto} auto-fixable, {manual} somente manual
**Corrigidos:** {fixed_count}/{auto} achados auto-fixable
**Falharam:** {failed_count} (revertidos)

| # | Achado | Status | Commit |
|---|---------|--------|--------|
| F-01 | Export faltando | Corrigido | abc1234 |
| F-03 | Stub de teste | Correção falhou | (revertido) |

### Achados somente manual (requerem atenção do desenvolvedor):
- F-02: Sem tratamento de erro no fluxo de pagamento — requer decisões de design
```
</step>

</process>

<success_criteria>
- Achados auto-fixable processados sequencialmente até --max ser atingido ou uma falha de teste parar o pipeline
- Testes passam após cada commit corrigido (sem commits quebrados)
- Correções falhadas são revertidas completamente (sem mudanças parciais restantes)
- Pipeline para após a primeira falha de teste (sem correções em cascata)
- Toda mensagem de commit contém o ID do achado
- Achados somente manual são apresentados para atenção do desenvolvedor
- --dry-run produz uma tabela de classificação útil independente
</success_criteria>
