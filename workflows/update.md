<purpose>
Verificar atualizações do GSD via npm, exibir o changelog entre as versões instalada e mais recente, obter confirmação do usuário e executar uma instalação limpa com limpeza de cache.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo `execution_context` do prompt de invocação antes de começar.
</required_reading>

<process>

<step name="get_installed_version">
Detectar se o GSD está instalado localmente ou globalmente, verificando ambos os locais e validando a integridade da instalação.

Primeiro, derive `PREFERRED_CONFIG_DIR` e `PREFERRED_RUNTIME` a partir do caminho `execution_context` do prompt de invocação:
- Se o caminho contiver `/get-shit-done/workflows/update.md`, remova esse sufixo e armazene o restante como `PREFERRED_CONFIG_DIR`
- Caminho contém `/.codex/` -> `codex`
- Caminho contém `/.gemini/` -> `gemini`
- Caminho contém `/.config/kilo/` ou `/.kilo/`, ou `PREFERRED_CONFIG_DIR` contém `kilo.json` / `kilo.jsonc` -> `kilo`
- Caminho contém `/.config/opencode/` ou `/.opencode/`, ou `PREFERRED_CONFIG_DIR` contém `opencode.json` / `opencode.jsonc` -> `opencode`
- Caso contrário -> `claude`

Use `PREFERRED_CONFIG_DIR` quando disponível para que instalações com `--config-dir` personalizado sejam verificadas antes dos locais padrão.
Use `PREFERRED_RUNTIME` como o primeiro runtime verificado para que `/gsd-update` tenha como alvo o runtime que o invocou.

A precedência de configuração do Kilo deve corresponder ao instalador: `KILO_CONFIG_DIR` -> `dirname(KILO_CONFIG)` -> `XDG_CONFIG_HOME/kilo` -> `~/.config/kilo`.

