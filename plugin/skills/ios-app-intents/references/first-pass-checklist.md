# Checklist de première passe

Utilise cette checklist pour décider ce qu'exposer dans la première release App Intents.

## Choisis les premières actions

Choisis des actions qui sont :

- utiles sans avoir à parcourir toute l'app d'abord
- faciles à décrire en une phrase
- utiles dans Shortcuts, Siri, Spotlight ou les widgets
- portées par de la logique app déjà existante plutôt que d'exiger une réécriture majeure

Bons premiers candidats :

- composer quelque chose
- ouvrir une destination ou un objet
- trouver ou filtrer un objet connu
- continuer un workflow existant
- démarrer une action ciblée

À éviter en première passe :

- les flows de setup géants
- les actions qui n'ont de sens qu'après de nombreux taps in-app
- les écrans à faible valeur exposés uniquement parce qu'ils existent

## Choisis les premières entités

Utilise des app entities quand le système a besoin d'identifier ou d'afficher des objets de l'app.

Bonnes premières entités :

- compte
- liste
- filtre
- destination
- brouillon
- élément média

Garde chaque entité centrée sur :

- l'identifiant
- la représentation d'affichage
- les quelques champs dont le système a besoin pour le routing ou la désambiguïsation

Ne mire pas tout le modèle de persistance si un type system-facing bien plus petit fait l'affaire.

## Décide du modèle de handoff

Pour chaque intent, demande-toi :

- Cette action peut-elle se terminer directement depuis la system surface ?
- Doit-elle ouvrir l'app vers un endroit précis ?
- Si elle ouvre l'app, quelle est l'unique route propre pour revenir dans la scène principale ?

Préfère un seul service de routing ou de handoff explicite à de nombreux canaux latéraux spécifiques à chaque fonctionnalité.
</content>
