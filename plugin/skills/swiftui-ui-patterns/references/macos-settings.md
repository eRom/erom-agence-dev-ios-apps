# macOS Settings

## Intention

Utilise ceci pour construire une fenêtre Settings macOS reposant sur la scene `Settings` de SwiftUI.

## Patterns essentiels

- Déclare la scene Settings dans l'`App` et compile-la uniquement pour macOS.
- Garde le contenu des settings dans une root view dédiée (`SettingsView`) et pilote les valeurs avec `@AppStorage`.
- Utilise `TabView` pour grouper les sections de settings quand tu as plus d'une catégorie.
- Utilise `Form` dans chaque tab pour garder les controls alignés et accessibles.
- Utilise `OpenSettingsAction` ou `SettingsLink` pour les points d'entrée in-app vers la fenêtre Settings.

## Exemple : scene settings

```swift
@main
struct MyApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
    #if os(macOS)
    Settings {
      SettingsView()
    }
    #endif
  }
}
```

## Exemple : settings view avec tabs

```swift
@MainActor
struct SettingsView: View {
  @AppStorage("showPreviews") private var showPreviews = true
  @AppStorage("fontSize") private var fontSize = 12.0

  var body: some View {
    TabView {
      Form {
        Toggle("Show Previews", isOn: $showPreviews)
        Slider(value: $fontSize, in: 9...96) {
          Text("Font Size (\(fontSize, specifier: "%.0f") pts)")
        }
      }
      .tabItem { Label("General", systemImage: "gear") }

      Form {
        Toggle("Enable Advanced Mode", isOn: .constant(false))
      }
      .tabItem { Label("Advanced", systemImage: "star") }
    }
    .scenePadding()
    .frame(maxWidth: 420, minHeight: 240)
  }
}
```

## Éviter la navigation

- Évite d'envelopper `SettingsView` dans un `NavigationStack`, sauf besoin réel de navigation push profonde.
- Privilégie les tabs ou les sections ; Settings est déjà présenté comme une fenêtre séparée et doit rester plat.
- Si tu dois montrer des settings hiérarchiques, utilise un unique `NavigationSplitView` avec une sidebar listant les catégories.

## Pièges

- Ne réutilise pas des layouts settings propres à iOS (stacks plein écran, flux riches en toolbar).
- Évite les grandes hiérarchies de view custom dans un `Form` ; garde les rows ciblées et accessibles.
