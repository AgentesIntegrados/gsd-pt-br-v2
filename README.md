# GSD pt-BR — Tradução do GSD para Português do Brasil

Plugin para o [Claude Code](https://claude.ai/code) que adiciona comandos GSD (Get Shit Done) traduzidos para português brasileiro.

## O que é

O GSD é um sistema de workflow para desenvolvimento solo com Claude Code. Este plugin adiciona **74 comandos em pt-BR** como alternativa aos comandos originais em inglês — ambos funcionam lado a lado.

Exemplos:
- `/gsd-novo-projeto` (pt-BR) = `/gsd-new-project` (original)
- `/gsd-executar-fase` = `/gsd-execute-phase`
- `/gsd-planejar-fase` = `/gsd-plan-phase`
- `/gsd-ajuda` = `/gsd-help`

## Instalação

### 1. Instalar o GSD (pré-requisito)

```bash
npx get-shit-done-cc@latest
```

### 2. Adicionar o marketplace no settings.json

Abra `~/.claude/settings.json` e adicione:

```json
{
  "extraKnownMarketplaces": {
    "gsd-ptbr": {
      "source": {
        "source": "settings",
        "name": "gsd-ptbr",
        "plugins": [
          {
            "name": "gsd-pt-br",
            "source": {
              "source": "github",
              "repo": "AgentesIntegrados/gsd-pt-br-v2"
            }
          }
        ]
      }
    }
  },
  "enabledPlugins": {
    "gsd-pt-br@gsd-ptbr": true
  }
}
```

### 3. Instalar o plugin

```bash
claude plugin install gsd-pt-br@gsd-ptbr
```

### 4. Reiniciar o Claude Code

Feche e abra o Claude Code para carregar os comandos traduzidos.

## Comandos disponíveis

| Comando pt-BR | Equivalente original | Descrição |
|---|---|---|
| `/gsd-novo-projeto` | `/gsd-new-project` | Inicializar novo projeto |
| `/gsd-planejar-fase` | `/gsd-plan-phase` | Criar plano detalhado da fase |
| `/gsd-executar-fase` | `/gsd-execute-phase` | Executar fase com paralelização |
| `/gsd-discutir-fase` | `/gsd-discuss-phase` | Coletar contexto antes de planejar |
| `/gsd-progresso` | `/gsd-progress` | Verificar progresso do projeto |
| `/gsd-proximo` | `/gsd-next` | Avançar para o próximo passo |
| `/gsd-ajuda` | `/gsd-help` | Referência completa de comandos |
| `/gsd-autonomo` | `/gsd-autonomous` | Executar fases automaticamente |
| `/gsd-rapido` | `/gsd-fast` | Executar tarefa trivial inline |
| `/gsd-tarefa-rapida` | `/gsd-quick` | Tarefa rápida com garantias GSD |
| `/gsd-depurar` | `/gsd-debug` | Depuração sistemática |
| `/gsd-revisao-codigo` | `/gsd-code-review` | Revisão de código da fase |
| `/gsd-entregar` | `/gsd-ship` | Criar PR e preparar merge |
| `/gsd-estatisticas` | `/gsd-stats` | Estatísticas do projeto |
| `/gsd-atualizar-traducao` | — | Sincronizar traduções após update |

[Ver lista completa de 74 comandos na pasta `skills/`]

## Atualização

Quando o GSD original atualizar (via `/gsd-update`), rode:

```
/gsd-atualizar-traducao
```

Isso detecta o delta (arquivos novos ou alterados) e traduz apenas o necessário.

## Estrutura do plugin

```
gsd-pt-br/
  plugin.json          — Manifesto do plugin
  README.md            — Este arquivo
  skills/              — 74 skills traduzidas (comandos slash)
  workflows/           — 71 workflows traduzidos (lógica de execução)
  references/          — 41 referências traduzidas (documentação)
  templates/           — 43+ templates traduzidos (modelos de artefatos)
```

## Compatibilidade

- Claude Code 2.1+
- GSD v1.34.0+
- Os comandos pt-BR coexistem com os originais em inglês

## Licença

MIT
