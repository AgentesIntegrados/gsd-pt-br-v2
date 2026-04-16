# Template de Entrada de Milestone

Adicione esta entrada ao `.planning/MILESTONES.md` ao concluir um milestone:

```markdown
## v[X.Y] [Nome] (Entregue em: YYYY-MM-DD)

**Entregue:** [Uma frase descrevendo o que foi entregue]

**Fases concluídas:** [X-Y] ([Z] planos no total)

**Principais conquistas:**
- [Grande conquista 1]
- [Grande conquista 2]
- [Grande conquista 3]
- [Grande conquista 4]

**Estatísticas:**
- [X] arquivos criados/modificados
- [Y] linhas de código (linguagem principal)
- [Z] fases, [N] planos, [M] tarefas
- [D] dias do início à entrega (ou de milestone a milestone)

**Intervalo git:** `feat(XX-XX)` → `feat(YY-YY)`

**Próximos passos:** [Breve descrição dos objetivos do próximo milestone, ou "Projeto concluído"]

---
```

<structure>
Se o MILESTONES.md não existir, crie-o com o cabeçalho:

```markdown
# Milestones do Projeto: [Nome do Projeto]

[Entradas em ordem cronológica reversa — mais recentes primeiro]
```
</structure>

<guidelines>
**Quando criar milestones:**
- MVP v1.0 inicial entregue
- Versões maiores (v2.0, v3.0)
- Milestones de funcionalidades significativas (v1.1, v1.2)
- Antes de arquivar o planejamento (registrar o que foi entregue)

**Não criar milestones para:**
- Conclusões de fases individuais (fluxo de trabalho normal)
- Trabalho em progresso (aguardar até a entrega)
- Correções de bugs menores que não constituem um release

**Estatísticas a incluir:**
- Contar arquivos modificados: `git diff --stat feat(XX-XX)..feat(YY-YY) | tail -1`
- Contar LOC: `find . -name "*.swift" -o -name "*.ts" | xargs wc -l` (ou extensão relevante)
- Contagens de fases/planos/tarefas do ROADMAP
- Cronograma do primeiro commit da fase ao último commit da fase

**Formato do intervalo git:**
- Primeiro commit do milestone → último commit do milestone
- Exemplo: `feat(01-01)` → `feat(04-01)` para as fases 1-4
</guidelines>

<example>
```markdown
# Milestones do Projeto: WeatherBar

## v1.1 Segurança e Polimento (Entregue em: 2025-12-10)

**Entregue:** Endurecimento de segurança com integração ao Keychain e tratamento de erros abrangente

**Fases concluídas:** 5-6 (3 planos no total)

**Principais conquistas:**
- Migração do armazenamento de chave de API de texto simples para o Keychain do macOS
- Implementação de tratamento abrangente de erros para falhas de rede
- Adicionada integração de relatório de falhas com Sentry
- Corrigido vazamento de memória no temporizador de atualização automática

**Estatísticas:**
- 23 arquivos modificados
- 650 linhas de Swift adicionadas
- 2 fases, 3 planos, 12 tarefas
- 8 dias de v1.0 a v1.1

**Intervalo git:** `feat(05-01)` → `feat(06-02)`

**Próximos passos:** Redesign v2.0 em SwiftUI com suporte a widgets

---

## v1.0 MVP (Entregue em: 2025-11-25)

**Entregue:** Aplicativo de previsão do tempo na barra de menus com condições atuais e previsão de 3 dias

**Fases concluídas:** 1-4 (7 planos no total)

**Principais conquistas:**
- Aplicativo na barra de menus com UI em popover (AppKit)
- Integração com a API OpenWeather com atualização automática
- Exibição do clima atual com ícone de condições
- Lista de previsão para 3 dias com temperaturas máximas e mínimas
- Assinado e notarizado para distribuição

**Estatísticas:**
- 47 arquivos criados
- 2.450 linhas de Swift
- 4 fases, 7 planos, 28 tarefas
- 12 dias do início à entrega

**Intervalo git:** `feat(01-01)` → `feat(04-01)`

**Próximos passos:** Auditoria e endurecimento de segurança para v1.1
```
</example>
