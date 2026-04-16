# Template de Contexto de Fase

Template para `.planning/phases/XX-name/{phase_num}-CONTEXT.md` — captura decisões de implementação de uma fase.

**Propósito:** Documentar decisões que agentes downstream precisam. O Pesquisador usa isto para saber O QUE investigar. O Planejador usa isto para saber quais escolhas estão bloqueadas vs. flexíveis.

**Princípio fundamental:** As categorias NÃO são predefinidas. Elas emergem do que foi realmente discutido PARA ESTA fase. Uma fase de CLI tem seções relevantes para CLI, uma fase de UI tem seções relevantes de UI.

**Consumidores downstream:**
- `gsd-phase-researcher` — Lê decisões para focar a pesquisa (ex.: "layout de cartão" → pesquisar padrões de componente de cartão)
- `gsd-planner` — Lê decisões para criar tarefas específicas (ex.: "scroll infinito" → tarefa inclui virtualização)

---

## Template do Arquivo

```markdown
# Fase [X]: [Nome] - Contexto

**Coletado em:** [data]
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

[Declaração clara do que esta fase entrega — a âncora de escopo. Vem do ROADMAP.md e está fixada. A discussão esclarece a implementação dentro deste limite.]

</domain>

<decisions>
## Decisões de Implementação

### [Área 1 que foi discutida]
- **D-01:** [Decisão específica tomada]
- **D-02:** [Outra decisão, se aplicável]

### [Área 2 que foi discutida]
- **D-03:** [Decisão específica tomada]

### [Área 3 que foi discutida]
- **D-04:** [Decisão específica tomada]

### Critério do Claude
[Áreas onde o usuário explicitamente disse "você decide" — o Claude tem flexibilidade aqui durante o planejamento/implementação]

</decisions>

<specifics>
## Ideias Específicas

[Quaisquer referências, exemplos ou momentos de "eu quero como X" da discussão. Referências de produto, comportamentos específicos, padrões de interação.]

[Se nenhum: "Sem requisitos específicos — aberto a abordagens padrão"]

</specifics>

<canonical_refs>
## Referências Canônicas

**Agentes downstream DEVEM ler estas antes de planejar ou implementar.**

[Liste cada spec, ADR, doc de feature ou doc de design que define requisitos ou restrições para esta fase. Use caminhos relativos completos para que os agentes possam lê-los diretamente. Agrupe por área de tópico quando a fase tem múltiplas preocupações.]

### [Área de tópico 1]
- `caminho/para/spec-ou-adr.md` — [O que este documento decide/define que é relevante]
- `caminho/para/doc.md` §N — [Seção específica e o que cobre]

### [Área de tópico 2]
- `caminho/para/feature-doc.md` — [Qual capacidade este define]

[Se o projeto não tem specs externas: "Sem specs externas — requisitos estão completamente capturados nas decisões acima"]

</canonical_refs>

<code_context>
## Insights do Código Existente

### Ativos Reutilizáveis
- [Componente/hook/utilitário]: [Como poderia ser usado nesta fase]

### Padrões Estabelecidos
- [Padrão]: [Como restringe/habilita esta fase]

### Pontos de Integração
- [Onde o novo código se conecta ao sistema existente]

</code_context>

<deferred>
## Ideias Adiadas

[Ideias que surgiram durante a discussão mas pertencem a outras fases. Capturadas aqui para não se perder, mas explicitamente fora do escopo desta fase.]

[Se nenhuma: "Nenhuma — a discussão ficou dentro do escopo da fase"]

</deferred>

---

*Fase: XX-name*
*Contexto coletado em: [data]*
```

<good_examples>

**Exemplo 1: Funcionalidade visual (Feed de Posts)**

```markdown
# Fase 3: Feed de Posts - Contexto

**Coletado em:** 2025-01-20
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

Exibir posts de usuários seguidos em um feed com scroll. Usuários podem visualizar posts e ver contagens de engajamento. Criar posts e interações são fases separadas.

</domain>

<decisions>
## Decisões de Implementação

### Estilo de layout
- Layout em cartões, não em timeline ou lista
- Cada cartão mostra: avatar do autor, nome, timestamp, conteúdo completo do post, contagens de reação
- Cartões têm sombras sutis, cantos arredondados — visual moderno

### Comportamento de carregamento
- Scroll infinito, não paginação
- Pull-to-refresh no mobile
- Indicador de novos posts no topo ("3 novos posts") em vez de inserção automática

### Estado vazio
- Ilustração amigável + "Siga pessoas para ver posts aqui"
- Sugerir 3-5 contas para seguir com base em interesses

### Critério do Claude
- Design do skeleton de carregamento
- Espaçamento e tipografia exatos
- Tratamento do estado de erro

</decisions>

<canonical_refs>
## Referências Canônicas

### Exibição do feed
- `docs/features/social-feed.md` — Requisitos do feed, campos do cartão de post, regras de exibição de engajamento
- `docs/decisions/adr-012-infinite-scroll.md` — Decisão de estratégia de scroll, requisitos de virtualização

### Estados vazios
- `docs/design/empty-states.md` — Padrões de estado vazio, diretrizes de ilustração

</canonical_refs>

<specifics>
## Ideias Específicas

- "Gosto de como o Twitter mostra o indicador de novos posts sem interromper a posição do scroll"
- Cartões devem parecer com os cards de issue do Linear — limpos, sem poluição visual

</specifics>

<deferred>
## Ideias Adiadas

- Comentários em posts — Fase 5
- Salvar posts nos favoritos — adicionar ao backlog

</deferred>

---

*Fase: 03-post-feed*
*Contexto coletado em: 2025-01-20*
```

**Exemplo 2: Ferramenta CLI (Backup de banco de dados)**

