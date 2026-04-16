---
name: gsd-mapear-codebase
description: "Analisar o codebase com agentes mapeadores paralelos para produzir documentos em .planning/codebase/"
argument-hint: "[opcional: área específica para mapear, ex.: 'api' ou 'auth']"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Write
  - Task
---


<objective>
Analisar o codebase existente usando agentes gsd-codebase-mapper paralelos para produzir documentos estruturados sobre o codebase.

Cada agente mapeador explora uma área de foco e **escreve documentos diretamente** em `.planning/codebase/`. O orquestrador recebe apenas confirmações, mantendo o uso de contexto mínimo.

Saída: pasta .planning/codebase/ com 7 documentos estruturados sobre o estado do codebase.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/map-codebase.md
</execution_context>

<context>
Área de foco: $ARGUMENTS (opcional - se fornecido, orienta os agentes a focarem em um subsistema específico)

**Carregar estado do projeto se existir:**
Verificar se existe .planning/STATE.md - carrega contexto se o projeto já foi inicializado

**Este comando pode ser executado:**
- Antes de /gsd-new-project (codebases brownfield) - cria o mapa do codebase primeiro
- Após /gsd-new-project (codebases greenfield) - atualiza o mapa do codebase conforme o código evolui
- A qualquer momento para atualizar o entendimento do codebase
</context>

<when_to_use>
**Use map-codebase para:**
- Projetos brownfield antes da inicialização (entender o código existente primeiro)
- Atualizar o mapa do codebase após mudanças significativas
- Onboarding em um codebase desconhecido
- Antes de grandes refatorações (entender o estado atual)
- Quando STATE.md referencia informações desatualizadas do codebase

**Pule o map-codebase para:**
- Projetos greenfield sem código ainda (nada para mapear)
- Codebases triviais (menos de 5 arquivos)
</when_to_use>

<process>
1. Verificar se .planning/codebase/ já existe (oferecer atualização ou pular)
2. Criar estrutura de diretório .planning/codebase/
3. Iniciar 4 agentes gsd-codebase-mapper em paralelo:
   - Agente 1: foco em tecnologia → escreve STACK.md, INTEGRATIONS.md
   - Agente 2: foco em arquitetura → escreve ARCHITECTURE.md, STRUCTURE.md
   - Agente 3: foco em qualidade → escreve CONVENTIONS.md, TESTING.md
   - Agente 4: foco em preocupações → escreve CONCERNS.md
4. Aguardar conclusão dos agentes, coletar confirmações (NÃO o conteúdo dos documentos)
5. Verificar se todos os 7 documentos existem com contagem de linhas
6. Fazer commit do mapa do codebase
7. Sugerir próximos passos (tipicamente: /gsd-new-project ou /gsd-plan-phase)
</process>

<success_criteria>
- [ ] Diretório .planning/codebase/ criado
- [ ] Todos os 7 documentos do codebase escritos pelos agentes mapeadores
- [ ] Documentos seguem a estrutura dos templates
- [ ] Agentes paralelos concluídos sem erros
- [ ] Usuário conhece os próximos passos
</success_criteria>
</content>
