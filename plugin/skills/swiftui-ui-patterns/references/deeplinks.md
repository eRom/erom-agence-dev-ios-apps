# Deep links et navigation

## Intention

Router les URL externes vers des destinations in-app, avec un fallback vers la gestion système quand nécessaire.

## Patterns essentiels

- Centralise la gestion des URL dans le router (`handle(url:)`, `handleDeepLink(url:)`).
- Injecte un handler `OpenURLAction` qui délègue au router.
- Utilise `.onOpenURL` pour les liens de scheme de l'app, et convertis-les en URL web si nécessaire.
- Laisse le router décider s'il faut naviguer ou ouvrir en externe.

## Exemple : points d'entrée du router

```swift
@MainActor
final class RouterPath {
  var path: [Route] = []
  var urlHandler: ((URL) -> OpenURLAction.Result)?

  func handle(url: URL) -> OpenURLAction.Result {
    if isInternal(url) {
      navigate(to: .status(id: url.lastPathComponent))
      return .handled
    }
    return urlHandler?(url) ?? .systemAction
  }

  func handleDeepLink(url: URL) -> OpenURLAction.Result {
    // Résout les URL fédérées, puis navigue.
    navigate(to: .status(id: url.lastPathComponent))
    return .handled
  }
}
```

## Exemple : attacher à une root view

```swift
extension View {
  func withLinkRouter(_ router: RouterPath) -> some View {
    self
      .environment(
        \.openURL,
        OpenURLAction { url in
          router.handle(url: url)
        }
      )
      .onOpenURL { url in
        router.handleDeepLink(url: url)
      }
  }
}
```

## Choix de design à garder

- Garde le parsing d'URL et la logique de décision à l'intérieur du router.
- Évite de gérer les deep links en plusieurs endroits ; un seul point d'entrée suffit.
- Prévois toujours un fallback vers `OpenURLAction` ou `UIApplication.shared.open`.

## Pièges

- Ne présume pas que l'URL est interne ; valide d'abord.
- Évite de bloquer l'UI en résolvant des liens distants ; utilise `Task`.
