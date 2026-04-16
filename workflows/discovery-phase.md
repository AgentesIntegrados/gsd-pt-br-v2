<purpose>
Executar descoberta de base de código em 3 níveis de profundidade: verificar (rápido), padrão e profundo. Produz DISCOVERY.md com mapa da arquitetura, padrões, dívida técnica e recomendações.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt que invocou antes de começar.
</required_reading>

<process>

<step name="initialize">
Analisar $ARGUMENTS para o modo de profundidade:

- `--verify` → DEPTH=verify (varredura rápida, ~2 min)
- `--standard` (padrão) → DEPTH=standard (análise padrão, ~5 min)
- `--deep` → DEPTH=deep (investigação profunda, ~15 min)

Carregar configuração:
```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
DISCOVERY_MODEL=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" resolve-model gsd-discoverer --raw)
AGENT_SKILLS=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" agent-skills gsd-discoverer 2>/dev/null)
```

Exibir banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DESCOBERTA {depth} — FASE {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="depth_verify">
**Se DEPTH=verify:**

Executar varredura de verificação rápida:

```bash
# Estrutura de diretórios (2 níveis)
find . -maxdepth 2 -type d -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | sort

# Stack tecnológico
cat package.json 2>/dev/null | head -30
cat requirements.txt 2>/dev/null | head -20
cat go.mod 2>/dev/null | head -10
cat Cargo.toml 2>/dev/null | head -10

# Pontos de entrada
ls src/index.* src/main.* app.* main.* server.* 2>/dev/null

# Contagem de arquivos por extensão
find . -not -path "*/node_modules/*" -not -path "*/.git/*" -name "*.ts" -o -name "*.tsx" -o -name "*.py" -o -name "*.go" 2>/dev/null | wc -l
```

Exibir resumo rápido:
```
Stack: {tecnologias detectadas}
Estrutura: {descrição de 1 linha}
Arquivos: {contagem}
Pontos de entrada: {lista}
```

**Sem geração de DISCOVERY.md no modo verify — apenas saída do console.**
</step>

<step name="depth_standard">
**Se DEPTH=standard:**

```
◆ Iniciando descoberta padrão...
```

Iniciar agente gsd-discoverer:

```
Task(
  prompt="""
Leia $HOME/.claude/agents/gsd-discoverer.md para instruções.

<objetivo>
Descoberta padrão da base de código — mapear arquitetura, padrões e estado.
Profundidade: standard
</objetivo>

<escopo>
- Mapa de arquitetura (módulos principais, fluxos de dados)
- Padrões de design em uso
- Qualidade do código e dívida técnica
- Lacunas de cobertura de testes
- Dependências externas e integrações
</escopo>

${AGENT_SKILLS}

<saida>
Escrever DISCOVERY.md na raiz do projeto ou em .planning/
</saida>
""",
  subagent_type="gsd-discoverer",
  model="{DISCOVERY_MODEL}",
  description="Descoberta padrão da base de código"
)
```
</step>

<step name="depth_deep">
**Se DEPTH=deep:**

```
◆ Iniciando descoberta profunda... (pode levar ~15 minutos)
```

Iniciar agente gsd-discoverer em modo deep:

```
Task(
  prompt="""
Leia $HOME/.claude/agents/gsd-discoverer.md para instruções.

<objetivo>
Descoberta profunda da base de código — análise completa para planejamento de refatoração ou migração.
Profundidade: deep
</objetivo>

<escopo>
- Tudo do padrão +
- Análise de dependências entre módulos (acoplamento/coesão)
- Caminhos de execução críticos
- Gargalos de desempenho identificados
- Problemas de segurança superficiais
- Oportunidades de simplificação
- Cobertura de testes por área funcional
- Qualidade da documentação
</escopo>

${AGENT_SKILLS}

<saida>
Escrever DISCOVERY.md completo com análise detalhada
</saida>
""",
  subagent_type="gsd-discoverer",
  model="{DISCOVERY_MODEL}",
  description="Descoberta profunda da base de código"
)
```
</step>

<step name="present_results">
**Para modos standard e deep:**

Após o retorno do agente, exibir resumo:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► DESCOBERTA CONCLUÍDA ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Arquivo de descoberta: {caminho para DISCOVERY.md}

## Destaques

**Arquitetura:** {descrição de 1 linha}
**Stack:** {tecnologias}
**Dívida técnica:** {nível: baixa/média/alta}
**Cobertura de testes:** {porcentagem estimada}

## Principais descobertas

{3-5 descobertas mais importantes}

───────────────────────────────────────────────────────────────

## ▶ Próximos passos

- `/gsd-plan-phase {N}` — planejar próxima fase usando contexto de descoberta
- `/gsd-discuss-phase {N}` — discutir abordagem com base nas descobertas
- `/gsd-discovery-phase --deep` — aprofundar análise

───────────────────────────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Profundidade de descoberta determinada pelos argumentos
- [ ] Modo verify: varredura rápida executada, resumo exibido
- [ ] Modo standard/deep: agente gsd-discoverer iniciado
- [ ] DISCOVERY.md criado (standard/deep)
- [ ] Destaques e descobertas principais apresentados
- [ ] Próximos passos sugeridos
</success_criteria>
