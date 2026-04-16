# Anti-Padrões do Planejador e Exemplos de Especificidade

> Arquivo de referência para o agente gsd-planner. Carregado sob demanda via referência `@`.
> Para janelas de contexto menores que 200K, este conteúdo é removido do prompt do agente e está disponível aqui para carregamento sob demanda.

## Anti-Padrões de Checkpoint

### Ruim — Pedir ao humano para automatizar

```xml
<task type="checkpoint:human-action">
  <action>Deploy to Vercel</action>
  <instructions>Visit vercel.com, import repo, click deploy...</instructions>
</task>
```

**Por que é ruim:** Vercel tem uma CLI. O Claude deve executar `vercel --yes`. Nunca peça ao usuário para fazer o que o Claude pode automatizar via CLI/API.

### Ruim — Checkpoints em excesso

```xml
<task type="auto">Create schema</task>
<task type="checkpoint:human-verify">Check schema</task>
<task type="auto">Create API</task>
<task type="checkpoint:human-verify">Check API</task>
```

**Por que é ruim:** Fadiga de verificação. Os usuários não devem ser solicitados a verificar cada pequeno passo. Combine em um único checkpoint ao final de um trabalho significativo.

### Bom — Checkpoint de verificação único

```xml
<task type="auto">Create schema</task>
<task type="auto">Create API</task>
<task type="auto">Create UI</task>
<task type="checkpoint:human-verify">
  <what-built>Complete auth flow (schema + API + UI)</what-built>
  <how-to-verify>Test full flow: register, login, access protected page</how-to-verify>
</task>
```

### Ruim — Misturar checkpoints com implementação

Um plano não deve intercalar múltiplos tipos de checkpoint com tarefas de implementação. Os checkpoints pertencem a fronteiras naturais de verificação, não espalhados por todo o plano.

## Exemplos de Especificidade

| VAGO DEMAIS | NA MEDIDA CERTA |
|-------------|-----------------|
| "Add authentication" | "Add JWT auth with refresh rotation using jose library, store in httpOnly cookie, 15min access / 7day refresh" |
| "Create the API" | "Create POST /api/projects endpoint accepting {name, description}, validates name length 3-50 chars, returns 201 with project object" |
| "Style the dashboard" | "Add Tailwind classes to Dashboard.tsx: grid layout (3 cols on lg, 1 on mobile), card shadows, hover states on action buttons" |
| "Handle errors" | "Wrap API calls in try/catch, return {error: string} on 4xx/5xx, show toast via sonner on client" |
| "Set up the database" | "Add User and Project models to schema.prisma with UUID ids, email unique constraint, createdAt/updatedAt timestamps, run prisma db push" |

**Teste de especificidade:** Uma instância diferente do Claude conseguiria executar a tarefa sem fazer perguntas de esclarecimento? Se não, adicione mais detalhes.

## Anti-Padrões da Seção de Contexto

### Ruim — Encadeamento reflexivo de SUMMARY

```markdown
<context>
@.planning/phases/01-foundation/01-01-SUMMARY.md
@.planning/phases/01-foundation/01-02-SUMMARY.md  <!-- O Plano 02 realmente precisa da saída do Plano 01? -->
@.planning/phases/01-foundation/01-03-SUMMARY.md  <!-- A cadeia cresce, o contexto incha -->
</context>
```

**Por que é ruim:** Os planos frequentemente são independentes. O encadeamento reflexivo (02 referencia 01, 03 referencia 02...) desperdiça contexto. Referencie arquivos SUMMARY anteriores apenas quando o plano genuinamente usa tipos/exportações desse plano anterior ou quando uma decisão desse plano afeta o atual.

### Bom — Contexto seletivo

```markdown
<context>
@.planning/PROJECT.md
@.planning/STATE.md
@.planning/phases/01-foundation/01-01-SUMMARY.md  <!-- Usa o tipo User definido no Plano 01 -->
</context>
```

## Anti-Padrões de Redução de Escopo

**Linguagem proibida em ações de tarefa:**
- "v1", "v2", "simplified version", "static for now", "hardcoded for now"
- "future enhancement", "placeholder", "basic version", "minimal implementation"
- "will be wired later", "dynamic in future phase", "skip for now"

Se uma decisão do CONTEXT.md diz "exibir custo calculado a partir da tabela de faturamento em impulsos", o plano deve entregar exatamente isso. Não "rótulo estático /min" como uma "v1". Se a fase for muito complexa, recomende uma divisão de fase em vez de reduzir o escopo silenciosamente.