```bash
expand_home() {
  case "$1" in
    "~/"*) printf '%s/%s\n' "$HOME" "${1#~/}" ;;
    *) printf '%s\n' "$1" ;;
  esac
}

# Runtime candidates: "<runtime>:<config-dir>" stored as an array.
# Using an array instead of a space-separated string ensures correct
# iteration in both bash and zsh (zsh does not word-split unquoted
# variables by default). Fixes #1173.
RUNTIME_DIRS=( "claude:.claude" "opencode:.config/opencode" "opencode:.opencode" "gemini:.gemini" "kilo:.config/kilo" "kilo:.kilo" "codex:.codex" )
ENV_RUNTIME_DIRS=()

# PREFERRED_CONFIG_DIR / PREFERRED_RUNTIME should be set from execution_context
# before running this block.
if [ -n "$PREFERRED_CONFIG_DIR" ]; then
  PREFERRED_CONFIG_DIR="$(expand_home "$PREFERRED_CONFIG_DIR")"
  if [ -z "$PREFERRED_RUNTIME" ]; then
    if [ -f "$PREFERRED_CONFIG_DIR/kilo.json" ] || [ -f "$PREFERRED_CONFIG_DIR/kilo.jsonc" ]; then
      PREFERRED_RUNTIME="kilo"
    elif [ -f "$PREFERRED_CONFIG_DIR/opencode.json" ] || [ -f "$PREFERRED_CONFIG_DIR/opencode.jsonc" ]; then
      PREFERRED_RUNTIME="opencode"
    elif [ -f "$PREFERRED_CONFIG_DIR/config.toml" ]; then
      PREFERRED_RUNTIME="codex"
    fi
  fi
fi

# If runtime is still unknown, infer from runtime env vars; fallback to claude.
if [ -z "$PREFERRED_RUNTIME" ]; then
  if [ -n "$CODEX_HOME" ]; then
    PREFERRED_RUNTIME="codex"
  elif [ -n "$GEMINI_CONFIG_DIR" ]; then
    PREFERRED_RUNTIME="gemini"
  elif [ -n "$KILO_CONFIG_DIR" ]; then
    PREFERRED_RUNTIME="kilo"
  elif [ -n "$KILO_CONFIG" ]; then
    PREFERRED_RUNTIME="kilo"
  elif [ -n "$OPENCODE_CONFIG_DIR" ] || [ -n "$OPENCODE_CONFIG" ]; then
    PREFERRED_RUNTIME="opencode"
  elif [ -n "$CLAUDE_CONFIG_DIR" ]; then
    PREFERRED_RUNTIME="claude"
  else
    PREFERRED_RUNTIME="claude"
  fi
fi

# If execution_context already points at an installed config dir, trust it first.
# This covers custom --config-dir installs that do not live under the default
# runtime directories.
if [ -n "$PREFERRED_CONFIG_DIR" ] && { [ -f "$PREFERRED_CONFIG_DIR/get-shit-done/VERSION" ] || [ -f "$PREFERRED_CONFIG_DIR/get-shit-done/workflows/update.md" ]; }; then
  INSTALL_SCOPE="GLOBAL"
  for dir in .claude .config/opencode .opencode .gemini .config/kilo .kilo .codex; do
    resolved_local="$(cd "./$dir" 2>/dev/null && pwd)"
    if [ -n "$resolved_local" ] && [ "$resolved_local" = "$PREFERRED_CONFIG_DIR" ]; then
      INSTALL_SCOPE="LOCAL"
      break
    fi
  done

  if [ -f "$PREFERRED_CONFIG_DIR/get-shit-done/VERSION" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$PREFERRED_CONFIG_DIR/get-shit-done/VERSION"; then
    INSTALLED_VERSION="$(cat "$PREFERRED_CONFIG_DIR/get-shit-done/VERSION")"
  else
    INSTALLED_VERSION="0.0.0"
  fi

  echo "$INSTALLED_VERSION"
  echo "$INSTALL_SCOPE"
  echo "${PREFERRED_RUNTIME:-claude}"
  exit 0
fi

# Absolute global candidates from env overrides (covers custom config dirs).
if [ -n "$CLAUDE_CONFIG_DIR" ]; then
  ENV_RUNTIME_DIRS+=( "claude:$(expand_home "$CLAUDE_CONFIG_DIR")" )
fi
if [ -n "$GEMINI_CONFIG_DIR" ]; then
  ENV_RUNTIME_DIRS+=( "gemini:$(expand_home "$GEMINI_CONFIG_DIR")" )
fi
if [ -n "$KILO_CONFIG_DIR" ]; then
  ENV_RUNTIME_DIRS+=( "kilo:$(expand_home "$KILO_CONFIG_DIR")" )
elif [ -n "$KILO_CONFIG" ]; then
  ENV_RUNTIME_DIRS+=( "kilo:$(dirname "$(expand_home "$KILO_CONFIG")")" )
elif [ -n "$XDG_CONFIG_HOME" ]; then
  ENV_RUNTIME_DIRS+=( "kilo:$(expand_home "$XDG_CONFIG_HOME")/kilo" )
fi
if [ -n "$OPENCODE_CONFIG_DIR" ]; then
  ENV_RUNTIME_DIRS+=( "opencode:$(expand_home "$OPENCODE_CONFIG_DIR")" )
elif [ -n "$OPENCODE_CONFIG" ]; then
  ENV_RUNTIME_DIRS+=( "opencode:$(dirname "$(expand_home "$OPENCODE_CONFIG")")" )
elif [ -n "$XDG_CONFIG_HOME" ]; then
  ENV_RUNTIME_DIRS+=( "opencode:$(expand_home "$XDG_CONFIG_HOME")/opencode" )
fi
if [ -n "$CODEX_HOME" ]; then
  ENV_RUNTIME_DIRS+=( "codex:$(expand_home "$CODEX_HOME")" )
fi

# Reorder entries so preferred runtime is checked first.
ORDERED_RUNTIME_DIRS=()
for entry in "${RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  if [ "$runtime" = "$PREFERRED_RUNTIME" ]; then
    ORDERED_RUNTIME_DIRS+=( "$entry" )
  fi
done
ORDERED_ENV_RUNTIME_DIRS=()
for entry in "${ENV_RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  if [ "$runtime" = "$PREFERRED_RUNTIME" ]; then
    ORDERED_ENV_RUNTIME_DIRS+=( "$entry" )
  fi
done
for entry in "${ENV_RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  if [ "$runtime" != "$PREFERRED_RUNTIME" ]; then
    ORDERED_ENV_RUNTIME_DIRS+=( "$entry" )
  fi
done
for entry in "${RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  if [ "$runtime" != "$PREFERRED_RUNTIME" ]; then
    ORDERED_RUNTIME_DIRS+=( "$entry" )
  fi
done

# Check local first (takes priority only if valid and distinct from global)
LOCAL_VERSION_FILE="" LOCAL_MARKER_FILE="" LOCAL_DIR="" LOCAL_RUNTIME=""
for entry in "${ORDERED_RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  dir="${entry#*:}"
  if [ -f "./$dir/get-shit-done/VERSION" ] || [ -f "./$dir/get-shit-done/workflows/update.md" ]; then
    LOCAL_RUNTIME="$runtime"
    LOCAL_VERSION_FILE="./$dir/get-shit-done/VERSION"
    LOCAL_MARKER_FILE="./$dir/get-shit-done/workflows/update.md"
    LOCAL_DIR="$(cd "./$dir" 2>/dev/null && pwd)"
    break
  fi
done

GLOBAL_VERSION_FILE="" GLOBAL_MARKER_FILE="" GLOBAL_DIR="" GLOBAL_RUNTIME=""
for entry in "${ORDERED_ENV_RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  dir="${entry#*:}"
  if [ -f "$dir/get-shit-done/VERSION" ] || [ -f "$dir/get-shit-done/workflows/update.md" ]; then
    GLOBAL_RUNTIME="$runtime"
    GLOBAL_VERSION_FILE="$dir/get-shit-done/VERSION"
    GLOBAL_MARKER_FILE="$dir/get-shit-done/workflows/update.md"
    GLOBAL_DIR="$(cd "$dir" 2>/dev/null && pwd)"
    break
  fi
done

if [ -z "$GLOBAL_RUNTIME" ]; then
  for entry in "${ORDERED_RUNTIME_DIRS[@]}"; do
    runtime="${entry%%:*}"
    dir="${entry#*:}"
    if [ -f "$HOME/$dir/get-shit-done/VERSION" ] || [ -f "$HOME/$dir/get-shit-done/workflows/update.md" ]; then
      GLOBAL_RUNTIME="$runtime"
      GLOBAL_VERSION_FILE="$HOME/$dir/get-shit-done/VERSION"
      GLOBAL_MARKER_FILE="$HOME/$dir/get-shit-done/workflows/update.md"
      GLOBAL_DIR="$(cd "$HOME/$dir" 2>/dev/null && pwd)"
      break
    fi
  done
fi

# Only treat as LOCAL if the resolved paths differ (prevents misdetection when CWD=$HOME)
IS_LOCAL=false
if [ -n "$LOCAL_VERSION_FILE" ] && [ -f "$LOCAL_VERSION_FILE" ] && [ -f "$LOCAL_MARKER_FILE" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$LOCAL_VERSION_FILE"; then
  if [ -z "$GLOBAL_DIR" ] || [ "$LOCAL_DIR" != "$GLOBAL_DIR" ]; then
    IS_LOCAL=true
  fi
fi

if [ "$IS_LOCAL" = true ]; then
  INSTALLED_VERSION="$(cat "$LOCAL_VERSION_FILE")"
  INSTALL_SCOPE="LOCAL"
  TARGET_RUNTIME="$LOCAL_RUNTIME"
elif [ -n "$GLOBAL_VERSION_FILE" ] && [ -f "$GLOBAL_VERSION_FILE" ] && [ -f "$GLOBAL_MARKER_FILE" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$GLOBAL_VERSION_FILE"; then
  INSTALLED_VERSION="$(cat "$GLOBAL_VERSION_FILE")"
  INSTALL_SCOPE="GLOBAL"
  TARGET_RUNTIME="$GLOBAL_RUNTIME"
elif [ -n "$LOCAL_RUNTIME" ] && [ -f "$LOCAL_MARKER_FILE" ]; then
  # Runtime detected but VERSION missing/corrupt: treat as unknown version, keep runtime target
  INSTALLED_VERSION="0.0.0"
  INSTALL_SCOPE="LOCAL"
  TARGET_RUNTIME="$LOCAL_RUNTIME"
elif [ -n "$GLOBAL_RUNTIME" ] && [ -f "$GLOBAL_MARKER_FILE" ]; then
  INSTALLED_VERSION="0.0.0"
  INSTALL_SCOPE="GLOBAL"
  TARGET_RUNTIME="$GLOBAL_RUNTIME"
else
  INSTALLED_VERSION="0.0.0"
  INSTALL_SCOPE="UNKNOWN"
  TARGET_RUNTIME="claude"
fi

echo "$INSTALLED_VERSION"
echo "$INSTALL_SCOPE"
echo "$TARGET_RUNTIME"
```

