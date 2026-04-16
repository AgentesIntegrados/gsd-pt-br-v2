<purpose>
Orquestrar o fluxo completo de perfilamento do desenvolvedor: consentimento, análise de sessão (ou fallback de questionário), geração de perfil, exibição de resultados e criação de artefatos.

Este fluxo conecta a Fase 1 (pipeline de sessão) e a Fase 2 (motor de perfilamento) em uma experiência coesa para o usuário. Todo o trabalho pesado é feito pelos subcomandos existentes de gsd-tools.cjs e pelo agente gsd-user-profiler -- este fluxo orquestra a sequência, lida com ramificações e fornece a UX.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

Referências importantes:
- @$HOME/.claude/get-shit-done/references/ui-brand.md (padrões de exibição)
- @$HOME/.claude/get-shit-done/agents/gsd-user-profiler.md (definição do agente perfilador)
- @$HOME/.claude/get-shit-done/references/user-profiling.md (documento de referência de perfilamento)
</required_reading>

<process>

## 1. Inicializar

Analise flags de $ARGUMENTS:
- Detecte flag `--questionnaire` (pular análise de sessão, apenas questionário)
- Detecte flag `--refresh` (reconstruir perfil mesmo quando um existe)

Verifique se há perfil existente:

```bash
PROFILE_PATH="$HOME/.claude/get-shit-done/USER-PROFILE.md"
[ -f "$PROFILE_PATH" ] && echo "EXISTS" || echo "NOT_FOUND"
```

**Se o perfil existir E --refresh NÃO estiver definido E --questionnaire NÃO estiver definido:**


**Modo texto (`workflow.text_mode: true` na config ou flag `--text`):** Defina `TEXT_MODE=true` se `--text` estiver presente em `$ARGUMENTS` OU `text_mode` do JSON de init for `true`. Quando TEXT_MODE estiver ativo, substitua cada chamada `AskUserQuestion` por uma lista numerada em texto simples e peça ao usuário para digitar o número da escolha. Isso é necessário para runtimes não-Claude (OpenAI Codex, Gemini CLI, etc.) onde `AskUserQuestion` não está disponível.
Use AskUserQuestion:
- header: "Perfil Existente"
- question: "Você já tem um perfil. O que você gostaria de fazer?"
- options:
  - "Ver" -- Exibir cartão de resumo dos dados do perfil existente, depois sair
  - "Atualizar" -- Continuar com comportamento de --refresh
  - "Cancelar" -- Sair do fluxo

Se "Ver": Leia USER-PROFILE.md, exiba seu conteúdo formatado como cartão de resumo, depois saia.
Se "Atualizar": Defina comportamento de --refresh e continue.
Se "Cancelar": Exiba "Nenhuma alteração feita." e saia.

**Se o perfil existir E --refresh ESTIVER definido:**

Faça backup do perfil existente:
```bash
cp "$HOME/.claude/get-shit-done/USER-PROFILE.md" "$HOME/.claude/USER-PROFILE.backup.md"
```

Exiba: "Re-analisando suas sessões para atualizar seu perfil."
Continue para a etapa 2.

**Se nenhum perfil existir:** Continue para a etapa 2.

---

## 2. Gate de Consentimento (ACTV-06)

**Pule se** a flag `--questionnaire` estiver definida (nenhuma leitura de JSONL ocorre -- vá diretamente para a etapa 4b).

Exiba tela de consentimento:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD > PERFIL DO SEU ESTILO DE CODIFICAÇÃO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Claude começa cada conversa de forma genérica. Um perfil ensina ao Claude
como VOCÊ realmente trabalha -- não como você acha que trabalha.

## O Que Vamos Analisar

Suas sessões recentes do Claude Code, procurando padrões nestas
8 dimensões comportamentais:

| Dimensão             | O Que Mede                                          |
|----------------------|-----------------------------------------------------|
| Estilo de Comunicação | Como você formula pedidos (conciso vs. detalhado)  |
| Velocidade de Decisão | Como você escolhe entre opções                     |
| Profundidade de Explicação | Quanta explicação você quer com o código      |
| Abordagem de Depuração | Como você lida com erros e bugs                  |
| Filosofia de UX      | Quanto você se importa com design vs. função        |
| Filosofia de Fornecedor | Como você avalia bibliotecas e ferramentas       |
| Gatilhos de Frustração | O que te faz corrigir o Claude                   |
| Estilo de Aprendizado | Como você prefere aprender coisas novas           |

## Tratamento de Dados

