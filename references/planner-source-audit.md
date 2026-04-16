# Auditoria de Fontes e Limites de Autoridade do Planejador

Referência para `agents/gsd-planner.md` — regras estendidas para auditorias de cobertura de múltiplas fontes e restrições de autoridade do planejador.

## Formato de Auditoria de Cobertura Multi-Fonte

Antes de finalizar os planos, produza uma **auditoria de fontes** cobrindo TODOS os quatro tipos de artifacts:

```
FONTE     | ID      | Funcionalidade/Requisito             | Plano | Status    | Notas
--------- | ------- | ------------------------------------ | ----- | --------- | ------
GOAL      | —       | {objetivo da fase do ROADMAP.md}     | 01-03 | COVERED   |
REQ       | REQ-14  | Login OAuth com Google + GH          | 02    | COVERED   |
REQ       | REQ-22  | Fluxo de verificação de e-mail       | 03    | COVERED   |
RESEARCH  | —       | Rate limiting em rotas de auth       | 01    | COVERED   |
RESEARCH  | —       | Rotação de refresh token             | NONE  | ⚠ MISSING | Nenhum plano cobre isso
CONTEXT   | D-01    | Usar biblioteca jose para JWT        | 02    | COVERED   |
CONTEXT   | D-04    | 15min access / 7day refresh          | 02    | COVERED   |
```

### Quatro Tipos de Fontes

1. **GOAL** — O campo `goal:` do ROADMAP.md para esta fase. A condição primária de sucesso.
2. **REQ** — Cada REQ-ID em `phase_req_ids`. Faça referência cruzada com REQUIREMENTS.md para descrições.
3. **RESEARCH** — Abordagens técnicas, restrições descobertas e funcionalidades identificadas no RESEARCH.md. Exclua itens explicitamente marcados como "fora do escopo" ou "trabalho futuro" pelo pesquisador.
4. **CONTEXT** — Cada decisão D-XX da seção `<decisions>` do CONTEXT.md.

### O que NÃO é uma Lacuna

Não sinalize estes como MISSING:
- Itens em `## Deferred Ideas` no CONTEXT.md — o desenvolvedor optou por adiá-los
- Itens com escopo para uma fase diferente via `phase_req_ids` — não atribuídos a esta fase
- Itens no RESEARCH.md explicitamente marcados como "fora do escopo" ou "trabalho futuro" pelo pesquisador

### Tratamento de Itens MISSING

Se QUALQUER linha estiver como `⚠ MISSING`, NÃO finalize o conjunto de planos silenciosamente. Retorne ao orquestrador:

```
## ⚠ Auditoria de Fontes: Itens Sem Plano Encontrados

Os seguintes itens dos artifacts de origem não possuem plano correspondente:

1. **{FONTE}: {descrição do item}** (em {arquivo do artifact}, seção "{seção}")
   - {por que foi identificado como necessário}

   Opções:
   A) Adicionar um plano para cobrir este item
   B) Dividir a fase: mover para uma sub-fase
   C) Adiar explicitamente: adicionar ao backlog com confirmação do desenvolvedor

   → Aguardando decisão do desenvolvedor antes de finalizar o conjunto de planos.
```

Se TODAS as linhas estiverem COVERED → retorne `## PLANNING COMPLETE` normalmente.

---

## Limites de Autoridade — Exemplos de Restrições

As únicas razões legítimas do planejador para dividir ou sinalizar uma funcionalidade são **restrições**, não julgamentos sobre dificuldade:

**Válido (restrições):**
- ✓ "Esta tarefa toca 9 arquivos e consumiria ~45% do contexto — dividir em duas tarefas"
- ✓ "Nenhuma chave de API ou endpoint está definido em nenhum artifact de origem — necessário input do desenvolvedor"
- ✓ "Esta funcionalidade depende do sistema de auth construído na Fase 03, que ainda não está completo"

**Inválido (julgamentos de dificuldade):**
- ✗ "Isso é complexo e seria difícil de implementar corretamente"
- ✗ "Integrar com um serviço externo pode levar muito tempo"
- ✗ "Esta é uma funcionalidade desafiadora que talvez devesse ser deixada para uma fase futura"

Se uma funcionalidade não tem nenhuma das três restrições legítimas (custo de contexto, informação ausente, conflito de dependência), ela é planejada. Ponto final.
