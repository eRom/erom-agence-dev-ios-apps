# Référence des patterns MV

Guidance condensée pour décider si une feature SwiftUI doit rester en MV pur ou introduire un view model.

Inspiré de la source fournie par l'utilisateur, "SwiftUI in 2025: Forget MVVM" (Thomas Ricouard), mais réécrit ici comme référence pratique de refactoring.

## Position par défaut

- Défaut sur MV : les views sont des expressions de state légères et des points d'orchestration.
- Privilégie `@State`, `@Environment`, `@Query`, `.task`, `.task(id:)`, et `onChange` avant de recourir à un view model.
- Garde la logique métier dans les services, modèles, ou types de domaine, pas dans le body de la view.
- Découpe les gros écrans en types de view plus petits avant d'inventer une couche de view model.
- Évite le fetching manuel ou la plomberie de state qui duplique les mécanismes de SwiftUI ou SwiftData.
- Teste d'abord les services, modèles, et transformations ; les views doivent rester simples et déclaratives.

## Quand éviter un view model

N'introduis pas de view model quand il ferait surtout :
- refléter un state local de la view,
- wrapper des valeurs déjà disponibles via `@Environment`,
- dupliquer un data flow basé sur `@Query`, `@State`, ou `Binding`,
- exister uniquement parce que le body de la view est trop long,
- porter une logique de chargement async ponctuelle qui peut vivre dans `.task` plus un state local de la view.

Dans ces cas, simplifie la view et le data flow plutôt que d'ajouter de l'indirection.

## Quand un view model peut se justifier

Un view model peut être raisonnable quand au moins une de ces conditions est vraie :
- l'utilisateur en demande explicitement un,
- le codebase standardise déjà un pattern de view model pour cette feature,
- l'écran a besoin d'un reference model à longue durée de vie avec un comportement qui ne rentre pas naturellement dans les seuls services,
- la feature adapte une API non-SwiftUI qui a besoin d'un objet de bridge dédié,
- plusieurs views partagent le même state spécifique à la présentation et ce state n'est pas mieux modélisé comme donnée d'environment au niveau app.

Même dans ces cas, garde le view model petit, explicite, et non-optionnel quand c'est possible.

## Pattern préféré : state local plus environment

```swift
struct FeedView: View {
    @Environment(BlueSkyClient.self) private var client

    enum ViewState {
        case loading
        case error(String)
        case loaded([Post])
    }

    @State private var viewState: ViewState = .loading

    var body: some View {
        List {
            switch viewState {
            case .loading:
                ProgressView("Loading feed...")
            case .error(let message):
                ErrorStateView(message: message, retryAction: { await loadFeed() })
            case .loaded(let posts):
                ForEach(posts) { post in
                    PostRowView(post: post)
                }
            }
        }
        .task { await loadFeed() }
    }

    private func loadFeed() async {
        do {
            let posts = try await client.getFeed()
            viewState = .loaded(posts)
        } catch {
            viewState = .error(error.localizedDescription)
        }
    }
}
```

Pourquoi c'est préférable :
- le state reste proche de l'UI qui le rend,
- les dépendances viennent de l'environment plutôt que d'un objet wrapper,
- la view coordonne le flux UI tandis que le service porte le vrai travail.

## Pattern préféré : utiliser les modifiers comme orchestration légère

```swift
.task(id: searchText) {
    guard !searchText.isEmpty else {
        results = []
        return
    }
    await searchFeed(query: searchText)
}

.onChange(of: isInSearch, initial: false) {
    guard !isInSearch else { return }
    Task { await fetchSuggestedFeed() }
}
```

Utilise les modifiers de lifecycle de la view pour une orchestration simple et locale. Ne convertis pas cela en view model par défaut, sauf si le comportement dépasse clairement la view.

## Note sur SwiftData

SwiftData est un argument fort pour garder le data flow à l'intérieur de la view quand c'est possible.

Préfère :

```swift
struct BookListView: View {
    @Query private var books: [Book]
    @Environment(\.modelContext) private var modelContext

    var body: some View {
        List {
            ForEach(books) { book in
                BookRowView(book: book)
                    .swipeActions {
                        Button("Delete", role: .destructive) {
                            modelContext.delete(book)
                        }
                    }
            }
        }
    }
}
```

Évite d'ajouter un view model qui fetch et reflète manuellement le même state, sauf si la feature a une raison explicite de le faire.

## Guidance sur les tests

Préfère tester :
- les services et les règles métier,
- les modèles et les transformations de state,
- les workflows async au niveau de la couche service,
- le comportement UI avec des previews ou des tests UI de plus haut niveau.

N'introduis pas de view model principalement pour rendre une view SwiftUI simple "testable". Cela ajoute généralement de la cérémonie sans améliorer l'architecture.

## Checklist de refactor

Pour refactorer vers MV :
- Retire les view models qui se contentent de wrapper des dépendances d'environment ou un state local de la view.
- Remplace les view models optionnels ou à initialisation différée quand un simple state de view suffit.
- Sors la logique métier du body de la view vers les services/modèles.
- Garde la view comme un coordinateur léger du state UI, de la navigation, et des actions utilisateur.
- Découpe les gros bodies en types de view plus petits avant d'ajouter de nouvelles couches d'indirection.

## En résumé

Traite les view models comme l'exception, pas le défaut.

Dans le SwiftUI moderne, la stack par défaut est :
- `@State` pour le state local,
- `@Environment` pour les dépendances partagées,
- `@Query` pour les collections backées par SwiftData,
- des lifecycle modifiers pour une orchestration légère,
- des services et modèles pour la logique métier.

Ne recours à un view model que quand la feature en a clairement besoin.
