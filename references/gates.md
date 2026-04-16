# Taxonomia de Gates

Tipos canônicos de gates usados nos fluxos de trabalho do GSD. Todo checkpoint de validação se enquadra em um destes quatro tipos.

---

## Tipos de Gates

### Gate de Pré-voo (Pre-flight)
**Propósito:** Valida pré-condições antes de iniciar uma operação.
**Comportamento:** Bloqueia a entrada se as condições não forem atendidas. Nenhum trabalho parcial é criado.
**Recuperação:** Corrija a pré-condição ausente e tente novamente.
**Exemplos:**
- plan-phase verifica a existência de REQUIREMENTS.md antes de planejar
- execute-phase valida se PLAN.md existe antes da execução
- discuss-phase confirma que a fase existe em ROADMAP.md

### Gate de Revisão (Revision)
**Propósito:** Avalia a qualidade da saída e encaminha para revisão se for insuficiente.
**Comportamento:** Retorna ao produtor com feedback específico. Limitado por um teto de iterações.
**Recuperação:** O produtor trata o feedback; o verificador reavalia. O loop também escala antecipadamente se a contagem de problemas não diminuir entre iterações consecutivas (detecção de estagnação). Após o máximo de iterações, escala incondicionalmente.
**Exemplos:**
- Plan-checker revisando PLAN.md (máximo 3 iterações)
- Verificador conferindo entregas da fase em relação aos critérios de sucesso

### Gate de Escalação (Escalation)
**Propósito:** Expõe problemas irresolvíveis ao desenvolvedor para uma decisão.
**Comportamento:** Pausa o fluxo de trabalho, apresenta opções, aguarda a entrada humana.
**Recuperação:** O desenvolvedor escolhe uma ação; o fluxo de trabalho retoma no caminho selecionado.
**Exemplos:**
- Loop de revisão esgotado após 3 iterações
- Conflito de merge durante a limpeza da worktree
- Requisito ambíguo que precisa de esclarecimento

### Gate de Aborto (Abort)
**Propósito:** Encerra a operação para evitar danos ou desperdício.
**Comportamento:** Para imediatamente, preserva o estado, reporta o motivo.
**Recuperação:** O desenvolvedor investiga a causa raiz, corrige e reinicia a partir do checkpoint.
**Exemplos:**
- Janela de contexto criticamente baixa durante a execução
- STATE.md em estado de erro bloqueando /gsd-next
- Verificação encontra entregas críticas ausentes

---

## Matriz de Gates

| Fluxo de Trabalho | Fase | Tipo de Gate | Artefatos Verificados | Comportamento em Falha |
|-------------------|------|--------------|----------------------|------------------------|
| plan-phase | Entrada | Pre-flight | REQUIREMENTS.md, ROADMAP.md | Bloquear com mensagem de arquivo ausente |
| plan-phase | Passo 12 | Revision | Qualidade do PLAN.md | Loop para o planejador (máx. 3) |
| plan-phase | Pós-revisão | Escalation | Problemas não resolvidos | Expor ao desenvolvedor |
| execute-phase | Entrada | Pre-flight | PLAN.md | Bloquear com mensagem de plano ausente |
| execute-phase | Conclusão | Revision | Completude do SUMMARY.md | Reexecutar tarefas incompletas |
| verify-work | Entrada | Pre-flight | SUMMARY.md | Bloquear com mensagem de resumo ausente |
| verify-work | Avaliação | Escalation | Critérios com falha | Expor lacunas ao desenvolvedor |
| next | Entrada | Abort | Estado de erro, checkpoints | Parar com diagnóstico |

---

## Implementando Gates

Use esta taxonomia ao projetar ou auditar pontos de validação de fluxos de trabalho:

- Gates **Pre-flight** pertencem aos pontos de entrada do fluxo de trabalho. São verificações baratas e determinísticas que evitam trabalho desperdiçado. Se você pode verificar uma pré-condição com uma verificação de existência de arquivo ou uma leitura de configuração, use um gate de pré-voo.
- Gates **Revision** pertencem após uma etapa do produtor onde a qualidade varia. Sempre os acompanhe com um teto de iterações para evitar loops infinitos. O teto deve refletir o custo de cada iteração — operações caras recebem menos tentativas.
- Gates **Escalation** pertencem onde a resolução automatizada é impossível ou ambígua. São a válvula de segurança entre os loops de revisão e o aborto. Apresente ao desenvolvedor opções claras e contexto suficiente para decidir.
- Gates **Abort** pertencem a pontos onde continuar causaria danos, desperdiçaria recursos significativos ou produziria saída sem sentido. Eles devem preservar o estado para que o trabalho possa ser retomado após a correção da causa raiz.

**Heurística de seleção:** Comece com pre-flight. Se a verificação ocorre após a produção do trabalho, é um gate de revisão. Se o loop de revisão não consegue resolver o problema, escale. Se continuar é perigoso, aborte.
