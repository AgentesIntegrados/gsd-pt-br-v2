# Template de Preocupações do Codebase

Template para `.planning/codebase/CONCERNS.md` - captura problemas conhecidos e áreas que requerem cuidado.

**Propósito:** Expor avisos acionáveis sobre o codebase. Focado em "o que observar ao fazer alterações."

---

## Template do Arquivo

```markdown
# Preocupações do Codebase

**Data da Análise:** [YYYY-MM-DD]

## Dívida Técnica

**[Área/Componente]:**
- Problema: [Qual é o atalho/solução alternativa]
- Por quê: [Por que foi feito dessa forma]
- Impacto: [O que quebra ou degrada por causa disso]
- Abordagem de correção: [Como tratar adequadamente]

**[Área/Componente]:**
- Problema: [Qual é o atalho/solução alternativa]
- Por quê: [Por que foi feito dessa forma]
- Impacto: [O que quebra ou degrada por causa disso]
- Abordagem de correção: [Como tratar adequadamente]

## Bugs Conhecidos

**[Descrição do bug]:**
- Sintomas: [O que acontece]
- Gatilho: [Como reproduzir]
- Solução alternativa: [Mitigação temporária, se houver]
- Causa raiz: [Se conhecida]
- Bloqueado por: [Se aguardando algo]

**[Descrição do bug]:**
- Sintomas: [O que acontece]
- Gatilho: [Como reproduzir]
- Solução alternativa: [Mitigação temporária, se houver]
- Causa raiz: [Se conhecida]

## Considerações de Segurança

**[Área que requer cuidado de segurança]:**
- Risco: [O que pode dar errado]
- Mitigação atual: [O que está em vigor agora]
- Recomendações: [O que deve ser adicionado]

**[Área que requer cuidado de segurança]:**
- Risco: [O que pode dar errado]
- Mitigação atual: [O que está em vigor agora]
- Recomendações: [O que deve ser adicionado]

## Gargalos de Performance

**[Operação/endpoint lento]:**
- Problema: [O que está lento]
- Medição: [Números reais: "500ms p95", "2s de carregamento"]
- Causa: [Por que está lento]
- Caminho de melhoria: [Como acelerar]

**[Operação/endpoint lento]:**
- Problema: [O que está lento]
- Medição: [Números reais]
- Causa: [Por que está lento]
- Caminho de melhoria: [Como acelerar]

## Áreas Frágeis

**[Componente/Módulo]:**
- Por que frágil: [O que o faz quebrar facilmente]
- Falhas comuns: [O que tipicamente dá errado]
- Modificação segura: [Como alterá-lo sem quebrar]
- Cobertura de testes: [Está testado? Lacunas?]

**[Componente/Módulo]:**
- Por que frágil: [O que o faz quebrar facilmente]
- Falhas comuns: [O que tipicamente dá errado]
- Modificação segura: [Como alterá-lo sem quebrar]
- Cobertura de testes: [Está testado? Lacunas?]

## Limites de Escala

**[Recurso/Sistema]:**
- Capacidade atual: [Números: "100 req/seg", "10k usuários"]
- Limite: [Onde quebra]
- Sintomas no limite: [O que acontece]
- Caminho de escala: [Como aumentar a capacidade]

## Dependências em Risco

**[Pacote/Serviço]:**
- Risco: [ex.: "obsoleto", "sem manutenção", "mudanças disruptivas chegando"]
- Impacto: [O que quebra se falhar]
- Plano de migração: [Alternativa ou caminho de atualização]

## Funcionalidades Críticas Ausentes

**[Lacuna de funcionalidade]:**
- Problema: [O que está faltando]
- Solução alternativa atual: [Como os usuários contornam]
- Bloqueia: [O que não pode ser feito sem isso]
- Complexidade de implementação: [Estimativa de esforço]

## Lacunas na Cobertura de Testes

**[Área não testada]:**
- O que não está testado: [Funcionalidade específica]
- Risco: [O que pode quebrar despercebido]
- Prioridade: [Alta/Média/Baixa]
- Dificuldade de testar: [Por que ainda não está testado]

---

*Auditoria de preocupações: [data]*
*Atualize quando os problemas forem corrigidos ou novos forem descobertos*
```

