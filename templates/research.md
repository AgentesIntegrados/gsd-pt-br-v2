# Template de Pesquisa

Template para `.planning/phases/XX-name/{phase_num}-RESEARCH.md` - pesquisa abrangente do ecossistema antes do planejamento.

**Propósito:** Documentar o que Claude precisa saber para implementar uma fase bem — não apenas "qual biblioteca", mas "como especialistas constroem isso".

---

## Template do Arquivo

```markdown
# Fase [X]: [Nome] - Pesquisa

**Pesquisada em:** [data]
**Domínio:** [tecnologia principal/domínio do problema]
**Confiança:** [ALTA/MÉDIA/BAIXA]

<user_constraints>
## Restrições do Usuário (do CONTEXT.md)

**CRÍTICO:** Se o CONTEXT.md existir do /gsd-discuss-phase, copie as decisões bloqueadas aqui verbatim. Estas DEVEM ser respeitadas pelo planejador.

### Decisões Bloqueadas
[Copie da seção `## Decisions` do CONTEXT.md - estas são NÃO-NEGOCIÁVEIS]
- [Decisão 1]
- [Decisão 2]

### Critério do Claude
[Copie do CONTEXT.md - áreas onde o pesquisador/planejador pode escolher]
- [Área 1]
- [Área 2]

### Ideias Adiadas (FORA DO ESCOPO)
[Copie do CONTEXT.md - NÃO pesquise ou planeje estas]
- [Adiada 1]
- [Adiada 2]

**Se não houver CONTEXT.md:** Escreva "Sem restrições do usuário - todas as decisões a critério do Claude"
</user_constraints>

<architectural_responsibility_map>
## Mapa de Responsabilidade Arquitetural

Mapeie cada capacidade da fase para seu proprietário de camada arquitetural padrão antes de mergulhar na pesquisa de frameworks. Isso evita que erros de atribuição de camada se propaguem para os planos.

| Capacidade | Camada Primária | Camada Secundária | Justificativa |
|------------|-----------------|-------------------|---------------|
| [capacidade da descrição da fase] | [Navegador/Cliente, Servidor Frontend, API/Backend, CDN/Estático, ou Banco de Dados/Armazenamento] | [camada secundária ou —] | [por que esta camada é responsável] |

**Se aplicação de camada única:** Escreva "Aplicação de camada única — todas as capacidades residem em [camada]" e omita a tabela.
</architectural_responsibility_map>

<research_summary>
## Resumo

[Resumo executivo de 2-3 parágrafos]
- O que foi pesquisado
- Qual é a abordagem padrão
- Principais recomendações

**Recomendação principal:** [orientação acionável em uma linha]
</research_summary>

<standard_stack>
## Stack Padrão

As bibliotecas/ferramentas estabelecidas para este domínio:

### Principais
| Biblioteca | Versão | Propósito | Por Que É Padrão |
|------------|--------|-----------|-----------------|
| [nome] | [ver] | [o que faz] | [por que especialistas usam] |
| [nome] | [ver] | [o que faz] | [por que especialistas usam] |

### Suporte
| Biblioteca | Versão | Propósito | Quando Usar |
|------------|--------|-----------|-------------|
| [nome] | [ver] | [o que faz] | [caso de uso] |
| [nome] | [ver] | [o que faz] | [caso de uso] |

### Alternativas Consideradas
| Em vez de | Poderia Usar | Compensação |
|-----------|--------------|-------------|
| [padrão] | [alternativa] | [quando a alternativa faz sentido] |

**Instalação:**
```bash
npm install [pacotes]
# ou
yarn add [pacotes]
```
</standard_stack>

<architecture_patterns>
## Padrões de Arquitetura

### Diagrama de Arquitetura do Sistema

Os diagramas de arquitetura DEVEM mostrar o fluxo de dados pelos componentes conceituais, não listagens de arquivos.

Requisitos:
- Mostrar pontos de entrada (como dados/requisições entram no sistema)
- Mostrar estágios de processamento (quais transformações acontecem, em qual ordem)
- Mostrar pontos de decisão e caminhos de ramificação
- Mostrar dependências externas e limites de serviço
- Usar setas para indicar a direção do fluxo de dados
- Um leitor deve conseguir rastrear o caso de uso principal de entrada a saída seguindo as setas

