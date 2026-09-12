# Performance guardrails

## Intention

Applique ces règles quand un écran SwiftUI est large, très scrollé, fréquemment mis à jour, ou à risque de recalculs inutiles.

## Règles essentielles

- Donne à `ForEach` et au contenu de liste une identité stable. N'utilise pas des indices instables comme identité quand la collection peut se réordonner ou muter.
- Garde le filtrage, le tri et le formatage coûteux hors du `body` ; précalcule-les ou déplace-les dans un model/helper quand ce n'est pas trivial.
- Réduis la portée d'observation pour que seules les views qui lisent le state changeant se mettent à jour.
- Privilégie les conteneurs lazy pour le contenu scrollable volumineux, et extrais des subviews quand seule une partie de l'écran change fréquemment.
- Évite de remplacer des arbres de views top-level entiers pour de petits changements de state ; garde une root view stable et fais varier des sections ou modifiers localisés.

## Exemple : identité stable

```swift
ForEach(items) { item in
  Row(item: item)
}
```

Préfère ça à une identité basée sur l'index quand la collection peut changer d'ordre :

```swift
ForEach(Array(items.enumerated()), id: \.offset) { _, item in
  Row(item: item)
}
```

## Exemple : sortir le travail coûteux du body

```swift
struct FeedView: View {
  let items: [FeedItem]

  private var sortedItems: [FeedItem] {
    items.sorted(using: KeyPathComparator(\.createdAt, order: .reverse))
  }

  var body: some View {
    List(sortedItems) { item in
      FeedRow(item: item)
    }
  }
}
```

Si le travail est plus coûteux qu'une petite propriété dérivée, déplace-le dans un model, un store ou un helper qui se met à jour moins souvent.

## Quand creuser plus loin

- Scroll saccadé dans des feeds ou des grids longues
- Lag de frappe dans une recherche ou une validation de formulaire
- Mises à jour de view trop larges quand un seul petit bout de state change
- Grands écrans avec beaucoup de conditionnels ou de formatage répété

## Pièges

- Recalculer des transformations lourdes à chaque render
- Observer un gros objet depuis de nombreux descendants alors qu'un seul champ compte
- Construire des conteneurs de scroll custom quand `List`, `LazyVStack` ou `LazyHGrid` résoudraient déjà le problème