<good_examples>
```markdown
# Preocupações do Codebase

**Data da Análise:** 2025-01-20

## Dívida Técnica

**Consultas ao banco de dados em componentes React:**
- Problema: Consultas diretas ao Supabase em 15+ componentes de página em vez de server actions
- Arquivos: `app/dashboard/page.tsx`, `app/profile/page.tsx`, `app/courses/[id]/page.tsx`, `app/settings/page.tsx` (e 11 mais em `app/`)
- Por quê: Prototipagem rápida durante a fase de MVP
- Impacto: Impossível implementar RLS adequadamente, expõe a estrutura do BD ao cliente
- Abordagem de correção: Mover todas as consultas para server actions em `app/actions/`, adicionar políticas RLS adequadas

**Validação manual de assinatura de webhook:**
- Problema: Código de verificação de webhook Stripe copiado e colado em 3 endpoints diferentes
- Arquivos: `app/api/webhooks/stripe/route.ts`, `app/api/webhooks/checkout/route.ts`, `app/api/webhooks/subscription/route.ts`
- Por quê: Cada webhook adicionado ad-hoc sem abstração
- Impacto: Fácil de esquecer a verificação em novos webhooks (risco de segurança)
- Abordagem de correção: Criar middleware compartilhado `lib/stripe/validate-webhook.ts`

## Bugs Conhecidos

**Race condition em atualizações de assinatura:**
- Sintomas: Usuário aparece como nível "free" por 5-10 segundos após pagamento bem-sucedido
- Gatilho: Navegação rápida após redirecionamento do checkout Stripe, antes do processamento do webhook
- Arquivos: `app/checkout/success/page.tsx` (handler de redirecionamento), `app/api/webhooks/stripe/route.ts` (webhook)
- Solução alternativa: O webhook do Stripe eventualmente atualiza o status (auto-corrige)
- Causa raiz: Processamento de webhook mais lento que a navegação do usuário, sem atualização otimista da UI
- Correção: Adicionar polling em `app/checkout/success/page.tsx` após o redirecionamento

**Estado de sessão inconsistente após logout:**
- Sintomas: Usuário redirecionado para /dashboard após logout em vez de /login
- Gatilho: Logout via botão no nav mobile (desktop funciona bem)
- Arquivo: `components/MobileNav.tsx` (linha ~45, handler de logout)
- Solução alternativa: Navegação manual para /login via URL funciona
- Causa raiz: Componente de nav mobile não aguarda supabase.auth.signOut()
- Correção: Adicionar await ao handler de logout em `components/MobileNav.tsx`

## Considerações de Segurança

**Verificação de role de admin apenas no cliente:**
- Risco: Páginas do painel admin verificam isAdmin do cliente Supabase, sem verificação no servidor
- Arquivos: `app/admin/page.tsx`, `app/admin/users/page.tsx`, `components/AdminGuard.tsx`
- Mitigação atual: Nenhuma (dependendo da ocultação na UI)
- Recomendações: Adicionar middleware nas rotas admin em `middleware.ts`, verificar role no servidor

**Uploads de arquivo sem validação:**
- Risco: Usuários podem enviar qualquer tipo de arquivo para o bucket de avatar (sem validação de tamanho/tipo)
- Arquivo: `components/AvatarUpload.tsx` (handler de upload)
- Mitigação atual: Bucket Supabase limita a 2MB (configurado no painel)
- Recomendações: Adicionar validação de tipo de arquivo (apenas image/*) em `lib/storage/validate.ts`

## Gargalos de Performance

**Endpoint /api/courses:**
- Problema: Buscando todos os cursos com aulas e autores aninhados
- Arquivo: `app/api/courses/route.ts`
- Medição: 1,2s p95 de tempo de resposta com mais de 50 cursos
- Causa: Padrão N+1 (consulta separada por curso para as aulas)
- Caminho de melhoria: Usar include do Prisma para carregar as aulas com eager loading em `lib/db/courses.ts`, adicionar cache Redis

**Carregamento inicial do dashboard:**
- Problema: Cascata de 5 chamadas de API seriais no mount
- Arquivo: `app/dashboard/page.tsx`
- Medição: 3,5s até interativo em 3G lento
- Causa: Cada componente busca seus próprios dados independentemente
- Caminho de melhoria: Converter para Server Component com uma única busca paralela

## Áreas Frágeis

**Cadeia de middleware de autenticação:**
- Arquivo: `middleware.ts`
- Por que frágil: 4 funções de middleware diferentes rodam em ordem específica (auth → role → subscription → logging)
- Falhas comuns: Mudança na ordem do middleware quebra tudo, difícil de depurar
- Modificação segura: Adicionar testes antes de mudar a ordem, documentar dependências em comentários
- Cobertura de testes: Sem testes de integração para a cadeia de middleware (apenas testes unitários)

**Tratamento de eventos webhook Stripe:**
- Arquivo: `app/api/webhooks/stripe/route.ts`
- Por que frágil: Giant switch statement com 12 tipos de evento, lógica de transação compartilhada
- Falhas comuns: Novo tipo de evento adicionado sem tratamento, atualizações parciais do BD em caso de erro
- Modificação segura: Extrair cada handler de evento para `lib/stripe/handlers/*.ts`
- Cobertura de testes: Apenas 3 dos 12 tipos de evento têm testes

## Limites de Escala

**Plano Gratuito do Supabase:**
- Capacidade atual: 500MB de banco de dados, 1GB de armazenamento de arquivos, 2GB de largura de banda/mês
- Limite: ~5.000 usuários estimados antes de atingir os limites
- Sintomas no limite: Erros 429 de rate limit, escritas no BD falham
- Caminho de escala: Atualizar para Pro ($25/mês) expande para 8GB de BD, 100GB de armazenamento

**Renderização bloqueante no servidor:**
- Capacidade atual: ~50 usuários simultâneos antes de lentidão
- Limite: Plano Hobby do Vercel (timeout de função de 10s, 100GB-hrs/mês)
- Sintomas no limite: Timeouts 504 do gateway em páginas de cursos
- Caminho de escala: Atualizar para Vercel Pro ($20/mês), adicionar cache de edge

## Dependências em Risco

**react-hot-toast:**
- Risco: Sem manutenção (última atualização há 18 meses), compatibilidade com React 19 desconhecida
- Impacto: Notificações toast quebram, sem degradação elegante
- Plano de migração: Trocar por sonner (ativamente mantido, API similar)

## Funcionalidades Críticas Ausentes

**Tratamento de falhas de pagamento:**
- Problema: Sem mecanismo de tentativa novamente ou notificação ao usuário quando o pagamento de assinatura falha
- Solução alternativa atual: Usuários reinserem as informações de pagamento manualmente (se perceberem)
- Bloqueia: Não é possível reter usuários com cartões expirados, sem processo de dunning
- Complexidade de implementação: Média (webhooks Stripe + fluxo de e-mail + UI)

**Rastreamento de progresso do curso:**
- Problema: Sem estado persistente para quais aulas foram concluídas
- Solução alternativa atual: Usuários rastreiam o progresso manualmente
- Bloqueia: Impossível mostrar percentual de conclusão, impossível recomendar a próxima aula
- Complexidade de implementação: Baixa (adicionar tabela de junção completed_lessons)

## Lacunas na Cobertura de Testes

**Fluxo de pagamento ponta a ponta:**
- O que não está testado: Fluxo completo de checkout Stripe → webhook → ativação de assinatura
- Risco: Processamento de pagamentos pode quebrar silenciosamente (aconteceu duas vezes)
- Prioridade: Alta
- Dificuldade de testar: Necessidade de fixtures de teste do Stripe e configuração de simulação de webhook

**Comportamento de error boundary:**
- O que não está testado: Como o aplicativo se comporta quando componentes lançam erros
- Risco: Tela branca da morte para usuários, sem relatório de erros
- Prioridade: Média
- Dificuldade de testar: Necessidade de acionar erros intencionalmente no ambiente de teste

---

*Auditoria de preocupações: 2025-01-20*
*Atualize quando os problemas forem corrigidos ou novos forem descobertos*
```
</good_examples>

<guidelines>
**O que pertence ao CONCERNS.md:**
- Dívida técnica com impacto claro e abordagem de correção
- Bugs conhecidos com etapas de reprodução
- Lacunas de segurança e recomendações de mitigação
- Gargalos de performance com medições
- Código frágil que quebra facilmente
- Limites de escala com números
- Dependências que precisam de atenção
- Funcionalidades ausentes que bloqueiam fluxos de trabalho
- Lacunas na cobertura de testes

**O que NÃO pertence aqui:**
- Opiniões sem evidências ("o código está bagunçado")
- Reclamações sem soluções ("auth é péssimo")
- Ideias de funcionalidades futuras (isso é para o planejamento de produto)
- TODOs normais (esses ficam nos comentários de código)
- Decisões arquiteturais que estão funcionando bem
- Problemas menores de estilo de código

**Ao preencher este template:**
- **Sempre inclua caminhos de arquivo** - Preocupações sem localizações não são acionáveis. Use backticks: `src/file.ts`
- Seja específico com medições ("500ms p95" e não "lento")
- Inclua etapas de reprodução para bugs
- Sugira abordagens de correção, não apenas problemas
- Foque em itens acionáveis
- Priorize por risco/impacto
- Atualize quando os problemas forem resolvidos
- Adicione novas preocupações conforme descobertas

**Diretrizes de tom:**
- Profissional, não emocional ("padrão N+1" e não "consultas terríveis")
- Orientado a soluções ("Correção: adicionar índice" e não "precisa de correção")
- Focado em risco ("Pode expor dados do usuário" e não "a segurança é ruim")
- Factual ("3,5s de tempo de carregamento" e não "muito lento")

**Útil para planejamento de fase quando:**
- Decidindo o que trabalhar a seguir
- Estimando o risco de mudanças
- Entendendo onde ser cuidadoso
- Priorizando melhorias
- Integrando novos contextos do Claude
- Planejando trabalho de refatoração

**Como isso é preenchido:**
Os agentes de exploração detectam esses problemas durante o mapeamento do codebase. Adições manuais são bem-vindas para problemas descobertos por humanos. Esta é documentação viva, não uma lista de reclamações.
</guidelines>
