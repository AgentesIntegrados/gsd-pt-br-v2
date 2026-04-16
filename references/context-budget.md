# Regras de Orçamento de Contexto

Regras padrão para manter o contexto do orquestrador enxuto. Referencie em workflows que geram subagentes ou leem conteúdo significativo.

Veja também: `references/universal-anti-patterns.md` para o conjunto completo de regras universais.

---

## Regras Universais

Todo workflow que gera agentes ou lê conteúdo significativo deve seguir estas regras:

1. **Nunca** leia arquivos de definição de agentes (`agents/*.md`) -- `subagent_type` os carrega automaticamente
2. **Nunca** insira arquivos grandes inline em prompts de subagentes -- diga aos agentes para ler os arquivos do disco
3. **A profundidade de leitura escala com a janela de contexto** -- verifique `context_window_tokens` em `.planning/config.json`:
   - Em < 500000 tokens (padrão 200k): leia apenas frontmatter, campos de status ou resumos. Nunca leia os corpos completos de SUMMARY.md, VERIFICATION.md ou RESEARCH.md.
   - Em >= 500000 tokens (modelo 1M): PODE ler corpos completos de saída de subagentes quando o conteúdo é necessário para apresentação inline ou tomada de decisão. Ainda evite leituras desnecessárias.
4. **Delegue** trabalho pesado a subagentes -- o orquestrador roteia, não executa
5. **Aviso proativo**: Se você já consumiu contexto significativo (leituras de arquivos grandes, múltiplos resultados de subagentes), avise o usuário: "O orçamento de contexto está ficando pesado. Considere fazer um checkpoint do progresso."

## Profundidade de Leitura por Janela de Contexto

| Janela de Contexto | Leitura de Saída de Subagente | SUMMARY.md | VERIFICATION.md | PLAN.md (outras fases) |
|---------------|------------------------|------------|-----------------|------------------------|
| < 500k (modelo 200k) | Somente frontmatter | Somente frontmatter | Somente frontmatter | Somente fase atual |
| >= 500k (modelo 1M) | Corpo completo permitido | Corpo completo permitido | Corpo completo permitido | Somente fase atual |

**Como verificar:** Leia `.planning/config.json` e inspecione `context_window_tokens`. Se o campo estiver ausente, trate como 200k (padrão conservador).

## Níveis de Degradação de Contexto

Monitore o uso de contexto e ajuste o comportamento de acordo:

| Nível | Uso | Comportamento |
|------|-------|----------|
| PICO | 0-30% | Operações completas. Leia corpos, gere múltiplos agentes, apresente resultados inline. |
| BOM | 30-50% | Operações normais. Prefira leituras de frontmatter, delegue agressivamente. |
| DEGRADANDO | 50-70% | Economize. Leituras apenas de frontmatter, inlining mínimo, avise o usuário sobre o orçamento. |
| BAIXO | 70%+ | Modo de emergência. Faça checkpoint do progresso imediatamente. Sem novas leituras, a menos que crítico. |

## Sinais de Alerta de Degradação de Contexto

A qualidade degrada gradualmente antes que os limites de pânico sejam acionados. Fique atento a estes sinais precoces:

- **Conclusão parcial silenciosa** -- o agente afirma que a tarefa está concluída, mas a implementação está incompleta. O self-check captura existência de arquivos, mas não completude semântica. Sempre verifique se a saída do agente atende aos must_haves do plano, não apenas que os arquivos existam.
- **Vagueza crescente** -- o agente começa a usar frases como "tratamento adequado" ou "padrões padrão" em vez de código específico. Isso indica pressão de contexto mesmo antes que avisos de orçamento sejam acionados.
- **Passos pulados** -- o agente omite passos de protocolo que normalmente seguiria. Se os critérios de sucesso de um agente têm 8 itens, mas ele reporta apenas 5, suspeite de pressão de contexto.

Ao delegar a agentes, o orquestrador não pode verificar a correção semântica da saída do agente -- apenas completude estrutural. Essa é uma limitação fundamental. Mitigue com must_haves.truths e verificação por amostragem.
