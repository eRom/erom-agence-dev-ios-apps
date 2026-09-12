---
name: ios-simulator-browser
description: Affiche un Simulator iOS dans le Browser pane de Claude Code Desktop (ou dans Chrome depuis le CLI) grâce à serve-sim, et rend les previews SwiftUI d'un package Swift dans ce simulateur avec hot reload. À utiliser quand il faut voir ou piloter l'app iOS à côté du chat, voir une preview SwiftUI hors du Canvas Xcode, itérer en direct sur une preview, ou capturer une preuve visuelle de ce qu'affiche le simulateur.
---

# Simulator iOS dans le navigateur

Portée depuis `build-ios-apps` 0.1.2 (OpenAI, MIT), qui visait le navigateur
intégré de Codex. Ici la cible est le Browser pane de Claude Code Desktop, avec
Claude in Chrome en repli dans le CLI.

## Prérequis

- **Un runtime iOS installé.** `xcrun simctl runtime list` doit lister au moins
  une image disque. Sinon tous les simulateurs sont « Unavailable » et rien ne
  démarre : `xcodebuild -downloadPlatform iOS`.
- **Mac Apple Silicon.** Le helper de serve-sim n'existe qu'en arm64.
- **`bunx`, jamais `npx`.** Le garde-fou local bloque `npx`. Versions épinglées :
  `serve-sim@0.1.46`.

## 1. Choisir le simulateur

Toujours un UDID explicite, jamais « le simulateur démarré » : un autre fil peut
en utiliser un autre.

```bash
xcrun simctl list devices available
SIM="<udid>"
xcrun simctl boot "$SIM"    # « current state: Booted » veut dire qu'il tourne déjà
```

Si l'app se lance déjà via XcodeBuildMCP (`build_run_sim`), reprendre l'UDID de
ce flux : il démarre le simulateur lui-même.

Avant de lancer un miroir, tuer un éventuel helper resté d'une session
précédente, **pour ce simulateur seulement** :

```bash
bunx serve-sim@0.1.46 --kill "$SIM"
```

Jamais `--kill` sans UDID : ça coupe les miroirs des autres fils.

## 2a. Claude Code Desktop : le Browser pane

Déclarer serve-sim comme serveur de preview dans `.claude/launch.json`, **à la
racine du projet de l'app** (le dossier ouvert dans Desktop), pas dans ce plugin.
Si le fichier existe, ajouter l'entrée au tableau `configurations` sans toucher
aux autres.

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "simulator",
      "runtimeExecutable": "bunx",
      "runtimeArgs": ["serve-sim@0.1.46", "--port", "3200", "<udid>"],
      "port": 3200,
      "autoPort": false
    }
  ]
}
```

- **`autoPort: false` est obligatoire.** Sur conflit de port, Desktop passe le
  nouveau port par la variable `PORT`, que serve-sim ignore : il ne lit que
  `--port`. Le Browser pane pointerait alors dans le vide.
- **L'UDID est propre à la machine.** Si le `launch.json` est commité, l'entrée
  ne marchera que sur ce Mac.

Démarrer ensuite le serveur `simulator` depuis Desktop : la preview s'ouvre sur
`http://localhost:3200` dans le Browser pane, où Claude peut prendre des
captures, cliquer et lire la page. Pour l'arrêter, le menu des serveurs de la
barre de session.

## 2b. CLI : Claude in Chrome

Pas de Browser pane dans le terminal. Lancer serve-sim en tâche de fond (Bash
avec `run_in_background`), puis ouvrir `http://localhost:3200` avec les outils
`mcp__claude-in-chrome__*`.

```bash
bunx serve-sim@0.1.46 --port 3200 "$SIM"
```

Vérifier qu'il écoute avant d'ouvrir la page : `bunx serve-sim@0.1.46 --list "$SIM"`
doit répondre `"running":true` avec l'URL. Si rien n'écoute après une
vingtaine de secondes, `--kill "$SIM"` puis relancer. Vu le 2026-09-12 sur un
simulateur démarré quelques secondes plus tôt : processus vivant, aucune
sortie, aucun port ouvert. Le second lancement a marché, et le blocage ne s'est
pas reproduit ensuite.

## 3. Piloter le simulateur

Pour agir sur l'app, préférer les commandes de serve-sim aux clics dans la
page. Le 2026-09-12, un clic placé d'après une capture précédente est tombé à
côté : la mise en page de la preview avait bougé entre-temps. Les commandes
visent l'écran du téléphone lui-même, en coordonnées normalisées de 0 à 1.

```bash
bunx serve-sim@0.1.46 tap 0.84 0.49 -d "$SIM"      # x, y sur l'écran du téléphone
bunx serve-sim@0.1.46 button home -d "$SIM"
bunx serve-sim@0.1.46 type "texte" -d "$SIM"
bunx serve-sim@0.1.46 event-log -d "$SIM"          # confirme ce que l'appareil a reçu
```

Pour calculer x et y, diviser la position visée par la taille d'une capture
`simctl` (voir ci-dessous).

## 4. Prouver que ça tourne

Une page chargée ne prouve rien : le flux vidéo peut être mort derrière.
Prendre une capture qui montre une vraie image du simulateur avant de dire que
le miroir marche.

Si l'image est noire ou bizarre, trancher entre le miroir et l'appareil avec
une capture prise directement sur le simulateur :

```bash
xcrun simctl io "$SIM" screenshot /tmp/sim.png
```

Même image des deux côtés : le miroir est fidèle, c'est l'appareil qui est
dans cet état.

## 5. Arrêter proprement

Après l'arrêt du serveur (menu Desktop ou fin de la tâche de fond), vérifier que
le helper du simulateur est bien parti, et le tuer sinon :

```bash
bunx serve-sim@0.1.46 --list "$SIM"
bunx serve-sim@0.1.46 --kill "$SIM"
```

## Previews SwiftUI avec hot reload

Pour des previews qui vivent dans un package Swift importable. Le lanceur
fourni génère un projet hôte jetable hors du code de l'utilisateur, l'installe
et le lance dans le simulateur, puis surveille le package.

`<skill-root>` est le « Base directory for this skill » injecté au chargement
de cette skill. Le script est écrit pour Node, il se lance avec `node`.

```bash
node <skill-root>/scripts/swiftui-preview-browser.mjs \
  /chemin/absolu/vers/Package.swift \
  --package-target "<target>" \
  --device "<udid>"
```

- Le mode watch est actif par défaut. À chaque modification d'un source du
  package, le lanceur recompile une dylib et la remplace à chaud dans l'hôte,
  sans relancer l'app.
- L'hôte affiche toutes les variantes de preview trouvées dans la cible, avec
  une pagination dans le simulateur. Pour n'en garder que certaines :
  `--preview-filter <regex[, ...]>`, qui filtre sur les noms affichés et les
  identifiants comme `StatusRowView_Previews`.
- Dès que le lanceur affiche l'UDID choisi, lancer serve-sim sur ce même UDID
  (étape 2a ou 2b).

## Limites

- Couvre les `PreviewProvider` et `#Preview` d'un package Swift, via l'hôte généré.
- Ne jamais modifier le `.xcodeproj`, le `.xcworkspace`, le `Package.swift`, les
  schemes ou les réglages de build de l'utilisateur pour forcer une preview.

## Preuve à rendre

- Miroir ou preview : une capture du Browser pane (ou de Chrome) montrant
  l'image du simulateur.
- Hot reload : la ligne `hot reloaded package preview ... in pid ...` du
  lanceur, plus une capture de l'image modifiée après l'édition.