O mapeamento arquivo-para-implementação pertence à tabela de Responsabilidades dos Componentes, não ao diagrama.

### Estrutura de Projeto Recomendada
```
src/
├── [pasta]/        # [propósito]
├── [pasta]/        # [propósito]
└── [pasta]/        # [propósito]
```

### Padrão 1: [Nome do Padrão]
**O que é:** [descrição]
**Quando usar:** [condições]
**Exemplo:**
```typescript
// [exemplo de código do Context7/documentação oficial]
```

### Padrão 2: [Nome do Padrão]
**O que é:** [descrição]
**Quando usar:** [condições]
**Exemplo:**
```typescript
// [exemplo de código]
```

### Anti-Padrões a Evitar
- **[Anti-padrão]:** [por que é ruim, o que fazer em vez disso]
- **[Anti-padrão]:** [por que é ruim, o que fazer em vez disso]
</architecture_patterns>

<dont_hand_roll>
## Não Construa do Zero

Problemas que parecem simples, mas têm soluções existentes:

| Problema | Não Construa | Use Em Vez Disso | Por Quê |
|----------|--------------|------------------|---------|
| [problema] | [o que você construiria] | [biblioteca] | [casos extremos, complexidade] |
| [problema] | [o que você construiria] | [biblioteca] | [casos extremos, complexidade] |
| [problema] | [o que você construiria] | [biblioteca] | [casos extremos, complexidade] |

**Insight chave:** [por que soluções customizadas são piores neste domínio]
</dont_hand_roll>

<common_pitfalls>
## Armadilhas Comuns

### Armadilha 1: [Nome]
**O que dá errado:** [descrição]
**Por que acontece:** [causa raiz]
**Como evitar:** [estratégia de prevenção]
**Sinais de alerta:** [como detectar cedo]

### Armadilha 2: [Nome]
**O que dá errado:** [descrição]
**Por que acontece:** [causa raiz]
**Como evitar:** [estratégia de prevenção]
**Sinais de alerta:** [como detectar cedo]

### Armadilha 3: [Nome]
**O que dá errado:** [descrição]
**Por que acontece:** [causa raiz]
**Como evitar:** [estratégia de prevenção]
**Sinais de alerta:** [como detectar cedo]
</common_pitfalls>

<code_examples>
## Exemplos de Código

Padrões verificados de fontes oficiais:

### [Operação Comum 1]
```typescript
// Fonte: [Context7/URL da documentação oficial]
[código]
```

### [Operação Comum 2]
```typescript
// Fonte: [Context7/URL da documentação oficial]
[código]
```

### [Operação Comum 3]
```typescript
// Fonte: [Context7/URL da documentação oficial]
[código]
```
</code_examples>

<sota_updates>
## Estado da Arte (2024-2025)

O que mudou recentemente:

| Abordagem Antiga | Abordagem Atual | Quando Mudou | Impacto |
|------------------|-----------------|--------------|---------|
| [antiga] | [nova] | [data/versão] | [o que significa para a implementação] |

**Novas ferramentas/padrões a considerar:**
- [Ferramenta/Padrão]: [o que habilita, quando usar]
- [Ferramenta/Padrão]: [o que habilita, quando usar]

**Obsoleto/desatualizado:**
- [Coisa]: [por que está desatualizada, o que a substituiu]
</sota_updates>

<open_questions>
## Perguntas Abertas

Coisas que não puderam ser totalmente resolvidas:

1. **[Pergunta]**
   - O que sabemos: [informação parcial]
   - O que não está claro: [a lacuna]
   - Recomendação: [como lidar durante o planejamento/execução]

2. **[Pergunta]**
   - O que sabemos: [informação parcial]
   - O que não está claro: [a lacuna]
   - Recomendação: [como lidar]
</open_questions>

<sources>
## Fontes

### Primárias (confiança ALTA)
- [ID da biblioteca Context7] - [tópicos consultados]
- [URL da documentação oficial] - [o que foi verificado]

### Secundárias (confiança MÉDIA)
- [WebSearch verificada com fonte oficial] - [descoberta + verificação]

### Terciárias (confiança BAIXA - precisa de validação)
- [Apenas WebSearch] - [descoberta, marcada para validação durante a implementação]
</sources>

<metadata>
## Metadados

**Escopo da pesquisa:**
- Tecnologia principal: [o quê]
- Ecossistema: [bibliotecas exploradas]
- Padrões: [padrões pesquisados]
- Armadilhas: [áreas verificadas]

