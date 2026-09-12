---
name: swiftui-view-refactor
description: "Refactore des fichiers de view SwiftUI vers une structure stable et testable. À utiliser pour découper de grosses views, resserrer le data flow, ou nettoyer l'ownership d'Observation."
---

# SwiftUI View Refactor

## Vue d'ensemble
Refactore les views SwiftUI vers des types de view petits, explicites et stables. Défaut sur du SwiftUI vanilla : state local dans la view, dépendances partagées dans l'environment, logique métier dans les services/modèles, et view models uniquement quand la demande ou le code existant l'exige clairement.

## Directives fondamentales

### 1) Ordre des éléments de la view (haut → bas)
- Impose cet ordre, sauf si le fichier existant a une convention locale plus forte à préserver.
- Environment
- `private`/`public` `let`
- `@State` / autres stored properties
- computed `var` (non-view)
- `init`
- `body`
- computed view builders / autres helpers de view
- fonctions helper / async

### 2) Défaut sur MV, pas MVVM
- Les views doivent être des expressions de state légères et des points d'orchestration, pas des conteneurs de logique métier.
- Privilégie `@State`, `@Environment`, `@Query`, `.task`, `.task(id:)`, et `onChange` avant de recourir à un view model.
- Injecte les services et modèles partagés via `@Environment` ; garde la logique de domaine dans les services/modèles, pas dans le body de la view.
- N'introduis pas de view model juste pour refléter un state local de la view ou wrapper des dépendances d'environment.
- Si un écran devient gros, découpe l'UI en subviews avant d'inventer une nouvelle couche de view model.

### 3) Préfère fortement des types de subview dédiés aux helpers `some View` calculés
- Signale les propriétés `body` plus longues qu'environ un écran, ou qui contiennent plusieurs sections logiques.
- Préfère extraire des types `View` dédiés pour les sections non triviales, surtout quand elles ont du state, du travail async, du branching, ou méritent leur propre preview.
- Garde les helpers `some View` calculés rares et petits. Ne construis pas un écran entier à partir de fragments façon `private var header: some View`.
- Passe des inputs petits et explicites (données, bindings, callbacks) aux subviews extraites, plutôt que de leur transmettre tout le state du parent.
- Si une subview extraite devient réutilisable ou porte un sens propre, déplace-la dans son propre fichier.

Préfère :

```swift
var body: some View {
    List {
        HeaderSection(title: title, subtitle: subtitle)
        FilterSection(
            filterOptions: filterOptions,
            selectedFilter: $selectedFilter
        )
        ResultsSection(items: filteredItems)
        FooterSection()
    }
}

private struct HeaderSection: View {
    let title: String
    let subtitle: String

    var body: some View {
        VStack(alignment: .leading, spacing: 6) {
            Text(title).font(.title2)
            Text(subtitle).font(.subheadline)
        }
    }
}

private struct FilterSection: View {
    let filterOptions: [FilterOption]
    @Binding var selectedFilter: FilterOption

    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack {
                ForEach(filterOptions, id: \.self) { option in
                    FilterChip(option: option, isSelected: option == selectedFilter)
                        .onTapGesture { selectedFilter = option }
                }
            }
        }
    }
}
```

Évite :

```swift
var body: some View {
    List {
        header
        filters
        results
        footer
    }
}

private var header: some View {
    VStack(alignment: .leading, spacing: 6) {
        Text(title).font(.title2)
        Text(subtitle).font(.subheadline)
    }
}
```

### 3b) Extrais les actions et effets de bord hors de `body`
- Ne garde pas d'actions de bouton non triviales inline dans le body de la view.
- N'enfouis pas de logique métier dans `.task`, `.onAppear`, `.onChange`, ou `.refreshable`.
- Préfère appeler de petites méthodes privées depuis la view, et déplace la vraie logique métier dans les services/modèles.
- Le body doit se lire comme de l'UI, pas comme un view controller.

```swift
Button("Save", action: save)
    .disabled(isSaving)

.task(id: searchText) {
    await reload(for: searchText)
}

private func save() {
    Task { await saveAsync() }
}

private func reload(for searchText: String) async {
    guard !searchText.isEmpty else {
        results = []
        return
    }
    await searchService.search(searchText)
}
```

