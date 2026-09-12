---
name: ios-memgraph-leaks
description: "Capturer et inspecter des leaks et memgraphs iOS. À utiliser pour débugger des objets leakés, des retain cycles, une croissance mémoire, ou produire des preuves de leak avant/après."
---

# iOS Memgraph Leaks

Utilise cette skill pour prouver des leaks iOS à partir d'un process simulator vivant ou d'un `.memgraph` existant. Combine-la avec `../ios-debugger-agent/SKILL.md` quand la tâche a aussi besoin de build, install, launch, UI driving, logs ou screenshots sur simulator.

## Workflow principal

1. Build, lance, et pilote le flow exact qui devrait libérer des objets.
2. Capture un memgraph depuis le process simulator en cours avec `scripts/capture_sim_memgraph.sh`.
3. Résume les leaks avec `scripts/summarize_memgraph_leaks.py`.
4. Pour chaque type leaké app-owned, inspecte l'ownership avec `leaks --traceTree=<address> <file.memgraph>` et les preuves de leak groupées.
5. Fais le patch root-cause le plus petit possible, puis recapture le même flow sur le même simulator quand c'est possible.
6. Rapporte la preuve : comptes de leaks avant/après, types root disparus, leaks restants, chemins de memgraph, et résultats de test/build.

Ne prétends jamais avoir corrigé un leak sur la seule base d'un memgraph plus petit. Un fix crédible explique le chemin d'ownership qui maintenait l'objet en vie et montre que ce même chemin ou type disparaît après le patch.

## Capture

Préfère capturer depuis le simulator déjà utilisé pour la reproduction. Résous l'UDID du simulator et le bundle identifier de l'app, puis capture l'app en cours :

```bash
SKILL_DIR="<absolute path to this loaded skill folder>"
SIM="<simulator-udid>"
BUNDLE_ID="<app.bundle.identifier>"
MEMGRAPH_DIR="$(mktemp -d "${TMPDIR:-/tmp}/erom-ios-memgraph.XXXXXX")"

"$SKILL_DIR/scripts/capture_sim_memgraph.sh" \
  --udid "$SIM" \
  --bundle-id "$BUNDLE_ID" \
  --out-dir "$MEMGRAPH_DIR"
```

Ne dérive pas `SKILL_DIR` du `pwd` du repo de l'app cible ; les plugins installés vivent en général hors de l'app débuggée. Stocke les captures dans un temp spécifique au run ou un dossier choisi par l'utilisateur, jamais sous `SKILL_DIR`.

Si le process n'est pas trouvé, vérifie le bundle identifier et utilise `xcrun simctl spawn "$SIM" launchctl list` pour inspecter les labels en cours d'exécution.

## Résumer

Résume un memgraph existant :

```bash
"$SKILL_DIR/scripts/summarize_memgraph_leaks.py" \
  /path/to/app.memgraph \
  --trace-limit 5 \
  --out /path/to/leak-summary.md
```

Utilise `--trace-limit` avec parcimonie. Les trace trees sont une preuve de root-cause utile, mais les gros memgraphs peuvent produire une sortie bruyante. Si un trace tree indique `Found 0 roots referencing`, traite-le comme un candidat de leak unreachable/self-retained et utilise le grouped leak tree du résumé ou `leaks --groupByType <file.memgraph>` pour identifier les champs retenus et la chaîne de payload.

## Règles de root cause

- Identifie le premier type leaké app-owned dans la sortie de leak ou la trace.
- Détermine la durée de vie voulue : process, session, account, view, request, ou task.
- Traite une allocation lazy ou différée comme une réduction de scope, pas comme un fix de leak, sauf si l'allocation eager originale violait déjà elle-même la durée de vie voulue.
- Prouve les affirmations de retain cycle avec un chemin d'ownership `traceTree` ou une reproduction isolée.
- Pour les leaks unreachable/self-cycle, `traceTree` peut n'avoir aucun chemin root ; utilise `leaks --groupByType` plus une vérification du source pour trouver l'edge auto-retenant.
- Ne prétends pas au succès juste parce que le compte total de leaks a baissé ; prouve que le type ou le chemin spécifique a disparu.
- Sépare les vraies branches de root cause des branches candidates/bruit.
- Préfère supprimer l'edge retenant plutôt qu'ajouter du code de cleanup large.

## Rapport

Un rapport de leak utile inclut :

- le flow exact et le build simulator/app
- les chemins du memgraph et du résumé
- les types leakés app-owned et leurs comptes
- au moins un chemin d'ownership, ou une preuve de grouped leak tree quand l'objet est unreachable depuis les roots
- le plus petit fix d'edge retenant proposé ou appliqué
- une preuve avant/après quand un fix a été effectué

Si le memgraph ne montre que du bruit framework/runtime, dis-le clairement et recommande la prochaine capture plus ciblée plutôt que d'inventer un leak d'app.
