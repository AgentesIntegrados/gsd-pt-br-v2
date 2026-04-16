---
name: gsd-forense
description: "Investigação post-mortem para fluxos GSD com falha — analisa histórico git, artefatos e estado para diagnosticar o que deu errado"
argument-hint: "[descrição do problema]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
---


<objective>
Investiga o que deu errado durante a execução de um fluxo GSD. Analisa o histórico git, artefatos de `.planning/` e o estado do sistema de arquivos para detectar anomalias e gerar um relatório de diagnóstico estruturado.

Propósito: Diagnosticar fluxos com falha ou travados para que o usuário compreenda a causa raiz e tome ação corretiva.
Saída: Relatório forense salvo em `.planning/forensics/`, apresentado de forma inline, com criação de issue opcional.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/forensics.md
</execution_context>

<context>
**Fontes de dados:**
- `git log` (commits recentes, padrões, lacunas de tempo)
- `git status` / `git diff` (trabalho não commitado, conflitos)
- `.planning/STATE.md` (posição atual, histórico de sessões)
- `.planning/ROADMAP.md` (escopo e progresso da fase)
- `.planning/phases/*/` (PLAN.md, SUMMARY.md, VERIFICATION.md, CONTEXT.md)
- `.planning/reports/SESSION_REPORT.md` (resultados da última sessão)

**Entrada do usuário:**
- Descrição do problema: $ARGUMENTS (opcional — será solicitado se não fornecido)
</context>

<process>
Leia e execute o fluxo forensics a partir de @$HOME/.claude/get-shit-done/workflows/forensics.md do início ao fim.
</process>

<success_criteria>
- Evidências coletadas de todas as fontes de dados disponíveis
- Pelo menos 4 tipos de anomalia verificados (loop travado, artefatos ausentes, trabalho abandonado, crash/interrupção)
- Relatório forense estruturado escrito em `.planning/forensics/report-{timestamp}.md`
- Relatório apresentado de forma inline com descobertas, anomalias e recomendações
- Investigação interativa oferecida para análise mais aprofundada
- Criação de issue no GitHub oferecida se houver descobertas acionáveis
</success_criteria>

<critical_rules>
- **Investigação somente leitura:** Não modifique arquivos-fonte do projeto durante a investigação forense. Apenas escreva o relatório forense e atualize o rastreamento de sessões do STATE.md.
- **Remova dados sensíveis:** Elimine caminhos absolutos, chaves de API e tokens dos relatórios e issues.
- **Baseie descobertas em evidências:** Cada anomalia deve citar commits, arquivos ou dados de estado específicos.
- **Sem especulação sem evidências:** Se os dados forem insuficientes, diga isso — não invente causas raiz.
</critical_rules>
