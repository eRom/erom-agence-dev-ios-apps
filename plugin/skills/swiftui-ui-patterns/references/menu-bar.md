# Menu Bar

## Intention

Utilise ce pattern pour ajouter ou personnaliser la menu bar macOS/iPadOS avec des commands SwiftUI.

## Patterns essentiels

- Ajoute les commands au niveau `Scene` avec `.commands { ... }`.
- Utilise `SidebarCommands()` quand ton UI inclut une sidebar de navigation.
- Utilise `CommandMenu` pour les menus spécifiques à l'app et pour grouper des actions liées.
- Utilise `CommandGroup` pour insérer des items avant/après les groupes système ou les remplacer.
- Utilise `FocusedValue` pour des items de menu contextuels qui dépendent de la scene active.

## Exemple : command menu basique

```swift
@main
struct MyApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
    .commands {
      CommandMenu("Actions") {
        Button("Run", action: run)
          .keyboardShortcut("R")
        Button("Stop", action: stop)
          .keyboardShortcut(".")
      }
    }
  }

  private func run() {}
  private func stop() {}
}
```

## Exemple : insérer et remplacer des groupes

```swift
WindowGroup {
  ContentView()
}
.commands {
  CommandGroup(before: .systemServices) {
    Button("Check for Updates") { /* open updater */ }
  }

  CommandGroup(after: .newItem) {
    Button("New from Clipboard") { /* create item */ }
  }

  CommandGroup(replacing: .help) {
    Button("User Manual") { /* open docs */ }
  }
}
```

## Exemple : state de menu focused

```swift
@Observable
final class DataModel {
  var items: [String] = []
}

struct ContentView: View {
  @State private var model = DataModel()

  var body: some View {
    List(model.items, id: \.self) { item in
      Text(item)
    }
    .focusedSceneValue(model)
  }
}

struct ItemCommands: Commands {
  @FocusedValue(DataModel.self) private var model: DataModel?

  var body: some Commands {
    CommandGroup(after: .newItem) {
      Button("New Item") {
        model?.items.append("Untitled")
      }
      .disabled(model == nil)
    }
  }
}
```

## Menu bar et Settings

- Définir une scene `Settings` ajoute automatiquement l'item de menu Settings sur macOS.
- Si tu as besoin d'un point d'entrée custom dans l'app, utilise `OpenSettingsAction` ou `SettingsLink`.

## Pièges

- Évite d'enregistrer le même keyboard shortcut dans plusieurs command groups.
- N'utilise pas les items de menu comme seul point d'entrée découvrable pour des fonctionnalités critiques.
