# Modo de Fechamento de Lacunas — Referência do Planejador

Ativado pela flag `--gaps`. Cria planos para resolver falhas de verificação ou de UAT.

**Importante: Ignore itens adiados.** Ao ler VERIFICATION.md, apenas a seção `gaps:` contém itens acionáveis que precisam de planos de fechamento. A seção `deferred:` (se presente) lista itens explicitamente endereçados em fases futuras do milestone — esses NÃO são lacunas e devem ser ignorados durante o planejamento de fechamento. Criar planos para itens adiados desperdiça esforço em trabalho já agendado para fases futuras.

**1. Encontre as fontes das lacunas:**

Use o contexto de inicialização (de load_project_state) que fornece `phase_dir`:

```bash
# Verificar VERIFICATION.md (lacunas de verificação de código)
ls "$phase_dir"/*-VERIFICATION.md 2>/dev/null

# Verificar UAT.md com status diagnosticado (lacunas de teste de usuário)
grep -l "status: diagnosed" "$phase_dir"/*-UAT.md 2>/dev/null
```

**2. Analise as lacunas:** Cada lacuna tem: truth (comportamento com falha), reason, artifacts (arquivos com problemas), missing (coisas a adicionar/corrigir).

**3. Carregue os SUMMARYs existentes** para entender o que já foi construído.

**4. Encontre o próximo número de plano:** Se os planos 01-03 existem, o próximo é 04.

**5. Agrupe as lacunas em planos** por: mesmo artifact, mesma preocupação, ordem de dependência (não é possível conectar se o artifact é um stub → corrija o stub primeiro).

**6. Crie tarefas de fechamento de lacunas:**

```xml
<task name="{fix_description}" type="auto">
  <files>{artifact.path}</files>
  <action>
    {Para cada item em gap.missing:}
    - {missing item}

    Referenciar código existente: {from SUMMARYs}
    Motivo da lacuna: {gap.reason}
  </action>
  <verify>{Como confirmar que a lacuna foi fechada}</verify>
  <done>{Verdade observável agora alcançável}</done>
</task>
```

**7. Atribua waves usando análise padrão de dependências** (igual ao passo `assign_waves`):
- Planos sem dependências → wave 1
- Planos que dependem de outros planos de fechamento de lacunas → max(waves de dependência) + 1
- Considere também dependências em planos existentes (não-lacuna) da fase

**8. Escreva os arquivos PLAN.md:**

```yaml
---
phase: XX-name
plan: NN              # Sequencial após os existentes
type: execute
wave: N               # Calculado a partir de depends_on (veja assign_waves)
depends_on: [...]     # Outros planos dos quais este depende (lacuna ou existente)
files_modified: [...]
autonomous: true
gap_closure: true     # Flag de rastreamento
---
```
