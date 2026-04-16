# Template de Debug

Template para `.planning/debug/[slug].md` — rastreamento de sessão de debug ativa.

---

## Template do Arquivo

```markdown
---
status: gathering | investigating | fixing | verifying | awaiting_human_verify | resolved
trigger: "[entrada verbatim do usuário]"
created: [timestamp ISO]
updated: [timestamp ISO]
---

## Foco Atual
<!-- SOBRESCREVER em cada atualização - sempre reflete o AGORA -->

hypothesis: [teoria atual sendo testada]
test: [como está sendo testada]
expecting: [o que o resultado significa se verdadeiro/falso]
next_action: [próximo passo imediato — seja específico, não "continuar investigando"]
reasoning_checkpoint: null  <!-- preenchido antes de cada tentativa de correção — veja structured_returns -->
tdd_checkpoint: null  <!-- preenchido quando tdd_mode está ativo após causa raiz confirmada -->

## Sintomas
<!-- Escrito durante a coleta, depois imutável -->

expected: [o que deveria acontecer]
actual: [o que realmente acontece]
errors: [mensagens de erro, se houver]
reproduction: [como acionar]
started: [quando quebrou / sempre esteve quebrado]

## Eliminados
<!-- SOMENTE ACRESCENTAR - evita re-investigar após /clear -->

- hypothesis: [teoria que estava errada]
  evidence: [o que a refutou]
  timestamp: [quando foi eliminada]

## Evidências
<!-- SOMENTE ACRESCENTAR - fatos descobertos durante a investigação -->

- timestamp: [quando encontrado]
  checked: [o que foi examinado]
  found: [o que foi observado]
  implication: [o que isso significa]

## Resolução
<!-- SOBRESCREVER conforme o entendimento evolui -->

root_cause: [vazio até encontrar]
fix: [vazio até aplicar]
verification: [vazio até verificar]
files_changed: []
```

---

<section_rules>

**Frontmatter (status, trigger, timestamps):**
- `status`: SOBRESCREVER — reflete a fase atual
- `trigger`: IMUTÁVEL — entrada verbatim do usuário, nunca muda
- `created`: IMUTÁVEL — definido uma vez
- `updated`: SOBRESCREVER — atualizar a cada mudança

**Foco Atual:**
- SOBRESCREVER completamente a cada atualização
- Sempre reflete o que o Claude está fazendo AGORA
- Se o Claude lê isso após /clear, sabe exatamente onde retomar
- Campos: hypothesis, test, expecting, next_action, reasoning_checkpoint, tdd_checkpoint
- `next_action`: deve ser concreto e acionável — ruim: "continuar investigando"; bom: "Adicionar log na linha 47 de auth.js para observar o valor do token antes de jwt.verify()"
- `reasoning_checkpoint`: SOBRESCREVER antes de cada fix_and_verify — registro de raciocínio estruturado de cinco campos (hypothesis, confirming_evidence, falsification_test, fix_rationale, blind_spots)
- `tdd_checkpoint`: SOBRESCREVER durante as fases red/green de TDD — arquivo de teste, nome, status, saída de falha

**Sintomas:**
- Escritos durante a fase inicial de coleta
- IMUTÁVEIS após a coleta completa
- Ponto de referência para o que estamos tentando corrigir
- Campos: expected, actual, errors, reproduction, started

**Eliminados:**
- SOMENTE ACRESCENTAR — nunca remover entradas
- Evita re-investigar becos sem saída após reset de contexto
- Cada entrada: hypothesis, evidência que a refutou, timestamp
- Crítico para eficiência após limites de /clear

**Evidências:**
- SOMENTE ACRESCENTAR — nunca remover entradas
- Fatos descobertos durante a investigação
- Cada entrada: timestamp, o que foi verificado, o que foi encontrado, implicação
- Constrói o caso para a causa raiz

**Resolução:**
- SOBRESCREVER conforme o entendimento evolui
- Pode ser atualizada múltiplas vezes conforme correções são tentadas
- Estado final mostra causa raiz confirmada e correção verificada
- Campos: root_cause, fix, verification, files_changed

</section_rules>

<lifecycle>

**Criação:** Imediatamente quando /gsd-debug é chamado
- Criar arquivo com trigger da entrada do usuário
- Definir status como "gathering"
- Foco Atual: next_action = "coletar sintomas"
- Sintomas: vazio, a ser preenchido

**Durante a coleta de sintomas:**
- Atualizar a seção Sintomas conforme o usuário responde às perguntas
- Atualizar o Foco Atual com cada pergunta
- Quando completo: status → "investigating"

**Durante a investigação:**
- SOBRESCREVER o Foco Atual com cada hipótese
- ACRESCENTAR às Evidências com cada descoberta
- ACRESCENTAR aos Eliminados quando a hipótese é refutada
- Atualizar o timestamp no frontmatter

**Durante a correção:**
- status → "fixing"
- Atualizar Resolution.root_cause quando confirmado
- Atualizar Resolution.fix quando aplicado
- Atualizar Resolution.files_changed

**Durante a verificação:**
- status → "verifying"
- Atualizar Resolution.verification com os resultados
- Se a verificação falhar: status → "investigating", tentar novamente

**Após a auto-verificação passar:**
- status → "awaiting_human_verify"
- Solicitar confirmação explícita do usuário em um checkpoint
- NÃO mover o arquivo para resolvido ainda

**Na resolução:**
- status → "resolved"
- Mover arquivo para .planning/debug/resolved/ (somente após o usuário confirmar a correção)

</lifecycle>

<resume_behavior>

Quando o Claude lê este arquivo após /clear:

1. Analisar o frontmatter → saber o status
2. Ler o Foco Atual → saber exatamente o que estava acontecendo
3. Ler Eliminados → saber o que NÃO tentar novamente
4. Ler Evidências → saber o que foi aprendido
5. Continuar a partir do next_action

O arquivo É o cérebro do debugging. O Claude deve conseguir retomar perfeitamente de qualquer ponto de interrupção.

</resume_behavior>

<size_constraint>

Mantenha os arquivos de debug focados:
- Entradas de Evidências: 1-2 linhas cada, apenas os fatos
- Eliminados: breve — hypothesis + por que falhou
- Sem prosa narrativa — somente dados estruturados

Se as evidências crescerem muito (10+ entradas), considere se você está andando em círculos. Verifique os Eliminados para garantir que não está retrabalhando o mesmo terreno.

</size_constraint>
