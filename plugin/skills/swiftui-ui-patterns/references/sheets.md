# Sheets

## Intention

Utilise un pattern de routing de sheets centralisé pour que n'importe quelle view puisse présenter des modales sans prop-drilling. Ça garde le state de sheet à un seul endroit et scale à mesure que l'app grandit.

## Architecture essentielle

- Définis un enum `SheetDestination` qui décrit chaque modale et qui est `Identifiable`.
- Stocke la sheet courante dans un objet router (`presentedSheet: SheetDestination?`).
- Crée un view modifier comme `withSheetDestinations(...)` qui mappe l'enum vers des views de sheet concrètes.
- Injecte le router dans l'environment pour que les child views puissent set `presentedSheet` directement.

## Exemple : sheet locale pilotée par un item

Utilise ça quand le state de sheet est local à un seul écran et n'a pas besoin de routing centralisé.

```swift
@State private var selectedItem: Item?

.sheet(item: $selectedItem) { item in
  EditItemSheet(item: item)
}
```

## Exemple : enum SheetDestination

```swift
enum SheetDestination: Identifiable, Hashable {
  case composer
  case editProfile
  case settings
  case report(itemID: String)

  var id: String {
    switch self {
    case .composer, .editProfile:
      // Utilise le même id pour garantir qu'une seule sheet de type éditeur est active à la fois.
      return "editor"
    case .settings:
      return "settings"
    case .report:
      return "report"
    }
  }
}
```

## Exemple : modifier withSheetDestinations

```swift
extension View {
  func withSheetDestinations(
    sheet: Binding<SheetDestination?>
  ) -> some View {
    sheet(item: sheet) { destination in
      Group {
        switch destination {
        case .composer:
          ComposerView()
        case .editProfile:
          EditProfileView()
        case .settings:
          SettingsView()
        case .report(let itemID):
          ReportView(itemID: itemID)
        }
      }
    }
  }
}
```

## Exemple : présentation depuis une child view

```swift
struct StatusRow: View {
  @Environment(RouterPath.self) private var router

  var body: some View {
    Button("Report") {
      router.presentedSheet = .report(itemID: "123")
    }
  }
}
```

## Wiring requis

Pour que la child view fonctionne, une parent view doit :
- posséder l'instance du router,
- attacher `withSheetDestinations(sheet: $router.presentedSheet)` (ou un handler `sheet(item:)` équivalent), et
- l'injecter avec `.environment(router)` après le sheet modifier pour que le contenu de la modale en hérite.

Ça permet à l'assignation de la child view à `router.presentedSheet` de piloter la présentation depuis la root.

## Exemple : sheets qui ont besoin de leur propre navigation

Enveloppe le contenu de la sheet dans une `NavigationStack` pour qu'elle puisse push à l'intérieur de la modale.

```swift
struct NavigationSheet<Content: View>: View {
  var content: () -> Content

  var body: some View {
    NavigationStack {
      content()
        .toolbar { CloseToolbarItem() }
    }
  }
}
```

## Exemple : la sheet possède ses actions

Garde la logique de dismissal et de confirmation à l'intérieur de la sheet quand les actions appartiennent à la modale elle-même.

```swift
struct EditItemSheet: View {
  @Environment(\.dismiss) private var dismiss
  @Environment(Store.self) private var store

  let item: Item
  @State private var isSaving = false

  var body: some View {
    VStack {
      Button(isSaving ? "Saving..." : "Save") {
        Task { await save() }
      }
    }
  }

  private func save() async {
    isSaving = true
    await store.save(item)
    dismiss()
  }
}
```

## Choix de design à garder

- Centralise le routing des sheets pour que les features puissent présenter des modales sans faire transiter des bindings à travers de nombreuses couches.
- Utilise `sheet(item:)` pour garantir qu'une seule sheet est active et pour piloter la présentation depuis l'enum.
- Groupe les sheets liées sous le même `id` quand elles sont mutuellement exclusives (par exemple les flows d'édition).
- Garde les views de sheet légères et composées de views plus petites ; évite les gros monolithes.
- Laisse les sheets posséder leurs actions et appeler `dismiss()` en interne plutôt que de faire transiter des closures `onCancel` ou `onConfirm` à travers de nombreuses couches.

## Pièges

- Évite de mélanger `sheet(isPresented:)` et `sheet(item:)` pour la même préoccupation ; privilégie un seul enum.
- Évite un `if let` à l'intérieur d'un body de sheet quand le state de présentation porte déjà le modèle sélectionné ; privilégie `sheet(item:)`.
- Ne stocke pas de state lourd dans `SheetDestination` ; passe des identifiants ou des modèles légers.
- Si plusieurs sheets peuvent apparaître depuis le même écran, donne-leur des valeurs `id` distinctes.
