# Análise de Argumentos de Fase

Analise e normalize argumentos de fase para comandos que operam em fases.

## Extração

A partir de `$ARGUMENTS`:
- Extraia o número da fase (primeiro argumento numérico)
- Extraia flags (prefixadas com `--`)
- O texto restante é a descrição (para comandos insert/add)

## Usando gsd-tools

O comando `find-phase` lida com normalização e validação em uma única etapa:

```bash
PHASE_INFO=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase "${PHASE}")
```

Retorna JSON com:
- `found`: true/false
- `directory`: Caminho completo para o diretório da fase
- `phase_number`: Número normalizado (ex.: "06", "06.1")
- `phase_name`: Parte do nome (ex.: "foundation")
- `plans`: Array de arquivos PLAN.md
- `summaries`: Array de arquivos SUMMARY.md

## Normalização Manual (Legado)

Preencha fases inteiras com zeros à esquerda até 2 dígitos. Preserve sufixos decimais.

```bash
# Normalizar número de fase
if [[ "$PHASE" =~ ^[0-9]+$ ]]; then
  # Inteiro: 8 → 08
  PHASE=$(printf "%02d" "$PHASE")
elif [[ "$PHASE" =~ ^([0-9]+)\.([0-9]+)$ ]]; then
  # Decimal: 2.1 → 02.1
  PHASE=$(printf "%02d.%s" "${BASH_REMATCH[1]}" "${BASH_REMATCH[2]}")
fi
```

## Validação

Use `roadmap get-phase` para validar se a fase existe:

```bash
PHASE_CHECK=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" --pick found)
if [ "$PHASE_CHECK" = "false" ]; then
  echo "ERROR: Phase ${PHASE} not found in roadmap"
  exit 1
fi
```

## Busca de Diretório

Use `find-phase` para busca de diretório:

```bash
PHASE_DIR=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" find-phase "${PHASE}" --raw)
```
