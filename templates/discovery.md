# Template de Discovery

Template para `.planning/phases/XX-name/DISCOVERY.md` — pesquisa superficial para decisões de biblioteca/opção.

**Propósito:** Responder perguntas do tipo "qual biblioteca/opção devemos usar" durante a discovery obrigatória na fase de planejamento.

Para pesquisa profunda de ecossistema ("como especialistas constroem isso"), use `/gsd-research-phase` que produz RESEARCH.md.

---

## Template do Arquivo

```markdown
---
phase: XX-name
type: discovery
topic: [tópico-de-discovery]
---

<session_initialization>
Antes de iniciar a discovery, verifique a data de hoje:
!`date +%Y-%m-%d`

Use esta data ao pesquisar informações "atuais" ou "mais recentes".
Exemplo: Se hoje é 2025-11-22, pesquise "2025" e não "2024".
</session_initialization>

<discovery_objective>
Descobrir [tópico] para informar a implementação de [nome da fase].

Propósito: [Qual decisão/implementação isso habilita]
Escopo: [Limites]
Saída: DISCOVERY.md com recomendação
</discovery_objective>

<discovery_scope>
<include>
- [Pergunta a responder]
- [Área a investigar]
- [Comparação específica se necessário]
</include>

<exclude>
- [Fora do escopo desta discovery]
- [Adie para a fase de implementação]
</exclude>
</discovery_scope>

<discovery_protocol>

**Prioridade de Fontes:**
1. **Context7 MCP** — Para documentação de biblioteca/framework (atual, autoritativa)
2. **Docs Oficiais** — Para plataformas específicas ou bibliotecas não indexadas
3. **WebSearch** — Para comparações, tendências, padrões da comunidade (verifique todos os achados)

**Checklist de Qualidade:**
Antes de concluir a discovery, verifique:
- [ ] Todas as afirmações têm fontes autoritativas (Context7 ou docs oficiais)
- [ ] Afirmações negativas ("X não é possível") verificadas com documentação oficial
- [ ] Sintaxe de API/configuração de Context7 ou docs oficiais (nunca apenas WebSearch)
- [ ] Achados do WebSearch cruzados com fontes autoritativas
- [ ] Atualizações recentes/changelogs verificados para breaking changes
- [ ] Abordagens alternativas consideradas (não apenas a primeira solução encontrada)

**Níveis de Confiança:**
- ALTO: Context7 ou docs oficiais confirmam
- MÉDIO: WebSearch + Context7/docs oficiais confirmam
- BAIXO: Apenas WebSearch ou apenas conhecimento de treinamento (marcar para validação)

</discovery_protocol>


<output_structure>
Criar `.planning/phases/XX-name/DISCOVERY.md`:

```markdown
# Discovery: [Tópico]

## Resumo
[2-3 parágrafos de resumo executivo - o que foi pesquisado, o que foi encontrado, o que é recomendado]

## Recomendação Principal
[O que fazer e por quê - seja específico e acionável]

## Alternativas Consideradas
[O que mais foi avaliado e por que não foi escolhido]

## Principais Achados

### [Categoria 1]
- [Achado com URL de fonte e relevância para nosso caso]

### [Categoria 2]
- [Achado com URL de fonte e relevância]

## Exemplos de Código
[Padrões de implementação relevantes, se aplicável]

## Metadados

<metadata>
<confidence level="high|medium|low">
[Por que este nível de confiança - com base na qualidade e verificação das fontes]
</confidence>

<sources>
- [Fontes autoritativas primárias utilizadas]
</sources>

<open_questions>
[O que não pôde ser determinado ou precisa de validação durante a implementação]
</open_questions>

<validation_checkpoints>
[Se a confiança for BAIXA ou MÉDIA, liste coisas específicas a verificar durante a implementação]
</validation_checkpoints>
</metadata>
```
</output_structure>

<success_criteria>
- Todas as perguntas do escopo respondidas com fontes autoritativas
- Itens do checklist de qualidade concluídos
- Recomendação principal clara
- Achados de baixa confiança marcados com checkpoints de validação
- Pronto para informar a criação do PLAN.md
</success_criteria>

<guidelines>
**Quando usar discovery:**
- Escolha de tecnologia incerta (biblioteca A vs B)
- Boas práticas necessárias para integração desconhecida
- Investigação de API/biblioteca necessária
- Decisão única pendente

**Quando NÃO usar:**
- Padrões estabelecidos (CRUD, auth com biblioteca conhecida)
- Detalhes de implementação (adie para execução)
- Perguntas respondíveis a partir do contexto existente do projeto

**Quando usar RESEARCH.md em vez disso:**
- Domínios de nicho/complexos (3D, jogos, áudio, shaders)
- Necessidade de conhecimento de ecossistema, não apenas escolha de biblioteca
- Perguntas do tipo "como especialistas constroem isso"
- Use `/gsd-research-phase` para esses casos
</guidelines>
