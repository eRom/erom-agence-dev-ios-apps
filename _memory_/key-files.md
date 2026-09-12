# Fichiers clés

_Mis à jour : 2026-09-12_

## Contrat et état

- `CLAUDE.md` : invariants du dépôt (français, crédit de la source, rien de Codex, MCP seulement s'il tourne) et tableau « État actuel », la seule vérité sur ce qui est testé.
- `README.md` : vitrine GitHub, carte `assets/erom-dev-ios-apps.png`, tableau des skills, section « Origine ».
- `plugin/README.md` : même contenu sans image, distribué avec le plugin.

## Plugin distribué

- `plugin/.claude-plugin/plugin.json` : nom `erom-dev-ios-apps`, version 0.1.0, keywords, `skills: ./skills/`.
- `plugin/.mcp.json` : `bunx xcodebuildmcp@2.7.0 mcp`, workflows `simulator,ui-automation,debugging`.
- `plugin/skills/ios-simulator-browser/SKILL.md` : seule skill réécrite (pas seulement traduite) pour Claude Code : `launch.json` Desktop, `bunx serve-sim`, commandes `tap`/`button`, preuve par `simctl screenshot`.
- `plugin/skills/ios-simulator-browser/scripts/swiftui-preview-browser.mjs` : hôte de previews SwiftUI avec hot reload, lancé par `node`.
- `plugin/skills/ios-ettrace-performance/SKILL.md` : passage tmux (`new-session`, `send-keys`, `capture-pane`) à la place du `write_stdin` de Codex.
- `plugin/skills/swiftui-performance-audit/references/report-template.md` : modèle de rapport, traduit en entier (intitulés compris).

## Hors dépôt

- `.claude/settings.local.json` : profil de session `plugin` (skill `session-profile`), gitignoré globalement.
- `~/dev/erom-marketplace/.claude-plugin/marketplace.json` : entrée du plugin, metadata 0.27.0.
- `~/dev/erom-agence-dev-plugin/plugin/skills/illustrate/references/GABARIT.md` : table des cartes livrées (ligne `erom-dev-ios-apps` ajoutée, registre mur d'outils).