Interpretar a saída:
- Linha 1 = versão instalada (`0.0.0` significa versão desconhecida)
- Linha 2 = escopo da instalação (`LOCAL`, `GLOBAL` ou `UNKNOWN`)
- Linha 3 = runtime alvo (`claude`, `opencode`, `gemini`, `kilo` ou `codex`)
- Se o escopo for `UNKNOWN`, prosseguir para o passo de instalação usando o fallback `--claude --global`.

Se múltiplas instalações de runtime forem detectadas e o runtime de invocação não puder ser determinado a partir do execution_context, perguntar ao usuário qual runtime atualizar antes de executar a instalação.

**Se o arquivo VERSION estiver ausente:**
```
## GSD Update

**Versão instalada:** Desconhecida

Sua instalação não inclui rastreamento de versão.

Executando instalação limpa...
```

Prosseguir para o passo de instalação (tratar como versão 0.0.0 para comparação).
</step>

<step name="check_latest_version">
Verificar no npm a versão mais recente:

```bash
npm view get-shit-done-cc version 2>/dev/null
```

**Se a verificação do npm falhar:**
```
Não foi possível verificar atualizações (sem conexão ou npm indisponível).

Para atualizar manualmente: `npx get-shit-done-cc --global`
```

Encerrar.
</step>

