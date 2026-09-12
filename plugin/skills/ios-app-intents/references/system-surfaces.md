# System surfaces

Pense en points d'entrée système, pas seulement en shortcuts.

## Shortcuts

- Idéal pour les actions directes et les chaînes d'automatisation.
- Expose les actions que les utilisateurs voudraient vraiment réutiliser.
- Ajoute des entrées `AppShortcutsProvider` pour les premiers intents à forte valeur.

## Siri

- Idéal pour des verbes clairs et des actions deep-linkable.
- Formule les titres et les paramètres pour que le système puisse les présenter et les désambiguïser clairement.

## Spotlight

- Idéal pour la découvrabilité des actions comme des entités.
- Utilise des représentations d'affichage fortes et des noms de type clairs.

## Widgets, Live Activities et controls

- Idéal quand les mêmes actions font déjà sens comme points d'entrée pilotés par intent.
- Réutilise la même surface d'intents quand c'est pratique plutôt que d'inventer des modèles d'action séparés.

## Consignes générales

- Conçois une seule petite couche d'action capable de servir plusieurs surfaces.
- Garde des noms d'action concrets et orientés utilisateur.
- Préfère des entités et des paramètres structurés plutôt que d'essayer de tout encoder en texte libre.
- Commence étroit, livre un ensemble utile, puis étends selon l'usage réel.
</content>
