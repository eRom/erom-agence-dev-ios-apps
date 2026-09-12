---
name: ios-ettrace-performance
description: "Capturer et interpréter des profils ETTrace sur iOS Simulator. À utiliser pour profiler la latence de launch ou de runtime, comparer des traces, ou trouver des stacks CPU-heavy."
---

# iOS ETTrace Performance

Utilise cette skill pour capturer un profil ETTrace ciblé et symbolicated depuis une app sur simulator iOS. Combine-la avec `../ios-debugger-agent/SKILL.md` quand la tâche a aussi besoin de build, install, launch, UI driving, logs ou screenshots sur simulator.

## Workflow principal

1. Choisis un seul flow ciblé et note les points de départ et d'arrêt attendus.
2. Build l'app simulator exacte qui sera installée et profilée.
3. Link temporairement ETTrace dans ce target d'app pour le profiling simulator/debug.
4. Collecte les dSYMs UUID-matched pour l'executable de l'app et les frameworks dynamiques embarqués.
5. Capture une trace de launch ou de runtime.
6. Préserve le JSON de flamegraph processed immédiatement après le run.
7. Analyse uniquement le JSON processed et rapporte le flow, les artifacts, les hotspots et les caveats.

Évite les captures larges du type « utilise l'app un moment ». Une trace doit correspondre à un seul flow visible par l'utilisateur.

## Setup

Utilise un dossier de run writable pour chaque session de profiling :

```bash
if [ -z "${RUN_DIR:-}" ]; then
  RUN_DIR="$(mktemp -d "${TMPDIR:-/tmp}/erom-ios-ettrace.XXXXXX")"
fi
mkdir -p "$RUN_DIR"
```

Installe le runner CLI ETTrace s'il n'est pas déjà disponible :

```bash
brew install emergetools/homebrew-tap/ettrace
```

`ettrace` est le runner macOS côté host. L'app doit aussi linker un `ETTrace.xcframework` pour l'architecture iOS Simulator.
Ce workflow est validé pour les fichiers `output_<thread>.json` processed par ETTrace v1.1.0, avec `nodes` au niveau top-level.

## Linker ETTrace dans l'app

Câble ETTrace dans le target d'app exact qui est profilé. Garde l'intégration dans un patch clairement temporaire et retire-le une fois la tâche de profiling terminée, sauf demande explicite de l'utilisateur de le conserver.

Options préférées :

- Réutilise un `ETTrace.xcframework` compatible simulator existant si le repo en vendor déjà un.
- S'il n'en existe pas, build une copie simulator-only dans `RUN_DIR` à partir du package ETTrace upstream.
- Link le framework directement dans le target de l'app, pas seulement dans les tests, les resources, les data files, ou un target launcher imbriqué.
- Confirme que les logs de launch affichent `Starting ETTrace`.
- Profile une seule app simulator instrumentée par ETTrace à la fois, car le mode simulator écoute sur un port localhost fixe.

Build un framework simulator si nécessaire :

```bash
ETTRACE_TAG="${ETTRACE_TAG:-v1.1.0}" # À surcharger pour matcher le runner installé quand Homebrew fait une mise à jour.
ETTRACE_SRC="$RUN_DIR/ETTrace-src"
if [ ! -d "$ETTRACE_SRC" ]; then
  git clone --depth 1 --branch "$ETTRACE_TAG" https://github.com/EmergeTools/ETTrace "$ETTRACE_SRC"
fi

rm -rf "$RUN_DIR/ETTrace-iphonesimulator.xcarchive" "$RUN_DIR/ETTrace.xcframework"
pushd "$ETTRACE_SRC" >/dev/null
xcodebuild archive \
  -scheme ETTrace \
  -archivePath "$RUN_DIR/ETTrace-iphonesimulator.xcarchive" \
  -sdk iphonesimulator \
  -destination 'generic/platform=iOS Simulator' \
  BUILD_LIBRARY_FOR_DISTRIBUTION=YES \
  INSTALL_PATH='Library/Frameworks' \
  SKIP_INSTALL=NO \
  CLANG_CXX_LANGUAGE_STANDARD=c++17

xcodebuild -create-xcframework \
  -framework "$RUN_DIR/ETTrace-iphonesimulator.xcarchive/Products/Library/Frameworks/ETTrace.framework" \
  -output "$RUN_DIR/ETTrace.xcframework"
popd >/dev/null
```

