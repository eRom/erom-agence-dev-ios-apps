# Template de sortie d'audit

## Intention

Utilise cette structure pour rapporter les résultats de l'audit de performance SwiftUI, afin que l'utilisateur voie rapidement le symptôme, la preuve, la cause probable et la prochaine étape de validation.

## Template

```markdown
## Résumé

[Un court paragraphe sur le bottleneck le plus probable et si la conclusion est appuyée par le code ou par une trace.]

## Constats

1. [Titre du problème]
   - Symptôme : [ce que voit l'utilisateur]
   - Cause probable : [cause racine]
   - Preuve : [référence au code ou preuve de profiling]
   - Correctif : [changement précis]
   - Validation : [ce qu'il faut mesurer après le correctif]

2. [Titre du problème]
   - Symptôme : ...
   - Cause probable : ...
   - Preuve : ...
   - Correctif : ...
   - Validation : ...

## Mesures

| Mesure | Avant | Après | Notes |
| --- | --- | --- | --- |
| CPU | [valeur] | [valeur] | [note] |
| Frame drops / hitching | [valeur] | [valeur] | [note] |
| Pic mémoire | [valeur] | [valeur] | [note] |

## Prochaine étape

[Une action concrète suivante : appliquer un correctif, capturer une meilleure trace, ou valider sur device.]
```

## Notes

- Classe les findings par impact, pas par ordre dans le fichier.
- Précise explicitement quand une conclusion est encore une hypothèse.
- Si aucune métrique n'est disponible, omets le tableau et indique ce qu'il faudrait mesurer ensuite.
