# Understanding Hangs in Your App (Résumé)

Contexte : guidance Apple sur l'identification des hangs causés par un travail long sur le main thread, et sur la compréhension du main run loop.

## Concepts clés

- Un hang est un délai perceptible dans une interaction discrète (typiquement >100 ms).
- Les hangs viennent presque toujours d'un travail long sur le main thread.
- Le main run loop traite séquentiellement les événements UI, les timers, et le travail sur la main queue.

## Étapes du travail sur le main thread

- Livraison de l'événement à la bonne view/au bon handler.
- Ton code : mises à jour de state, récupération de données, changements d'UI.
- Commit Core Animation vers le render server.

## Pourquoi le main run loop compte

- Seul le main thread peut mettre à jour l'UI en toute sécurité.
- Le run loop est le fondement qui exécute le travail de la main queue.
- Si le run loop est occupé, il ne peut pas traiter de nouveaux événements ; cela cause des hangs.

## Diagnostiquer les hangs

- Observer les périodes d'activité du main run loop : un loop sain dort la plupart du temps.
- La détection de hangs signale typiquement les périodes d'activité >250 ms.
- L'instrument Hangs peut être configuré pour abaisser les seuils.

## Points pratiques à retenir

- Garder le travail sur le main thread court ; décharger le travail lourd des event handlers.
- Éviter les tâches longues sur la main dispatch queue ou le main actor.
- Utiliser le comportement du run loop comme indicateur de la réactivité perçue par l'utilisateur.
