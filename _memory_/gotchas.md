# Gotchas

_Mis à jour : 2026-09-12_

## Outillage iOS

- **XcodeBuildMCP via `npx` ne se connecte pas.** `npx -y xcodebuildmcp@latest mcp` sort `✘ Failed to connect - CONNECTION_CLOSED` deux fois de suite ; `bunx xcodebuildmcp@2.7.0 mcp` sort `✔ Connected`. Le workflow `logging` n'existe pas en 2.7.0 (warning au démarrage). Recheck : `claude --plugin-dir plugin mcp list | grep xcodebuild`.
- **Pas de runtime iOS = tous les simulateurs « Unavailable ».** `xcrun simctl runtime list` montrait 0 image disque ; réglé par `xcodebuild -downloadPlatform iOS` (iOS 26.5, 7,9 Go).
- **serve-sim ignore la variable `PORT`.** Il ne lit que `--port` (`dist/serve-sim.js` 0.1.46 : `Z??3200`, aucune occurrence de `env.PORT`). Dans `launch.json` : `--port 3200` explicite et `"autoPort": false`, sinon le Browser pane pointe dans le vide.
- **serve-sim bloqué au premier lancement.** Processus vivant, aucune sortie, aucun port, sur un simulateur démarré quelques secondes plus tôt ; relancé, il marche ; non reproduit. Remède : `--kill "$SIM"` puis relance. [candidat 1x - session 2026-09-12]
- **Clic dans la page serve-sim tombé à côté.** La mise en page de la preview a bougé entre la capture et le clic. Utiliser `serve-sim tap x y -d "$SIM"` (coordonnées normalisées) et vérifier par `event-log`. [candidat 1x - session 2026-09-12]
- **Simulateur bloqué sur écran noir** après ouverture de Réglages sur un runtime neuf, Home sans effet ; `simctl io screenshot` identique au miroir, donc l'appareil et non le stream. Remède proposé, non testé : `xcrun simctl erase <UDID>`. [candidat 1x - session 2026-09-12]
- **`ettrace` exige un TTY.** L'outil Bash n'en a pas : tmux fonctionne, `capture-pane` lit « Press return when ready... ». Capture complète pas encore faite (il faut une app liée à `ETTrace.xcframework`).

## Harnais et outils de session

- **Le hook `guard-tools` bloque toute commande contenant le mot `npx`**, même comme motif de `grep` (refus `BLOCKED BY POLICY` le 2026-09-12). Ne pas écrire ce mot dans une commande Bash.
- **`sips --cropOffset` a rendu de mauvaises zones** au lieu des bandes demandées ; recadrer avec Pillow : `uv run --with pillow python -c "..."`. [candidat 1x - session 2026-09-12]
- **Brief de traduction ambigu.** « Mêmes titres » a été lu comme « titres non traduits » par un agent : écrire « même nombre et même niveau, texte traduit ». [candidat 1x - session 2026-09-12]
- **Le GABARIT d'`illustrate` s'édite dans `~/dev/erom-agence-dev-plugin`**, jamais dans le cache `~/.claude/plugins/cache/...` qu'une mise à jour écrase. La version installée (0.1.4) ne voit la nouvelle ligne qu'après la prochaine release d'`erom-dev-plugin`.

## Publication

- **Dépôt privé = plugin installable par Romain seul.** Passé en public le 2026-09-12 après scan de l'historique (aucun `_memory_`, `settings.local`, `.env`, clé ni ID Linear/Slack). `_memory_/ONBOARD.md` reste gitignoré parce qu'il porte ces IDs.
