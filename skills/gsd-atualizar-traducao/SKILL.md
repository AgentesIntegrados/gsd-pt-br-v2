---
name: gsd-atualizar-traducao
description: "Sincronizar traduções pt-BR com a versão mais recente do GSD original. Detecta arquivos novos ou alterados e traduz apenas o delta."
argument-hint: "[--dry-run] [--force]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
---

<objective>
Sincronizar o plugin gsd-pt-br com a versão atual do GSD original após um `/gsd-update`.

Detecta o que mudou entre o GSD original e as traduções existentes, traduz apenas o delta (arquivos novos ou modificados), e comita no repositório do plugin.

**Quando usar:** Após rodar `/gsd-update` e o GSD subir de versão.
</objective>

<process>

## 1. Detectar versão e estado

```bash
GSD_VERSION=$(cat "$HOME/.claude/get-shit-done/VERSION")
echo "Versão atual do GSD: $GSD_VERSION"
```

Verificar se o repositório local do plugin existe:
```bash
PLUGIN_DIR="$HOME/.claude/plugins/gsd-pt-br"
ls "$PLUGIN_DIR/plugin.json" 2>/dev/null || echo "ERRO: Plugin não encontrado em $PLUGIN_DIR"
```

## 2. Mapear diferenças

### 2.1 Skills novas ou removidas

Comparar skills do GSD original com as do plugin. O mapeamento de nomes segue a convenção:
- `gsd-new-project` → `gsd-novo-projeto`
- `gsd-execute-phase` → `gsd-executar-fase`

```bash
# Listar skills originais
ls "$HOME/.claude/skills/" | grep "^gsd-" | sort > /tmp/gsd-original-skills.txt

# Listar skills traduzidas
ls "$PLUGIN_DIR/skills/" | sort > /tmp/gsd-translated-skills.txt
```

Identificar:
- **Skills novas no original** que não têm correspondente traduzido
- **Skills removidas do original** que ainda existem no plugin

### 2.2 Workflows novos ou alterados

```bash
# Contar workflows originais vs traduzidos
ORIG_WF=$(ls "$HOME/.claude/get-shit-done/workflows/"*.md 2>/dev/null | wc -l | tr -d ' ')
TRAD_WF=$(ls "$PLUGIN_DIR/workflows/"*.md 2>/dev/null | wc -l | tr -d ' ')
echo "Workflows: $ORIG_WF originais, $TRAD_WF traduzidos"
```

Para cada workflow original, verificar se existe tradução. Para os que existem, comparar tamanho — diferença > 20% indica mudança significativa.

### 2.3 Referências e templates

Mesma lógica: detectar novos e verificar mudanças por tamanho.

## 3. Apresentar relatório ao usuário

Mostrar o delta encontrado:

```
## Relatório de Sincronização — GSD v{versão}

### Skills
- Novas: {lista}
- Removidas: {lista}

### Workflows
- Novos: {lista}
- Possivelmente alterados: {lista}

### Referências
- Novas: {lista}

### Templates
- Novos: {lista}

Deseja prosseguir com a tradução? (sim/não)
```

Se `--dry-run` estiver nos argumentos, parar aqui e não traduzir.

## 4. Traduzir o delta

Para cada arquivo novo ou alterado, usar Agent para traduzir:

**Regras de tradução (mesmas do projeto original):**
- Traduzir: texto legível, descrições, headers, notas, mensagens de erro/sucesso
- NÃO traduzir: blocos de código, variáveis, paths, comandos, tags XML, chaves JSON, nomes de tools/agents
- Português brasileiro natural
- Manter formatação markdown intacta

**Para skills novas:**
1. Ler o SKILL.md original
2. Traduzir
3. Gerar nome em pt-BR seguindo a convenção (ex: `gsd-new-feature` → `gsd-nova-funcionalidade`)
4. Escrever em `$PLUGIN_DIR/skills/{nome-ptbr}/SKILL.md`

**Para workflows/referências/templates novos:**
1. Ler o arquivo original
2. Traduzir
3. Escrever com o mesmo nome no diretório correspondente do plugin

**Para arquivos alterados:**
1. Ler a versão original atualizada
2. Traduzir do zero (re-tradução completa)
3. Sobrescrever o arquivo existente no plugin

Se houver muitos arquivos, usar agentes em paralelo (lotes de ~10).

## 5. Atualizar plugin.json

Se a versão do plugin não refletir a versão do GSD:
```bash
# Atualizar versão no plugin.json para refletir a base GSD
```

## 6. Commit e push

```bash
cd "$PLUGIN_DIR"
git add -A
git commit -m "sync: atualizar traduções para GSD v$GSD_VERSION

Arquivos novos: {N}
Arquivos atualizados: {N}
Arquivos removidos: {N}

Co-Authored-By: Claude <noreply@anthropic.com>"
```

Perguntar ao usuário se deseja fazer push:
```
Deseja enviar as atualizações para o GitHub? (sim/não)
```

Se sim:
```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519_agentesintegrados" git push origin main
```

## 7. Relatório final

```
## Sincronização Concluída

GSD v{versão anterior} → v{versão atual}

- {N} arquivos traduzidos
- {N} arquivos atualizados
- {N} arquivos removidos
- Commit: {hash}
- Push: {sim/não}

Reinicie a sessão do Claude Code para carregar as traduções atualizadas.
```

</process>

<flags>
- `--dry-run` — Apenas mostra o relatório de diferenças sem traduzir
- `--force` — Re-traduz todos os arquivos, não apenas o delta
</flags>

<notes>
- Esta skill deve ser executada APÓS `/gsd-update`, nunca antes
- A detecção de mudanças usa comparação de tamanho como heurística — não é perfeita mas cobre a maioria dos casos
- Skills renomeadas no upstream podem aparecer como "nova + removida" — verificar manualmente
- O push usa a chave SSH id_ed25519_agentesintegrados para o repo AgentesIntegrados/gsd-pt-br-v2
</notes>
