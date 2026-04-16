---
name: gsd-reaplicar-patches
description: "Reaplicar modificações locais após uma atualização do GSD"
---


<purpose>
Após uma atualização do GSD apagar e reinstalar arquivos, este comando mescla as modificações locais salvas anteriormente pelo usuário na nova versão. Utiliza comparação em três vias (baseline original, backup modificado pelo usuário, versão recém-instalada) para distinguir de forma confiável as personalizações do usuário das mudanças de versão.

**Invariante crítico:** Todo arquivo em `gsd-local-patches/` foi salvo em backup porque a comparação de hash do instalador detectou que ele foi modificado. O fluxo NUNCA deve concluir "sem conteúdo personalizado" para qualquer arquivo salvo — isso é uma contradição lógica. Em caso de dúvida, classifique como CONFLITO exigindo revisão do usuário, não como IGNORADO.
</purpose>

<process>

## Etapa 1: Detectar patches salvos

Verificar o diretório de patches locais:

```bash
expand_home() {
  case "$1" in
    "~/"*) printf '%s/%s\n' "$HOME" "${1#~/}" ;;
    *) printf '%s\n' "$1" ;;
  esac
}

PATCHES_DIR=""

# Env overrides first — covers custom config directories used with --config-dir
if [ -n "$KILO_CONFIG_DIR" ]; then
  candidate="$(expand_home "$KILO_CONFIG_DIR")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
elif [ -n "$KILO_CONFIG" ]; then
  candidate="$(dirname "$(expand_home "$KILO_CONFIG")")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
elif [ -n "$XDG_CONFIG_HOME" ]; then
  candidate="$(expand_home "$XDG_CONFIG_HOME")/kilo/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
fi

if [ -z "$PATCHES_DIR" ] && [ -n "$OPENCODE_CONFIG_DIR" ]; then
  candidate="$(expand_home "$OPENCODE_CONFIG_DIR")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
elif [ -z "$PATCHES_DIR" ] && [ -n "$OPENCODE_CONFIG" ]; then
  candidate="$(dirname "$(expand_home "$OPENCODE_CONFIG")")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
elif [ -z "$PATCHES_DIR" ] && [ -n "$XDG_CONFIG_HOME" ]; then
  candidate="$(expand_home "$XDG_CONFIG_HOME")/opencode/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
fi

if [ -z "$PATCHES_DIR" ] && [ -n "$GEMINI_CONFIG_DIR" ]; then
  candidate="$(expand_home "$GEMINI_CONFIG_DIR")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
fi

if [ -z "$PATCHES_DIR" ] && [ -n "$CODEX_HOME" ]; then
  candidate="$(expand_home "$CODEX_HOME")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
fi

if [ -z "$PATCHES_DIR" ] && [ -n "$CLAUDE_CONFIG_DIR" ]; then
  candidate="$(expand_home "$CLAUDE_CONFIG_DIR")/gsd-local-patches"
  if [ -d "$candidate" ]; then
    PATCHES_DIR="$candidate"
  fi
fi

# Global install — detect runtime config directory defaults
if [ -z "$PATCHES_DIR" ]; then
  if [ -d "$HOME/.config/kilo/gsd-local-patches" ]; then
    PATCHES_DIR="$HOME/.config/kilo/gsd-local-patches"
  elif [ -d "$HOME/.config/opencode/gsd-local-patches" ]; then
    PATCHES_DIR="$HOME/.config/opencode/gsd-local-patches"
  elif [ -d "$HOME/.opencode/gsd-local-patches" ]; then
    PATCHES_DIR="$HOME/.opencode/gsd-local-patches"
  elif [ -d "$HOME/.gemini/gsd-local-patches" ]; then
    PATCHES_DIR="$HOME/.gemini/gsd-local-patches"
  elif [ -d "$HOME/.codex/gsd-local-patches" ]; then
    PATCHES_DIR="$HOME/.codex/gsd-local-patches"
  else
    PATCHES_DIR="$HOME/.claude/gsd-local-patches"
  fi
fi
# Local install fallback — check all runtime directories
if [ ! -d "$PATCHES_DIR" ]; then
  for dir in .config/kilo .kilo .config/opencode .opencode .gemini .codex .claude; do
    if [ -d "./$dir/gsd-local-patches" ]; then
      PATCHES_DIR="./$dir/gsd-local-patches"
      break
    fi
  done
fi
```