**Detalhamento de confiança:**
- Stack padrão: [ALTA/MÉDIA/BAIXA] - [motivo]
- Arquitetura: [ALTA/MÉDIA/BAIXA] - [motivo]
- Armadilhas: [ALTA/MÉDIA/BAIXA] - [motivo]
- Exemplos de código: [ALTA/MÉDIA/BAIXA] - [motivo]

**Data da pesquisa:** [data]
**Válida até:** [estimativa - 30 dias para tecnologia estável, 7 dias para tecnologia em rápida evolução]
</metadata>

---

*Fase: XX-name*
*Pesquisa concluída em: [data]*
*Pronta para planejamento: [sim/não]*
```

---

## Bom Exemplo

```markdown
# Fase 3: Direção em Cidade 3D - Pesquisa

**Pesquisada em:** 2025-01-20
**Domínio:** Jogo web 3D com Three.js e mecânicas de direção
**Confiança:** ALTA

<research_summary>
## Resumo

Pesquisado o ecossistema Three.js para construir um jogo de direção em cidade 3D. A abordagem padrão usa Three.js com React Three Fiber para arquitetura de componentes, Rapier para física e drei para helpers comuns.

Descoberta chave: Não construa física ou detecção de colisão do zero. Rapier (via @react-three/rapier) lida com física de veículos, colisão com terreno e interações com objetos da cidade eficientemente. Código de física customizado leva a bugs e problemas de performance.

**Recomendação principal:** Use a stack R3F + Rapier + drei. Comece com o controlador de veículo do drei, adicione física de veículo Rapier, construa a cidade com malhas instanciadas para performance.
</research_summary>

<standard_stack>
## Stack Padrão

### Principais
| Biblioteca | Versão | Propósito | Por Que É Padrão |
|------------|--------|-----------|-----------------|
| three | 0.160.0 | Renderização 3D | O padrão para 3D na web |
| @react-three/fiber | 8.15.0 | Renderer React para Three.js | 3D declarativo, melhor DX |
| @react-three/drei | 9.92.0 | Helpers e abstrações | Resolve problemas comuns |
| @react-three/rapier | 1.2.1 | Bindings do motor de física | Melhor física para R3F |

### Suporte
| Biblioteca | Versão | Propósito | Quando Usar |
|------------|--------|-----------|-------------|
| @react-three/postprocessing | 2.16.0 | Efeitos visuais | Bloom, DOF, motion blur |
| leva | 0.9.35 | UI de debug | Ajuste de parâmetros |
| zustand | 4.4.7 | Gerenciamento de estado | Estado do jogo, estado da UI |
| use-sound | 4.0.1 | Áudio | Sons do motor, ambiente |

### Alternativas Consideradas
| Em vez de | Poderia Usar | Compensação |
|-----------|--------------|-------------|
| Rapier | Cannon.js | Cannon mais simples, mas menos performático para veículos |
| R3F | Vanilla Three | Vanilla se sem React, mas DX do R3F é muito melhor |
| drei | Helpers customizados | drei é battle-tested, não reinvente |

**Instalação:**
```bash
npm install three @react-three/fiber @react-three/drei @react-three/rapier zustand
```
</standard_stack>

<architecture_patterns>
## Padrões de Arquitetura

### Estrutura de Projeto Recomendada
```
src/
├── components/
│   ├── Vehicle/          # Carro do jogador com física
│   ├── City/             # Geração de cidade e prédios
│   ├── Road/             # Rede de estradas
│   └── Environment/      # Céu, iluminação, névoa
├── hooks/
│   ├── useVehicleControls.ts
│   └── useGameState.ts
├── stores/
│   └── gameStore.ts      # Estado Zustand
└── utils/
    └── cityGenerator.ts  # Helpers de geração procedural
```

### Padrão 1: Veículo com Física Rapier
**O que é:** Use RigidBody com configurações específicas de veículo, não física customizada
**Quando usar:** Qualquer veículo terrestre
**Exemplo:**
```typescript
// Fonte: docs @react-three/rapier
import { RigidBody, useRapier } from '@react-three/rapier'

function Vehicle() {
  const rigidBody = useRef()

  return (
    <RigidBody
      ref={rigidBody}
      type="dynamic"
      colliders="hull"
      mass={1500}
      linearDamping={0.5}
      angularDamping={0.5}
    >
      <mesh>
        <boxGeometry args={[2, 1, 4]} />
        <meshStandardMaterial />
      </mesh>
    </RigidBody>
  )
}
```