✓ Lê arquivos de sessão localmente (somente leitura, nada é modificado)
✓ Analisa padrões de mensagens (não o significado do conteúdo)
✓ Armazena perfil em $HOME/.claude/get-shit-done/USER-PROFILE.md
✗ Nada é enviado a serviços externos
✗ Conteúdo sensível (chaves de API, senhas) é automaticamente excluído
```

**Se caminho --refresh:**
Mostre consentimento abreviado:

```
Re-analisando suas sessões para atualizar seu perfil.
Seu perfil existente foi salvo como backup em USER-PROFILE.backup.md.
```

Use AskUserQuestion:
- header: "Atualizar"
- question: "Continuar com a atualização do perfil?"
- options:
  - "Continuar" -- Ir para a etapa 3
  - "Cancelar" -- Sair do fluxo

**Se caminho padrão (sem --refresh):**

Use AskUserQuestion:
- header: "Pronto?"
- question: "Pronto para analisar suas sessões?"
- options:
  - "Vamos lá" -- Ir para a etapa 3 (análise de sessão)
  - "Usar questionário" -- Ir para a etapa 4b (caminho de questionário)
  - "Agora não" -- Exibir "Sem problema. Execute /gsd-profile-user quando estiver pronto." e sair

---

## 3. Varredura de Sessões

Exiba: "◆ Varrendo sessões..."

Execute varredura de sessões:
```bash
SCAN_RESULT=$(node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs scan-sessions --json 2>/dev/null)
```

Analise o output JSON para obter contagem de sessões e contagem de projetos.

Exiba: "✓ Encontradas N sessões em M projetos"

**Determine suficiência dos dados:**
- Conte o total de mensagens disponíveis do resultado da varredura (soma de sessões entre projetos)
- Se 0 sessões encontradas: Exiba "Nenhuma sessão encontrada. Mudando para questionário." e vá para a etapa 4b
- Se sessões encontradas: Continue para a etapa 4a

---

## 4a. Caminho de Análise de Sessão

Exiba: "◆ Amostrando mensagens..."

Execute amostragem de perfil:
```bash
SAMPLE_RESULT=$(node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs profile-sample --json 2>/dev/null)
```

Analise o output JSON para obter o caminho do diretório temporário e a contagem de mensagens.

Exiba: "✓ N mensagens amostradas de M projetos"

Exiba: "◆ Analisando padrões..."

**Spawne o agente gsd-user-profiler usando a ferramenta Task:**

Use a ferramenta Task para spawnar o agente `gsd-user-profiler`. Forneça:
- O caminho do arquivo JSONL amostrado do output de profile-sample
- O documento de referência de perfilamento em `$HOME/.claude/get-shit-done/references/user-profiling.md`

O prompt do agente deve seguir esta estrutura:
```
Leia o documento de referência de perfilamento e as mensagens de sessão amostradas, depois analise os padrões comportamentais do desenvolvedor em todas as 8 dimensões.

Referência: @$HOME/.claude/get-shit-done/references/user-profiling.md
Dados de sessão: @{temp_dir}/profile-sample.jsonl

