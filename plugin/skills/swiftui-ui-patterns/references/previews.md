# Previews

## Intention

Utilise les previews pour valider le layout, le wiring de state et les dépendances injectées sans dépendre d'une app en cours d'exécution ou de services live.

## Règles essentielles

- Ajoute une couverture `#Preview` pour le state principal plus les états secondaires importants comme loading, empty et error.
- Utilise des fixtures, mocks et sample data déterministes. Ne fais pas dépendre les previews d'appels réseau live, de vraies bases de données ou de singletons globaux.
- Installe les dépendances d'environment requises directement dans la preview pour que la view puisse se render en isolation.
- Garde le setup de preview proche de la view jusqu'à ce que ça devienne bruyant ; extrais alors des helpers ou fixtures de preview légers.
- Si une preview crash, corrige l'initialisation du state ou le wiring des dépendances avant d'étendre davantage la feature.

## Exemple : états de preview simples

```swift
#Preview("Loaded") {
  ProfileView(profile: .fixture)
}

#Preview("Empty") {
  ProfileView(profile: nil)
}
```

## Exemple : preview avec dépendances injectées

```swift
#Preview("Search results") {
  SearchView()
    .environment(SearchClient.preview(results: [.fixture, .fixture2]))
    .environment(Theme.preview)
}
```

## Checklist de preview

- La preview installe-t-elle chaque dépendance d'environment requise ?
- Couvre-t-elle au moins un chemin de succès et un cas non-happy path ?
- Les fixtures sont-elles stables et assez petites pour être lues rapidement ?
- La preview peut-elle se render sans réseau, sans auth, sans initialisation globale de l'app ?

## Pièges

- Ne masque pas les crashs de preview en rendant les dépendances optionnelles si la view de production les requiert.
- Évite les grosses fixtures inline quand un sample nommé est plus facile à lire.
- Ne couple pas les previews à des singletons globaux partagés sauf si le projet n'a pas d'alternative.
