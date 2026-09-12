# Demystify SwiftUI Performance (WWDC23) (Résumé)

Contexte : session WWDC23 sur la construction d'un modèle mental pour la performance SwiftUI et le triage des hangs/hitches.

## Boucle de performance

- Mesurer -> Identifier -> Optimiser -> Re-mesurer.
- Se concentrer sur des symptômes concrets (navigation lente, animations cassées, curseur qui tourne).

## Dépendances et updates

- Les views forment un graphe de dépendances ; les dynamic properties sont une source fréquente d'updates.
- Utiliser `Self._printChanges()` en debug uniquement pour inspecter les dépendances supplémentaires.
- Éliminer les dépendances inutiles en extrayant des views ou en réduisant la portée du state.
- Envisager `@Observable` pour un tracking des propriétés plus granulaire.

## Causes courantes d'updates lents

- Bodies de view coûteux (interpolation de string, filtering, formatting).
- Instanciation de dynamic properties et initialisation de state dans `body`.
- Résolution d'identity lente dans les listes/tables.
- Travail caché : lookups de bundle, allocations heap, construction répétée de strings.

## Éviter une initialisation lente dans les bodies de view

- Ne pas créer de modèles lourds de façon synchrone dans les bodies de view.
- Utiliser `.task` pour récupérer les données async et garder `init` léger.

## Règles d'identity pour les listes et les tables

- Une identity stable est critique pour la performance et l'animation.
- S'assurer d'un nombre constant de views par élément dans `ForEach`.
- Éviter le filtering inline dans `ForEach` ; préfiltrer et mettre en cache les collections.
- Éviter `AnyView` dans les lignes de liste ; cela masque l'identity et augmente le coût.
- Aplatir les `ForEach` imbriqués quand c'est possible pour réduire l'overhead.

## Spécificités de Table

- `TableRow` se résout en une seule ligne ; le nombre de lignes doit être constant.
- Préférer l'initialiseur `Table` simplifié pour imposer un nombre de lignes constant.
- Utiliser des IDs explicites pour le back deployment quand nécessaire.

## Aides au debugging

- Utiliser Instruments pour les hangs et les hitches.
- Utiliser `_printChanges` pour valider les hypothèses de dépendances pendant le debug.