### 4) Garde un view tree stable (évite la permutation conditionnelle de view au niveau racine)
- Évite les `body` ou computed views qui retournent des branches racines complètement différentes via `if/else`.
- Préfère une seule base view stable avec des conditions à l'intérieur des sections/modifiers (`overlay`, `opacity`, `disabled`, `toolbar`, etc.).
- Le branch swapping au niveau racine cause du churn d'identity, une invalidation plus large, et du recalcul supplémentaire.

Préfère :

```swift
var body: some View {
    List {
        documentsListContent
    }
    .toolbar {
        if canEdit {
            editToolbar
        }
    }
}
```

Évite :

```swift
var documentsListView: some View {
    if canEdit {
        editableDocumentsList
    } else {
        readOnlyDocumentsList
    }
}
```

### 5) Gestion du view model (uniquement si déjà présent ou explicitement demandé)
- Traite les view models comme un pattern legacy ou de besoin explicite, pas comme le défaut.
- N'introduis pas de view model sauf si la demande ou le code existant l'exige clairement.
- Si un view model existe, rends-le non-optionnel quand c'est possible.
- Passe les dépendances à la view via `init`, puis crée le view model dans l'`init` de la view.
- Évite les patterns `bootstrapIfNeeded` et autres contournements de setup différé.

Exemple (basé sur Observation) :

```swift
@State private var viewModel: SomeViewModel

init(dependency: Dependency) {
    _viewModel = State(initialValue: SomeViewModel(dependency: dependency))
}
```

### 6) Usage d'Observation
- Pour les reference types `@Observable` sur iOS 17+, stocke-les comme `@State` dans la view propriétaire.
- Passe les observables explicitement en aval ; évite le state optionnel sauf si l'UI en a réellement besoin.
- Si la cible de déploiement inclut iOS 16 ou antérieur, utilise `@StateObject` chez le propriétaire et `@ObservedObject` lors de l'injection de modèles observables legacy.

## Workflow

1. Réordonne la view pour respecter les règles d'ordre.
2. Retire les actions et effets de bord inline de `body` ; déplace la logique métier dans les services/modèles et ne garde qu'une orchestration légère dans la view.
3. Raccourcis les bodies longs en extrayant des types de subview dédiés ; évite de reconstruire l'écran à partir de nombreux helpers `some View` calculés.
4. Assure une structure de view stable : évite le branch swapping via `if` au niveau racine ; déplace les conditions vers des sections/modifiers localisés.
5. Si un view model existe ou est explicitement requis, remplace les view models optionnels par un view model `@State` non-optionnel initialisé dans `init`.
6. Confirme l'usage d'Observation : `@State` pour les modèles `@Observable` racines sur iOS 17+, wrappers legacy uniquement quand la cible de déploiement l'exige.
7. Garde le comportement intact : ne change pas le layout ou la logique métier sauf demande explicite.

## Notes

- Préfère des types de view petits et explicites aux gros blocs conditionnels et aux grosses propriétés `some View` calculées.
- Garde les computed view builders sous `body` et les computed vars non-view au-dessus de `init`.
- Un bon refactor SwiftUI doit faire lire la view de haut en bas comme du data flow plus du layout, pas comme du layout mélangé à de la logique impérative.
- Pour la guidance et le raisonnement MV-first, voir `references/mv-patterns.md`.
- En complément des références ci-dessus, utilise une recherche web pour consulter la documentation Apple Developer actuelle quand les API SwiftUI, le comportement d'Observation, ou les recommandations de plateforme ont pu changer.

## Gestion des grosses views

Quand un fichier de view SwiftUI dépasse environ 300 lignes, découpe-le agressivement. Extrais les sections significatives en types `View` dédiés plutôt que de cacher la complexité dans de nombreuses computed properties. Utilise des extensions `private` avec des commentaires `// MARK: -` pour les actions et les helpers, mais ne traite pas les extensions comme un substitut au découpage d'un écran géant en types de view plus petits. Si une subview extraite est réutilisée ou porte un sens propre, déplace-la dans son propre fichier.