Analise estas mensagens e retorne sua análise no formato JSON <analysis> especificado no documento de referência.
```

**Analise o output do agente:**
- Extraia o bloco JSON `<analysis>` da resposta do agente
- Salve o JSON de análise em um arquivo temporário (no mesmo diretório temporário criado por profile-sample)

```bash
ANALYSIS_PATH="{temp_dir}/analysis.json"
```

Escreva o JSON de análise em `$ANALYSIS_PATH`.

Exiba: "✓ Análise completa (N dimensões pontuadas)"

**Verifique dados escassos:**
- Leia o JSON de análise e verifique a contagem total de mensagens
- Se < 50 mensagens foram analisadas: Observe que um suplemento de questionário poderia melhorar a precisão. Exiba: "Nota: Dados de sessão limitados (N mensagens). Os resultados podem ter menor confiabilidade."

Continue para a etapa 5.

---

## 4b. Caminho do Questionário

Exiba: "Usando questionário para construir seu perfil."

**Obtenha as perguntas:**
```bash
QUESTIONS=$(node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs profile-questionnaire --json 2>/dev/null)
```

Analise o JSON de perguntas. Contém 8 perguntas, uma por dimensão.

**Apresente cada pergunta ao usuário via AskUserQuestion:**

Para cada pergunta no array de perguntas:
- header: O nome da dimensão (ex: "Estilo de Comunicação")
- question: O texto da pergunta
- options: As opções de resposta da definição da pergunta

Colete todas as respostas em um objeto JSON de respostas mapeando chaves de dimensão para valores de resposta selecionados.

**Salve as respostas em arquivo temporário:**
```bash
ANSWERS_PATH=$(mktemp /tmp/gsd-profile-answers-XXXXXX.json)
```

Escreva o JSON de respostas em `$ANSWERS_PATH`.

**Converta respostas em análise:**
```bash
ANALYSIS_RESULT=$(node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs profile-questionnaire --answers "$ANSWERS_PATH" --json 2>/dev/null)
```

Analise o JSON de análise do resultado.

Salve o JSON de análise em um arquivo temporário:
```bash
ANALYSIS_PATH=$(mktemp /tmp/gsd-profile-analysis-XXXXXX.json)
```

Escreva o JSON de análise em `$ANALYSIS_PATH`.

Continue para a etapa 5 (pule a resolução de split já que o questionário lida com ambiguidade internamente).

---

## 5. Resolução de Split

**Pule se** caminho apenas de questionário (splits já tratados internamente).

Leia o JSON de análise de `$ANALYSIS_PATH`.

Verifique cada dimensão para `cross_project_consistent: false`.

**Para cada split detectado:**

Use AskUserQuestion:
- header: O nome da dimensão (ex: "Estilo de Comunicação")
- question: "Suas sessões mostram padrões diferentes:" seguido do contexto do split (ex: "Projetos CLI/backend -> terse-direct, Projetos Frontend/UI -> detailed-structured")
- options:
  - Opção de avaliação A (ex: "terse-direct")
  - Opção de avaliação B (ex: "detailed-structured")
  - "Dependente do contexto (manter ambos)"

**Se o usuário escolher uma avaliação específica:** Atualize o campo `rating` da dimensão no JSON de análise para o valor selecionado.

**Se o usuário escolher "Dependente do contexto":** Mantenha a avaliação dominante no campo `rating`. Adicione um `context_note` ao resumo da dimensão descrevendo o split (ex: "Dependente do contexto: conciso em projetos CLI, detalhado em projetos frontend").

Escreva o JSON de análise atualizado de volta em `$ANALYSIS_PATH`.

---

## 6. Escrita do Perfil

Exiba: "◆ Escrevendo perfil..."

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs write-profile --input "$ANALYSIS_PATH" --json 2>/dev/null
```

Exiba: "✓ Perfil escrito em $HOME/.claude/get-shit-done/USER-PROFILE.md"

---

## 7. Exibição de Resultados

Leia o JSON de análise de `$ANALYSIS_PATH` para construir a exibição.

**Mostre tabela de cartão de relatório:**

```
## Seu Perfil

| Dimensão              | Avaliação            | Confiabilidade |
|-----------------------|----------------------|----------------|
| Estilo de Comunicação | detailed-structured  | ALTA           |
| Velocidade de Decisão | deliberate-informed  | MÉDIA          |
| Profundidade de Expl. | concise              | ALTA           |
| Abordagem de Depuração| hypothesis-driven    | MÉDIA          |
| Filosofia de UX       | pragmatic            | BAIXA          |
| Filosofia de Fornecedor| thorough-evaluator  | ALTA           |
| Gatilhos de Frustração| scope-creep          | MÉDIA          |
| Estilo de Aprendizado | self-directed        | ALTA           |
```

(Preencha com valores reais do JSON de análise.)

**Mostre destaques:**

Escolha 3-4 dimensões com maior confiabilidade e sinais de evidência mais fortes. Formate como:

```
## Destaques

- **Comunicação (ALTA):** Você consistentemente fornece contexto estruturado com
  cabeçalhos e declarações de problema antes de fazer pedidos
- **Escolha de Fornecedores (ALTA):** Você pesquisa alternativas minuciosamente -- comparando
  documentação, atividade no GitHub e tamanhos de bundle antes de se comprometer
- **Frustrações (MÉDIA):** Você corrige o Claude com mais frequência quando ele faz coisas
  que você não pediu -- o escopo excessivo é seu gatilho principal
```

Construa destaques a partir do array `evidence` e campos `summary` no JSON de análise. Use as citações de evidência mais convincentes. Formate cada uma como "Você tende a..." ou "Você consistentemente..." com atribuição de evidência.

**Ofereça visualização do perfil completo:**

Use AskUserQuestion:
- header: "Perfil"
- question: "Quer ver o perfil completo?"
- options:
  - "Sim" -- Ler e exibir o conteúdo completo de USER-PROFILE.md, depois continuar para a etapa 8
  - "Continuar para artefatos" -- Ir diretamente para a etapa 8

---

## 8. Seleção de Artefatos (ACTV-05)

Use AskUserQuestion com multiSelect:
- header: "Artefatos"
- question: "Quais artefatos devo gerar?"
- options (TODOS pré-selecionados por padrão):
  - "Arquivo de comando /gsd-dev-preferences" -- "Carregue suas preferências em qualquer sessão"
  - "Seção de perfil no CLAUDE.md" -- "Adicione perfil ao CLAUDE.md deste projeto"
  - "CLAUDE.md Global" -- "Adicione perfil ao $HOME/.claude/CLAUDE.md para todos os projetos"

