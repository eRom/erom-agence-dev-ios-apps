---
name: swiftui-ui-patterns
description: Construit et refactore l'UI SwiftUI avec des patterns de composants et des exemples. À utiliser pour façonner la navigation, le state, les layouts, les controls ou la composition d'écran.
---

# SwiftUI UI Patterns

## Démarrage rapide

Choisis une piste selon ton objectif :

### Projet existant

- Identifie la feature ou l'écran et le modèle d'interaction principal (list, detail, editor, settings, tabbed).
- Trouve un exemple proche dans le repo avec `rg "TabView\("` ou équivalent, puis lis la view SwiftUI la plus proche.
- Applique les conventions locales : privilégie le state SwiftUI natif, garde le state local quand c'est possible, et utilise l'injection via l'environment pour les dépendances partagées.
- Choisis la référence de composant pertinente dans `references/components-index.md` et suis ses indications.
- Si l'interaction révèle du contenu secondaire en faisant glisser ou scroller le contenu principal, lis `references/scroll-reveal.md` avant d'implémenter les gestures à la main.
- Construis la view avec des subviews petites et ciblées, et un flux de données SwiftUI natif.

### Nouveau projet (scaffolding)

- Commence par `references/app-wiring.md` pour brancher TabView + NavigationStack + sheets.
- Ajoute un `AppTab` et un `RouterPath` minimaux à partir des skeletons fournis.
- Choisis la prochaine référence de composant selon l'UI dont tu as besoin en premier (TabView, NavigationStack, Sheets).
- Étends les enums de route et de sheet à mesure que de nouveaux écrans arrivent.

## Règles générales à suivre

- Utilise le state SwiftUI moderne (`@State`, `@Binding`, `@Observable`, `@Environment`) et évite les view models inutiles.
- Si la deployment target inclut iOS 16 ou antérieur et ne peut pas utiliser l'Observation API introduite avec iOS 17, retombe sur `ObservableObject` avec `@StateObject` pour l'ownership racine, `@ObservedObject` pour l'observation injectée, et `@EnvironmentObject` uniquement pour du state vraiment partagé au niveau de l'app.
- Privilégie la composition ; garde les views petites et ciblées.
- Utilise async/await avec `.task` et des états explicites de loading/error. Pour le redémarrage, l'annulation et le debouncing, lis `references/async-state.md`.
- Garde les services partagés de l'app dans `@Environment`, mais privilégie l'injection explicite via initializer pour les dépendances et modèles propres à une feature. Pour les patterns de wiring racine, lis `references/app-wiring.md`.
- Privilégie l'API SwiftUI la plus récente compatible avec la deployment target, et précise l'OS minimum chaque fois qu'un pattern en dépend.
- Ne conserve les patterns legacy que dans les fichiers déjà legacy.
- Suis le formatter et le style guide du projet.
- **Sheets** : privilégie `.sheet(item:)` à `.sheet(isPresented:)` quand le state représente un modèle sélectionné. Évite le `if let` dans le body d'une sheet. Les sheets doivent posséder leurs propres actions et appeler `dismiss()` en interne plutôt que de transmettre des closures `onCancel`/`onConfirm`.
- **Scroll-driven reveals** : privilégie une valeur de progression normalisée dérivée de l'offset de scroll, et pilote le state visuel depuis cette unique source de vérité. Évite les machines à état de gesture en parallèle, sauf si le scroll seul ne peut pas exprimer l'interaction.

## Résumé de l'ownership du state

Utilise l'outil de state le plus étroit correspondant au modèle d'ownership :

| Scénario | Pattern préféré |
| --- | --- |
| State UI local possédé par une seule view | `@State` |
| Un enfant mute une valeur possédée par le parent | `@Binding` |
| Modèle référence possédé à la racine sur iOS 17+ | `@State` avec un type `@Observable` |
| Un enfant lit ou mute un modèle `@Observable` injecté sur iOS 17+ | Le passer explicitement en stored property |
| Service ou configuration partagé au niveau de l'app | `@Environment(Type.self)` |
| Modèle référence legacy sur iOS 16 et antérieur | `@StateObject` à la racine, `@ObservedObject` quand injecté |

