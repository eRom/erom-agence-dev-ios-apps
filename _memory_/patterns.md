# Patterns

_Mis à jour : 2026-09-12_

## Rédaction des skills

- Noms de skills, champ `name:`, code, API, commandes et chemins en anglais ; prose, titres de section et commentaires de code en français.
- Jargon qu'un dev iOS français dit en anglais gardé tel quel (view, sheet, build, leak, flamegraph, hang...). Titre `#` de niveau 1 laissé en anglais, titres `##` traduits (« Intention », « Pièges », « Exemple : ... »).
- `description:` du frontmatter en français, entre guillemets YAML, mots-clés techniques anglais conservés pour le déclenchement.
- Aucun tiret cadratin dans ce qui est distribué (skills, README).
- Toute commande externe passe par `bunx` avec version épinglée (`serve-sim@0.1.46`, `xcodebuildmcp@2.7.0`), jamais `npx`.

## Traduction en lot

- Un agent par lot de fichiers, même lexique de jargon dans chaque brief, puis contrôle automatique contre `HEAD` : fences, titres, blocs de code (commentaires neutralisés), spans inline, cibles de liens, tirets cadratins. Frontmatter validé par `yaml.safe_load`.

## Git et publication

- Messages de commit en français, préfixe conventionnel (`feat:`, `docs:`, `chore:`), corps factuel, lignes `Co-Authored-By` et `Claude-Session`.
- Publication uniquement par `/erom-dev-plugin:release`, depuis ce dépôt : le plugin poussé d'abord, la marketplace ensuite, CI vérifiée sur le SHA poussé. Première publication = bump mineur de `metadata.version`.
- Carte du README par `/erom-dev-plugin:illustrate` : brouillon `low`, tirage `high`, textes vérifiés au zoom ; retouche par `gpt_image_edit` pour ajouter un élément.
