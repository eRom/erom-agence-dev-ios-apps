# State asynchrone et lifecycle des tasks

## Intention

Utilise ce pattern quand une view charge des data, réagit à un input changeant, ou coordonne du travail asynchrone qui doit suivre le lifecycle de la view SwiftUI.

## Règles essentielles

- Utilise `.task` pour le travail de chargement au appear qui appartient au lifecycle de la view.
- Utilise `.task(id:)` quand le travail asynchrone doit redémarrer pour un input changeant, comme une query, une sélection, ou un identifier.
- Traite l'annulation comme un chemin normal pour les tasks pilotées par la view. Vérifie `Task.isCancelled` dans les flux plus longs et évite d'exposer l'annulation comme une erreur user-facing.
- Debounce ou regroupe le travail asynchrone piloté par l'utilisateur, comme la recherche, avant qu'il ne se transforme en requêtes répétées.
- Garde les modèles et mutations UI-facing main-actor-safe ; fais le travail en arrière-plan dans les services, puis publie le résultat vers le state UI.

## Exemple : chargement au appear

```swift
struct DetailView: View {
  let id: String
  @State private var state: LoadState<Item> = .idle
  @Environment(ItemClient.self) private var client

  var body: some View {
    content
      .task {
        await load()
      }
  }

  @ViewBuilder
  private var content: some View {
    switch state {
    case .idle, .loading:
      ProgressView()
    case .loaded(let item):
      ItemContent(item: item)
    case .failed(let error):
      ErrorView(error: error)
    }
  }

  private func load() async {
    state = .loading
    do {
      state = .loaded(try await client.fetch(id: id))
    } catch is CancellationError {
      return
    } catch {
      state = .failed(error)
    }
  }
}
```

## Exemple : redémarrage au changement d'input

```swift
struct SearchView: View {
  @State private var query = ""
  @State private var results: [ResultItem] = []
  @Environment(SearchClient.self) private var client

  var body: some View {
    List(results) { item in
      Text(item.title)
    }
    .searchable(text: $query)
    .task(id: query) {
      try? await Task.sleep(for: .milliseconds(250))
      guard !Task.isCancelled, !query.isEmpty else {
        results = []
        return
      }
      do {
        results = try await client.search(query)
      } catch is CancellationError {
        return
      } catch {
        results = []
      }
    }
  }
}
```

## Quand sortir le travail de la view

- Si le flux asynchrone s'étend sur plusieurs écrans ou doit survivre à la dismissal de la view, déplace-le dans un service ou un modèle.
- Si la view coordonne surtout le lifecycle au niveau de l'app ou les changements de compte, branche-la dans la coquille d'app, voir `app-wiring.md`.
- Si la politique de retry, de cache, ou de mode offline devient complexe, garde la politique dans le client/service et laisse la view avec des transitions de state simples.

## Pièges

- Ne démarre pas de travail réseau directement depuis `body`.
- N'ignore pas l'annulation pour les recherches, le typeahead, ou les sélections qui changent rapidement.
- Évite de stocker du state asynchrone dérivé à plusieurs endroits quand une seule source de vérité suffit.
