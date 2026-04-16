<purpose>
Avaliação leve da base de código. Instancia um único agente gsd-codebase-mapper para uma área de foco,
produzindo documentos direcionados em `.planning/codebase/`.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt que invocou este fluxo antes de começar.
</required_reading>

<available_agent_types>
Tipos de subagentes GSD válidos (use os nomes exatos — não use 'general-purpose' como alternativa):
- gsd-codebase-mapper — Mapeia estrutura do projeto e dependências
</available_agent_types>

<process>

## Mapeamento de Foco para Documento

| Foco | Documentos Produzidos |
|------|-----------------------|
| `tech` | STACK.md, INTEGRATIONS.md |
| `arch` | ARCHITECTURE.md, STRUCTURE.md |
| `quality` | CONVENTIONS.md, TESTING.md |
| `concerns` | CONCERNS.md |
| `tech+arch` | STACK.md, INTEGRATIONS.md, ARCHITECTURE.md, STRUCTURE.md |

## Passo 1: Analisar argumentos e resolver foco

Analisar a entrada do usuário para `--focus <area>`. Padrão para `tech+arch` se não especificado.

Validar que o foco é um dos seguintes: `tech`, `arch`, `quality`, `concerns`, `tech+arch`.

Se inválido:
```
Área de foco desconhecida: "{input}". Opções válidas: tech, arch, quality, concerns, tech+arch
```
Sair.

## Passo 2: Verificar documentos existentes

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init map-codebase 2>/dev/null || echo "{}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Verificar quais documentos seriam produzidos para o foco selecionado (da tabela de mapeamento acima).

Para cada documento alvo, verificar se já existe em `.planning/codebase/`:
```bash
ls -la .planning/codebase/{DOCUMENT}.md 2>/dev/null
```

Se algum existir, mostrar as datas de modificação e perguntar:
```
Documentos existentes encontrados:
  - STACK.md (modificado em 2026-04-03)
  - INTEGRATIONS.md (modificado em 2026-04-01)

Sobrescrever com nova varredura? [s/N]
```

Se o usuário disser não, sair.

## Passo 3: Criar diretório de saída

```bash
mkdir -p .planning/codebase
```

## Passo 4: Instanciar agente mapper

Instanciar um único agente `gsd-codebase-mapper` com a área de foco selecionada:

```
Task(
  prompt="Scan this codebase with focus: {focus}. Write results to .planning/codebase/. Produce only: {document_list}",
  subagent_type="gsd-codebase-mapper",
  model="{resolved_model}"
)
```

## Passo 5: Relatório

```
## Varredura Concluída

**Foco:** {focus}
**Documentos produzidos:**
{lista de documentos escritos com contagem de linhas}

Use `/gsd-map-codebase` para uma varredura paralela abrangente de 4 áreas.
```

</process>

<success_criteria>
- [ ] Área de foco corretamente analisada (padrão: tech+arch)
- [ ] Documentos existentes detectados com datas de modificação exibidas
- [ ] Usuário consultado antes de sobrescrever
- [ ] Único agente mapper instanciado com foco correto
- [ ] Documentos de saída escritos em .planning/codebase/
</success_criteria>