Pour les apps Bazel, un import temporaire ressemble en général à ceci :

```python
load("@rules_apple//apple:apple.bzl", "apple_dynamic_xcframework_import")

package(default_visibility = ["//visibility:public"])

apple_dynamic_xcframework_import(
    name = "ETTrace",
    xcframework_imports = glob(["ETTrace.xcframework/**"]),
)
```

Pour les projets Xcode, ajoute temporairement le `ETTrace.xcframework` simulator aux phases Link Binary With Libraries / Embed Frameworks du target de l'app, pour le build debug simulator que tu profiles, puis retire ce câblage après le profiling.

## Verrou de symbolication

Ne tire aucune conclusion d'un flamegraph unsymbolicated. Avant chaque capture, prépare un dossier de dSYM qui inclut le dSYM de l'app et les dSYMs des frameworks dynamiques first-party embarqués.

Collecte les dSYMs après le build final qui a produit l'app installée :

```bash
SKILL_DIR="<absolute path to this loaded skill folder>"
APP="<path-to-built-simulator-App.app>"
DSYMS="$RUN_DIR/dsyms"

"$SKILL_DIR/scripts/collect_ios_dsyms.sh" \
  --app "$APP" \
  --out-dir "$DSYMS" \
  --search-root "$(dirname "$APP")" \
  --search-root "$PWD" \
  --extra-dsym "$RUN_DIR/ETTrace-iphonesimulator.xcarchive/dSYMs/ETTrace.framework.dSYM"
```

Ajoute `--require-framework <FrameworkName>` pour les frameworks dynamiques app-owned qui doivent symboliser ; utilise `--require-all-frameworks` uniquement quand tous les frameworks embarqués sont app-owned ou censés avoir des symboles. Si le helper signale un dSYM d'app ou de framework requis manquant, rebuild l'app simulator exacte avec génération de dSYM avant de tracer, ou ajoute le dossier de sortie de build qui contient ces dSYMs comme `--search-root` supplémentaire.

Vérifie les UUIDs importants avant de tracer quand le rapport semble suspect :

```bash
dwarfdump --uuid "$APP/$(/usr/libexec/PlistBuddy -c 'Print :CFBundleExecutable' "$APP/Info.plist")"
find "$DSYMS" -maxdepth 1 -type d -name '*.dSYM' -print -exec dwarfdump --uuid {} \;
```

Une fois ETTrace terminé, lis son résumé de symbolication. Traite toute ligne first-party significative « have library but no symbol » comme une trace échouée, sauf s'il s'agit de bruit négligeable. Les buckets unsymbolicated de system-framework ou internes à ETTrace sont généralement acceptables.

## Capture

Pour les traces de launch :

```bash
cd "$RUN_DIR"
CAPTURE_MARKER="$RUN_DIR/.ettrace-capture-start"
: > "$CAPTURE_MARKER"
find "$RUN_DIR" -maxdepth 1 \( -name 'output.json' -o -name 'output_*.json' \) -delete
ettrace --simulator --launch --verbose --dsyms "$DSYMS"
```

Utilise `--launch` uniquement pour mesurer le startup ou le premier render. La première connexion de launch peut force quit l'app ; relance depuis l'écran d'accueil du simulator plutôt que depuis Xcode si demandé. Pour les traces first-launch-after-install, mets temporairement `ETTraceRunAtStartup=YES` dans l'Info.plist de l'app, puis lance `ettrace --simulator` et démarre depuis l'écran d'accueil.

Pour les traces de flow runtime :

