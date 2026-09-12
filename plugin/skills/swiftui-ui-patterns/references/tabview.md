# TabView

## Intention

Utilise ce pattern pour une architecture de tabs scalable et multi-plateforme avec :
- une seule source de vérité pour l'identité et le contenu des tabs,
- des jeux de tabs et des sections de sidebar spécifiques à la plateforme,
- des tabs dynamiques sourcées depuis de la donnée,
- un hook d'interception pour les tabs spéciales (par exemple compose).

## Architecture essentielle

- L'enum `AppTab` définit l'identité, les labels, les icônes et le content builder.
- L'enum `SidebarSections` groupe les tabs pour les sections de sidebar.
- `AppView` possède la `TabView` et le binding de sélection, et route les changements de tab via `updateTab`.

## Exemple : binding custom avec effets de bord

Utilise ça quand la sélection de tab a besoin d'effets de bord, comme intercepter une tab spéciale pour effectuer une action au lieu de changer la sélection.

```swift
@MainActor
struct AppView: View {
  @Binding var selectedTab: AppTab

  var body: some View {
    TabView(selection: .init(
      get: { selectedTab },
      set: { updateTab(with: $0) }
    )) {
      ForEach(availableSections) { section in
        TabSection(section.title) {
          ForEach(section.tabs) { tab in
            Tab(value: tab) {
              tab.makeContentView(
                homeTimeline: $timeline,
                selectedTab: $selectedTab,
                pinnedFilters: $pinnedFilters
              )
            } label: {
              tab.label
            }
            .tabPlacement(tab.tabPlacement)
          }
        }
        .tabPlacement(.sidebarOnly)
      }
    }
  }

  private func updateTab(with newTab: AppTab) {
    if newTab == .post {
      // Intercepte les tabs spéciales (compose) au lieu de changer la sélection.
      presentComposer()
      return
    }
    selectedTab = newTab
  }
}
```

## Exemple : binding direct sans effets de bord

Utilise ça quand la sélection est purement pilotée par du state.

```swift
@MainActor
struct AppView: View {
  @Binding var selectedTab: AppTab

  var body: some View {
    TabView(selection: $selectedTab) {
      ForEach(availableSections) { section in
        TabSection(section.title) {
          ForEach(section.tabs) { tab in
            Tab(value: tab) {
              tab.makeContentView(
                homeTimeline: $timeline,
                selectedTab: $selectedTab,
                pinnedFilters: $pinnedFilters
              )
            } label: {
              tab.label
            }
            .tabPlacement(tab.tabPlacement)
          }
        }
        .tabPlacement(.sidebarOnly)
      }
    }
  }
}
```

## Choix de design à garder

- Centralise l'identité et le contenu des tabs dans `AppTab` avec `makeContentView(...)`.
- Utilise `Tab(value:)` avec un binding `selection` pour une sélection de tab pilotée par du state.
- Route les changements de sélection via `updateTab` pour gérer les tabs spéciales et le comportement scroll-to-top.
- Utilise `TabSection` + `.tabPlacement(.sidebarOnly)` pour la structure de sidebar.
- Utilise `.tabPlacement(.pinned)` dans `AppTab.tabPlacement` pour une seule tab épinglée ; c'est couramment utilisé pour le contenu de tab `.searchable` sur iOS 26, mais peut servir pour n'importe quelle tab.

## Pattern de tabs dynamiques

- `SidebarSections` gère les tabs de données dynamiques.
- `AppTab.anyTimelineFilter(filter:)` enveloppe les tabs dynamiques dans un seul case d'enum.
- L'enum fournit label/icon/title pour les tabs dynamiques via le type de filtre.

## Pièges

- Évite d'ajouter des ViewModels pour les tabs ; garde le state local ou dans des services `@Observable`.
- N'imbrique pas des objets `@Observable` à l'intérieur d'autres objets `@Observable`.
- Assure-toi que les valeurs `AppTab.id` sont stables ; les cases dynamiques doivent hasher sur des IDs stables.
- Les tabs spéciales (compose) ne doivent pas changer la sélection.
