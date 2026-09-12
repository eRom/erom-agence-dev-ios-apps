# ScrollView and Lazy stacks

## Intention

Utilise `ScrollView` avec `LazyVStack`, `LazyHStack` ou `LazyVGrid` quand tu as besoin d'un layout custom, de contenu mixte, ou d'un scroll horizontal / basé sur une grid.

## Patterns essentiels

- Privilégie `ScrollView` + `LazyVStack` pour des layouts type chat ou des feeds custom.
- Utilise `ScrollView(.horizontal)` + `LazyHStack` pour les chips, tags, avatars et media strips.
- Utilise `LazyVGrid` pour les grids d'icônes/media ; privilégie des colonnes adaptive quand c'est possible.
- Utilise `ScrollViewReader` pour le scroll-to-top/bottom et les jumps basés sur des ancres.
- Utilise `safeAreaInset(edge:)` pour les input bars qui doivent rester collées au-dessus du keyboard.

## Exemple : feed vertical custom

```swift
@MainActor
struct ConversationView: View {
  private enum Constants { static let bottomAnchor = "bottom" }
  @State private var scrollProxy: ScrollViewProxy?

  var body: some View {
    ScrollViewReader { proxy in
      ScrollView {
        LazyVStack {
          ForEach(messages) { message in
            MessageRow(message: message)
              .id(message.id)
          }
          Color.clear.frame(height: 1).id(Constants.bottomAnchor)
        }
        .padding(.horizontal, .layoutPadding)
      }
      .safeAreaInset(edge: .bottom) {
        MessageInputBar()
      }
      .onAppear {
        scrollProxy = proxy
        withAnimation {
          proxy.scrollTo(Constants.bottomAnchor, anchor: .bottom)
        }
      }
    }
  }
}
```

## Exemple : chips horizontales

```swift
ScrollView(.horizontal, showsIndicators: false) {
  LazyHStack(spacing: 8) {
    ForEach(chips) { chip in
      ChipView(chip: chip)
    }
  }
}
```

## Exemple : grid adaptive

```swift
let columns = [GridItem(.adaptive(minimum: 120))]

ScrollView {
  LazyVGrid(columns: columns, spacing: 8) {
    ForEach(items) { item in
      GridItemView(item: item)
    }
  }
  .padding(8)
}
```

## Choix de design à garder

- Utilise les stacks `Lazy*` quand le nombre d'items est grand ou inconnu.
- Utilise des stacks non-lazy pour du contenu petit et de taille fixe, pour éviter l'overhead du lazy.
- Garde les IDs stables quand tu utilises `ScrollViewReader`.
- Privilégie des animations explicites (`withAnimation`) quand tu scrolles vers un ID.

## Pièges

- Évite d'imbriquer des scroll views sur le même axe ; ça crée des conflits de gesture.
- Ne combine pas `List` et `ScrollView` dans la même hiérarchie sans raison claire.
- Un usage excessif de `LazyVStack` pour du contenu minuscule peut ajouter une complexité inutile.
