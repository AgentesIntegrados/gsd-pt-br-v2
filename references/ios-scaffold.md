# Referência de Scaffold para Apps iOS

Regras e padrões para estruturar aplicativos iOS. Aplique quando qualquer plano envolver a criação de um novo alvo de app iOS.

---

## Regra Crítica: Nunca Use Package.swift como Sistema de Build Principal para Apps iOS

**NUNCA use `Package.swift` com `.executableTarget` (ou `.target`) para estruturar um app iOS.** Alvos executáveis do Swift Package Manager compilam como ferramentas de linha de comando macOS — eles não produzem bundles `.app`, não podem ser assinados para dispositivos iOS e não podem ser enviados à App Store.

**Padrão proibido:**
```swift
// Package.swift — NÃO USE para apps iOS
.executableTarget(name: "MyApp", dependencies: [])
// ou
.target(name: "MyApp", dependencies: [])
```

Usar esse padrão produz um binário CLI macOS, não um app iOS. O app não irá compilar para nenhum simulador ou dispositivo iOS.

---

## Padrão Obrigatório: XcodeGen

Todo scaffold de app iOS DEVE usar XcodeGen para gerar o `.xcodeproj`.

### Passo 1 — Instalar XcodeGen (se não estiver presente)

```bash
brew install xcodegen
```

### Passo 2 — Criar `project.yml`

`project.yml` é a especificação do XcodeGen que descreve a estrutura do projeto. Especificação mínima viável:

```yaml
name: MyApp
options:
  bundleIdPrefix: com.example
  deploymentTarget:
    iOS: "17.0"
settings:
  SWIFT_VERSION: "5.10"
  IPHONEOS_DEPLOYMENT_TARGET: "17.0"
targets:
  MyApp:
    type: application
    platform: iOS
    sources: [Sources/MyApp]
    settings:
      PRODUCT_BUNDLE_IDENTIFIER: com.example.MyApp
      INFOPLIST_FILE: Sources/MyApp/Info.plist
    scheme:
      testTargets:
        - MyAppTests
  MyAppTests:
    type: bundle.unit-test
    platform: iOS
    sources: [Tests/MyAppTests]
    dependencies:
      - target: MyApp
```

### Passo 3 — Gerar o .xcodeproj

```bash
xcodegen generate
```

Isso cria `MyApp.xcodeproj` na raiz do projeto. Faça commit de `project.yml` mas adicione `*.xcodeproj` ao `.gitignore` (regenere ao fazer checkout).

### Passo 4 — Layout padrão do projeto

```
MyApp/
├── project.yml              # Spec do XcodeGen — faça commit deste
├── .gitignore               # inclui *.xcodeproj
├── Sources/
│   └── MyApp/
│       ├── MyAppApp.swift   # ponto de entrada @main
│       ├── ContentView.swift
│       └── Info.plist
└── Tests/
    └── MyAppTests/
        └── MyAppTests.swift
```

---

## Compatibilidade de Alvo de Implantação iOS

Sempre verifique a disponibilidade da API SwiftUI em relação ao `IPHONEOS_DEPLOYMENT_TARGET` do projeto antes de usar qualquer componente SwiftUI.

| API | iOS Mínimo |
|-----|------------|
| `NavigationView` | iOS 13 |
| `NavigationStack` | iOS 16 |
| `NavigationSplitView` | iOS 16 |
| `List(selection:)` com multi-select | iOS 17 |
| APIs de posição de scroll do `ScrollView` | iOS 17 |
| Macro `Observable` (`@Observable`) | iOS 17 |
| `SwiftData` | iOS 17 |
| `@Bindable` | iOS 17 |
| `TipKit` | iOS 17 |

**Regra:** Se um plano requer uma API SwiftUI que excede o alvo de implantação do projeto, ou:
1. Eleve o alvo de implantação em `project.yml` (e documente a decisão), ou
2. Envolva a chamada em `if #available(iOS NN, *) { ... }` com uma implementação de fallback.

NÃO use silenciosamente uma API que requer uma versão iOS superior à declarada em `IPHONEOS_DEPLOYMENT_TARGET` — o app irá travar em dispositivos mais antigos.

---

## Verificação

Após executar `xcodegen generate`, verifique se o projeto compila:

```bash
xcodebuild -project MyApp.xcodeproj -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16' build
```

Um build bem-sucedido (código de saída 0) confirma que o scaffold é válido para iOS.
