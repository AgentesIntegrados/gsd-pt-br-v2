# Instruções para o GSD

- Use a skill get-shit-done quando o usuário pedir GSD ou usar um comando `gsd-*`.
- Trate `/gsd-...` ou `gsd-...` como invocações de comando e carregue o arquivo correspondente de `.github/skills/gsd-*`.
- Quando um comando diz para spawnar um subagente, prefira um agente personalizado correspondente de `.github/agents`.
- Não aplique workflows GSD a menos que o usuário solicite explicitamente.
- Após completar qualquer comando `gsd-*` (ou qualquer entregável que ele dispare: funcionalidade, correção de bug, testes, documentação, etc.), SEMPRE: (1) ofereça ao usuário o próximo passo via `ask_user`; repita este ciclo de feedback até que o usuário indique explicitamente que terminou.
