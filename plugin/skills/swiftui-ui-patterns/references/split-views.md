# Split views and columns

## Intention

Fournis un layout multi-colonnes léger et personnalisable pour iPad/macOS sans dépendre de `NavigationSplitView`.

## Pattern de split column custom (HStack manuel)

Utilise ça quand tu veux un contrôle total sur le dimensionnement des colonnes, le comportement et les ajustements d'environment.

```swift
@MainActor
struct AppView: View {
  @Environment(\.horizontalSizeClass) private var horizontalSizeClass
  @AppStorage("showSecondaryColumn") private var showSecondaryColumn = true

  var body: some View {
    HStack(spacing: 0) {
      primaryColumn
      if shouldShowSecondaryColumn {
        Divider().edgesIgnoringSafeArea(.all)
        secondaryColumn
      }
    }
  }

  private var shouldShowSecondaryColumn: Bool {
    horizontalSizeClass == .regular
      && showSecondaryColumn
  }

  private var primaryColumn: some View {
    TabView { /* tabs */ }
  }

  private var secondaryColumn: some View {
    NotificationsTab()
      .environment(\.isSecondaryColumn, true)
      .frame(maxWidth: .secondaryColumnWidth)
  }
}
```

## Notes sur l'approche custom

- Utilise un preference ou setting partagé pour activer/désactiver la colonne secondaire.
- Injecte un flag d'environment (par exemple `isSecondaryColumn`) pour que les child views puissent adapter leur comportement.
- Privilégie une largeur fixe ou plafonnée pour la colonne secondaire afin d'éviter le layout thrash.

## Alternative : NavigationSplitView

`NavigationSplitView` peut gérer sidebar + detail + colonnes supplémentaires pour toi, mais est plus difficile à personnaliser dans des cas comme :\n- une colonne de notifications dédiée indépendante de la sélection,\n- un dimensionnement custom, ou\n- des comportements de toolbar différents par colonne.

```swift
@MainActor
struct AppView: View {
  var body: some View {
    NavigationSplitView {
      SidebarView()
    } content: {
      MainContentView()
    } detail: {
      NotificationsView()
    }
  }
}
```

## Quand choisir quoi

- Utilise le split manuel en HStack quand tu as besoin d'un contrôle total ou d'une colonne secondaire non standard.
- Utilise `NavigationSplitView` quand tu veux un layout système standard avec une personnalisation minimale.
