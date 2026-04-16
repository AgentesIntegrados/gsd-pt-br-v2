# Perfis de Modelo

Os perfis de modelo controlam qual modelo Claude cada agente GSD usa. Isso permite equilibrar qualidade versus gasto de tokens, ou herdar o modelo de sessão atualmente selecionado.

## Definições de Perfil

| Agente | `quality` | `balanced` | `budget` | `adaptive` | `inherit` |
|--------|-----------|------------|----------|------------|-----------|
| gsd-planner | opus | opus | sonnet | opus | inherit |
| gsd-roadmapper | opus | sonnet | sonnet | sonnet | inherit |
| gsd-executor | opus | sonnet | sonnet | sonnet | inherit |
| gsd-phase-researcher | opus | sonnet | haiku | sonnet | inherit |
| gsd-project-researcher | opus | sonnet | haiku | sonnet | inherit |
| gsd-research-synthesizer | sonnet | sonnet | haiku | haiku | inherit |
| gsd-debugger | opus | sonnet | sonnet | opus | inherit |
| gsd-codebase-mapper | sonnet | haiku | haiku | haiku | inherit |
| gsd-verifier | sonnet | sonnet | haiku | sonnet | inherit |
| gsd-plan-checker | sonnet | sonnet | haiku | haiku | inherit |
| gsd-integration-checker | sonnet | sonnet | haiku | haiku | inherit |
| gsd-nyquist-auditor | sonnet | sonnet | haiku | haiku | inherit |

## Filosofia dos Perfis

**quality** - Máximo poder de raciocínio
- Opus para todos os agentes de tomada de decisão
- Sonnet para verificação somente leitura
- Use quando: quota disponível, trabalho crítico de arquitetura

**balanced** (padrão) - Alocação inteligente
- Opus apenas para planejamento (onde decisões de arquitetura acontecem)
- Sonnet para execução e pesquisa (segue instruções explícitas)
- Sonnet para verificação (precisa de raciocínio, não apenas correspondência de padrões)
- Use quando: desenvolvimento normal, bom equilíbrio entre qualidade e custo

**budget** - Uso mínimo de Opus
- Sonnet para qualquer coisa que escreva código
- Haiku para pesquisa e verificação
- Use quando: economizando quota, trabalho de alto volume, fases menos críticas

**adaptive** — Otimização de custo baseada em função
- Opus para planejamento e depuração (onde a qualidade do raciocínio tem maior impacto)
- Sonnet para execução, pesquisa e verificação (segue instruções explícitas)
- Haiku para mapeamento, verificação e auditoria (alto volume, saída estruturada)
- Use quando: otimizando custo sem sacrificar a qualidade do plano, desenvolvimento solo em tiers de API pagos

**inherit** - Seguir o modelo de sessão atual
- Todos os agentes resolvem para `inherit`
- Melhor quando você alterna modelos interativamente (por exemplo OpenCode ou Kilo `/model`)
- **Obrigatório ao usar provedores não-Anthropic** (OpenRouter, modelos locais, etc.) — caso contrário, o GSD pode chamar modelos Anthropic diretamente, incorrendo em custos inesperados
- Use quando: você quer que o GSD siga o modelo de runtime atualmente selecionado

## Usando Runtimes Não-Claude (Codex, OpenCode, Gemini CLI, Kilo)

Quando instalado para um runtime não-Claude, o instalador do GSD define `resolve_model_ids: "omit"` em `~/.gsd/defaults.json`. Isso retorna um parâmetro de modelo vazio para todos os agentes, fazendo com que cada agente use o modelo padrão do runtime. Nenhuma configuração manual é necessária.

Para atribuir modelos diferentes a agentes diferentes, adicione `model_overrides` com IDs de modelo que seu runtime reconhece:

```json
{
  "resolve_model_ids": "omit",
  "model_overrides": {
    "gsd-planner": "o3",
    "gsd-executor": "o4-mini",
    "gsd-debugger": "o3",
    "gsd-codebase-mapper": "o4-mini"
  }
}
```

A mesma lógica de camadas se aplica: modelos mais fortes para planejamento e depuração, modelos mais baratos para execução e mapeamento.

## Usando Claude Code com Provedores Não-Anthropic (OpenRouter, Local)

Se você está usando Claude Code com OpenRouter, um modelo local, ou qualquer provedor não-Anthropic, defina o perfil `inherit` para evitar que o GSD chame modelos Anthropic para subagentes:

```bash
# Via comando de configurações
/gsd-settings
# → Selecione "Inherit" para perfil de modelo

# Ou manualmente em .planning/config.json
{
  "model_profile": "inherit"
}
```

Sem `inherit`, o perfil `balanced` padrão do GSD gera modelos Anthropic específicos (`opus`, `sonnet`, `haiku`) para cada tipo de agente, o que pode resultar em custos adicionais de API através do seu provedor não-Anthropic.

## Lógica de Resolução

Os orquestradores resolvem o modelo antes de fazer o spawn:

```
1. Ler .planning/config.json
2. Verificar model_overrides para override específico do agente
3. Se não houver override, consultar o agente na tabela de perfis
4. Passar parâmetro de modelo para a chamada de Task
```

## Overrides por Agente

Substitua agentes específicos sem alterar o perfil inteiro:

```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "gsd-executor": "opus",
    "gsd-planner": "haiku"
  }
}
```

Overrides têm precedência sobre o perfil. Valores válidos: `opus`, `sonnet`, `haiku`, `inherit`, ou qualquer ID de modelo totalmente qualificado (ex.: `"o3"`, `"openai/o3"`, `"google/gemini-2.5-pro"`).

## Alternando Perfis

Em tempo de execução: `/gsd-set-profile <profile>`

Padrão por projeto: Defina em `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Justificativa do Design

**Por que Opus para gsd-planner?**
O planejamento envolve decisões de arquitetura, decomposição de objetivos e design de tarefas. É aqui que a qualidade do modelo tem o maior impacto.

**Por que Sonnet para gsd-executor?**
Os executores seguem instruções explícitas do PLAN.md. O plano já contém o raciocínio; a execução é implementação.

**Por que Sonnet (não Haiku) para verificadores em balanced?**
A verificação requer raciocínio orientado a objetivos — verificar se o código *entrega* o que a fase prometeu, não apenas correspondência de padrões. O Sonnet lida bem com isso; o Haiku pode perder lacunas sutis.

**Por que Haiku para gsd-codebase-mapper?**
Exploração somente leitura e extração de padrões. Nenhum raciocínio necessário, apenas saída estruturada a partir do conteúdo dos arquivos.

**Por que `inherit` em vez de passar `opus` diretamente?**
O alias `"opus"` do Claude Code mapeia para uma versão específica do modelo. Organizações podem bloquear versões mais antigas do opus enquanto permitem as mais novas. O GSD retorna `"inherit"` para agentes de nível opus, fazendo com que usem qualquer versão do opus que o usuário configurou em sua sessão. Isso evita conflitos de versão e fallbacks silenciosos para Sonnet.

**Por que o perfil `inherit`?**
Alguns runtimes (incluindo OpenCode) permitem que usuários alternem modelos em tempo de execução (`/model`). O perfil `inherit` mantém todos os subagentes do GSD alinhados com essa seleção ao vivo.
