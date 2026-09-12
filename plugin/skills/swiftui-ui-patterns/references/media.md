# Media (images, video, viewer)

## Intention

Utilise des patterns cohérents pour charger des images, prévisualiser du media et présenter un viewer full-screen.

## Patterns essentiels

- Utilise `LazyImage` (ou `AsyncImage`) pour les images distantes avec des états de chargement.
- Privilégie un composant de preview léger pour le media inline.
- Utilise un state de viewer partagé (par exemple `QuickLook`) pour présenter un viewer media full-screen.
- Utilise `openWindow` pour desktop/visionOS et une sheet pour iOS.

## Exemple : preview media inline

```swift
struct MediaPreviewRow: View {
  @Environment(QuickLook.self) private var quickLook

  let attachments: [MediaAttachment]

  var body: some View {
    ScrollView(.horizontal, showsIndicators: false) {
      HStack {
        ForEach(attachments) { attachment in
          LazyImage(url: attachment.previewURL) { state in
            if let image = state.image {
              image.resizable().aspectRatio(contentMode: .fill)
            } else {
              ProgressView()
            }
          }
          .frame(width: 120, height: 120)
          .clipped()
          .onTapGesture {
            quickLook.prepareFor(
              selectedMediaAttachment: attachment,
              mediaAttachments: attachments
            )
          }
        }
      }
    }
  }
}
```

## Exemple : sheet de viewer media global

```swift
struct AppRoot: View {
  @State private var quickLook = QuickLook.shared

  var body: some View {
    content
      .environment(quickLook)
      .sheet(item: $quickLook.selectedMediaAttachment) { selected in
        MediaUIView(selectedAttachment: selected, attachments: quickLook.mediaAttachments)
      }
  }
}
```

## Choix de design à garder

- Garde les previews légers ; charge le media complet dans le viewer.
- Utilise un state de viewer partagé pour que n'importe quelle view puisse ouvrir du media sans prop-drilling.
- Garde un seul point d'entrée pour le viewer (sheet/window) pour éviter les doublons.

## Pièges

- Évite de charger des images en pleine résolution dans les lignes de liste ; utilise des previews redimensionnées.
- Ne présente pas plusieurs sheets de viewer en même temps ; garde une seule source de vérité.