Ler `backup-meta.json` do diretório de patches.

**Se nenhum patch for encontrado:**
```
Nenhum patch local encontrado. Nada a reaplicar.

Os patches locais são salvos automaticamente quando você executa /gsd-update
após modificar qualquer arquivo de fluxo de trabalho, comando ou agente do GSD.
```
Encerrar.

## Etapa 2: Determinar baseline para comparação em três vias

A qualidade da mesclagem depende de ter um **baseline original** — a versão original não modificada de cada arquivo da versão do GSD anterior à atualização. Isso permite a comparação em três vias:
- **Baseline original** (arquivo GSD original antes de qualquer edição do usuário)
- **Versão do usuário** (salva em `gsd-local-patches/`)
- **Nova versão** (recém-instalada após a atualização)

Verificar fontes de baseline em ordem de prioridade:

### Opção A: Histórico git (mais confiável)
Se o diretório de configuração for um repositório git:
```bash
CONFIG_DIR=$(dirname "$PATCHES_DIR")
if git -C "$CONFIG_DIR" rev-parse --git-dir >/dev/null 2>&1; then
  HAS_GIT=true
fi
```
Quando `HAS_GIT=true`, usar `git log` para encontrar o commit onde o GSD foi originalmente instalado (antes das edições do usuário). Para cada arquivo, o baseline original pode ser extraído com:
```bash
git -C "$CONFIG_DIR" log --diff-filter=A --format="%H" -- "{file_path}"
```
Isso fornece o commit que adicionou o arquivo pela primeira vez (o commit de instalação). Extrair a versão original:
```bash
git -C "$CONFIG_DIR" show {install_commit}:{file_path}
```

### Opção B: Diretório de snapshot original
Verificar se um diretório `gsd-pristine/` existe ao lado de `gsd-local-patches/`:
```bash
PRISTINE_DIR="$CONFIG_DIR/gsd-pristine"
```
Se existir, o instalador salvou cópias originais no momento da instalação. Usar essas como baseline.

### Opção C: Sem baseline disponível (fallback em duas vias)
Se nem o histórico git nem os snapshots originais estiverem disponíveis, usar comparação em duas vias — mas com **heurísticas reforçadas** (ver Etapa 3).

## Etapa 3: Exibir resumo dos patches

```
## Patches Locais a Reaplicar

**Salvo de:** v{from_version}
**Versão atual:** {ler arquivo VERSION}
**Arquivos modificados:** {count}
**Estratégia de mesclagem:** {três vias (git) | três vias (pristine) | duas vias (aprimorada)}

| # | Arquivo | Status |
|---|---------|--------|
| 1 | {file_path} | Pendente |
| 2 | {file_path} | Pendente |
```

## Etapa 4: Mesclar cada arquivo

Para cada arquivo em `backup-meta.json`:

1. **Ler a versão salva** (cópia modificada pelo usuário de `gsd-local-patches/`)
2. **Ler a versão recém-instalada** (arquivo atual após atualização)
3. **Se disponível, ler o baseline original** (do histórico git ou `gsd-pristine/`)

### Mesclagem em três vias (quando o baseline estiver disponível)

Comparar as três versões para isolar as alterações:
- **Alterações do usuário** = diff(original → versão do usuário) — estas são as personalizações a preservar
- **Alterações upstream** = diff(original → nova versão) — estas são as atualizações de versão a aceitar

