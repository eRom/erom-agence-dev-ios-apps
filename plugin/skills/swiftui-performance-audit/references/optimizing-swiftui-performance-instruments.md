# Optimizing SwiftUI Performance with Instruments (Résumé)

Contexte : session WWDC introduisant le nouvel instrument SwiftUI dans Instruments 26 et comment diagnostiquer les bottlenecks spécifiques à SwiftUI.

## Points clés à retenir

- Profiler les problèmes SwiftUI avec le SwiftUI template (SwiftUI instrument + Time Profiler + Hangs/Hitches).
- Les long view body updates sont un bottleneck fréquent ; utiliser "Long View Body Updates" pour identifier les bodies lents.
- Fixer une inspection range sur un update long et corréler avec le Time Profiler pour trouver les frames coûteuses.
- Sortir le travail de `body` : déplacer le formatting, le sorting, le décodage d'images et les autres travaux coûteux vers des chemins mis en cache ou précalculés.
- Utiliser le Cause & Effect Graph pour diagnostiquer *pourquoi* les updates se produisent ; SwiftUI est déclaratif, donc les backtraces sont souvent peu utiles.
- Éviter les dépendances larges qui déclenchent de nombreux updates (par exemple des arrays `@Observable` ou des lectures d'environment globales).
- Préférer des view models granulaires et un state scopé pour que seule la view affectée se mette à jour.
- Les vérifications d'update des environment values coûtent quand même du temps ; éviter de placer des valeurs qui changent vite (timers, géométrie) dans l'environment.
- Profiler tôt et souvent pendant le développement des features pour attraper les régressions.

## Workflow suggéré (condensé)

1. Enregistrer une trace en mode Release avec le SwiftUI template.
2. Inspecter "Long View Body Updates" et "Other Long Updates".
3. Zoomer sur un update long, puis inspecter le Time Profiler pour trouver les frames chaudes.
4. Corriger le travail lent dans body en déplaçant la logique lourde vers des chemins précalculés/en cache.
5. Utiliser le Cause & Effect Graph pour identifier un fan-out d'updates non intentionnel.
6. Ré-enregistrer et comparer le nombre d'updates et la fréquence des hitches.

## Exemples de patterns issus de la session

- Mettre en cache des strings de distance formatées dans un location manager au lieu de les calculer dans `body`.
- Remplacer une dépendance sur un array de favoris global par des view models par item pour réduire le fan-out d'updates.
