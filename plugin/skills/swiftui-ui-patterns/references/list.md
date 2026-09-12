# List et Section

## Intention

Utilise `List` pour du contenu façon feed et des rows façon settings, là où la réutilisation de row native, la sélection, et l'accessibilité comptent.

## Patterns essentiels

- Privilégie `List` pour du contenu long, à scroll vertical, avec des rows répétées.
- Utilise des headers de `Section` pour grouper les rows liées.
- Associe avec `ScrollViewReader` quand tu as besoin de scroll-to-top ou de jump-to-id.
- Utilise `.listStyle(.plain)` pour des layouts de feed modernes.
- Utilise `.listStyle(.grouped)` pour des pages de découverte/recherche multi-sections où le regroupement en sections aide.
- Applique `.scrollContentBackground(.hidden)` + un background custom quand tu as besoin d'une surface themée.
- Utilise `.listRowInsets(...)` et `.listRowSeparator(.hidden)` pour ajuster l'espacement des rows et les separators.
- Utilise `.environment(\\.defaultMinListRowHeight, ...)` pour contrôler des layouts de liste denses.

## Exemple : feed list avec scroll-to-top

```swift
@MainActor
struct TimelineListView: View {
  @Environment(\.selectedTabScrollToTop) private var selectedTabScrollToTop
  @State private var scrollToId: String?

  var body: some View {
    ScrollViewReader { proxy in
      List {
        ForEach(items) { item in
          TimelineRow(item: item)
            .id(item.id)
            .listRowInsets(.init(top: 12, leading: 16, bottom: 6, trailing: 16))
            .listRowSeparator(.hidden)
        }
      }
      .listStyle(.plain)
      .environment(\\.defaultMinListRowHeight, 1)
      .onChange(of: scrollToId) { _, newValue in
        if let newValue {
          proxy.scrollTo(newValue, anchor: .top)
          scrollToId = nil
        }
      }
      .onChange(of: selectedTabScrollToTop) { _, newValue in
        if newValue == 0 {
          withAnimation {
            proxy.scrollTo(ScrollToView.Constants.scrollToTop, anchor: .top)
          }
        }
      }
    }
  }
}
```

## Exemple : list façon settings

```swift
@MainActor
struct SettingsView: View {
  var body: some View {
    List {
      Section("General") {
        NavigationLink("Display") { DisplaySettingsView() }
        NavigationLink("Haptics") { HapticsSettingsView() }
      }
      Section("Account") {
        Button("Sign Out", role: .destructive) {}
      }
    }
    .listStyle(.insetGrouped)
  }
}
```

## Choix de design à garder

- Utilise `List` pour les feeds dynamiques, les settings, et toute UI où la sémantique de row aide.
- Utilise des IDs stables pour les rows afin de garder les animations et le positionnement de scroll fiables.
- Privilégie `.contentShape(Rectangle())` sur les rows qui doivent être tappables de bout en bout.
- Utilise `.refreshable` pour le pull-to-refresh sur les feeds quand la source de data le supporte.

## Pièges

- Évite les layouts custom lourds dans une row de `List` ; utilise plutôt `ScrollView` + `LazyVStack`.
- Fais attention en mélangeant `List` et `ScrollView` imbriquée ; cela peut causer des conflits de gesture.
