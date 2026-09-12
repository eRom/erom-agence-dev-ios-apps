# Searchable

## Intention

Utilise `searchable` pour ajouter une UI de recherche native avec scopes optionnels et résultats asynchrones.

## Patterns essentiels

- Bind `searchable(text:)` à du state local.
- Utilise `.searchScopes` pour plusieurs modes de recherche.
- Utilise `.task(id: searchQuery)` ou des tasks debounced pour éviter le overfetching.
- Affiche des placeholders ou des états de progress pendant le chargement des résultats.

## Exemple : searchable avec scopes

```swift
@MainActor
struct ExploreView: View {
  @State private var searchQuery = ""
  @State private var searchScope: SearchScope = .all
  @State private var isSearching = false
  @State private var results: [SearchResult] = []

  var body: some View {
    List {
      if isSearching {
        ProgressView()
      } else {
        ForEach(results) { result in
          SearchRow(result: result)
        }
      }
    }
    .searchable(
      text: $searchQuery,
      placement: .navigationBarDrawer(displayMode: .always),
      prompt: Text("Search")
    )
    .searchScopes($searchScope) {
      ForEach(SearchScope.allCases, id: \.self) { scope in
        Text(scope.title)
      }
    }
    .task(id: searchQuery) {
      await runSearch()
    }
  }

  private func runSearch() async {
    guard !searchQuery.isEmpty else {
      results = []
      return
    }
    isSearching = true
    defer { isSearching = false }
    try? await Task.sleep(for: .milliseconds(250))
    results = await fetchResults(query: searchQuery, scope: searchScope)
  }
}
```

## Choix de design à garder

- Affiche un placeholder quand la recherche est vide ou sans résultat.
- Debounce la saisie pour éviter de spammer le réseau.
- Garde le state de recherche local à la view.

## Pièges

- Évite de lancer des recherches pour des chaînes vides.
- Ne bloque pas le main thread pendant le fetch.
