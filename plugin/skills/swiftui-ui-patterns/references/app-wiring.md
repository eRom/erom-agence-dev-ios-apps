# Wiring de l'app et graphe de dépendances

## Intention

Montrer comment brancher la coquille d'app (TabView + NavigationStack + sheets) et installer un graphe de dépendances global (environment objects, services, streaming clients, SwiftData ModelContainer) en un seul endroit.

## Structure recommandée

1) La root view met en place les tabs, les routers par tab, et les sheets.
2) Un modifier dédié installe les dépendances globales et les tâches de lifecycle (état d'auth, watchers de streaming, push tokens, containers de data).
3) Les feature views ne récupèrent depuis l'environment que ce dont elles ont besoin ; le state propre à une feature reste local.

## Choix des dépendances

- Utilise `@Environment` pour les services au niveau de l'app, les clients partagés, le theme/la configuration, et les valeurs dont beaucoup de descendants ont réellement besoin.
- Privilégie l'injection via initializer pour les dépendances et modèles propres à une feature. Ne déplace pas une dépendance dans l'environment juste pour éviter de passer un ou deux arguments.
- Garde le state mutable d'une feature hors de l'environment, sauf s'il est intentionnellement partagé sur de larges parties de l'app.
- Utilise `@EnvironmentObject` uniquement comme fallback legacy ou quand le projet le standardise déjà pour un objet réellement partagé.

## Exemple de coquille racine (générique)

```swift
@MainActor
struct AppView: View {
  @State private var selectedTab: AppTab = .home
  @State private var tabRouter = TabRouter()

  var body: some View {
    TabView(selection: $selectedTab) {
      ForEach(AppTab.allCases) { tab in
        let router = tabRouter.router(for: tab)
        NavigationStack(path: tabRouter.binding(for: tab)) {
          tab.makeContentView()
        }
        .withSheetDestinations(sheet: Binding(
          get: { router.presentedSheet },
          set: { router.presentedSheet = $0 }
        ))
        .environment(router)
        .tabItem { tab.label }
        .tag(tab)
      }
    }
    .withAppDependencyGraph()
  }
}
```

Exemple minimal d'`AppTab` :

```swift
@MainActor
enum AppTab: Identifiable, Hashable, CaseIterable {
  case home, notifications, settings
  var id: String { String(describing: self) }

  @ViewBuilder
  func makeContentView() -> some View {
    switch self {
    case .home: HomeView()
    case .notifications: NotificationsView()
    case .settings: SettingsView()
    }
  }

  @ViewBuilder
  var label: some View {
    switch self {
    case .home: Label("Home", systemImage: "house")
    case .notifications: Label("Notifications", systemImage: "bell")
    case .settings: Label("Settings", systemImage: "gear")
    }
  }
}
```

Skeleton de router :

```swift
@MainActor
@Observable
final class RouterPath {
  var path: [Route] = []
  var presentedSheet: SheetDestination?
}

enum Route: Hashable {
  case detail(id: String)
}
```

## Modifier du graphe de dépendances (générique)

Utilise un seul modifier pour installer les environment objects et gérer les hooks de lifecycle quand le compte/client actif change. Cela garde le wiring cohérent et évite d'oublier une dépendance dans les call sites.

```swift
extension View {
  func withAppDependencyGraph(
    accountManager: AccountManager = .shared,
    currentAccount: CurrentAccount = .shared,
    currentInstance: CurrentInstance = .shared,
    userPreferences: UserPreferences = .shared,
    theme: Theme = .shared,
    watcher: StreamWatcher = .shared,
    pushNotifications: PushNotificationsService = .shared,
    intentService: AppIntentService = .shared,
    quickLook: QuickLook = .shared,
    toastCenter: ToastCenter = .shared,
    namespace: Namespace.ID? = nil,
    isSupporter: Bool = false
  ) -> some View {
    environment(accountManager)
      .environment(accountManager.currentClient)
      .environment(quickLook)
      .environment(currentAccount)
      .environment(currentInstance)
      .environment(userPreferences)
      .environment(theme)
      .environment(watcher)
      .environment(pushNotifications)
      .environment(intentService)
      .environment(toastCenter)
      .environment(\.isSupporter, isSupporter)
      .task(id: accountManager.currentClient.id) {
        let client = accountManager.currentClient
        if let namespace { quickLook.namespace = namespace }
        currentAccount.setClient(client: client)
        currentInstance.setClient(client: client)
        userPreferences.setClient(client: client)
        await currentInstance.fetchCurrentInstance()
        watcher.setClient(client: client, instanceStreamingURL: currentInstance.instance?.streamingURL)
        if client.isAuth {
          watcher.watch(streams: [.user, .direct])
        } else {
          watcher.stopWatching()
        }
      }
      .task(id: accountManager.pushAccounts.map(\.token)) {
        pushNotifications.tokens = accountManager.pushAccounts.map(\.token)
      }
  }
}
```

Notes :
- Les hooks `.task(id:)` réagissent aux changements de compte/client, en réamorçant les services et le state du watcher.
- Garde le modifier concentré sur le wiring global ; le state propre à une feature reste dans les features.
- Ajuste les types (AccountManager, StreamWatcher, etc.) pour correspondre à ton projet.

## SwiftData / ModelContainer

Installe ton `ModelContainer` à la racine pour que toutes les feature views partagent le même store. Garde la liste réduite aux modèles qui ont besoin de persistence.

```swift
extension View {
  func withModelContainer() -> some View {
    modelContainer(for: [Draft.self, LocalTimeline.self, TagGroup.self])
  }
}
```

Pourquoi : un seul container évite des stores dupliqués par sheet ou par tab et garde les data cohérentes.

## Routing des sheets (piloté par enum)

Centralise les sheets avec un petit enum et un modifier helper.

```swift
enum SheetDestination: Identifiable {
  case composer
  case settings
  var id: String { String(describing: self) }
}

extension View {
  func withSheetDestinations(sheet: Binding<SheetDestination?>) -> some View {
    sheet(item: sheet) { destination in
      switch destination {
      case .composer:
        ComposerView().withEnvironments()
      case .settings:
        SettingsView().withEnvironments()
      }
    }
  }
}
```

Pourquoi : des sheets pilotées par enum gardent la présentation centralisée et testable ; ajouter une nouvelle sheet revient à ajouter un cas d'enum et une branche de switch.

## Quand l'utiliser

- Des apps avec plusieurs packages/modules qui partagent des environment objects et des services.
- Des apps qui doivent réagir aux changements de compte/client et rebrancher le streaming/push en sécurité.
- Toute app qui veut un wiring TabView + NavigationStack + sheet cohérent sans répéter le setup de l'environment.

## Points d'attention

- Garde le modifier de dépendances léger ; ne mets pas de state de feature ou de logique lourde là-dedans.
- Assure-toi que le travail `.task(id:)` est léger ou annulé correctement ; le travail de longue durée doit rester dans les services.
- Si des clients non authentifiés existent, verrouille les appels de streaming/watch pour éviter le spam de reconnexion.
