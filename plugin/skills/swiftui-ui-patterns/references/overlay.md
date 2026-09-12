# Overlay and toasts

## Intention

Utilise les overlays pour de l'UI transitoire (toasts, banners, loaders) sans affecter le layout.

## Patterns essentiels

- Utilise `.overlay(alignment:)` pour placer de l'UI globale sans changer le layout sous-jacent.
- Garde les overlays légers et dismissibles.
- Utilise un `ToastCenter` dédié (ou équivalent) pour un state global si plusieurs features déclenchent des toasts.

## Exemple : overlay de toast

```swift
struct AppRootView: View {
  @State private var toast: Toast?

  var body: some View {
    content
      .overlay(alignment: .top) {
        if let toast {
          ToastView(toast: toast)
            .transition(.move(edge: .top).combined(with: .opacity))
            .onAppear {
              DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
                withAnimation { self.toast = nil }
              }
            }
        }
      }
  }
}
```

## Choix de design à garder

- Privilégie les overlays pour l'UI transitoire plutôt que de les embarquer dans des stacks de layout.
- Utilise des transitions et des timers d'auto-dismiss courts.
- Garde l'overlay aligné sur un bord clair (`.top` ou `.bottom`).

## Pièges

- Évite les overlays qui bloquent toute interaction sauf si c'est explicitement nécessaire.
- N'empile pas plusieurs overlays ; utilise une queue ou remplace le toast courant.