Choisis d'abord l'emplacement de l'ownership, puis le wrapper. N'introduis pas de modèle référence quand un state par valeur suffit.

## Références transverses

- En complément des références ci-dessous, utilise la recherche web pour consulter la documentation Apple Developer actuelle quand les API SwiftUI, leur disponibilité, ou les recommandations de la plateforme ont pu changer.
- `references/navigationstack.md` : ownership de la navigation, historique par tab, et routing par enum.
- `references/sheets.md` : présentation modale centralisée et sheets pilotées par enum.
- `references/deeplinks.md` : gestion des URL et routing des liens externes vers les destinations de l'app.
- `references/app-wiring.md` : graphe de dépendances racine, usage de l'environment, et wiring de la coquille d'app.
- `references/async-state.md` : `.task`, `.task(id:)`, annulation, debouncing, et state UI asynchrone.
- `references/previews.md` : `#Preview`, fixtures, environments mockés, et setup de preview isolé.
- `references/performance.md` : identité stable, scope d'observation, containers lazy, et garde-fous de coût de rendu.

## Anti-patterns

- Des views géantes qui mélangent layout, logique métier, réseau, routing et formatting dans un seul fichier.
- Plusieurs flags booléens pour des sheets, alerts ou destinations de navigation mutuellement exclusives.
- Des appels de service en direct à l'intérieur de chemins de code pilotés par `body`, au lieu des hooks de lifecycle de la view ou de modèles/services injectés.
- Recourir à `AnyView` pour contourner des incompatibilités de type qui devraient se résoudre par une meilleure composition.
- Faire retomber par défaut chaque dépendance partagée sur `@EnvironmentObject` ou un router global sans raison d'ownership claire.

## Workflow pour une nouvelle view SwiftUI

1. Définis le state de la view, son emplacement d'ownership, et les hypothèses d'OS minimum avant d'écrire le code UI.
2. Identifie les dépendances qui doivent aller dans `@Environment` et celles qui doivent rester des entrées explicites de l'initializer.
3. Esquisse la hiérarchie de views, le modèle de routing, et les points de présentation ; extrais les parties répétées en subviews. Pour une navigation complexe, lis `references/navigationstack.md`, `references/sheets.md`, ou `references/deeplinks.md`. **Build et vérifie l'absence d'erreurs de compilation avant de continuer.**
4. Implémente le chargement asynchrone avec `.task` ou `.task(id:)`, plus des états explicites de loading et error si besoin. Lis `references/async-state.md` quand le travail dépend d'inputs changeants ou d'annulation.
5. Ajoute des previews pour les états principaux et secondaires, puis ajoute des labels ou identifiers d'accessibilité quand l'UI est interactive. Lis `references/previews.md` quand la view a besoin de fixtures ou de dépendances mockées injectées.
6. Valide avec un build : vérifie l'absence d'erreurs de compilation, vérifie que les previews rendent sans crasher, assure-toi que les changements de state se propagent correctement, et vérifie que l'identité de liste et le scope d'observation ne provoqueront pas de re-renders évitables. Lis `references/performance.md` si l'écran est grand, riche en scroll, ou mis à jour fréquemment. Pour les erreurs de compilation SwiftUI courantes (annotations `@State` manquantes, closures `ViewBuilder` ambiguës, ou types génériques mal assortis), résous-les avant de mettre à jour les callsites. **Si le build échoue :** lis le message d'erreur attentivement, corrige le problème identifié, puis rebuild avant de passer à l'étape suivante. Si une preview crashe, isole la subview fautive, vérifie que son initialisation de state est valide, et relance la preview avant de continuer.

## Références de composants

Utilise `references/components-index.md` comme point d'entrée. Chaque référence de composant doit inclure :
- L'intention et les scénarios où elle s'applique le mieux.
- Un pattern d'usage minimal avec les conventions locales.
- Les pièges et notes de performance.
- Des chemins vers des exemples existants dans le repo courant.

## Ajouter une nouvelle référence de composant

- Crée `references/<component>.md`.
- Garde-la courte et actionnable ; renvoie vers des fichiers concrets du repo courant.
- Mets à jour `references/components-index.md` avec la nouvelle entrée.
