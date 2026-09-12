# Grids

## Intention

Utilise `LazyVGrid` pour les icon pickers, les galeries média, et les sélections visuelles denses où les items s'alignent en colonnes.

## Patterns essentiels

- Utilise des colonnes `.adaptive` pour des layouts qui doivent s'adapter selon la taille de l'appareil.
- Utilise plusieurs colonnes `.flexible` quand tu veux un nombre de colonnes fixe.
- Garde un espacement cohérent et réduit pour éviter des gouttières irrégulières.
- Utilise `GeometryReader` dans les cells de grid quand tu as besoin de thumbnails carrées.

## Exemple : grid d'icons adaptative

```swift
let columns = [GridItem(.adaptive(minimum: 120, maximum: 1024))]

LazyVGrid(columns: columns, spacing: 6) {
  ForEach(icons) { icon in
    Button {
      select(icon)
    } label: {
      ZStack(alignment: .bottomTrailing) {
        Image(icon.previewName)
          .resizable()
          .aspectRatio(contentMode: .fit)
          .cornerRadius(6)
        if icon.isSelected {
          Image(systemName: "checkmark.seal.fill")
            .padding(4)
            .tint(.green)
        }
      }
    }
    .buttonStyle(.plain)
  }
}
```

## Exemple : grid média fixe à 3 colonnes

```swift
LazyVGrid(
  columns: [
    .init(.flexible(minimum: 100), spacing: 4),
    .init(.flexible(minimum: 100), spacing: 4),
    .init(.flexible(minimum: 100), spacing: 4),
  ],
  spacing: 4
) {
  ForEach(items) { item in
    GeometryReader { proxy in
      ThumbnailView(item: item)
        .frame(width: proxy.size.width, height: proxy.size.width)
    }
    .aspectRatio(1, contentMode: .fit)
  }
}
```

## Choix de design à garder

- Utilise `LazyVGrid` pour les grandes collections ; évite les grids non-lazy pour de gros ensembles.
- Garde des tap targets full-bleed avec `.contentShape(Rectangle())` si besoin.
- Privilégie les grids adaptatives pour les pickers de settings et les layouts flexibles.

## Pièges

- Évite les overlays lourds dans chaque cell de grid ; cela peut coûter cher.
- N'imbrique pas de grids dans d'autres grids sans raison claire.
