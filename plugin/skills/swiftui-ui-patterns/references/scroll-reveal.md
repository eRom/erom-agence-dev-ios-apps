# Scroll-reveal detail surfaces

## Intention

Utilise ce pattern quand un écran de detail a une surface principale devant et du contenu secondaire derrière, et que tu veux que l'utilisateur révèle cette couche secondaire en scrollant ou en swipant plutôt qu'en tapant un bouton séparé.

Cas typiques :

- écrans de detail media qui révèlent des actions ou des métadonnées
- maps, cards ou canvases qui transitionnent vers du detail structuré
- viewers full-screen avec une seconde page "actions" ou "insights"

## Pattern essentiel

Construis l'interaction comme une `ScrollView` verticale paginée avec deux sections :

1. une section principale dimensionnée au viewport
2. une section secondaire en dessous

Dérive une valeur `progress` normalisée à partir du content offset vertical, et pilote tous les changements visuels depuis cette seule valeur.

Évite de traiter le reveal comme un système de gesture séparé, sauf si le scroll seul ne peut pas l'exprimer.

## Structure minimale

```swift
private enum DetailSection: Hashable {
  case primary
  case secondary
}

struct DetailSurface: View {
  @State private var revealProgress: CGFloat = 0
  @State private var secondaryHeight: CGFloat = 1

  var body: some View {
    GeometryReader { geometry in
      ScrollViewReader { proxy in
        ScrollView(.vertical, showsIndicators: false) {
          VStack(spacing: 0) {
            PrimaryContent(progress: revealProgress)
              .frame(height: geometry.size.height)
              .id(DetailSection.primary)

            SecondaryContent(progress: revealProgress)
              .id(DetailSection.secondary)
              .onGeometryChange(for: CGFloat.self) { geo in
                geo.size.height
              } action: { newHeight in
                secondaryHeight = max(newHeight, 1)
              }
          }
          .scrollTargetLayout()
        }
        .scrollTargetBehavior(.paging)
        .onScrollGeometryChange(for: CGFloat.self, of: { scroll in
          scroll.contentOffset.y + scroll.contentInsets.top
        }) { _, offset in
          revealProgress = (offset / secondaryHeight).clamped(to: 0...1)
        }
        .safeAreaInset(edge: .bottom) {
          ChevronAffordance(progress: revealProgress) {
            withAnimation(.smooth) {
              let target: DetailSection = revealProgress < 0.5 ? .secondary : .primary
              proxy.scrollTo(target, anchor: .top)
            }
          }
        }
      }
    }
  }
}
```

## Choix de design à garder

- Fais en sorte que la section principale soit exactement de la taille du viewport quand l'interaction doit ressembler à une pagination entre états.
- Calcule `progress` à partir du vrai scroll offset, pas de booléens dupliqués comme `isExpanded`, `isShowingSecondary` et `isSnapped`.
- Utilise `progress` pour piloter `offset`, `opacity`, `blur`, `scaleEffect` et le state de toolbar afin que toute la surface reste synchronisée.
- Utilise `ScrollViewReader` pour le snapping programmatique depuis des taps sur le contenu principal ou les chevron affordances.
- Utilise `onScrollTargetVisibilityChange` quand tu as besoin d'un state de section stabilisé pour les haptics, la fermeture d'un tooltip, l'analytics ou les annonces d'accessibilité.

## Morphing d'un control partagé

Si un control semble se déplacer de la surface principale vers le contenu secondaire, ne rends pas deux copies pleinement visibles.

À la place :

- expose une ancre source dans la zone principale
- expose une ancre destination dans le contenu secondaire
- rends un seul overlay qui interpole position et taille avec `progress`

```swift
Color.clear
  .anchorPreference(key: ControlAnchorKey.self, value: .bounds) { anchor in
    ["source": anchor]
  }

Color.clear
  .anchorPreference(key: ControlAnchorKey.self, value: .bounds) { anchor in
    ["destination": anchor]
  }

.overlayPreferenceValue(ControlAnchorKey.self) { anchors in
  MorphingControlOverlay(anchors: anchors, progress: revealProgress)
}
```

Ça garde le mouvement cohérent et évite les bugs de hit-target dupliqués.

## Haptics et affordances

- Utilise des haptics de seuil légers quand le reveal commence, et des haptics plus forts proche de l'état engagé.
- Garde une affordance visible comme un chevron ou une pill tant que `progress` est proche de zéro.
- Flip, fade ou blur l'affordance à mesure que la section secondaire devient active.

## Garde-fous d'interaction

- Désactive le scroll vertical quand un mode conflictuel est actif, comme le pinch-to-zoom, le crop ou la manipulation media full-screen.
- Désactive le hit testing sur les overlays qui doivent disparaître une fois le contenu secondaire révélé.
- Évite les scroll views imbriquées sur le même axe, sauf si la view interne est effectivement statique ou désactivée pendant le reveal.

## Pièges

- Ne code pas en dur le diviseur de progress. Mesure la hauteur de la section secondaire ou une autre vraie distance de reveal.
- Ne mélange pas plusieurs sources d'animation pour la même propriété. Si `progress` la pilote, coupe les autres animations sur cette propriété.
- Ne stocke pas de state dérivé comme `isSecondaryVisible` sauf si une autre API l'exige. Préfère le dériver de `progress` ou des scroll targets visibles.
- Attention aux boucles de feedback de layout lors de la mesure des hauteurs. Clamp les valeurs à zéro et ne mets à jour que quand la hauteur mesurée change réellement.

## Exemple concret

- Pool iOS tile detail reveal: `/Users/dimillian/Documents/Dev/Pool/pool-ios/Pool/Sources/Features/Tile/Detail/TileDetailView.swift`
- Secondary content anchor example: `/Users/dimillian/Documents/Dev/Pool/pool-ios/Pool/Sources/Features/Tile/Detail/TileDetailIntentListView.swift`