<step name="compare_versions">
Comparar a versão instalada com a mais recente:

**Se instalada == mais recente:**
```
## GSD Update

**Instalada:** X.Y.Z
**Mais recente:** X.Y.Z

Você já está na versão mais recente.
```

Encerrar.

**Se instalada > mais recente:**
```
## GSD Update

**Instalada:** X.Y.Z
**Mais recente:** A.B.C

Você está à frente da versão mais recente — isso parece uma instalação de desenvolvimento.

Se você vir um aviso "⚠ dev install — re-run installer to sync hooks" na
sua statusline, seus arquivos de hook são mais antigos que seu arquivo VERSION. Corrija
executando novamente o instalador local a partir de seu branch de desenvolvimento:

    node bin/install.js --global --claude

Executar /gsd-update instalaria a versão do npm (A.B.C) e rebaixaria
sua versão de desenvolvimento — NÃO use isso para resolver este aviso.
```

Encerrar.
</step>

<step name="show_changes_and_confirm">
**Se houver atualização disponível**, buscar e exibir as novidades ANTES de atualizar:

1. Buscar o changelog da URL raw do GitHub
2. Extrair entradas entre as versões instalada e mais recente
3. Exibir prévia e solicitar confirmação:

```
## Atualização GSD Disponível

**Instalada:** 1.5.10
**Mais recente:** 1.5.15

### Novidades
────────────────────────────────────────────────────────────

## [1.5.15] - 2026-01-20

### Added
- Feature X

## [1.5.14] - 2026-01-18

### Fixed
- Bug fix Y

────────────────────────────────────────────────────────────

⚠️  **Atenção:** O instalador realiza uma instalação limpa das pastas do GSD:
- `commands/gsd/` será apagada e substituída
- `get-shit-done/` será apagada e substituída
- Arquivos `agents/gsd-*` serão substituídos

(Os caminhos são relativos ao local de instalação do runtime detectado:
global: `$HOME/.claude/`, `~/.config/opencode/`, `~/.opencode/`, `~/.gemini/`, `~/.config/kilo/`, ou `~/.codex/`
local: `./.claude/`, `./.config/opencode/`, `./.opencode/`, `./.gemini/`, `./.kilo/`, ou `./.codex/`)

Seus arquivos personalizados em outros locais são preservados:
- Comandos personalizados fora de `commands/gsd/` ✓
- Agentes personalizados sem o prefixo `gsd-` ✓
- Hooks personalizados ✓
- Seus arquivos CLAUDE.md ✓

Se você modificou algum arquivo do GSD diretamente, eles serão automaticamente copiados para `gsd-local-patches/` e poderão ser reaplicados com `/gsd-reapply-patches` após a atualização.
```