### Padrão 2: Malhas Instanciadas para a Cidade
**O que é:** Use InstancedMesh para objetos repetidos (prédios, árvores, adereços)
**Quando usar:** Mais de 100 objetos similares
**Exemplo:**
```typescript
// Fonte: docs drei
import { Instances, Instance } from '@react-three/drei'

function Buildings({ positions }) {
  return (
    <Instances limit={1000}>
      <boxGeometry />
      <meshStandardMaterial />
      {positions.map((pos, i) => (
        <Instance key={i} position={pos} scale={[1, Math.random() * 5 + 1, 1]} />
      ))}
    </Instances>
  )
}
```

### Anti-Padrões a Evitar
- **Criar malhas no loop de renderização:** Crie uma vez, atualize apenas transformações
- **Não usar InstancedMesh:** Malhas individuais para prédios prejudica a performance
- **Física matemática customizada:** Rapier lida melhor, sempre
</architecture_patterns>

<dont_hand_roll>
## Não Construa do Zero

| Problema | Não Construa | Use Em Vez Disso | Por Quê |
|----------|--------------|------------------|---------|
| Física de veículos | Velocidade/aceleração customizadas | Rapier RigidBody | Atrito de rodas, suspensão, colisões são complexos |
| Detecção de colisão | Raycasting manual | Rapier colliders | Performance, casos extremos, tunneling |
| Câmera seguindo | Lerp manual | drei CameraControls ou customizado com useFrame | Interpolação suave, limites |
| Geração de cidade | Posicionamento puramente aleatório | Grade com ruído para variação | Aleatório parece errado, grade é previsível |
| LOD | Verificações manuais de distância | drei <Detailed> | Lida com transições, histerese |

**Insight chave:** Desenvolvimento de jogos 3D tem mais de 40 anos de problemas resolvidos. Rapier implementa simulação de física adequada. drei implementa helpers 3D adequados. Lutar contra eles leva a bugs que parecem problemas de "sensação do jogo", mas são na verdade casos extremos de física.
</dont_hand_roll>

<common_pitfalls>
## Armadilhas Comuns

### Armadilha 1: Tunneling Físico
**O que dá errado:** Objetos rápidos passam através de paredes
**Por que acontece:** Passo de física padrão muito grande para a velocidade
**Como evitar:** Use CCD (Detecção de Colisão Contínua) no Rapier
**Sinais de alerta:** Objetos aparecendo aleatoriamente fora de prédios

### Armadilha 2: Morte de Performance por Draw Calls
**O que dá errado:** Jogo trava com muitos prédios
**Por que acontece:** Cada malha = 1 draw call, centenas de prédios = centenas de chamadas
**Como evitar:** InstancedMesh para objetos similares, unir geometria estática
**Sinais de alerta:** Limitado pela GPU, FPS baixo apesar de cena simples

### Armadilha 3: Sensação "Flutuante" do Veículo
**O que dá errado:** O carro não parece estar no chão
**Por que acontece:** Falta simulação adequada de rodas/suspensão
**Como evitar:** Use o controlador de veículo Rapier ou ajuste cuidadosamente massa/amortecimento
**Sinais de alerta:** Carro quica de forma estranha, não adere nas curvas
</common_pitfalls>

<code_examples>
## Exemplos de Código

### Configuração Básica R3F + Rapier
```typescript
// Fonte: primeiros passos @react-three/rapier
import { Canvas } from '@react-three/fiber'
import { Physics } from '@react-three/rapier'

function Game() {
  return (
    <Canvas>
      <Physics gravity={[0, -9.81, 0]}>
        <Vehicle />
        <City />
        <Ground />
      </Physics>
    </Canvas>
  )
}
```

