# Title menus

## Intention

Utilise un title menu dans la navigation bar pour fournir du filtrage contextuel ou des actions rapides sans ajouter de chrome supplémentaire.

## Patterns essentiels

- Utilise `ToolbarTitleMenu` pour attacher un menu au titre de navigation.
- Garde le contenu du menu compact et groupé avec des dividers.

## Exemple : title menu pour des filtres

```swift
@ToolbarContentBuilder
private var toolbarView: some ToolbarContent {
  ToolbarTitleMenu {
    Button("Latest") { timeline = .latest }
    Button("Resume") { timeline = .resume }
    Divider()
    Button("Local") { timeline = .local }
    Button("Federated") { timeline = .federated }
  }
}
```

## Exemple : attacher à une view

```swift
NavigationStack {
  TimelineView()
    .toolbar {
      toolbarView
    }
}
```

## Exemple : titre + menu ensemble

```swift
struct TimelineScreen: View {
  @State private var timeline: TimelineFilter = .home

  var body: some View {
    NavigationStack {
      TimelineView()
        .toolbar {
          ToolbarItem(placement: .principal) {
            VStack(spacing: 2) {
              Text(timeline.title)
                .font(.headline)
              Text(timeline.subtitle)
                .font(.caption)
                .foregroundStyle(.secondary)
            }
          }

          ToolbarTitleMenu {
            Button("Home") { timeline = .home }
            Button("Local") { timeline = .local }
            Button("Federated") { timeline = .federated }
          }
        }
        .navigationBarTitleDisplayMode(.inline)
    }
  }
}
```

## Exemple : titre + sous-titre avec menu

```swift
ToolbarItem(placement: .principal) {
  VStack(spacing: 2) {
    Text(title)
      .font(.headline)
    Text(subtitle)
      .font(.caption)
      .foregroundStyle(.secondary)
  }
}
```

## Choix de design à garder

- Affiche le title menu seulement quand le filtrage ou le changement de contexte est disponible.
- Garde le titre lisible ; évite les labels longs qui tronquent.
- Utilise du texte secondaire sous le titre si un contexte supplémentaire est nécessaire.

## Pièges

- Ne surcharge pas le menu avec trop d'options.
- Évite d'utiliser les title menus pour des actions destructives.