```markdown
# Fase 2: Comando de Backup - Contexto

**Coletado em:** 2025-01-20
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

Comando CLI para fazer backup do banco de dados em arquivo local ou S3. Suporta backups completos e incrementais. O comando de restauração é uma fase separada.

</domain>

<decisions>
## Decisões de Implementação

### Formato de saída
- JSON para uso programático, formato de tabela para humanos
- Padrão de tabela, flag --json para JSON
- Modo verbose (-v) mostra progresso, silencioso por padrão

### Design de flags
- Flags curtas para opções comuns: -o (output), -v (verbose), -f (forçar)
- Flags longas para clareza: --incremental, --compress, --encrypt
- Obrigatório: string de conexão com o banco (posicional ou --db)

### Recuperação de erros
- Tentar 3 vezes em falha de rede, então falhar com mensagem clara
- Flag --no-retry para falhar rápido
- Backups parciais são deletados em caso de falha (sem arquivos corrompidos)

### Critério do Claude
- Implementação exata da barra de progresso
- Escolha do algoritmo de compressão
- Tratamento de arquivos temporários

</decisions>

<canonical_refs>
## Referências Canônicas

### CLI de backup
- `docs/features/backup-restore.md` — Requisitos de backup, backends suportados, spec de criptografia
- `docs/decisions/adr-007-cli-conventions.md` — Nomenclatura de flags, códigos de saída, padrões de formato de saída

</canonical_refs>

<specifics>
## Ideias Específicas

- "Quero que pareça com pg_dump — familiar para pessoas de banco de dados"
- Deve funcionar em pipelines CI (códigos de saída, sem prompts interativos)

</specifics>

<deferred>
## Ideias Adiadas

- Backups agendados — fase separada
- Rotação/retenção de backups — adicionar ao backlog

</deferred>

---

*Fase: 02-backup-command*
*Contexto coletado em: 2025-01-20*
```

**Exemplo 3: Tarefa de organização (Biblioteca de fotos)**

```markdown
# Fase 1: Organização de Fotos - Contexto

**Coletado em:** 2025-01-20
**Status:** Pronto para planejamento

<domain>
## Limite da Fase

Organizar biblioteca de fotos existente em pastas estruturadas. Tratar duplicatas e aplicar nomenclatura consistente. Marcação e busca são fases separadas.

</domain>

<decisions>
## Decisões de Implementação

### Critérios de agrupamento
- Agrupamento primário por ano, depois por mês
- Eventos detectados por agrupamento temporal (fotos dentro de 2 horas = mesmo evento)
- Pastas de evento nomeadas por data + localização se disponível

### Tratamento de duplicatas
- Manter a versão de maior resolução
- Mover duplicatas para pasta _duplicates (não deletar)
- Registrar todas as decisões de duplicata para revisão

### Convenção de nomenclatura
- Formato: YYYY-MM-DD_HH-MM-SS_nomearquivooriginal.ext
- Preservar nome de arquivo original como sufixo para pesquisabilidade
- Tratar colisões de nome com sufixo incremental

### Critério do Claude
- Algoritmo exato de agrupamento
- Como tratar fotos sem dados EXIF
- Uso de emoji em pastas

</decisions>

<canonical_refs>
## Referências Canônicas

### Regras de organização
- `docs/features/photo-organization.md` — Regras de agrupamento, política de duplicatas, spec de nomenclatura
- `docs/decisions/adr-003-exif-handling.md` — Estratégia de extração EXIF, fallback para metadados ausentes

</canonical_refs>

<specifics>
## Ideias Específicas

- "Quero conseguir encontrar fotos por aproximadamente quando foram tiradas"
- Não deletar nada — no pior caso, mover para uma pasta de revisão

</specifics>

<deferred>
## Ideias Adiadas

- Agrupamento por reconhecimento facial — fase futura
- Sincronização com nuvem — fora do escopo por enquanto

</deferred>

---

*Fase: 01-photo-organization*
*Contexto coletado em: 2025-01-20*
```

</good_examples>

<guidelines>
**Este template captura DECISÕES para agentes downstream.**

A saída deve responder: "O que o pesquisador precisa investigar? Quais escolhas estão bloqueadas para o planejador?"

**Conteúdo bom (decisões concretas):**
- "Layout em cartões, não em timeline"
- "Tentar 3 vezes em falha de rede, então falhar"
- "Agrupar por ano, depois por mês"
- "JSON para uso programático, tabela para humanos"

**Conteúdo ruim (muito vago):**
- "Deve parecer moderno e limpo"
- "Boa experiência do usuário"
- "Rápido e responsivo"
- "Fácil de usar"

**Após a criação:**
- O arquivo vive no diretório da fase: `.planning/phases/XX-name/{phase_num}-CONTEXT.md`
- `gsd-phase-researcher` usa as decisões para focar a investigação E lê as canonical_refs para saber QUAIS docs estudar
- `gsd-planner` usa as decisões + pesquisa para criar tarefas executáveis E lê as canonical_refs para verificar alinhamento
- Agentes downstream NÃO devem precisar perguntar ao usuário novamente sobre decisões já capturadas

**CRÍTICO — Referências canônicas:**
- A seção `<canonical_refs>` é OBRIGATÓRIA. Todo CONTEXT.md deve ter uma.
- Se seu projeto tem specs externas, ADRs ou docs de design, liste-os com caminhos relativos completos agrupados por tópico
- Se o ROADMAP.md lista `Canonical refs:` por fase, extraia e expanda essas referências
- Menções inline como "veja ADR-019" espalhadas nas decisões são inúteis para agentes downstream — eles precisam de caminhos completos e referências de seção em uma seção dedicada que possam encontrar
- Se não existem specs externas, diga isso explicitamente — não omita a seção silenciosamente
</guidelines>
