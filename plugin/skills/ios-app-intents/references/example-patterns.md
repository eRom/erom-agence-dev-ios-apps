# Patterns d'exemples

Utilise ceux-ci comme points de départ pour décider quoi exposer en premier.

## 1) Intent de handoff open-app

Idéal pour :

- les flows de composition
- les éditeurs
- la navigation vers une destination
- les actions qui ont besoin de la scène app complète, de l'état d'auth, ou d'une UI plus riche

Pattern :

- `openAppWhenRun = true`
- collecter une entrée légère dans l'intent
- stocker un seul payload d'intent traité dans un router central ou un service de handoff
- laisser la scène app traduire ce payload en tabs, sheets, routes ou windows

Exemple :

- « Ouvrir l'app pour composer un brouillon »
- « Ouvrir l'app sur une section sélectionnée »
- « Ouvrir un éditeur prérempli avec le contenu de l'étape shortcut précédente »

## 2) Intent d'action inline en arrière-plan

Idéal pour :

- les actions rapides de create/update
- les opérations send, archive, mark, favorite ou toggle
- les actions qui peuvent se terminer sans l'UI principale de l'app

Pattern :

- `openAppWhenRun = false`
- effectuer l'opération directement dans `perform()`
- retourner un feedback dialog ou snippet pour que le résultat paraisse complet dans Shortcuts ou Siri

Exemple :

- « Créer une tâche »
- « Envoyer un message »
- « Archiver un document »

## 3) Variantes appariées open-app et inline

Idéal pour :

- les actions qui ont besoin à la fois d'automatisation et d'une revue manuelle plus riche
- les flows où certains utilisateurs veulent un shortcut en arrière-plan tandis que d'autres veulent atterrir dans l'app

Pattern :

- garder les noms de paramètres alignés entre les deux intents
- laisser la version open-app faire le handoff vers l'UI
- laisser la version inline appeler directement le même service de domaine
- exposer les deux dans `AppShortcutsProvider` avec des titres clairs

Exemple :

- « Composer dans l'app » et « Envoyer maintenant »
- « Ouvrir l'éditeur de post image » et « Poster des images en arrière-plan »

## 4) Choix fixe via `AppEnum`

Idéal pour :

- les tabs
- les modes
- les niveaux de visibilité
- les petits ensembles de filtres ou catégories

Pattern :

- définir un `AppEnum`
- donner à chaque case une `DisplayRepresentation` orientée utilisateur
- mapper les cases de l'enum vers des types spécifiques à l'app en un seul endroit

Exemple :

- ouvrir un tab sélectionné
- exécuter une action en mode « public », « private » ou « team »

## 5) Sélection portée par une entité via `AppEntity`

Idéal pour :

- les comptes
- les projets
- les listes
- les destinations
- les recherches enregistrées

Pattern :

- exposer uniquement les champs nécessaires à l'affichage et à la recherche
- ajouter `suggestedEntities()` pour l'UX du picker
- ajouter `defaultResult()` seulement quand il existe un défaut vraiment utile
- garder la logique de fetch réseau ou base de données dans le type query, pas dans la couche view

Exemple :

- choisir un compte depuis lequel poster
- choisir un projet à ouvrir
- sélectionner une liste enregistrée pour un widget

## 6) Dépendance de query entre paramètres

Idéal pour :

- quand un paramètre change les choix valides d'un autre
- la configuration de widget ou de control où « compte » déterrmine « projet »

Pattern :

- utiliser `@IntentParameterDependency` dans la query
- lire le paramètre en amont
- restreindre le fetch d'entités à la valeur parente choisie

Exemple :

- le workspace sélectionné filtre les documents disponibles
- le compte sélectionné filtre les listes disponibles

## 7) Intent de configuration de widget

Idéal pour :

- les widgets qui ont besoin d'un compte, projet, filtre ou destination sélectionné
- les controls pilotés par intent qui doivent réutiliser le même modèle de paramètres

Pattern :

- définir un `WidgetConfigurationIntent`
- utiliser les mêmes types `AppEntity` que ceux déjà utilisés par les shortcuts
- fournir des valeurs d'exemple adaptées aux previews quand le widget en a besoin

Exemple :

- choisir un compte plus une liste
- choisir un projet plus un filtre de statut

## 8) Design des phrases de shortcut

Idéal pour :

- rendre les actions découvrables dans Siri et Shortcuts

Pattern :

- garder les phrases courtes et menées par le verbe
- exposer une ou deux phrases canoniques, puis n'ajouter que quelques variantes naturelles
- utiliser un `shortTitle` et un `systemImageName` précis

Exemple :

- « Create a note with \(.applicationName) »
- « Open inbox in \(.applicationName) »
- « Send image with \(.applicationName) »
</content>
