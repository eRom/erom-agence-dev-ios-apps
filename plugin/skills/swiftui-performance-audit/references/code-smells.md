# Code smells courants et patterns de remédiation

## Intention

Utilise cette référence pendant une revue code-first pour faire correspondre les patterns SwiftUI visibles à leurs coûts runtime probables et à des recommandations de remédiation sûres.

## Smells prioritaires

### Formatters coûteux dans `body`

```swift
var body: some View {
    let number = NumberFormatter()
    let measure = MeasurementFormatter()
    Text(measure.string(from: .init(value: meters, unit: .meters)))
}
```

Préfère des formatters mis en cache dans un modèle ou un helper dédié :

```swift
final class DistanceFormatter {
    static let shared = DistanceFormatter()
    let number = NumberFormatter()
    let measure = MeasurementFormatter()
}
```

### Computed properties lourdes

```swift
var filtered: [Item] {
    items.filter { $0.isEnabled }
}
```

Préfère dériver cette valeur une fois par changement significatif de l'input, dans un modèle/helper, ou ne stocke un state dérivé possédé par la view que quand la view possède réellement le cycle de vie de la transformation.

### Sort ou filter à l'intérieur de `body`

```swift
List {
    ForEach(items.sorted(by: sortRule)) { item in
        Row(item)
    }
}
```

Préfère trier avant que le travail de render ne commence :

```swift
let sortedItems = items.sorted(by: sortRule)
```

### Filtering inline à l'intérieur de `ForEach`

```swift
ForEach(items.filter { $0.isEnabled }) { item in
    Row(item)
}
```

Préfère une collection préfiltrée avec une identity stable.

### Identity instable

```swift
ForEach(items, id: \.self) { item in
    Row(item)
}
```

Évite `id: \.self` pour des valeurs non stables ou des collections qui se réordonnent. Utilise un identifiant de domaine stable.

### Permutation conditionnelle de view au niveau racine

```swift
var content: some View {
    if isEditing {
        editingView
    } else {
        readOnlyView
    }
}
```

Préfère une seule base view stable et localise les conditions dans des sections ou des modifiers. Cela réduit le churn d'identity à la racine et rend le diffing moins coûteux.

### Décodage d'image sur le main thread

```swift
Image(uiImage: UIImage(data: data)!)
```

Préfère faire le décodage et le downsample hors du main thread, puis stocker l'image traitée.

## Fan-out d'Observation

### Lectures larges d'`@Observable` sur iOS 17+

```swift
@Observable final class Model {
    var items: [Item] = []
}

var body: some View {
    Row(isFavorite: model.items.contains(item))
}
```

Si beaucoup de views lisent la même collection large ou le même modèle racine, de petits changements peuvent fan-out en une invalidation large. Préfère des inputs dérivés plus étroits, des surfaces observables plus petites, ou un state par item plus proche des leaf views.

### Lectures larges d'`ObservableObject` sur iOS 16 et versions antérieures

```swift
final class Model: ObservableObject {
    @Published var items: [Item] = []
}
```

Le même avertissement s'applique à l'observation legacy. Évite d'avoir de nombreux descendants qui observent un gros objet partagé quand ils n'ont besoin que d'un seul champ dérivé.

## Notes de remédiation

### `@State` n'est pas un cache générique

Utilise `@State` pour le state possédé par la view et les valeurs dérivées qui appartiennent intentionnellement au cycle de vie de la view. Ne déplace pas un calcul arbitraire et coûteux dans `@State` sans définir aussi quand et pourquoi il se met à jour.

Meilleures alternatives :
- précalculer dans le modèle ou le store
- mettre à jour un state dérivé en réponse à un changement d'input spécifique
- mémoïser dans un helper dédié
- préprocesser dans une tâche en background avant le rendering

### `equatable()` est une recommandation conditionnelle

Utilise `equatable()` uniquement quand :
- l'égalité est moins coûteuse que de recalculer le subtree, et
- les inputs de la view sont value-semantic et suffisamment stables pour des vérifications d'égalité pertinentes

N'applique pas `equatable()` comme correctif générique à tous les redraws.

## Ordre de triage

Quand plusieurs smells apparaissent ensemble, priorise dans cet ordre :
1. Invalidation large et fan-out d'observation
2. Identity instable et churn de liste
3. Travail sur le main thread pendant le render
4. Coût du décodage ou du resize d'images
5. Complexité du layout et des animations