**Se nenhum artefato selecionado:** Exiba "Nenhum artefato gerado. Seu perfil está salvo em $HOME/.claude/get-shit-done/USER-PROFILE.md" e vá para a etapa 10.

---

## 9. Geração de Artefatos

Gere artefatos selecionados sequencialmente (E/S de arquivo é rápida, sem benefício de agentes paralelos):

**Para /gsd-dev-preferences (se selecionado):**

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs generate-dev-preferences --analysis "$ANALYSIS_PATH" --json 2>/dev/null
```

Exiba: "✓ /gsd-dev-preferences gerado em $HOME/.claude/commands/gsd/dev-preferences.md"

**Para seção de perfil no CLAUDE.md (se selecionado):**

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs generate-claude-profile --analysis "$ANALYSIS_PATH" --json 2>/dev/null
```

Exiba: "✓ Seção de perfil adicionada ao CLAUDE.md"

**Para CLAUDE.md Global (se selecionado):**

```bash
node $HOME/.claude/get-shit-done/bin/gsd-tools.cjs generate-claude-profile --analysis "$ANALYSIS_PATH" --global --json 2>/dev/null
```

Exiba: "✓ Seção de perfil adicionada ao $HOME/.claude/CLAUDE.md"

**Tratamento de erros:** Se alguma chamada de gsd-tools.cjs falhar, exiba a mensagem de erro e use AskUserQuestion para oferecer "Tentar novamente" ou "Pular este artefato". Na tentativa, re-execute o comando. Ao pular, continue para o próximo artefato.

---

## 10. Resumo e Diff de Atualização

**Se caminho --refresh:**

Leia tanto o backup antigo quanto a nova análise para comparar avaliações/confiabilidade das dimensões.

Leia o perfil de backup:
```bash
BACKUP_PATH="$HOME/.claude/USER-PROFILE.backup.md"
```

Compare a avaliação e confiabilidade de cada dimensão entre o antigo e o novo. Exiba tabela de diff mostrando apenas dimensões alteradas:

```
## Alterações

| Dimensão        | Antes                       | Depois                       |
|-----------------|-----------------------------|------------------------------|
| Comunicação     | terse-direct (BAIXA)        | detailed-structured (ALTA)   |
| Depuração       | fix-first (MÉDIA)           | hypothesis-driven (MÉDIA)    |
```

Se nada mudou: Exiba "Nenhuma alteração detectada -- seu perfil já está atualizado."

**Exiba resumo final:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD > PERFIL COMPLETO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Seu perfil:    $HOME/.claude/get-shit-done/USER-PROFILE.md
```

Em seguida, liste os caminhos para cada artefato gerado:
```
Artefatos:
  ✓ /gsd-dev-preferences   $HOME/.claude/commands/gsd/dev-preferences.md
  ✓ Seção CLAUDE.md         ./CLAUDE.md
  ✓ CLAUDE.md Global        $HOME/.claude/CLAUDE.md
```

(Mostre apenas artefatos que foram realmente gerados.)

**Limpe arquivos temporários:**

Remova o diretório temporário criado por profile-sample (contém JSONL de amostra e JSON de análise):
```bash
rm -rf "$TEMP_DIR"
```

Remova também quaisquer arquivos temporários independentes criados para respostas do questionário:
```bash
rm -f "$ANSWERS_PATH" 2>/dev/null
rm -f "$ANALYSIS_PATH" 2>/dev/null
```

(Limpe apenas caminhos temporários que foram realmente criados durante esta execução do fluxo.)

</process>

<success_criteria>
- [ ] Inicialização detecta perfil existente e trata todas as três respostas (ver/atualizar/cancelar)
- [ ] Gate de consentimento exibido para o caminho de análise de sessão, pulado para o caminho de questionário
- [ ] Varredura de sessão descobre sessões e relata estatísticas
- [ ] Caminho de análise de sessão: amostras mensagens, spawna agente perfilador, extrai JSON de análise
- [ ] Caminho de questionário: apresenta 8 perguntas, coleta respostas, converte para JSON de análise
- [ ] Resolução de split apresenta splits dependentes de contexto com opções de resolução do usuário
- [ ] Perfil escrito em USER-PROFILE.md via subcomando write-profile
- [ ] Exibição de resultados mostra tabela de cartão de relatório e destaques com evidências
- [ ] Seleção de artefatos usa multiSelect com todas as opções pré-selecionadas
- [ ] Artefatos gerados sequencialmente via subcomandos de gsd-tools.cjs
- [ ] Diff de atualização mostra dimensões alteradas quando --refresh foi usado
- [ ] Arquivos temporários limpos na conclusão
</success_criteria>
</output>
