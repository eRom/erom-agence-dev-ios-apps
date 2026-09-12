---
name: ios-app-intents
description: "Concevoir des App Intents, App Entities et App Shortcuts pour les system surfaces iOS. À utiliser pour exposer des actions ou du contenu de l'app à Shortcuts, Siri, Spotlight, aux widgets ou aux controls."
---

# iOS App Intents

## Vue d'ensemble
Expose la plus petite surface d'actions et d'entités utile au système. Commence par les verbes et les objets que les gens voudraient vraiment en dehors de l'app, puis implémente une couche App Intents étroite capable de faire un deep-link ou un handoff propre vers l'app principale quand c'est nécessaire.

Lis ces références selon le besoin :

- `references/first-pass-checklist.md` pour choisir la première surface d'intents et d'entités
- `references/example-patterns.md` pour des formes d'exemples concrets à copier et adapter
- `references/code-templates.md` pour des templates de code App Intents génériques
- `references/system-surfaces.md` pour penser Shortcuts, Siri, Spotlight, widgets et autres points d'entrée système

## Workflow principal

### 1) Commence par les actions, pas les écrans
- Identifie les 1 à 3 actions à plus forte valeur qui devraient fonctionner hors de l'UI de l'app.
- Préfère des verbes comme compose, open, find, filter, continue, inspect ou start.
- Ne mire pas tout l'arbre de navigation de l'app en intents.

### 2) Définis une petite surface d'entités
- Ajoute des types `AppEntity` uniquement pour les objets que le système a besoin de comprendre ou de router.
- Garde la forme de l'entité plus étroite que le modèle de persistance de l'app.
- N'ajoute `EntityQuery` ou d'autres types de query que là où la désambiguïsation ou les suggestions apportent une vraie valeur.

### 3) Décide si l'action se termine sur place ou ouvre l'app
- Utilise des intents non-opening pour les actions qui peuvent se terminer directement depuis la system surface.
- Utilise `openAppWhenRun` ou des intents de type open quand l'utilisateur doit atterrir dans un workflow in-app précis.
- Quand l'app doit réagir dans la scène principale, ajoute un seul chemin de handoff runtime clair plutôt que d'éparpiller de la logique de routing ad hoc.
- Si l'action peut fonctionner dans les deux modes, envisage de livrer à la fois une version inline et une version open-app plutôt que de forcer un compromis.

### 4) Rends les actions découvrables
- Ajoute des entrées `AppShortcutsProvider` pour le premier ensemble d'intents à forte valeur.
- Choisis des titres, phrases et symboles qui font sens dans Shortcuts, Siri et Spotlight.
- Garde les phrases de shortcut directes et orientées tâche.
- Réutilise le même modèle d'action pour les widgets et les controls quand une configuration de widget ou un control piloté par intent a déjà besoin des mêmes paramètres.

### 5) Valide le handoff runtime
- Build l'app et confirme que le target intents compile proprement.
- Vérifie que l'app s'ouvre ou route vers l'endroit attendu quand un intent s'exécute.
- Résume quelles actions sont désormais exposées, quelles entités les portent, et comment l'app gère l'invocation.

## Défauts solides

- Préfère un target ou module intents dédié pour la couche system-facing.
- Garde les types intent fins ; la logique métier doit rester dans les services de l'app ou les modèles de domaine.
- Garde les app entities petites et adaptées à l'affichage.
- Utilise `AppEnum` pour des choix fixes de l'app comme des tabs, des modes ou des niveaux de visibilité avant de passer à un vrai type entity.
- Préfère une seule surface de routing app-intent prévisible dans la scène principale ou le root router de l'app.
- Traite les App Intents comme de l'infrastructure d'intégration système, pas seulement comme une fonctionnalité Shortcuts.

## Anti-patterns

- Exposer chaque écran ou tab comme son propre intent sans vraie valeur utilisateur.
- Mirer tout le graphe de modèle en types `AppEntity`.
- Cacher le handoff runtime dans des effets de bord globaux sans chemin d'entrée app clair.
- Ajouter des App Shortcuts avec des phrases vagues ou des titres génériques.
- Traiter la première passe App Intents comme un projet de taxonomie large plutôt que comme une petite release utile.

## Notes

- Documentation Apple à utiliser comme références principales :
  - `https://developer.apple.com/documentation/appintents/making-actions-and-content-discoverable-and-widely-available`
  - `https://developer.apple.com/documentation/appintents/creating-your-first-app-intent`
  - `https://developer.apple.com/documentation/appintents/adopting-app-intents-to-support-system-experiences`
- En plus des liens ci-dessus, utilise la recherche web pour consulter la documentation Apple Developer actuelle, les API App Intents ou le comportement de la plateforme ayant pu changer.
- Une bonne première passe inclut souvent un open-app intent, un action intent, un ou deux types d'entité, et un petit `AppShortcutsProvider`.
- Bonnes familles d'exemples à couvrir :
  - ouvrir une destination ou un éditeur dans l'app
  - effectuer une action légère inline sans ouvrir l'app
  - choisir dans un enum fixe comme un tab ou un mode
  - résoudre une ou plusieurs entités via `EntityQuery`
  - alimenter une configuration de widget ou des controls depuis la même surface d'entités
</content>
