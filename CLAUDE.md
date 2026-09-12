# Contrat du dépôt

Dépôt de développement du plugin Claude Code `erom-dev-ios-apps` : Concevoir, déboguer et profiler des apps iOS natives en SwiftUI : App Intents et App Shortcuts, Liquid Glass d'iOS 26, patterns et refactor de vues, audit de performance, traces ETTrace, memgraphs de fuites mémoire, débogage sur simulateur via XcodeBuildMCP et miroir du Simulator dans le navigateur.
Ce fichier dit comment travailler ici. Il décrit un contrat, pas un état atteint :
la section « État actuel » dit ce qui n'y est pas encore conforme.

## Structure

```
plugin/                          seul dossier distribué (source marketplace git-subdir)
  .claude-plugin/plugin.json     manifeste
  skills/<nom>/SKILL.md          une skill par dossier
  skills/<nom>/references/       matière longue, lue seulement quand la skill tourne
  agents/<nom>.md                subagents, découverts automatiquement
docs/                            specs, plans, recherches, revues
_memory_/                        connaissance de session persistée, indexée par le vault RAG
.claude/                         notes et settings locaux, jamais distribués
```

Pas d'étape de build : les skills sont du Markdown pur, écrit à la main
directement dans `plugin/`.

## Invariants

1. **Seul `plugin/` est distribué.** Tout ce qui est hors de ce dossier reste
   local ou sert le développement : notes, matière de travail, mémoire.
2. **Français.** Le plugin sert des projets français. Skills, exemples et
   sorties en français, y compris les descriptions de `SKILL.md`.
3. **Aucune capacité dupliquée.** Avant d'ajouter une skill, vérifier qu'aucun
   autre plugin eRom ne la porte déjà (`~/dev/erom-marketplace/.claude-plugin/marketplace.json`
   liste les plugins publiés et ce qu'ils couvrent). Deux plugins qui font la
   même chose, c'est un plugin de trop.
4. **La règle vient de ce qui a déjà tourné.** Chaque consigne d'une skill doit
   pouvoir se rattacher à un artefact réel : une sortie observée, un test qui
   passe, un incident daté. Le générique ne décrit aucun usage et ne sert de
   source à rien.
5. **Le manifeste ne déclare pas ses agents.** La clé `agents` absente vaut
   découverte automatique de `plugin/agents/`. Une liste explicite fige les
   chemins et fait mentir `claude plugin details`, qui affiche alors « Agents (0) ».

6. **Crédit de la source.** Les skills sont portées depuis `build-ios-apps` 0.1.2
   d'OpenAI (`github.com/openai/plugins`, `plugins/build-ios-apps`), licence MIT
   déclarée dans son `.codex-plugin/plugin.json`. Toute skill reprise garde la
   mention de son origine dans les README.
7. **Rien de Codex ne passe tel quel.** La source vise Codex : `agents/openai.yaml`,
   le bloc `interface` du manifeste et le « Codex in-app browser » de
   `ios-simulator-browser` n'existent pas dans Claude Code. Chaque skill portée
   est relue pour ça avant d'entrer dans `plugin/`.
8. **Pas de `plugin/.mcp.json` avant un XcodeBuildMCP qui tourne.** La source
   lance `npx -y xcodebuildmcp@latest mcp`. Le serveur n'entre dans le manifeste
   qu'une fois lancé et vu répondre ici, version épinglée.

## Œuvre servie

Faire passer la PWA `erom-agence-control-plane` en app iOS native. Chaque skill
se justifie par ce chantier : une skill qui ne sert pas cette app n'entre pas.

## Vérifier

```bash
claude --plugin-dir plugin plugin details erom-dev-ios-apps
```

Affiche l'inventaire réel des composants chargés et le coût token projeté.
C'est la seule preuve que le manifeste charge ce qu'on croit : un dossier
`skills/` ou `agents/` peuplé mais invisible ici n'est pas chargé.

## Publication

Publié dans `erom-marketplace` depuis la 0.1.0 (2026-09-12), dépôt GitHub public. La publication ne se fait pas à la main : la skill `release`
du plugin `erom-dev-plugin` la porte de bout en bout, depuis ce dépôt.

```
/erom-dev-plugin:release
```

Elle lit le nom et la version dans le manifeste, choisit le bump SemVer, commite
et pousse ce dépôt, puis met à jour `~/dev/erom-marketplace` (entrée du plugin,
metadata, README), et vérifie la CI. Toujours dans cet ordre : le plugin d'abord, 
la marketplace ensuite, parce que l'entrée pointe `ref: main` sur ce dépôt.

Sur une **première** publication, elle s'arrête et demande : l'entrée à créer
réclame une description, une source `git-subdir` et un choix de `strict`. C'est
le moment de les préparer, pas avant.

## État actuel - 2026-09-12

Les 9 skills de la source sont copiées telles quelles. `claude plugin details`
voit 9 skills et 1 serveur MCP, ~778 tok always-on.

| Élément | État |
|---|---|
| `plugin/skills/` | 9 skills, toutes en français le 2026-09-12 (`SKILL.md` et `references/`, jargon et API gardés en anglais). Les scripts `.py`, `.sh`, `.mjs` sont restés tels quels |
| `ios-ettrace-performance` | le passage Codex (`write_stdin`) remplacé par une session tmux (`new-session`, `send-keys`, `capture-pane`). Testé en partie le 2026-09-12 avec `ettrace` 1.1.0 (Homebrew) : dans tmux, `capture-pane` lit bien le prompt « Press return when ready... ». Pas encore de capture complète, faute d'app liée à `ETTrace.xcframework` |
| `ios-simulator-browser` | portée vers le Browser pane de Desktop (`.claude/launch.json`, `bunx serve-sim@0.1.46`, `autoPort: false`). Sources : doc Desktop, README serve-sim (section « Claude Code Desktop »), `dist/serve-sim.js` 0.1.46 qui ne lit que `--port`. Voie CLI testée le 2026-09-12 sur iPhone 17 Pro / iOS 26.5 : preview sur 3200, image en direct dans Chrome, `tap` et `button home` reçus (`event-log`). Voie Desktop (Browser pane) pas encore testée |
| Traces Codex | aucune dans les skills, hors mention de la source. Préfixes `mktemp` passés de `codex-` à `erom-` |
| `plugin/.mcp.json` | `bunx xcodebuildmcp@2.7.0 mcp`, vu « Connected » par `claude --plugin-dir plugin mcp list` le 2026-09-12. `npx -y xcodebuildmcp@latest` échouait (CONNECTION_CLOSED), le workflow `logging` n'existe pas en 2.7.0 |
| Runtimes du Simulator | aucun installé sur la machine (`xcrun simctl runtime list` : 0 image disque), rien de ce qui touche au simulateur ne peut tourner |
| Keywords du manifeste | remplis |
| Image de tête | `assets/erom-dev-ios-apps.png`, 1536x1024, tirée le 2026-09-12 par `/erom-dev-plugin:illustrate` (mur d'outils d'horloger), puis un iPhone ajouté sur l'établi par `gpt_image_edit` à la demande de Romain (écran noir, sans logo). Textes vérifiés au zoom après l'édition |
| Publication marketplace | 0.1.0 publiée le 2026-09-12 (`git-subdir`, `strict: true`), dépôt passé en public le même jour |
