# Integração do Parceiro de Raciocínio

Raciocínio estendido condicional em pontos de decisão do fluxo de trabalho. Ativa quando `features.thinking_partner: true` em `.planning/config.json` (padrão: false).

---

## Sinais de Detecção de Trade-off

O parceiro de raciocínio ativa quando as respostas do desenvolvedor contêm sinais específicos indicando prioridades concorrentes:

**Sinais de palavras-chave:**
- "or" / "versus" / "vs" conectando duas abordagens
- "tradeoff" / "trade-off" / "tradeoffs"
- "on one hand" / "on the other hand"
- "pros and cons"
- "not sure between" / "torn between"

**Sinais estruturais:**
- Desenvolvedor lista 2+ opções concorrentes
- Desenvolvedor pergunta "which is better" ou "what would you recommend"
- Desenvolvedor reverte uma decisão anterior ("actually, maybe we should...")

**Quando NÃO ativar:**
- O desenvolvedor já fez uma escolha clara
- O "ou" é retórico ou trivial (ex.: "tabs ou espaços" — use a convenção do projeto)
- Perguntas simples de sim/não
- O desenvolvedor pede explicitamente para avançar

---

## Pontos de Integração

### 1. Fase de Discussão — Análise Aprofundada de Trade-off

**Quando:** Durante a etapa `discuss_areas`, após uma resposta do desenvolvedor revelar prioridades concorrentes.

**O que:** Pause o fluxo normal de perguntas e ofereça uma análise estruturada breve:
```
Noto prioridades concorrentes aqui — {X} otimiza para {A} enquanto {Y} otimiza para {B}.

Quer que eu pense nos trade-offs antes de decidirmos?
[Sim, analise os trade-offs] / [Não, já decidi]
```

Se sim, forneça uma análise breve (3-5 tópicos) cobrindo:
- O que cada abordagem otimiza
- O que cada abordagem sacrifica
- Qual se alinha melhor com os objetivos declarados do projeto (do PROJECT.md)
- Uma recomendação com justificativa

Depois retorne ao fluxo normal de discussão.

### 2. Fase de Planejamento — Análise de Decisão Arquitetural

**Quando:** Durante o passo 11 (Tratar Retorno do Verificador), quando o plan-checker sinaliza problemas contendo palavras-chave de trade-off arquitetural.

**O que:** Antes de enviar para o loop de revisão, analise a decisão arquitetural:
```
O plan-checker sinalizou um trade-off arquitetural: {descrição do problema}

Análise breve:
- Opção A: {abordagem} — {prós/contras}
- Opção B: {abordagem} — {prós/contras}
- Recomendação: {escolha} porque {justificativa alinhada com os objetivos da fase}

Aplicar esta recomendação à revisão? [Sim] / [Não, deixa eu decidir]
```

### 3. Exploração — Comparação de Abordagens (requer #1729)

**Quando:** Durante a conversa Socrática, quando múltiplas abordagens viáveis emergem.
**Nota:** Este ponto de integração será adicionado quando /gsd-explore (#1729) chegar.

---

## Configuração

```json
{
  "features": {
    "thinking_partner": true
  }
}
```

Padrão: `false`. O parceiro de raciocínio é opt-in porque adiciona latência a fluxos de trabalho interativos.

---

## Princípios de Design

1. **Leve** — análise inline, não uma sessão interativa separada
2. **Opt-in** — deve ser habilitado explicitamente, nunca ativa por padrão
3. **Pulável** — sempre ofereça "Não, já decidi" para contornar
4. **Breve** — 3-5 tópicos no máximo, não um relatório de pesquisa completo
5. **Alinhado** — recomendações fazem referência aos objetivos do PROJECT.md quando disponíveis