**Regras de mesclagem:**
- Seções alteradas apenas pelo usuário → aplicar versão do usuário
- Seções alteradas apenas pelo upstream → aceitar versão upstream
- Seções alteradas por ambos → marcar como CONFLITO, mostrar ambas, perguntar ao usuário
- Seções não alteradas por nenhum → usar nova versão (idêntica nas três)

### Mesclagem em duas vias (fallback quando não há baseline)

Quando nenhum baseline original estiver disponível, usar estas **heurísticas reforçadas**:

**REGRA CRÍTICA: Todo arquivo neste diretório de backup foi explicitamente detectado como modificado pela comparação SHA-256 do instalador. "Sem conteúdo personalizado" nunca é uma conclusão válida.**

Para cada arquivo:
a. Ler ambas as versões completamente
b. Identificar TODAS as diferenças e classificar cada uma como:
   - **Drift mecânico** — substituições de caminho (ex.: `/Users/xxx/.claude/` → `$HOME/.claude/`), adições de variáveis (`${GSD_WS}`, `${AGENT_SKILLS_*}`), adições de tratamento de erros (`|| true`)
   - **Personalização do usuário** — etapas/seções adicionadas, seções removidas, conteúdo reordenado, comportamento alterado, campos de frontmatter adicionados, instruções modificadas

c. **Se QUAISQUER diferenças permanecerem após filtrar o drift mecânico → essas são personalizações do usuário. Mesclar.**
d. **Se TODAS as diferenças parecerem ser drift mecânico → ainda assim marcar como CONFLITO.** A verificação de hash do instalador já provou que este arquivo foi modificado. Perguntar ao usuário: "Este arquivo parece ter apenas diferenças de caminho/variável. Havia personalizações intencionais?" NÃO ignorar silenciosamente.

### Mesclagem em duas vias aprimorada com git

Quando o diretório de configuração é um repositório git mas o commit de instalação original não pode ser encontrado, usar o histórico de commits para identificar as alterações do usuário:
```bash
# Find non-update commits that touched this file
git -C "$CONFIG_DIR" log --oneline --no-merges -- "{file_path}" | grep -v "gsd:update\|GSD update\|gsd-install"
```
Cada commit correspondente representa uma modificação intencional do usuário. Usar as mensagens e diffs de commits para entender o que foi alterado e por quê.

4. **Escrever resultado mesclado** no local instalado

### Verificação pós-mesclagem

Após escrever cada arquivo mesclado, verificar se as modificações do usuário sobreviveram à mesclagem:

1. **Verificação de contagem de linhas:** Contar linhas no backup e no resultado mesclado. Se o resultado mesclado tiver menos linhas do que o backup menos as remoções upstream esperadas, marcar para revisão.
2. **Verificação de presença de hunk:** Para cada seção adicionada pelo usuário identificada durante a análise de diff, pesquisar na saída mesclada pelo menos a primeira linha significativa (não em branco, não comentário) de cada adição. Linhas de assinatura ausentes indicam um hunk descartado.
3. **Relatar avisos inline** (sem bloquear):
   ```
   ⚠ Possível conteúdo descartado em {file_path}:
     - Hunk ausente perto da linha {N}: "{first_line_preview}..." ({line_count} linhas)
     - Backup disponível: {patches_dir}/{file_path}
   ```
4. **Produzir uma Tabela de Verificação de Hunks** — uma linha por hunk por arquivo. Esta tabela é **saída obrigatória** e deve ser produzida antes que a Etapa 5 possa prosseguir. Formato:

   | arquivo | hunk_id | linha_assinatura | contagem_linhas | verificado |
   |---------|---------|-----------------|-----------------|------------|
   | {file_path} | {N} | {primeira_linha_significativa} | {count} | sim |
   | {file_path} | {N} | {primeira_linha_significativa} | {count} | não |

   - `hunk_id` — inteiro sequencial por arquivo (1, 2, 3…)
   - `linha_assinatura` — primeira linha não em branco e não comentário da seção adicionada pelo usuário
   - `contagem_linhas` — total de linhas no hunk
   - `verificado` — `sim` se a linha_assinatura estiver presente na saída mesclada, `não` caso contrário

