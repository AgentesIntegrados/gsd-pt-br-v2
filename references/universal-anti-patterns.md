# Anti-Padrões Universais

Regras que se aplicam a TODOS os fluxos de trabalho e agentes. Fluxos de trabalho individuais podem ter anti-padrões específicos adicionais.

---

## Regras de Orçamento de Contexto

1. **Nunca** leia arquivos de definição de agente (`agents/*.md`) — `subagent_type` os carrega automaticamente. Ler definições de agente no orquestrador desperdiça contexto com conteúdo automaticamente injetado nas sessões de subagente.
2. **Nunca** incorpore arquivos grandes em prompts de subagente — diga aos agentes para ler arquivos do disco. Os agentes têm suas próprias janelas de contexto.
3. **Profundidade de leitura escala com a janela de contexto** — verifique `context_window_tokens` em `.planning/config.json`. Em < 500000: leia apenas o frontmatter, campos de status ou resumos. Em >= 500000 (modelo 1M): leituras completas do corpo são permitidas quando o conteúdo for necessário para decisões inline. Veja `references/context-budget.md` para a tabela completa.
4. **Delegue** trabalho pesado para subagentes — o orquestrador roteia, ele não constrói, analisa, pesquisa, investiga ou verifica.
5. **Aviso proativo de pausa**: Se você já consumiu contexto significativo (leituras grandes de arquivos, múltiplos resultados de subagentes), avise o usuário: "O orçamento de contexto está ficando pesado. Considere fazer um checkpoint do progresso."

## Regras de Leitura de Arquivos

6. **Profundidade de leitura do SUMMARY.md escala com a janela de contexto** — em context_window_tokens < 500000: leia apenas o frontmatter de SUMMARYs de fases anteriores. Em >= 500000: leituras completas do corpo são permitidas para fases de dependência direta. Dependências transitivas (2+ fases atrás) permanecem apenas frontmatter independentemente.
7. **Nunca** leia arquivos PLAN.md completos de outras fases — apenas os planos da fase atual.
8. **Nunca** leia arquivos `.planning/logs/` — apenas o fluxo de saúde lê esses.
9. **Não** releia conteúdo completo de arquivo quando o frontmatter for suficiente — o frontmatter contém status, key_files, commits e campos de provides. Exceção: em >= 500000, reler o corpo completo é aceitável quando o conteúdo semântico for necessário.

## Regras de Subagente

10. **NUNCA** use tipos de agente não-GSD (`general-purpose`, `Explore`, `Plan`, `Bash`, `feature-dev`, etc.) — SEMPRE use `subagent_type: "gsd-{agent}"` (ex.: `gsd-phase-researcher`, `gsd-executor`, `gsd-planner`). Agentes GSD têm prompts cientes do projeto, log de auditoria e contexto de fluxo de trabalho. Agentes genéricos ignoram tudo isso.
11. **Não** reabra decisões que já estão bloqueadas no CONTEXT.md (ou seção ## Context do PROJECT.md) — respeite decisões bloqueadas incondicionalmente.

## Anti-Padrões de Questionamento

Referência: `references/questioning.md` para a lista completa de anti-padrões.

12. **Não** percorra listas de verificação — percorrer listas de verificação (perguntar itens um por um de uma lista) é o anti-padrão número 1. Em vez disso, use profundidade progressiva: comece amplo, aprofunde onde for interessante.
13. **Não** use jargão corporativo — evite expressões como "alinhamento de stakeholders", "sinergia", "entregáveis". Use linguagem simples.
14. **Não** aplique restrições prematuras — não estreite o espaço de solução antes de entender o problema. Pergunte sobre o problema primeiro, depois restrinja.

## Anti-Padrões de Gerenciamento de Estado

15. **Sem Write/Edit direto em STATE.md ou ROADMAP.md para mutações.** Sempre use os comandos CLI do `gsd-tools.cjs` (`state update`, `state advance-plan`, `roadmap update-status`) para mutações. O uso direto da ferramenta Write ignora a lógica de atualização segura e é inseguro em ambientes multi-sessão. Exceção: a criação inicial do STATE.md a partir de um template é permitida.

## Regras de Comportamento

16. **Não** crie artefatos que o usuário não aprovou — sempre confirme antes de escrever novos documentos de planejamento.
17. **Não** modifique arquivos fora do escopo declarado do fluxo de trabalho — verifique a lista files_modified do plano.
18. **Não** sugira múltiplas próximas ações sem prioridade clara — uma sugestão primária, alternativas listadas secundariamente.
19. **Não** use `git add .` ou `git add -A` — faça staging apenas de arquivos específicos.
20. **Não** inclua informações sensíveis (chaves de API, senhas, tokens) em documentos de planejamento ou commits.

## Regras de Recuperação de Erros

21. **Detecção de bloqueio Git**: Antes de qualquer operação git, se ela falhar com "Unable to create lock file", verifique se há `.git/index.lock` obsoleto e oriente o usuário a removê-lo (não remova automaticamente).
22. **Consciência de fallback de configuração**: O carregamento de configuração retorna `null` silenciosamente em JSON inválido. Se seu fluxo de trabalho depende de valores de configuração, verifique se é null e avise o usuário: "config.json é inválido ou está ausente — executando com padrões."
23. **Recuperação de estado parcial**: Se STATE.md faz referência a um diretório de fase que não existe, não prossiga silenciosamente. Avise o usuário e sugira diagnosticar a incompatibilidade.

## Regras Específicas do GSD

24. **Não** verifique `mode === 'auto'` ou `mode === 'autonomous'` — o GSD usa a flag de configuração `yolo`. Verifique `yolo: true` para modo autônomo, ausência ou `false` para modo interativo.
25. **Sempre use `gsd-tools.cjs`** (não `gsd-tools.js` ou qualquer outra variante) — o GSD usa CommonJS para compatibilidade com CLI do Node.js.
26. **Arquivos de plano DEVEM seguir o padrão `{padded_phase}-{NN}-PLAN.md`** (ex.: `01-01-PLAN.md`). Nunca use `PLAN-01.md`, `plan-01.md` ou qualquer outra variação — a detecção do gsd-tools depende deste padrão exato.
27. **Não comece a executar o próximo plano antes de escrever o SUMMARY.md para o plano atual** — planos posteriores podem referenciá-lo via includes `@`.

## Regras para iOS / Plataforma Apple

28. **NUNCA use `Package.swift` + `.executableTarget` (ou `.target`) como sistema de build principal para apps iOS.** Alvos executáveis SPM produzem binários CLI macOS, não bundles iOS `.app`. Eles não podem ser instalados em dispositivos iOS ou enviados à App Store. Use XcodeGen (`project.yml` + `xcodegen generate`) para criar um `.xcodeproj` adequado. Veja `references/ios-scaffold.md` para o padrão completo.
29. **Verifique a disponibilidade da API SwiftUI antes de usar.** Muitas APIs SwiftUI requerem uma versão mínima específica do iOS (ex.: `NavigationSplitView` é iOS 16+, `List(selection:)` com multi-select e `@Observable` requerem iOS 17). Se um plano usa uma API que excede o `IPHONEOS_DEPLOYMENT_TARGET` declarado, eleve o alvo de implantação ou adicione guardas `#available`.
