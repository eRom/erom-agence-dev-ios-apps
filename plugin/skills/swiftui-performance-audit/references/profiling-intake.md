# Checklist d'intake et de collecte pour le profiling

## Intention

Utilise cette checklist quand la revue de code seule ne peut pas expliquer le problème de performance SwiftUI et qu'il faut une preuve runtime de la part de l'utilisateur.

## À demander en premier

- Le symptôme exact : pic de CPU, frames perdues, croissance mémoire, hangs, ou view updates excessifs.
- L'interaction exacte : scrolling, saisie, chargement initial, navigation push/pop, animation, présentation d'une sheet, ou refresh en background.
- Le device et la version d'OS cibles.
- Si le problème a été reproduit sur un device réel ou seulement sur Simulator.
- La configuration de build : Debug ou Release.
- Si l'utilisateur a déjà une baseline ou une comparaison avant/après.

## Demande de profiling par défaut

Demande à l'utilisateur de :
- Lancer l'app en build Release quand c'est possible.
- Utiliser le SwiftUI template d'Instruments.
- Reproduire l'interaction problématique exacte, juste assez longtemps pour capturer le problème.
- Capturer la timeline SwiftUI et le Time Profiler ensemble.
- Exporter la trace ou fournir des captures d'écran des lanes SwiftUI clés et du call tree du Time Profiler.

## Artefacts à demander

- Export de trace ou captures d'écran des lanes SwiftUI concernées
- Capture d'écran ou export du call tree du Time Profiler
- Configuration device/OS/build
- Une courte note décrivant ce qui se passait au moment de la capture
- Si la mémoire est en cause, le memory graph ou les données Allocations si disponibles

## Quand demander plus

- Demande une deuxième capture si la première mélange plusieurs interactions.
- Demande une paire avant/après si l'utilisateur a déjà tenté un correctif.
- Demande une capture sur device si le problème n'apparaît que sur Simulator ou si la fluidité du scrolling compte.

## Pièges courants

- Les builds Debug peuvent fausser le timing et le comportement d'allocation de SwiftUI.
- Les traces Simulator peuvent manquer des problèmes de rendering ou de mémoire propres au device.
- Des interactions mélangées dans une même capture rendent l'attribution plus difficile.
- Des captures d'écran sans la note de reproduction sont beaucoup plus difficiles à interpréter.
