# Lightweight Clients (à base de closures)

Utilise ce pattern pour garder les dépendances réseau ou service simples et testables sans introduire un view model complet ni un framework de DI lourd. Cela fonctionne bien pour les apps SwiftUI où tu veux une petite surface d'API composable, remplaçable dans les previews/tests.

## Intention
- Fournir un tiny "client", fait de closures asynchrones.
- Garder la logique métier dans un store ou une couche feature, pas dans la view.
- Faciliter le stubbing dans les previews/tests.

## Forme minimale
```swift
struct SomeClient {
    var fetchItems: (_ limit: Int) async throws -> [Item]
    var search: (_ query: String, _ limit: Int) async throws -> [Item]
}

extension SomeClient {
    static func live(baseURL: URL = URL(string: "https://example.com")!) -> SomeClient {
        let session = URLSession.shared
        return SomeClient(
            fetchItems: { limit in
                // construit l'URL, appelle la session, décode
            },
            search: { query, limit in
                // construit l'URL, appelle la session, décode
            }
        )
    }
}
```

## Pattern d'usage
```swift
@MainActor
@Observable final class ItemsStore {
    enum LoadState { case idle, loading, loaded, failed(String) }

    var items: [Item] = []
    var state: LoadState = .idle
    private let client: SomeClient

    init(client: SomeClient) {
        self.client = client
    }

    func load(limit: Int = 20) async {
        state = .loading
        do {
            items = try await client.fetchItems(limit)
            state = .loaded
        } catch {
            state = .failed(error.localizedDescription)
        }
    }
}
```

```swift
struct ContentView: View {
    @Environment(ItemsStore.self) private var store

    var body: some View {
        List(store.items) { item in
            Text(item.title)
        }
        .task { await store.load() }
    }
}
```

```swift
@main
struct MyApp: App {
    @State private var store = ItemsStore(client: .live())

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(store)
        }
    }
}
```

## Directives
- Garde le decoding et la construction d'URL dans le client ; garde les changements de state dans le store.
- Fais en sorte que le store accepte le client dans `init` et garde-le private.
- Évite les singletons globaux ; utilise `.environment` pour l'injection du store.
- Si tu as besoin de plusieurs variantes (mock/stub), ajoute `static func mock(...)`.

## Pièges
- Ne mets pas de state UI dans le client ; garde le state dans le store.
- Ne capture pas `self` ou du state de view dans les closures du client.
