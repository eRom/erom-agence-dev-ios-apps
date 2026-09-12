# Matched transitions

## Intention

Utilise les matched transitions pour créer une continuité fluide entre une source (thumbnail, avatar) et une destination (sheet, detail, viewer).

## Patterns essentiels

- Utilise un `Namespace` partagé et un ID stable pour la source.
- Utilise `matchedTransitionSource` + `navigationTransition(.zoom(...))` sur iOS 26+.
- Utilise `matchedGeometryEffect` pour des transitions in-place au sein d'une même hiérarchie de views.
- Garde les IDs stables entre les mises à jour de la view (évite les UUID aléatoires).

## Exemple : preview media vers un viewer full-screen (iOS 26+)

```swift
struct MediaPreview: View {
  @Namespace private var namespace
  @State private var selected: MediaAttachment?

  var body: some View {
    ThumbnailView()
      .matchedTransitionSource(id: selected?.id ?? "", in: namespace)
      .sheet(item: $selected) { item in
        MediaViewer(item: item)
          .navigationTransition(.zoom(sourceID: item.id, in: namespace))
      }
  }
}
```

## Exemple : matched geometry au sein d'une view

```swift
struct ToggleBadge: View {
  @Namespace private var space
  @State private var isOn = false

  var body: some View {
    Button {
      withAnimation(.spring) { isOn.toggle() }
    } label: {
      Image(systemName: isOn ? "eye" : "eye.slash")
        .matchedGeometryEffect(id: "icon", in: space)
    }
  }
}
```

## Choix de design à garder

- Privilégie `matchedTransitionSource` pour les transitions cross-screen.
- Garde des tailles source/destination raisonnables pour éviter des changements d'échelle brusques.
- Utilise `withAnimation` pour les transitions pilotées par du state.

## Pièges

- N'utilise pas d'IDs instables : ça casse la transition.
- Évite les formes non concordantes (carré vers cercle par exemple) sauf si le design l'attend explicitement.
