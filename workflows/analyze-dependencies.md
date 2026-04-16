<purpose>
Analisa as fases do ROADMAP.md em busca de relações de dependência antes da execução. Detecta sobreposição de arquivos entre fases, dependências semânticas de API/fluxo de dados, e sugere entradas de `Depends on` para prevenir conflitos de merge durante execução paralela pelo `/gsd-manager`.
</purpose>

<process>

## 1. Carregar ROADMAP.md

Leia `.planning/ROADMAP.md`. Se não existir, erro: "Nenhum ROADMAP.md encontrado — execute `/gsd-new-project` primeiro."

Extraia todas as fases. Para cada fase capture:
- Número e nome da fase
- Descrição do escopo/objetivo
- Arquivos listados nos campos `Files` ou `files_modified` (se presentes)
- Valor existente do campo `Depends on`

## 2. Inferir Prováveis Modificações de Arquivos

Para cada fase sem `files_modified` explícito, analise a descrição do escopo/objetivo para inferir quais arquivos provavelmente serão modificados. Use estas heurísticas:

- **Fases de banco de dados/schema** → arquivos de migração, definições de schema, arquivos de modelo
- **Fases de API/backend** → arquivos de rota, arquivos de controlador, arquivos de serviço, arquivos de handler
- **Fases de frontend/UI** → arquivos de componente, arquivos de página, arquivos de estilo
- **Fases de auth** → arquivos de middleware, arquivos de rota de auth, arquivos de sessão/token
- **Fases de config/infra** → arquivos de configuração, arquivos de ambiente, arquivos de CI/CD
- **Fases de teste** → arquivos de teste, arquivos spec, arquivos de fixture
- **Fases de utilitários compartilhados** → arquivos lib/utils, definições de tipo compartilhadas

Agrupe as fases por domínio de arquivo inferido (banco de dados, API, frontend, auth, config, compartilhado).

## 3. Detectar Relações de Dependência

Para cada par de fases (A, B), verifique os sinais de dependência:

### Detecção de Sobreposição de Arquivos
Se as fases A e B vão modificar arquivos no mesmo domínio ou nos mesmos arquivos específicos, uma deve rodar antes da outra. A fase que *fornece* a fundação roda primeiro.

### Detecção de Dependência Semântica
Leia o escopo/objetivo de cada fase em busca desses padrões:
- A fase B menciona consumir, usar ou chamar algo que a Fase A cria/implementa
- A fase B referencia uma "API", "schema", "modelo", "endpoint" ou "interface" que a Fase A constrói
- A fase B diz "após X estar concluído", "assim que X for construído", "usando o X da Fase N"
- A fase B estende ou modifica código que a Fase A estabelece

### Detecção de Fluxo de Dados
- A fase A cria estruturas de dados, schemas ou tipos → A fase B os consome ou transforma
- A fase A popula/migra o banco de dados → A fase B lê desse banco de dados
- A fase A expõe um contrato de API → A fase B implementa o cliente para esse contrato

## 4. Construir Tabela de Dependências

Gere uma tabela de sugestões de dependências:

```
Análise de Dependências de Fase
=========================

Fase N: <nome>
  Escopo: <escopo resumido>
  Provavelmente toca: <domínios de arquivo inferidos>

  Dependências sugeridas:
  → Depends on: <Fase M> — razão: <explicação de sobreposição/semântica/fluxo de dados>

  "Depends on" atual: <valor existente ou "(nenhum)">
```

Para pares de fases sem dependência detectada, declare: "Nenhuma dependência detectada entre a Fase X e a Fase Y."

## 5. Resumir Mudanças Sugeridas

Mostre um diff consolidado das mudanças propostas para `Depends on` no ROADMAP.md:

```
Atualizações sugeridas para ROADMAP.md:
  Fase 3: adicionar "Depends on: 1, 2"   (sobreposição de arquivo: schema do banco de dados)
  Fase 5: adicionar "Depends on: 3"      (semântico: usa API de auth da Fase 3)
  Fase 4: nenhuma mudança necessária     (escopo independente)
```

## 6. Confirmar e Aplicar

Pergunte ao usuário: "Aplicar estas sugestões de `Depends on` ao ROADMAP.md? (sim / não / editar)"

- **sim** — Escreva todas as entradas `Depends on` sugeridas no ROADMAP.md. Confirme cada escrita.
- **não** — Imprima as sugestões apenas como texto. O usuário atualiza manualmente.
- **editar** — Apresente cada sugestão individualmente com sim/não/pular por sugestão.

Ao escrever no ROADMAP.md:
- Localize a entrada da fase e adicione ou atualize o campo `Depends on:`
- Preserve todo o outro conteúdo da fase sem alterações
- Não reordene as fases

Após aplicar: "ROADMAP.md atualizado. Execute `/gsd-manager` para executar as fases na ordem correta."

</process>
