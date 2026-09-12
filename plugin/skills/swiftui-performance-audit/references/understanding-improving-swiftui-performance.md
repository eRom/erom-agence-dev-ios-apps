# Understanding and Improving SwiftUI Performance (Résumé)

Contexte : guidance Apple sur le diagnostic de la performance SwiftUI avec Instruments et l'application de design patterns pour réduire les updates longs ou fréquents.

## Concepts fondamentaux

- SwiftUI est déclaratif ; les view updates sont pilotés par le state, l'environment, et les dépendances de données observables.
- Les bodies de view doivent se calculer rapidement pour respecter les frame deadlines ; des updates lents ou fréquents provoquent des hitches.
- Instruments est l'outil principal pour trouver les updates longs et une fréquence d'update excessive.

## Workflow Instruments

1. Profiler via Product > Profile.
2. Choisir le SwiftUI template et enregistrer.
3. Exercer l'interaction cible.
4. Arrêter l'enregistrement et inspecter la track SwiftUI + le Time Profiler.

## Lanes de la timeline SwiftUI

- Update Groups : vue d'ensemble du temps que SwiftUI passe à calculer les updates.
- Long View Body Updates : orange >500us, rouge >1000us.
- Long Platform View Updates : hosting AppKit/UIKit dans SwiftUI.
- Other Long Updates : géométrie/texte/layout et autre travail SwiftUI.
- Hitches : frames manquées où l'UI n'était pas prête à temps.

## Diagnostiquer les long view body updates

- Développer la track SwiftUI ; inspecter les subtracks spécifiques à chaque module.
- Fixer une Inspection Range et corréler avec le Time Profiler.
- Utiliser le call tree ou le flame graph pour identifier les frames coûteuses.
- Répéter l'update pour rassembler assez d'échantillons pour l'analyse.
- Filtrer sur un update spécifique (Show Calls Made by `MySwiftUIView.body`).

## Diagnostiquer les updates fréquents

- Utiliser les Update Groups pour trouver des groupes actifs longs sans updates longs.
- Fixer une inspection range sur le groupe et analyser le nombre d'updates.
- Utiliser le Cause graph ("Show Causes") pour voir ce qui déclenche les updates.
- Comparer les causes avec le data flow attendu ; prioriser les causes les plus fréquentes.

## Patterns de remédiation

- Sortir le travail coûteux de `body` et mettre les résultats en cache.
- Utiliser la macro `Observable()` pour scoper les dépendances aux propriétés réellement lues.
- Éviter les dépendances larges qui font fan-out les updates vers de nombreuses views.
- Réduire le churn de layout ; isoler les subtrees dépendants du state des layout readers.
- Éviter de stocker des closures qui capturent le state du parent ; précalculer les child views.
- Gater les updates fréquents (par exemple les changements de géométrie) avec des seuils.

## Vérification

- Ré-enregistrer après les changements pour confirmer une réduction du nombre d'updates et moins de hitches.