### Hook de Controles de Veículo
```typescript
// Fonte: Padrão da comunidade, verificado com docs drei
import { useFrame } from '@react-three/fiber'
import { useKeyboardControls } from '@react-three/drei'

function useVehicleControls(rigidBodyRef) {
  const [, getKeys] = useKeyboardControls()

  useFrame(() => {
    const { forward, back, left, right } = getKeys()
    const body = rigidBodyRef.current
    if (!body) return

    const impulse = { x: 0, y: 0, z: 0 }
    if (forward) impulse.z -= 10
    if (back) impulse.z += 5

    body.applyImpulse(impulse, true)

    if (left) body.applyTorqueImpulse({ x: 0, y: 2, z: 0 }, true)
    if (right) body.applyTorqueImpulse({ x: 0, y: -2, z: 0 }, true)
  })
}
```
</code_examples>

<sota_updates>
## Estado da Arte (2024-2025)

| Abordagem Antiga | Abordagem Atual | Quando Mudou | Impacto |
|------------------|-----------------|--------------|---------|
| cannon-es | Rapier | 2023 | Rapier é mais rápido, melhor mantido |
| vanilla Three.js | React Three Fiber | 2020+ | R3F agora é padrão para apps React |
| InstancedMesh manual | drei <Instances> | 2022 | API mais simples, lida com atualizações |

**Novas ferramentas/padrões a considerar:**
- **WebGPU:** Chegando, mas ainda não pronto para produção em jogos (2025)
- **drei Gltf helpers:** <useGLTF.preload> para telas de carregamento

**Obsoleto/desatualizado:**
- **cannon.js (original):** Use o fork cannon-es ou, melhor ainda, Rapier
- **Raycasting manual para física:** Use apenas os colliders do Rapier
</sota_updates>

<sources>
## Fontes

### Primárias (confiança ALTA)
- /pmndrs/react-three-fiber - primeiros passos, hooks, performance
- /pmndrs/drei - instances, controles, helpers
- /dimforge/rapier-js - configuração de física, física de veículos

### Secundárias (confiança MÉDIA)
- Threads do discourse do Three.js "city driving game" - padrões verificados com a documentação
- Repositório de exemplos R3F - código verificado que funciona

### Terciárias (confiança BAIXA - precisa de validação)
- Nenhuma - todas as descobertas verificadas
</sources>

<metadata>
## Metadados

**Escopo da pesquisa:**
- Tecnologia principal: Three.js + React Three Fiber
- Ecossistema: Rapier, drei, zustand
- Padrões: Física de veículos, instanciamento, geração de cidade
- Armadilhas: Performance, física, sensação

**Detalhamento de confiança:**
- Stack padrão: ALTA - verificada com Context7, amplamente utilizada
- Arquitetura: ALTA - de exemplos oficiais
- Armadilhas: ALTA - documentadas no discourse, verificadas na documentação
- Exemplos de código: ALTA - de fontes Context7/oficiais

**Data da pesquisa:** 2025-01-20
**Válida até:** 2025-02-20 (30 dias - ecossistema R3F estável)
</metadata>

---

*Fase: 03-city-driving*
*Pesquisa concluída em: 2025-01-20*
*Pronta para planejamento: sim*
```

---

## Diretrizes

**Quando criar:**
- Antes de planejar fases em domínios de nicho/complexos
- Quando os dados de treinamento do Claude provavelmente estão desatualizados ou esparsos
- Quando "como especialistas fazem isso" importa mais do que "qual biblioteca"

**Estrutura:**
- Use tags XML para marcadores de seção (corresponde aos templates GSD)
- Sete seções principais: summary, standard_stack, architecture_patterns, dont_hand_roll, common_pitfalls, code_examples, sources
- Todas as seções são obrigatórias (conduz pesquisa abrangente)

**Qualidade do conteúdo:**
- Stack padrão: Versões específicas, não apenas nomes
- Arquitetura: Inclua exemplos de código reais de fontes autorizadas
- Não construa do zero: Seja explícito sobre quais problemas NÃO resolver por conta própria
- Armadilhas: Inclua sinais de alerta, não apenas "não faça isso"
- Fontes: Marque os níveis de confiança honestamente

**Integração com o planejamento:**
- RESEARCH.md carregado como referência @context no PLAN.md
- Stack padrão informa as escolhas de biblioteca
- Não construa do zero evita soluções customizadas
- Armadilhas informam os critérios de verificação
- Exemplos de código podem ser referenciados nas ações de tarefas

**Após a criação:**
- Arquivo reside no diretório da fase: `.planning/phases/XX-name/{phase_num}-RESEARCH.md`
- Referenciado durante o fluxo de trabalho de planejamento
- plan-phase o carrega automaticamente quando presente