```bash
cd "$RUN_DIR"
CAPTURE_MARKER="$RUN_DIR/.ettrace-capture-start"
: > "$CAPTURE_MARKER"
find "$RUN_DIR" -maxdepth 1 \( -name 'output.json' -o -name 'output_*.json' \) -delete
ettrace --simulator --verbose --dsyms "$DSYMS"
```

Démarre depuis un écran stable, lance ETTrace, effectue exactement un flow ciblé, attends que le travail visible soit terminé, puis arrête le runner. Pour une attribution plus large, ajoute `--multi-thread` ; sinon commence par le main thread.

`ettrace` a besoin d'un TTY : sans TTY, le runner peut se terminer sans trace utile. L'outil Bash de Claude Code n'en fournit pas, donc lance-le dans une session tmux, réponds aux prompts avec `send-keys` et lis la sortie avec `capture-pane` :

```bash
tmux new-session -d -s ettrace -c "$RUN_DIR" "ettrace --simulator --verbose --dsyms '$DSYMS'"
tmux capture-pane -p -t ettrace          # lire le prompt en cours
tmux send-keys -t ettrace Enter          # y répondre
tmux kill-session -t ettrace             # une fois la trace écrite
```

## Préserver les outputs

Le prochain run ETTrace peut overwrite les fichiers de flamegraph processed, donc préserve immédiatement les `output_<thread-id>.json` frais. N'analyse pas un `output.json` sauvegardé ; ETTrace sert aussi une route viewer sous ce nom, et les fichiers raw `emerge-output/output.json` ne sont pas les artifacts de flamegraph processed attendus par ce workflow.

```bash
PRESERVED_DIR="$(mktemp -d "$RUN_DIR/run-$(date +%Y%m%d-%H%M%S).XXXXXX")"
: > "$PRESERVED_DIR/summary.txt"
if [ ! -e "$CAPTURE_MARKER" ]; then
  echo "error: capture marker missing; start a fresh ETTrace capture before preserving outputs" >&2
  exit 1
fi
find "$RUN_DIR" -maxdepth 1 -name 'output_*.json' -newer "$CAPTURE_MARKER" -print | while IFS= read -r json; do
  preserved="$PRESERVED_DIR/${json##*/}"
  cp "$json" "$preserved"
  {
    echo "## ${preserved##*/}"
    python3 "$SKILL_DIR/scripts/analyze_flamegraph_json.py" "$preserved"
  } >> "$PRESERVED_DIR/summary.txt"
done
if [ ! -s "$PRESERVED_DIR/summary.txt" ]; then
  echo "error: no fresh processed ETTrace output JSON found in $RUN_DIR" >&2
  exit 1
fi
```

Analyse uniquement les fichiers `output_*.json` processed dans `RUN_DIR`. Ignore `output.json` et les fichiers raw `emerge-output/output.json`, sauf si tu debug ETTrace lui-même. Si l'analyzer rejette la forme du JSON, capture à nouveau avec le runner ETTrace Homebrew et le tag `ETTrace.xcframework` correspondant côté app, plutôt que d'essayer d'interpréter le fichier rejeté.

## Lire le profil

Pars de `run-*/summary.txt`, puis inspecte le JSON processed directement si nécessaire.

Rapporte :

- le flow exact, le build de l'app, le modèle/runtime du simulator, et le nombre de runs
- les chemins du JSON de flamegraph processed
- les leaves actives principales et les stacks first-party inclusifs avec leurs poids d'échantillons ou pourcentages
- si les symboles étaient complets pour les binaires app-owned
- les caveats tels que le setup de premier run, le coût simulator-only, la variance réseau, ou un faible nombre d'échantillons
- les deltas avant/après uniquement quand le même flow a été capturé avec un setup comparable


## Nettoyage

Retire le câblage temporaire d'ETTrace dans l'app une fois le profiling terminé, sauf demande explicite de l'utilisateur de le conserver. Garde ou jette les artifacts de run selon la tâche en cours.
