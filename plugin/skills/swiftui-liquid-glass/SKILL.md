---
name: swiftui-liquid-glass
description: "Implémenter et revoir une UI SwiftUI Liquid Glass iOS 26+. À utiliser pour adopter Liquid Glass ou vérifier sa correction, sa performance et son adéquation design."
---

# SwiftUI Liquid Glass

## Vue d'ensemble
Utilise cette skill pour construire ou revoir des fonctionnalités SwiftUI pleinement alignées avec l'API Liquid Glass d'iOS 26+. Priorise les API natives (`glassEffect`, `GlassEffectContainer`, glass button styles) et les guidelines de design Apple. Garde l'usage cohérent, interactif quand nécessaire, et attentif à la performance.

## Arbre de décision du workflow
Choisis le chemin qui correspond à la demande :

### 1) Revoir une fonctionnalité existante
- Inspecte où Liquid Glass devrait être utilisé et où il ne devrait pas l'être.
- Vérifie l'ordre correct des modifiers, l'usage des shapes et le placement des containers.
- Vérifie la gestion de la disponibilité iOS 26+ et l'existence de fallbacks sensés.

### 2) Améliorer une fonctionnalité avec Liquid Glass
- Identifie les composants cibles pour un traitement glass (surfaces, chips, boutons, cards).
- Refactore pour utiliser `GlassEffectContainer` là où plusieurs éléments glass apparaissent.
- Introduis du glass interactif uniquement pour les éléments tappable ou focusable.

### 3) Implémenter une nouvelle fonctionnalité avec Liquid Glass
- Conçois d'abord les surfaces et interactions glass (shape, prominence, regroupement).
- Ajoute les modifiers glass après les modifiers de layout/apparence.
- Ajoute des transitions de morphing uniquement quand la hiérarchie de vues change avec animation.

## Consignes de base
- Préfère les API natives Liquid Glass aux blurs custom.
- Utilise `GlassEffectContainer` quand plusieurs éléments glass coexistent.
- Applique `.glassEffect(...)` après les modifiers de layout et visuels.
- Utilise `.interactive()` pour les éléments qui répondent au touch/pointer.
- Garde des shapes cohérentes entre les éléments liés pour un look homogène.
- Protège avec `#available(iOS 26, *)` et fournis un fallback non-glass.

## Checklist de revue
- **Disponibilité** : `#available(iOS 26, *)` présent avec une UI de fallback.
- **Composition** : plusieurs vues glass regroupées dans un `GlassEffectContainer`.
- **Ordre des modifiers** : `glassEffect` appliqué après les modifiers de layout/apparence.
- **Interactivité** : `interactive()` uniquement là où il y a une interaction utilisateur.
- **Transitions** : `glassEffectID` utilisé avec `@Namespace` pour le morphing.
- **Cohérence** : shapes, teintes et espacements alignés sur toute la fonctionnalité.

## Checklist d'implémentation
- Définis les éléments cibles et la prominence glass souhaitée.
- Regroupe les éléments glass dans un `GlassEffectContainer` et ajuste l'espacement.
- Utilise `.glassEffect(.regular.tint(...).interactive(), in: .rect(cornerRadius: ...))` selon le besoin.
- Utilise `.buttonStyle(.glass)` / `.buttonStyle(.glassProminent)` pour les actions.
- Ajoute des transitions de morphing avec `glassEffectID` quand la hiérarchie change.
- Fournis des matériaux et visuels de fallback pour les versions iOS antérieures.

## Snippets rapides
Utilise ces patterns directement et ajuste shapes/teintes/espacements.

```swift
if #available(iOS 26, *) {
    Text("Hello")
        .padding()
        .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
} else {
    Text("Hello")
        .padding()
        .background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16))
}
```

```swift
GlassEffectContainer(spacing: 24) {
    HStack(spacing: 24) {
        Image(systemName: "scribble.variable")
            .frame(width: 72, height: 72)
            .font(.system(size: 32))
            .glassEffect()
        Image(systemName: "eraser.fill")
            .frame(width: 72, height: 72)
            .font(.system(size: 32))
            .glassEffect()
    }
}
```

```swift
Button("Confirm") { }
    .buttonStyle(.glassProminent)
```

## Ressources
- Guide de référence : `references/liquid-glass.md`
- Préfère la doc Apple pour les détails d'API à jour, et utilise la recherche web pour consulter la documentation Apple Developer actuelle en complément des références ci-dessus.
</content>
