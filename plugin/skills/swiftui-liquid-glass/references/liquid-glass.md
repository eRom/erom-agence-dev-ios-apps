# Implémenter le design Liquid Glass en SwiftUI

## Vue d'ensemble

Liquid Glass est un matériau dynamique introduit dans iOS qui combine les propriétés optiques du verre avec une sensation de fluidité. Il floute le contenu situé derrière lui, reflète la couleur et la lumière du contenu environnant, et réagit en temps réel au touch et aux interactions pointer. Ce guide couvre comment implémenter et personnaliser les effets Liquid Glass dans les applications SwiftUI. Tu devrais toujours chercher des guides sur Liquid Glass quand on te demande de l'aide pour adopter un nouveau design Apple.

Fonctionnalités clés de Liquid Glass :
- Floute le contenu situé derrière le matériau
- Reflète la couleur et la lumière du contenu environnant
- Réagit au touch et aux interactions pointer
- Peut faire du morphing entre les shapes pendant les transitions
- Disponible pour les composants standards et custom

## Implémentation de base

### Ajouter Liquid Glass à une vue

La façon la plus simple d'ajouter Liquid Glass à une vue est d'utiliser le modifier `glassEffect()` :

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect()
```

Par défaut, cela applique la variante regular du glass dans une shape Capsule derrière le contenu de la vue.

### Personnaliser la shape

Tu peux spécifier une shape différente pour l'effet Liquid Glass :

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16.0))
```

Options de shape courantes :
- `.capsule` (par défaut)
- `.rect(cornerRadius: CGFloat)`
- `.circle`

## Personnaliser les effets Liquid Glass

### Variantes et propriétés de Glass

Tu peux personnaliser l'effet Liquid Glass en configurant la structure `Glass` :

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect(.regular.tint(.orange).interactive())
```

Options de personnalisation clés :
- `.regular` - effet glass standard
- `.tint(Color)` - ajoute une teinte de couleur pour suggérer la prominence
- `.interactive(Bool)` - fait réagir le glass au touch et aux interactions pointer

### Rendre le glass interactif

Pour faire réagir Liquid Glass au touch et aux interactions pointer :

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect(.regular.interactive(true))
```

Ou plus concis :

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect(.regular.interactive())
```

## Travailler avec plusieurs effets Glass

### Utiliser GlassEffectContainer

Quand tu appliques des effets Liquid Glass à plusieurs vues, utilise `GlassEffectContainer` pour une meilleure performance de rendu et pour permettre les effets de blending et de morphing :

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80.0, height: 80.0)
            .font(.system(size: 36))
            .glassEffect()

        Image(systemName: "eraser.fill")
            .frame(width: 80.0, height: 80.0)
            .font(.system(size: 36))
            .glassEffect()
    }
}
```

Le paramètre `spacing` contrôle comment les effets Liquid Glass interagissent entre eux :
- Spacing plus petit : les vues doivent être plus proches pour fusionner les effets
- Spacing plus grand : les effets fusionnent à des distances plus grandes

### Unir plusieurs effets Glass

Pour combiner plusieurs vues en un seul effet Liquid Glass, utilise le modifier `glassEffectUnion` :

```swift
@Namespace private var namespace

// Plus loin dans ta vue :
GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        ForEach(symbolSet.indices, id: \.self) { item in
            Image(systemName: symbolSet[item])
                .frame(width: 80.0, height: 80.0)
                .font(.system(size: 36))
                .glassEffect()
                .glassEffectUnion(id: item < 2 ? "1" : "2", namespace: namespace)
        }
    }
}
```

C'est utile quand tu crées des vues dynamiquement ou avec des vues qui vivent hors d'un HStack ou VStack.

## Effets de morphing et transitions

### Créer des transitions de morphing

Pour créer des effets de morphing pendant les transitions entre vues avec Liquid Glass :

1. Crée un namespace avec le property wrapper `@Namespace`
2. Associe chaque effet Liquid Glass à un identifiant unique via `glassEffectID`
3. Utilise des animations quand tu changes la hiérarchie de vues

