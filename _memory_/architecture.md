# Architecture

_Mis à jour : 2026-09-12_

## Type et objectif

Dépôt de développement du plugin Claude Code `erom-dev-ios-apps` : construire, déboguer et mesurer une app iOS native en SwiftUI avec Claude, le simulateur sous les yeux.
Œuvre servie (réponse de Romain au chantier gate) : passer la PWA `erom-agence-control-plane` en app iOS native.

Publié dans `erom-marketplace` en 0.1.0 le 2026-09-12. Dépôt GitHub `eRom/erom-agence-dev-ios-apps`, public depuis le même jour.

## Stack

- Skills en Markdown pur, pas d'étape de build.
- Scripts hérités de la source, inchangés : `.py`, `.sh`, `.mjs` (Node pour `swiftui-preview-browser.mjs`).
- Serveur MCP embarqué : XcodeBuildMCP 2.7.0 lancé par `bunx`.
- Outils externes appelés par les skills : `serve-sim@0.1.46` (bunx), `ettrace` 1.1.0 (Homebrew), Xcode 26.6, runtime iOS 26.5.

## Arborescence

```
plugin/                      seul dossier distribué (git-subdir de la marketplace)
  .claude-plugin/plugin.json manifeste
  .mcp.json                  XcodeBuildMCP
  skills/<9 skills>/         SKILL.md + references/ + scripts/
  README.md, LICENSE
assets/erom-dev-ios-apps.png carte au fusain du README racine
CLAUDE.md                    contrat du dépôt + « État actuel » (source de vérité)
_memory_/                    cartographie de session (ONBOARD.md gitignoré)
```

## Composants et flux

- 9 skills : `ios-app-intents`, `ios-debugger-agent`, `ios-simulator-browser`, `ios-ettrace-performance`, `ios-memgraph-leaks`, `swiftui-liquid-glass`, `swiftui-performance-audit`, `swiftui-ui-patterns`, `swiftui-view-refactor`.
- Build / run / debug : skill `ios-debugger-agent` -> tools MCP XcodeBuildMCP -> Simulator.
- Miroir : `ios-simulator-browser` -> `serve-sim` (preview sur `localhost:3200`) -> Browser pane de Desktop (déclaré dans le `.claude/launch.json` du projet de l'app) ou Chrome via Claude in Chrome (CLI). Les deux voies validées le 2026-09-12.
- Profiling : `ios-ettrace-performance` lance `ettrace` dans une session tmux (TTY requis).

## Dépendances externes critiques

- Source des skills : `build-ios-apps` 0.1.2 d'OpenAI (MIT), copie locale `~/dev/openai-plugins/plugins/build-ios-apps`.
- Cycle de vie du plugin : plugin `erom-dev-plugin` (`scaffold`, `illustrate`, `release`), dépôt source `~/dev/erom-agence-dev-plugin`.
- Marketplace : `~/dev/erom-marketplace` (entrée `git-subdir`, `strict: true`, CI `validate.yml`).
