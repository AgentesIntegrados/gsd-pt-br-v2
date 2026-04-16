# Modo de Reviews — Referência do Planejador

Ativado quando o orquestrador define o Modo como `reviews`. Replanejamento do zero com o feedback do REVIEWS.md como contexto adicional.

**Mentalidade:** Planejador com novos insights de revisão — não um cirurgião fazendo correções pontuais, mas um arquiteto que leu as críticas dos pares.

### Passo 1: Carregar REVIEWS.md
Leia o arquivo de reviews em `<files_to_read>`. Analise:
- Feedback por revisor (pontos fortes, preocupações, sugestões)
- Resumo de Consenso (preocupações acordadas = maior prioridade para endereçar)
- Visões Divergentes (investigue, tome uma decisão)

### Passo 2: Categorizar o Feedback
Agrupe o feedback de revisão em:
- **Deve endereçar**: Preocupações de consenso de severidade ALTA
- **Deveria endereçar**: Preocupações de severidade MÉDIA de 2+ revisores
- **Considerar**: Sugestões individuais de revisores, itens de severidade BAIXA

### Passo 3: Planejar do Zero com Contexto de Revisão
Crie novos planos seguindo o processo padrão de planejamento, mas com o feedback de revisão como restrições adicionais:
- Cada preocupação de consenso de severidade ALTA DEVE ter uma tarefa que a endereça
- Preocupações MÉDIAS devem ser endereçadas onde for viável, sem over-engineering
- Anote nas ações das tarefas: "Endereça preocupação de revisão: {concern}" para rastreabilidade

### Passo 4: Retornar
Use o formato padrão de retorno PLANNING COMPLETE, adicionando uma seção de reviews:

```markdown
### Feedback de Revisão Endereçado

| Preocupação | Severidade | Como Endereçada |
|-------------|------------|-----------------|
| {concern} | ALTA | Plano {N}, Tarefa {M}: {como} |

### Feedback de Revisão Adiado
| Preocupação | Motivo |
|-------------|--------|
| {concern} | {por quê — fora de escopo, discordância, etc.} |
```
