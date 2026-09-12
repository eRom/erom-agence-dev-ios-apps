# Top bar overlays (iOS 26+ and fallback)

## Intention

Fournis un sélecteur top custom ou une pill row qui se place au-dessus du contenu scrollable, en utilisant `safeAreaBar(.top)` sur iOS 26 et un fallback compatible sur les versions d'OS antérieures.

## Approche iOS 26+

Utilise `safeAreaBar(edge: .top)` pour attacher la view à la safe area bar.

```swift
if #available(iOS 26.0, *) {
  content
    .safeAreaBar(edge: .top) {
      TopSelectorView()
        .padding(.horizontal, .layoutPadding)
    }
}
```

## Fallback pour les iOS antérieurs

Utilise `.safeAreaInset(edge: .top)` et masque le background de la toolbar pour éviter les doubles couches.

```swift
content
  .toolbarBackground(.hidden, for: .navigationBar)
  .safeAreaInset(edge: .top, spacing: 0) {
    VStack(spacing: 0) {
      TopSelectorView()
        .padding(.vertical, 8)
        .padding(.horizontal, .layoutPadding)
        .background(Color.primary.opacity(0.06))
        .background(Material.ultraThin)
      Divider()
    }
  }
```

## Choix de design à garder

- Utilise `safeAreaBar` quand disponible ; ça s'intègre mieux avec la navigation bar.
- Utilise un background discret + un divider dans le fallback pour garder la séparation avec le contenu.
- Garde la hauteur du sélecteur compacte pour éviter de trop repousser le contenu vers le bas.

## Pièges

- N'empile pas plusieurs top insets ; ça peut créer du padding supplémentaire.
- Évite les backgrounds lourds et opaques qui entrent en conflit avec la navigation bar.