5. **Rastrear status de verificação** — adicionar ao relatório por arquivo: `Mesclado (verificado)` vs `Mesclado (⚠ {N} hunks podem estar ausentes)`

6. **Relatar status por arquivo:**
   - `Mesclado` — modificações do usuário aplicadas sem conflitos (mostrar resumo do que foi preservado)
   - `Conflito` — usuário revisou e escolheu resolução
   - `Incorporado` — a modificação do usuário já foi adotada pelo upstream (válido apenas quando o baseline original confirma isso)

**Nunca reportar `Ignorado — sem conteúdo personalizado`.** Se um arquivo está no backup, ele tem conteúdo personalizado.

## Etapa 5: Portão de Verificação de Hunks

Antes de prosseguir para a limpeza, avaliar a Tabela de Verificação de Hunks produzida na Etapa 4.

**Se a Tabela de Verificação de Hunks estiver ausente** (a Etapa 4 não a produziu), PARAR imediatamente e informar ao usuário:
```
ERRO: A Tabela de Verificação de Hunks está ausente. A verificação pós-mesclagem não foi concluída.
Execute /gsd-reapply-patches novamente para tentar com verificação completa.
```

**Se qualquer linha na Tabela de Verificação de Hunks mostrar `verificado: não`**, PARAR e informar ao usuário:
```
ERRO: {N} hunk(s) falhou na verificação — o conteúdo pode ter sido descartado durante a mesclagem.

Hunks não verificados:
  {arquivo} hunk {hunk_id}: linha de assinatura "{linha_assinatura}" não encontrada na saída mesclada

O backup foi preservado em: {patches_dir}/{arquivo}
Revise o arquivo mesclado manualmente e então:
  (a) Remescle o conteúdo ausente manualmente, ou
  (b) Restaure do backup: cp {patches_dir}/{arquivo} {installed_path}
```

Não prosseguir para a limpeza até que o usuário confirme ter resolvido todos os hunks não verificados.

**Somente quando todas as linhas mostrarem `verificado: sim`** (ou quando todos os arquivos tiverem zero hunks adicionados pelo usuário) a execução pode continuar para a Etapa 6.

## Etapa 6: Opção de limpeza

Perguntar ao usuário:
- "Manter os backups de patches para referência?" → preservar `gsd-local-patches/`
- "Limpar os backups de patches?" → remover o diretório `gsd-local-patches/`

## Etapa 7: Relatório

```
## Patches Reaplicados

| # | Arquivo | Resultado | Alterações do Usuário Preservadas |
|---|---------|-----------|-----------------------------------|
| 1 | {file_path} | Mesclado | Adicionada etapa X, seção Y modificada |
| 2 | {file_path} | Incorporado | Já no upstream v{version} |
| 3 | {file_path} | Conflito resolvido | Usuário escolheu: manter seção personalizada |

{count} arquivo(s) atualizado(s). Suas modificações locais estão ativas novamente.
```

</process>

<success_criteria>
- [ ] Todos os patches salvos processados — zero arquivos deixados sem tratamento
- [ ] Nenhum arquivo classificado como "sem conteúdo personalizado" ou "IGNORADO" — todo arquivo salvo é definitivamente modificado
- [ ] Mesclagem em três vias usada quando o baseline original estiver disponível (histórico git ou gsd-pristine/)
- [ ] Modificações do usuário identificadas e mescladas na nova versão
- [ ] Conflitos apresentados ao usuário com ambas as versões mostradas
- [ ] Status relatado para cada arquivo com resumo do que foi preservado
- [ ] Verificação pós-mesclagem verifica cada arquivo em busca de hunks descartados e avisa se o conteúdo parece ausente
</success_criteria>