**Modo texto (`workflow.text_mode: true` na configuração ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU se `text_mode` do JSON de inicialização for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário que digite o número de sua escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Usar AskUserQuestion:
- Pergunta: "Deseja prosseguir com a atualização?"
- Opções:
  - "Sim, atualizar agora"
  - "Não, cancelar"

**Se o usuário cancelar:** Encerrar.
</step>

<step name="run_update">
Executar a atualização usando o tipo de instalação detectado no passo 1:

Construir o flag de runtime a partir do passo 1:
```bash
RUNTIME_FLAG="--$TARGET_RUNTIME"
```

**Se instalação LOCAL:**
```bash
npx -y get-shit-done-cc@latest "$RUNTIME_FLAG" --local
```

**Se instalação GLOBAL:**
```bash
npx -y get-shit-done-cc@latest "$RUNTIME_FLAG" --global
```

**Se instalação UNKNOWN:**
```bash
npx -y get-shit-done-cc@latest --claude --global
```

Capturar a saída. Se a instalação falhar, exibir o erro e encerrar.

Limpar o cache de atualização para que o indicador na statusline desapareça:

```bash
expand_home() {
  case "$1" in
    "~/"*) printf '%s/%s\n' "$HOME" "${1#~/}" ;;
    *) printf '%s\n' "$1" ;;
  esac
}

# Clear update cache across preferred, env-derived, and default runtime directories
CACHE_DIRS=()
if [ -n "$PREFERRED_CONFIG_DIR" ]; then
  CACHE_DIRS+=( "$(expand_home "$PREFERRED_CONFIG_DIR")" )
fi
if [ -n "$CLAUDE_CONFIG_DIR" ]; then
  CACHE_DIRS+=( "$(expand_home "$CLAUDE_CONFIG_DIR")" )
fi
if [ -n "$GEMINI_CONFIG_DIR" ]; then
  CACHE_DIRS+=( "$(expand_home "$GEMINI_CONFIG_DIR")" )
fi
if [ -n "$KILO_CONFIG_DIR" ]; then
  CACHE_DIRS+=( "$(expand_home "$KILO_CONFIG_DIR")" )
elif [ -n "$KILO_CONFIG" ]; then
  CACHE_DIRS+=( "$(dirname "$(expand_home "$KILO_CONFIG")")" )
elif [ -n "$XDG_CONFIG_HOME" ]; then
  CACHE_DIRS+=( "$(expand_home "$XDG_CONFIG_HOME")/kilo" )
fi
if [ -n "$OPENCODE_CONFIG_DIR" ]; then
  CACHE_DIRS+=( "$(expand_home "$OPENCODE_CONFIG_DIR")" )
elif [ -n "$OPENCODE_CONFIG" ]; then
  CACHE_DIRS+=( "$(dirname "$(expand_home "$OPENCODE_CONFIG")")" )
elif [ -n "$XDG_CONFIG_HOME" ]; then
  CACHE_DIRS+=( "$(expand_home "$XDG_CONFIG_HOME")/opencode" )
fi
if [ -n "$CODEX_HOME" ]; then
  CACHE_DIRS+=( "$(expand_home "$CODEX_HOME")" )
fi

for dir in "${CACHE_DIRS[@]}"; do
  if [ -n "$dir" ]; then
    rm -f "$dir/cache/gsd-update-check.json"
  fi
done

for dir in .claude .config/opencode .opencode .gemini .config/kilo .kilo .codex; do
  rm -f "./$dir/cache/gsd-update-check.json"
  rm -f "$HOME/$dir/cache/gsd-update-check.json"
done
```

O hook SessionStart (`gsd-check-update.js`) grava no diretório de cache do runtime detectado, portanto os caminhos preferidos/derivados de variáveis de ambiente e os caminhos padrão devem todos ser limpos para evitar indicadores de atualização desatualizados.
</step>

<step name="display_result">
Formatar a mensagem de conclusão (o changelog já foi exibido no passo de confirmação):

```
╔═══════════════════════════════════════════════════════════╗
║  GSD Atualizado: v1.5.10 → v1.5.15                        ║
╚═══════════════════════════════════════════════════════════╝

⚠️  Reinicie seu runtime para carregar os novos comandos.

[Ver changelog completo](https://github.com/gsd-build/get-shit-done/blob/main/CHANGELOG.md)
```
</step>


<step name="check_local_patches">
Após a conclusão da atualização, verificar se o instalador detectou e fez backup de arquivos modificados localmente:

Verificar a existência de gsd-local-patches/backup-meta.json no diretório de configuração.

**Se patches forem encontrados:**

```
Patches locais foram salvos antes da atualização.
Execute /gsd-reapply-patches para mesclar suas modificações na nova versão.
```

**Se não houver patches:** Continuar normalmente.
</step>
</process>

<success_criteria>
- [ ] Versão instalada lida corretamente
- [ ] Versão mais recente verificada via npm
- [ ] Atualização ignorada se já estiver na versão atual
- [ ] Changelog buscado e exibido ANTES da atualização
- [ ] Aviso de instalação limpa exibido
- [ ] Confirmação do usuário obtida
- [ ] Atualização executada com sucesso
- [ ] Lembrete de reinicialização exibido
</success_criteria>
