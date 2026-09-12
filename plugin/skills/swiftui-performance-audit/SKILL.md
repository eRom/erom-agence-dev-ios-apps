---
name: swiftui-performance-audit
description: "Audite les performances runtime de SwiftUI en partant du code. À utiliser pour diagnostiquer un rendering lent, un scrolling janky, des updates coûteux, ou un besoin de profiling."
---

# SwiftUI Performance Audit

## Démarrage rapide

Utilise cette skill pour diagnostiquer les problèmes de performance SwiftUI en partant du code, puis demande des preuves de profiling quand la revue de code seule ne suffit pas à expliquer les symptômes.

## Workflow

1. Classifie le symptôme : rendering lent, scrolling janky, CPU élevé, croissance mémoire, hangs, ou view updates excessifs.
2. Si le code est disponible, commence par une revue code-first avec `references/code-smells.md`.
3. Si le code n'est pas disponible, demande la plus petite tranche utile : la view cible, le data flow, les étapes de reproduction, et la cible de déploiement.
4. Si la revue de code n'est pas concluante ou qu'une preuve runtime est nécessaire, guide l'utilisateur dans le profiling avec `references/profiling-intake.md`.
5. Résume les causes probables, les preuves, les remédiations et les étapes de validation avec `references/report-template.md`.

## 1. Intake

Collecte :
- Le code de la view ou de la feature cible.
- Les symptômes et les étapes exactes de reproduction.
- Le data flow : `@State`, `@Binding`, dépendances d'environnement, et modèles observables.
- Si le problème apparaît sur device ou sur Simulator, et s'il a été observé en Debug ou en Release.

Demande à l'utilisateur de classifier le problème si possible :
- Pic de CPU ou drain de batterie
- Scrolling janky ou frames perdues
- Mémoire élevée ou pression sur les images
- Hangs ou interactions qui ne répondent plus
- View updates excessifs ou anormalement larges

Pour la checklist complète d'intake de profiling, lis `references/profiling-intake.md`.

## 2. Revue code-first

Concentre-toi sur :
- Les tempêtes d'invalidation dues à une observation trop large ou à des lectures d'environment.
- L'identity instable dans les listes et les `ForEach`.
- Le travail dérivé coûteux dans `body` ou dans les view builders.
- Le layout thrash causé par des hiérarchies complexes, `GeometryReader`, ou des chaînes de preferences.
- Le décodage ou le resize d'images lourdes sur le main thread.
- Le travail d'animation ou de transition appliqué trop largement.

Utilise `references/code-smells.md` pour le catalogue détaillé des code smells et les pistes de correction.

Fournis :
- Les causes racines probables avec des références au code.
- Des corrections et refactors suggérés.
- Si nécessaire, un repro minimal ou une suggestion d'instrumentation.

## 3. Guider l'utilisateur vers le profiling

Si la revue de code n'explique pas le problème, demande une preuve runtime :
- Un export de trace ou des captures d'écran de la timeline SwiftUI et du call tree du Time Profiler.
- La configuration device/OS/build.
- L'interaction exacte en cours de profiling.
- Des métriques avant/après si l'utilisateur compare un changement.

Utilise `references/profiling-intake.md` pour la checklist exacte et les étapes de collecte.

## 4. Analyser et diagnostiquer

- Fais correspondre la preuve à la catégorie la plus probable : invalidation, identity churn, layout thrash, travail sur le main thread, coût des images, ou coût des animations.
- Priorise les problèmes par impact, pas par facilité d'explication.
- Distingue une suspicion basée sur le code d'une preuve confirmée par une trace.
- Signale explicitement quand le profiling reste insuffisant et quelle preuve supplémentaire réduirait l'incertitude.

## 5. Remédier

Applique des corrections ciblées :
- Réduis la portée du state et limite le fan-out d'observation trop large.
- Stabilise les identités pour `ForEach` et les listes.
- Sors le travail lourd de `body` vers un state dérivé mis à jour depuis les inputs, un précalcul au niveau du modèle, des helpers mémoïsés, ou un préprocessing en background. Utilise `@State` uniquement pour le state possédé par la view, pas comme un cache ad hoc pour un calcul arbitraire.
- Utilise `equatable()` uniquement quand l'égalité est moins coûteuse que de recalculer le subtree et que les inputs sont réellement value-semantic.
- Downsample les images avant le rendering.
- Réduis la complexité du layout ou utilise un sizing fixe quand c'est possible.

Utilise `references/code-smells.md` pour des exemples, des indications spécifiques au fan-out d'Observation, et des patterns de remédiation.

## 6. Vérifier

Demande à l'utilisateur de relancer la même capture et de comparer avec les métriques de référence.
Résume le delta (CPU, frames perdues, pic mémoire) si disponible.

## Résultats attendus

Fournis :
- Un tableau de métriques court (avant/après si disponible).
- Les problèmes principaux (classés par impact).
- Les corrections proposées avec un effort estimé.

Utilise `references/report-template.md` pour formater l'audit final.

## Références

- Checklist d'intake et de collecte pour le profiling : `references/profiling-intake.md`
- Code smells courants et patterns de remédiation : `references/code-smells.md`
- Template de sortie d'audit : `references/report-template.md`
- Ajoute la documentation Apple et les ressources WWDC sous `references/` au fur et à mesure qu'elles sont fournies par l'utilisateur.
- Optimizing SwiftUI performance with Instruments : `references/optimizing-swiftui-performance-instruments.md`
- Understanding and improving SwiftUI performance : `references/understanding-improving-swiftui-performance.md`
- Understanding hangs in your app : `references/understanding-hangs-in-your-app.md`
- Demystify SwiftUI performance (WWDC23) : `references/demystify-swiftui-performance-wwdc23.md`
- En complément des références ci-dessus, utilise une recherche web pour consulter la documentation Apple Developer actuelle quand les workflows Instruments ou les recommandations de performance SwiftUI ont pu changer.
