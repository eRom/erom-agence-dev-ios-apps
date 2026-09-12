---
name: ios-debugger-agent
description: "Build, run et debug d'apps iOS sur Simulator avec XcodeBuildMCP. À utiliser pour lancer une app, inspecter l'UI ou les logs du simulator, ou diagnostiquer un comportement runtime."
---

# iOS Debugger Agent

## Vue d'ensemble
Utilise XcodeBuildMCP pour build et run le scheme du projet courant sur un simulator iOS booté, interagir avec l'UI, et capturer les logs. Préfère les tools MCP pour le contrôle du simulator, les logs et l'inspection des vues.

## Workflow principal
Suis cette séquence sauf si l'utilisateur demande une action plus ciblée.

### 1) Repérer le simulator booté
- Appelle `mcp__XcodeBuildMCP__list_sims` et sélectionne le simulator à l'état `Booted`.
- Si aucun n'est booté, demande à l'utilisateur d'en booter un (ne boote pas automatiquement sans demande).

### 2) Définir les defaults de session
- Appelle `mcp__XcodeBuildMCP__session-set-defaults` avec :
  - `projectPath` ou `workspacePath` (selon ce qu'utilise le repo)
  - `scheme` pour l'app courante
  - `simulatorId` du device booté
  - Optionnel : `configuration: "Debug"`, `useLatestOS: true`

### 3) Build + run (si demandé)
- Appelle `mcp__XcodeBuildMCP__build_run_sim`.
- **Si le build échoue**, vérifie la sortie d'erreur et retente (éventuellement avec `preferXcodebuild: true`) ou escalade vers l'utilisateur avant toute interaction UI.
- **Après un build réussi**, vérifie que l'app a bien lancé en appelant `mcp__XcodeBuildMCP__describe_ui` ou `mcp__XcodeBuildMCP__screenshot` avant de passer à l'interaction UI.
- Si l'app est déjà buildée et que seul le launch est demandé, utilise `mcp__XcodeBuildMCP__launch_app_sim`.
- Si le bundle id est inconnu :
  1) `mcp__XcodeBuildMCP__get_sim_app_path`
  2) `mcp__XcodeBuildMCP__get_app_bundle_id`

## Interaction UI & Debug
Utilise ces tools quand on te demande d'inspecter ou d'interagir avec l'app en cours d'exécution.

- **Describe UI** : `mcp__XcodeBuildMCP__describe_ui` avant de tap ou de swipe.
- **Tap** : `mcp__XcodeBuildMCP__tap` (préfère `id` ou `label` ; n'utilise les coordonnées qu'en dernier recours).
- **Type** : `mcp__XcodeBuildMCP__type_text` après avoir focus un champ.
- **Gestures** : `mcp__XcodeBuildMCP__gesture` pour les scrolls courants et les edge swipes.
- **Screenshot** : `mcp__XcodeBuildMCP__screenshot` pour confirmation visuelle.

## Logs & sortie console
- Démarrer les logs : `mcp__XcodeBuildMCP__start_sim_log_cap` avec le bundle id de l'app.
- Arrêter les logs : `mcp__XcodeBuildMCP__stop_sim_log_cap` et résume les lignes importantes.
- Pour la sortie console, passe `captureConsole: true` et relance si nécessaire.

## Dépannage
- Si le build échoue, demande s'il faut retenter avec `preferXcodebuild: true`.
- Si la mauvaise app se lance, vérifie le scheme et le bundle id.
- Si des éléments UI ne sont pas hittable, relance `describe_ui` après un changement de layout.
