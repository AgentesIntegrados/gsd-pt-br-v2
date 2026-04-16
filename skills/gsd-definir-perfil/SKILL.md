---
name: gsd-definir-perfil
description: "Alternar perfil de modelo para agentes GSD (quality/balanced/budget/inherit)"
argument-hint: "<profile (quality|balanced|budget|inherit)>"
allowed-tools:
  - Bash
---


Mostrar a seguinte saída ao usuário exatamente como está, sem comentários adicionais:

!`node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" config-set-model-profile $ARGUMENTS --raw`
