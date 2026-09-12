# NavigationStack

## Intention

Utilise ce pattern pour la navigation programmatique et les deep links, en particulier quand chaque tab a besoin d'un historique de navigation indépendant. L'idée clé : une `NavigationStack` par tab, chacune avec son propre path binding et son propre objet router.

## Architecture essentielle

- Définis un enum de route qui est `Hashable` et qui représente toutes les destinations.
- Crée un router léger (ou utilise une lib comme `https://github.com/Dimillian/AppRouter`) qui possède le `path` et tout state de sheet.
- Chaque tab possède sa propre instance de router et bind `NavigationStack(path:)` à elle.
- Injecte le router dans l'environment pour que les child views puissent naviguer de manière programmatique.
- Centralise le mapping des destinations avec un seul bloc `navigationDestination(for:)` (ou un modifier `withAppRouter()`).

## Exemple : router custom avec une stack par tab

```swift
@MainActor
@Observable
final class RouterPath {
  var path: [Route] = []
  var presentedSheet: SheetDestination?

  func navigate(to route: Route) {
    path.append(route)
  }

  func reset() {
    path = []
  }
}

enum Route: Hashable {
  case account(id: String)
  case status(id: String)
}

@MainActor
struct TimelineTab: View {
  @State private var routerPath = RouterPath()

  var body: some View {
    NavigationStack(path: $routerPath.path) {
      TimelineView()
        .navigationDestination(for: Route.self) { route in
          switch route {
          case .account(let id): AccountView(id: id)
          case .status(let id): StatusView(id: id)
          }
        }
    }
    .environment(routerPath)
  }
}
```

## Exemple : mapping de destinations centralisé

Utilise un view modifier partagé pour éviter de dupliquer les switch de route à travers les écrans.

```swift
extension View {
  func withAppRouter() -> some View {
    navigationDestination(for: Route.self) { route in
      switch route {
      case .account(let id):
        AccountView(id: id)
      case .status(let id):
        StatusView(id: id)
      }
    }
  }
}
```

Applique-le ensuite une fois par stack :

```swift
NavigationStack(path: $routerPath.path) {
  TimelineView()
    .withAppRouter()
}
```

## Exemple : binding par tab (tabs avec historique indépendant)

```swift
@MainActor
struct TabsView: View {
  @State private var timelineRouter = RouterPath()
  @State private var notificationsRouter = RouterPath()

  var body: some View {
    TabView {
      TimelineTab(router: timelineRouter)
      NotificationsTab(router: notificationsRouter)
    }
  }
}
```

## Exemple : tabs génériques avec une NavigationStack par tab

Utilise ça quand les tabs sont construites depuis de la donnée et que chacune a besoin de son propre path sans nom codé en dur.

```swift
@MainActor
struct TabsView: View {
  @State private var selectedTab: AppTab = .timeline
  @State private var tabRouter = TabRouter()

  var body: some View {
    TabView(selection: $selectedTab) {
      ForEach(AppTab.allCases) { tab in
        NavigationStack(path: tabRouter.binding(for: tab)) {
          tab.makeContentView()
        }
        .environment(tabRouter.router(for: tab))
        .tabItem { tab.label }
        .tag(tab)
      }
    }
  }
}
```

@MainActor
@Observable
final class TabRouter {
  private var routers: [AppTab: RouterPath] = [:]

  func router(for tab: AppTab) -> RouterPath {
    if let router = routers[tab] { return router }
    let router = RouterPath()
    routers[tab] = router
    return router
  }

  func binding(for tab: AppTab) -> Binding<[Route]> {
    let router = router(for: tab)
    return Binding(get: { router.path }, set: { router.path = $0 })
  }
}

## Choix de design à garder

- Une `NavigationStack` par tab pour préserver un historique indépendant.
- Une seule source de vérité pour le state de navigation (`RouterPath` ou router de lib).
- Utilise `navigationDestination(for:)` pour mapper les routes vers les views.
- Reset le path quand le contexte de l'app change (changement de compte, logout, etc.).
- Injecte le router dans l'environment pour que les child views puissent naviguer et présenter des sheets sans prop-drilling.
- Garde le state de présentation des sheets sur le router si tu veux un seul endroit pour gérer les modales.

## Pièges

- Ne partage pas un seul path entre toutes les tabs, sauf si tu veux un historique global.
- Assure-toi que les identifiants de route sont stables et `Hashable`.
- Évite de stocker des instances de view dans le path ; stocke plutôt des données de route légères.
- Si tu utilises un objet router, garde-le à l'écart des autres objets `@Observable` pour éviter l'observation imbriquée.
