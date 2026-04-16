# Padrões Comuns de Bugs

Lista de verificação de padrões frequentes de bugs para examinar antes de formular hipóteses. Ordenados por frequência. Verifique estes PRIMEIRO — eles cobrem ~80% dos bugs em todas as stacks de tecnologia.

<patterns>

## Acesso a Nulo / Indefinido

- **Acesso a propriedade nula** — acessar propriedade em `null` ou `undefined`, verificação de nulo ausente ou encadeamento opcional
- **Valor de retorno ausente** — função retorna `undefined` em vez do valor esperado, instrução `return` ausente ou branch errado
- **Desestruturação de nulo** — desestruturação de array/objeto em `null`/`undefined`, API retornou formato de erro em vez de dados
- **Parâmetro opcional sem padrão** — parâmetro opcional usado sem valor padrão, chamador omitiu o argumento

## Fora-por-Um / Limite

- **Limite de loop errado** — loop começa em 1 em vez de 0, ou termina em `length` em vez de `length - 1`
- **Erro de cerca** — "N itens precisam de N-1 separadores" contado incorretamente
- **Inclusivo vs. exclusivo** — limite de intervalo `<` vs. `<=`, índice final de slice/substring
- **Coleção vazia** — `.length === 0` passa para lógica que assume existência de itens

## Assíncrono / Temporização

- **await ausente** — função assíncrona chamada sem `await`, obtém objeto Promise em vez do valor resolvido
- **Condição de corrida** — duas operações assíncronas leem/escrevem o mesmo estado sem coordenação
- **Closure obsoleto** — callback captura valor antigo da variável, não o atual
- **Ordem de inicialização** — manipulador de evento dispara antes da configuração estar completa
- **Timer vazado** — timeout/interval não limpo, dispara após componente/contexto destruído

## Gerenciamento de Estado

- **Mutação compartilhada** — objeto/array modificado no lugar afeta outros consumidores
- **Renderização obsoleta** — estado atualizado mas UI não re-renderizada, gatilho reativo ausente ou referência errada
- **Estado do manipulador obsoleto** — closure captura estado no momento da vinculação, não o valor atual
- **Fonte dupla de verdade** — os mesmos dados armazenados em dois lugares, um fica fora de sincronia
- **Transição inválida** — máquina de estados permite transição sem condição de guarda

## Importação / Módulo

- **Dependência circular** — módulo A importa B, B importa A, um obtém `undefined`
- **Incompatibilidade de exportação** — export padrão vs. nomeado, `import X` vs. `import { X }`
- **Extensão errada** — `.js` vs. `.cjs` vs. `.mjs`, `.ts` vs. `.tsx`
- **Sensibilidade de maiúsculas no caminho** — funciona no Windows/macOS, falha no Linux
- **Extensão ausente** — ESM requer extensões de arquivo explícitas nas importações

## Tipo / Coerção

- **Comparação string vs. número** — `"5" > "10"` é `true` (lexicográfico), `5 > 10` é `false`
- **Coerção implícita** — `==` em vez de `===`, surpresas com truthy/falsy (`0`, `""`, `[]`)
- **Precisão numérica** — `0.1 + 0.2 !== 0.3`, inteiros grandes perdem precisão
- **Valor falsy válido** — valor é `0` ou `""` que é válido, mas falsy

## Ambiente / Configuração

- **Variável de ambiente ausente** — variável de ambiente ausente ou com valor errado em dev vs. prod vs. CI
- **Caminho hardcoded** — funciona em uma máquina, falha em outra
- **Conflito de porta** — porta já em uso, processo anterior ainda em execução
- **Permissão negada** — usuário/grupo diferente na implantação
- **Dependência ausente** — não está em package.json ou não instalada

## Forma dos Dados / Contrato de API

- **Formato de resposta alterado** — backend atualizado, frontend espera formato antigo
- **Tipo de contêiner errado** — array onde objeto é esperado ou vice-versa, `data` vs. `data.results` vs. `data[0]`
- **Campo obrigatório ausente** — campo obrigatório omitido no payload, backend retorna erro de validação
- **Incompatibilidade de formato de data** — string ISO vs. timestamp vs. string de localidade
- **Incompatibilidade de codificação** — UTF-8 vs. Latin-1, codificação de URL, entidades HTML

## Regex / String

- **lastIndex pegajoso** — flag `g` do regex com `.test()` depois `.exec()`, `lastIndex` não redefinido entre chamadas
- **Escape ausente** — `.` corresponde a qualquer caractere, `$` é especial, barra invertida precisa ser duplicada
- **Correspondência excessiva gulosa** — `.*` consome delimitadores, precisa de `.*?`
- **Tipo de aspas errado** — interpolação de string precisa de backticks para template literals

## Tratamento de Erros

- **Erro engolido** — `catch {}` vazio ou registra mas não relança/trata
- **Tipo de erro errado** — captura `Error` base quando tipo específico é necessário
- **Erro no manipulador** — código de limpeza lança exceção, mascarando o erro original
- **Rejeição não tratada** — `.catch()` ausente ou try/catch em torno de `await`

## Escopo / Closure

- **Sombreamento de variável** — escopo interno declara o mesmo nome, esconde variável externa
- **Captura de variável de loop** — todos os closures compartilham o mesmo `var i`, use `let` ou bind
- **Binding de `this` perdido** — callback perde contexto, precisa de `.bind()` ou função arrow
- **Confusão de escopo** — `var` elevado à função, `let`/`const` com escopo de bloco

</patterns>

<usage>

## Como Usar Esta Lista de Verificação

1. **Antes de formular qualquer hipótese**, examine as categorias relevantes com base no sintoma
2. **Combine o sintoma com o padrão** — se o bug envolve "undefined is not an object", verifique Nulo/Indefinido primeiro
3. **Cada padrão verificado é um candidato a hipótese** — confirme ou elimine com evidências
4. **Se nenhum padrão corresponder**, prossiga para investigação aberta

### Mapa Rápido de Sintoma para Categoria

| Sintoma | Verifique Primeiro |
|---------|------------|
| "Cannot read property of undefined/null" | Acesso a Nulo/Indefinido |
| "X is not a function" | Importação/Módulo, Tipo/Coerção |
| Funciona às vezes, falha às vezes | Assíncrono/Temporização, Gerenciamento de Estado |
| Funciona localmente, falha em CI/prod | Ambiente/Configuração |
| Dados errados exibidos | Forma dos Dados, Gerenciamento de Estado |
| Fora por um item / item final ausente | Fora-por-Um/Limite |
| "Unexpected token" / erro de parse | Forma dos Dados, Tipo/Coerção |
| Vazamento de memória / uso crescente de recursos | Assíncrono/Temporização (limpeza), Escopo/Closure |
| Loop infinito / max call stack | Gerenciamento de Estado, Assíncrono/Temporização |

</usage>
