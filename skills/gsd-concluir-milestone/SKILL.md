---
name: gsd-concluir-milestone
description: "Arquivar milestone concluído e preparar para a próxima versão"
argument-hint: "<version>"
allowed-tools:
  - Read
  - Write
  - Bash
---


<objective>
Marcar o milestone {{version}} como concluído, arquivar em milestones/ e atualizar ROADMAP.md e REQUIREMENTS.md.

Objetivo: Criar registro histórico da versão entregue, arquivar artefatos do milestone (roadmap + requisitos) e preparar para o próximo milestone.
Resultado: Milestone arquivado (roadmap + requisitos), PROJECT.md evoluído, tag git criada.
</objective>

<execution_context>
**Carregar estes arquivos AGORA (antes de prosseguir):**

- @$HOME/.claude/get-shit-done/workflows/complete-milestone.md (fluxo principal)
- @$HOME/.claude/get-shit-done/templates/milestone-archive.md (template de arquivo)
  </execution_context>

<context>
**Arquivos do projeto:**
- `.planning/ROADMAP.md`
- `.planning/REQUIREMENTS.md`
- `.planning/STATE.md`
- `.planning/PROJECT.md`

**Entrada do usuário:**

- Versão: {{version}} (ex.: "1.0", "1.1", "2.0")
  </context>

<process>

**Seguir o fluxo complete-milestone.md:**

0. **Verificar auditoria:**

   - Procurar `.planning/v{{version}}-MILESTONE-AUDIT.md`
   - Se ausente ou desatualizado: recomendar `/gsd-audit-milestone` primeiro
   - Se o status da auditoria for `gaps_found`: recomendar `/gsd-plan-milestone-gaps` primeiro
   - Se o status da auditoria for `passed`: prosseguir para o passo 1

   ```markdown
   ## Verificação Prévia

   {Se não houver v{{version}}-MILESTONE-AUDIT.md:}
   ⚠ Nenhuma auditoria de milestone encontrada. Execute `/gsd-audit-milestone` primeiro para verificar
   cobertura de requisitos, integração entre fases e fluxos de ponta a ponta.

   {Se a auditoria tiver lacunas:}
   ⚠ Auditoria de milestone encontrou lacunas. Execute `/gsd-plan-milestone-gaps` para criar
   fases que fechem as lacunas, ou prossiga assim mesmo para aceitar como dívida técnica.

   {Se a auditoria passou:}
   ✓ Auditoria de milestone aprovada. Prosseguindo com a conclusão.
   ```

1. **Verificar prontidão:**

   - Checar se todas as fases do milestone têm planos concluídos (SUMMARY.md existe)
   - Apresentar escopo e estatísticas do milestone
   - Aguardar confirmação

2. **Coletar estatísticas:**

   - Contar fases, planos, tarefas
   - Calcular intervalo git, mudanças de arquivo, LOC
   - Extrair cronograma do git log
   - Apresentar resumo e confirmar

3. **Extrair conquistas:**

   - Ler todos os arquivos SUMMARY.md das fases no intervalo do milestone
   - Extrair 4-6 conquistas principais
   - Apresentar para aprovação

4. **Arquivar milestone:**

   - Criar `.planning/milestones/v{{version}}-ROADMAP.md`
   - Extrair detalhes completos das fases do ROADMAP.md
   - Preencher o template milestone-archive.md
   - Atualizar ROADMAP.md para resumo de uma linha com link

5. **Arquivar requisitos:**

   - Criar `.planning/milestones/v{{version}}-REQUIREMENTS.md`
   - Marcar todos os requisitos da v1 como concluídos (checkboxes marcados)
   - Registrar resultados dos requisitos (validados, ajustados, removidos)
   - Excluir `.planning/REQUIREMENTS.md` (um novo será criado para o próximo milestone)

6. **Atualizar PROJECT.md:**

   - Adicionar seção "Estado Atual" com a versão entregue
   - Adicionar seção "Objetivos do Próximo Milestone"
   - Arquivar conteúdo anterior em `<details>` (se v1.1+)

7. **Commitar e criar tag:**

   - Staged: MILESTONES.md, PROJECT.md, ROADMAP.md, STATE.md, arquivos de arquivo
   - Commit: `chore: archive v{{version}} milestone`
   - Tag: `git tag -a v{{version}} -m "[resumo do milestone]"`
   - Perguntar sobre envio da tag

8. **Oferecer próximos passos:**
   - `/gsd-new-milestone` — iniciar próximo milestone (questionamento → pesquisa → requisitos → roadmap)

</process>

<success_criteria>

- Milestone arquivado em `.planning/milestones/v{{version}}-ROADMAP.md`
- Requisitos arquivados em `.planning/milestones/v{{version}}-REQUIREMENTS.md`
- `.planning/REQUIREMENTS.md` excluído (novo para o próximo milestone)
- ROADMAP.md recolhido para entrada de uma linha
- PROJECT.md atualizado com estado atual
- Tag git v{{version}} criada
- Commit bem-sucedido
- Usuário conhece os próximos passos (incluindo necessidade de novos requisitos)
  </success_criteria>

<critical_rules>

- **Carregar fluxo primeiro:** Ler complete-milestone.md antes de executar
- **Verificar conclusão:** Todas as fases devem ter arquivos SUMMARY.md
- **Confirmação do usuário:** Aguardar aprovação nos gates de verificação
- **Arquivar antes de excluir:** Sempre criar arquivos de arquivo antes de atualizar/excluir os originais
- **Resumo de uma linha:** O milestone recolhido no ROADMAP.md deve ser uma única linha com link
- **Eficiência de contexto:** O arquivo mantém ROADMAP.md e REQUIREMENTS.md com tamanho constante por milestone
- **Requisitos novos:** O próximo milestone começa com `/gsd-new-milestone`, que inclui definição de requisitos
  </critical_rules>