```swift
@State private var isExpanded: Bool = false
@Namespace private var namespace

var body: some View {
    GlassEffectContainer(spacing: 40.0) {
        HStack(spacing: 40.0) {
            Image(systemName: "scribble.variable")
                .frame(width: 80.0, height: 80.0)
                .font(.system(size: 36))
                .glassEffect()
                .glassEffectID("pencil", in: namespace)

            if isExpanded {
                Image(systemName: "eraser.fill")
                    .frame(width: 80.0, height: 80.0)
                    .font(.system(size: 36))
                    .glassEffect()
                    .glassEffectID("eraser", in: namespace)
            }
        }
    }

    Button("Toggle") {
        withAnimation {
            isExpanded.toggle()
        }
    }
    .buttonStyle(.glass)
}
```

L'effet de morphing se produit quand des vues avec Liquid Glass apparaissent ou disparaissent suite à des changements de hiérarchie de vues.

## Styling de boutons avec Liquid Glass

### Glass Button Style

SwiftUI fournit des styles de bouton intégrés pour Liquid Glass :

```swift
Button("Click Me") {
    // Action
}
.buttonStyle(.glass)
```

### Glass Prominent Button Style

Pour un bouton glass plus prononcé :

```swift
Button("Important Action") {
    // Action
}
.buttonStyle(.glassProminent)
```

## Techniques avancées

### Background Extension Effect

Pour étirer du contenu derrière une sidebar ou un inspector avec le background extension effect :

```swift
NavigationSplitView {
    // Contenu de la sidebar
} detail: {
    // Contenu du detail
        .background {
            // Contenu de fond qui s'étend sous la sidebar
        }
}
```

### Étendre le scroll horizontal sous la sidebar

Pour étendre les scroll views horizontales sous une sidebar ou un inspector :

```swift
ScrollView(.horizontal) {
    // Contenu scrollable
}
.scrollExtensionMode(.underSidebar)
```

## Bonnes pratiques

1. **Usage du container** : utilise toujours `GlassEffectContainer` quand tu appliques Liquid Glass à plusieurs vues, pour une meilleure performance et des effets de morphing.

2. **Ordre des effets** : applique le modifier `.glassEffect()` après les autres modifiers qui affectent l'apparence de la vue.

3. **Considération du spacing** : choisis soigneusement les valeurs de spacing dans les containers pour contrôler comment et quand les effets glass fusionnent.

4. **Animation** : utilise des animations quand tu changes les hiérarchies de vues pour permettre des transitions de morphing fluides.

5. **Interactivité** : ajoute `.interactive()` aux effets glass qui doivent répondre à l'interaction utilisateur.

6. **Design cohérent** : maintiens des shapes et styles cohérents dans ton app pour un look et une sensation homogènes.

## Exemple : badge custom avec Liquid Glass

```swift
struct BadgeView: View {
    let symbol: String
    let color: Color

    var body: some View {
        ZStack {
            Image(systemName: "hexagon.fill")
                .foregroundColor(color)
                .font(.system(size: 50))

            Image(systemName: symbol)
                .foregroundColor(.white)
                .font(.system(size: 30))
        }
        .glassEffect(.regular, in: .rect(cornerRadius: 16))
    }
}

// Usage :
GlassEffectContainer(spacing: 20) {
    HStack(spacing: 20) {
        BadgeView(symbol: "star.fill", color: .blue)
        BadgeView(symbol: "heart.fill", color: .red)
        BadgeView(symbol: "leaf.fill", color: .green)
    }
}
```

## Références

- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)
- [Landmarks: Building an app with Liquid Glass](https://developer.apple.com/documentation/SwiftUI/Landmarks-Building-an-app-with-Liquid-Glass)
- [SwiftUI View.glassEffect(_:in:isEnabled:)](https://developer.apple.com/documentation/SwiftUI/View/glassEffect(_:in:isEnabled:))
- [SwiftUI GlassEffectContainer](https://developer.apple.com/documentation/SwiftUI/GlassEffectContainer)
- [SwiftUI GlassEffectTransition](https://developer.apple.com/documentation/SwiftUI/GlassEffectTransition)
- [SwiftUI GlassButtonStyle](https://developer.apple.com/documentation/SwiftUI/GlassButtonStyle)
</content>
