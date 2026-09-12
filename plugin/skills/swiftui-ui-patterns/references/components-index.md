# Index des composants

Utilise ce fichier pour trouver les guides de composants et les références transverses. Chaque entrée indique quand l'utiliser.

## Composants disponibles

- TabView : `references/tabview.md` : à utiliser pour construire une app à tabs ou tout ensemble de features avec des tabs.
- NavigationStack : `references/navigationstack.md` : à utiliser quand tu as besoin de navigation push et de routing programmatique, notamment un historique par tab.
- Sheets et présentation : `references/sheets.md` : à utiliser pour des sheets locales pilotées par item, du routing modal centralisé, et des patterns d'action propres aux sheets.
- Form et Settings : `references/form.md` : à utiliser pour les settings, les inputs groupés, et la saisie de data structurée.
- macOS Settings : `references/macos-settings.md` : à utiliser pour construire une fenêtre Settings macOS avec la scene `Settings` de SwiftUI.
- Split views et colonnes : `references/split-views.md` : à utiliser pour des layouts multi-colonnes iPad/macOS ou des colonnes secondaires custom.
- List et Section : `references/list.md` : à utiliser pour du contenu façon feed et des rows de settings.
- ScrollView et lazy stacks : `references/scrollview.md` : à utiliser pour des layouts custom, des scrollers horizontaux, ou des grids.
- Surfaces de detail en scroll-reveal : `references/scroll-reveal.md` : à utiliser quand un écran de detail révèle du contenu ou des actions secondaires au fil du scroll ou du swipe entre des sections plein écran.
- Grids : `references/grids.md` : à utiliser pour des icon pickers, des galeries média, et des layouts en mosaïque.
- Theming et dynamic type : `references/theming.md` : à utiliser pour les tokens de theme à l'échelle de l'app, les couleurs, et le scaling du type.
- Controls (toggles, pickers, sliders) : `references/controls.md` : à utiliser pour les controls de settings et la sélection d'input.
- Input toolbar (ancrée en bas) : `references/input-toolbar.md` : à utiliser pour les écrans chat/composer avec une barre d'input collante.
- Overlays de top bar (iOS 26+ et fallback) : `references/top-bar.md` : à utiliser pour des sélecteurs ou pills épinglés au-dessus du contenu scrollé.
- Overlay et toasts : `references/overlay.md` : à utiliser pour de l'UI transitoire comme des bannières ou des toasts.
- Gestion du focus : `references/focus.md` : à utiliser pour chaîner des champs et gérer le focus clavier.
- Searchable : `references/searchable.md` : à utiliser pour une UI de recherche native avec scopes et résultats asynchrones.
- Images et média asynchrones : `references/media.md` : à utiliser pour du média distant, des previews, et des viewers de média.
- Haptics : `references/haptics.md` : à utiliser pour du feedback tactile lié à des actions clés.
- Matched transitions : `references/matched-transitions.md` : à utiliser pour des animations fluides source-vers-destination.
- Deep links et routing d'URL : `references/deeplinks.md` : à utiliser pour la navigation in-app depuis des URL.
- Title menus : `references/title-menus.md` : à utiliser pour des menus de filtre ou de contexte dans le titre de navigation.
- Menu bar commands : `references/menu-bar.md` : à utiliser pour ajouter ou personnaliser des commandes de menu bar macOS/iPadOS.
- Loading & placeholders : `references/loading-placeholders.md` : à utiliser pour des skeletons redacted, des empty states, et l'UX de loading.
- Lightweight clients : `references/lightweight-clients.md` : à utiliser pour des petits clients API à base de closures injectés dans des stores.

## Références transverses

- Wiring de l'app et graphe de dépendances : `references/app-wiring.md` : à utiliser pour brancher la coquille d'app, installer les dépendances partagées, et décider ce qui va dans l'environment.
- State asynchrone et lifecycle des tasks : `references/async-state.md` : à utiliser quand une view charge des data, réagit à un input changeant, ou a besoin de guidance sur l'annulation/le debouncing.
- Previews : `references/previews.md` : à utiliser en ajoutant `#Preview`, des fixtures, des environments mockés, ou un setup de preview isolé.
- Garde-fous de performance : `references/performance.md` : à utiliser quand un écran est grand, riche en scroll, mis à jour fréquemment, ou montre des signes de re-renders évitables.

## Composants prévus (créer les fichiers au besoin)

- Contenu web : créer `references/webview.md` : à utiliser pour du contenu web embarqué ou du in-app browsing.
- Patterns de composer de statut : créer `references/composer.md` : à utiliser pour des workflows de composition ou d'édition.
- Saisie de texte et validation : créer `references/text-input.md` : à utiliser pour les forms, la validation, et la saisie riche en texte.
- Usage du design system : créer `references/design-system.md` : à utiliser en appliquant des règles de style partagées.

## Ajouter des entrées

- Ajoute le fichier du composant et lie-le ici avec une courte description « quand l'utiliser ».
- Garde chaque référence de composant courte et actionnable.
